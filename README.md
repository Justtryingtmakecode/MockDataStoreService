<h1 align="center">MockDataStoreService</h1>
<div align="center">
	<a href="https://travis-ci.com/buildthomas/MockDataStoreService">
		<img src="https://travis-ci.com/buildthomas/MockDataStoreService.svg?branch=master" />
	</a>
	<a href="https://coveralls.io/github/buildthomas/MockDataStoreService?branch=master">
		<img src="https://coveralls.io/repos/github/buildthomas/MockDataStoreService/badge.svg?branch=master" />
	</a>
</div>

<div align="center">
	Emulation of Roblox's DataStoreService for seamless offline development & testing
</div>

<div>&nbsp;</div>

This is an open-source module that emulates datastores in Lua rather than using the actual service. This is useful for testing in offline projects / local place files with code/frameworks that need to have access to datastores.

The MockDataStoreService behaves exactly like DataStoreService -- it has the same API, and will also work even if the place is not currently published to a game with Studio API access enabled (it will act as a DataStoreService with no data stored on the back-end, unless you import some data from a json string manually using the module).

A small top-level helper module is provided (DataStoreService) that automatically detects and selects which datastores should be used (real datastores for published games with API access, mock datastores for offline games / published games without API access).

It is recommended to use this code in the structure that it is provided in, and to simply call  `require(path.to.DataStoreService)`  instead of  `game:GetService("DataStoreService")` anywhere in your code to use it properly.

-----

**Usage:**

```lua
local DataStoreService = require(the.path.to.DataStoreService)

-- Use as actual DataStoreService, i.e.:

local gds = DataStoreService:GetGlobalDataStore()
local ds = DataStoreService:GetDataStore("TestName", "TestScope")
local ods = DataStoreService:GetOrderedDataStore("TestName")

local value = ds:GetAsync("TestKey")
ds:SetAsync("TestKey", "TestValue")
local value = ds:IncrementAsync("IntegerKey", 3)
local value = ds:UpdateAsync("UpdateKey", function(oldValue) return newValue end)
local value = ds:RemoveAsync("TestKey")
local connection = ds:OnUpdate("UpdateKey", function(value) print(value) end)

local pages = ods:GetSortedAsync(true, 50, 1, 100)
repeat
	for _, pair in ipairs(pages:GetCurrentPage()) do
		local key, value = pair.key, pair.value
		-- (...)
	end
until pages.IsFinished or pages:AdvanceToNextPageAsync()

local budget = DataStoreService:GetRequestBudgetForRequestType(
	Enum.DataStoreRequestType.UpdateAsync
)

-- Import/export data to a specific datastore:

ds:ImportFromJSON({ -- feed table or json string representing contents of datastore
	TestKey = "Hello world!"; -- a key value pair
	AnotherKey = {a = 1, b = 2}; -- another key value pair
	-- (...)
})

print(ds:ExportToJSON())

-- Import/export entirety of DataStoreService:

DataStoreService:ImportFromJSON({ -- feed table or json string
	DataStore = { -- regular datastores
		TestName = { -- name of datastore
			TestScope = { -- scope of datastore
				TestKey = "Hello world!"; -- a key value pair
				AnotherKey = {1,2,3}; -- another key value pair
				-- (...)
			}
		}
	};
	GlobalDataStore = { -- the (one) globaldatastore
		TestKey = "Hello world!"; -- a key value pair
		AnotherKey = {1,2,3}; -- another key value pair
		-- (...)
	};
	OrderedDataStore = { -- ordered datastores
		TestName = { -- name of ordered datastore
			TestScope = { -- scope of ordered datastore
				TestKey = 15; -- a key value pair
				AnotherKey = 3; -- another key value pair
				-- (...)
			}
		}
	};
}

print(DataStoreService:ExportToJSON())

```

Review the API of datastores here:
- http://wiki.roblox.com/index.php?title=API:Class/DataStoreService
- http://wiki.roblox.com/index.php?title=API:Class/GlobalDataStore
- http://wiki.roblox.com/index.php?title=API:Class/OrderedDataStore
- http://wiki.roblox.com/index.php?title=API:Class/DataStorePages

-----

**Features:**
- Identical API and near-identical behaviour compared to real DataStoreService.
- Error messages are more verbose/accurate than the ones generated by actual datastores, which makes development/bug-fixing easier.
- Throws descriptive errors for attempts at storing invalid data, telling you exactly which part of the data is invalid. (credit to @Corecii's helper function)
- Emulates the yielding of datastore requests (waits a random amount of time before returning from the call).
- Extra API for json-exporting/importing contents of one/all datastores for easy testing.
- All operations safely deep-copy values where necessary (not possible to alter values in the datastore by externally altering tables, etc).
- Enforces the "6 seconds between writes on the same key" rule.
- Enforces datastore budgets correctly: budget are set and increased at the rates of the actual service, requests will be throttled if the budget is exceeded, and if there are too many throttled requests in the queue then new requests will error instead of throttling.
