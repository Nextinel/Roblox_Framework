# Validator Documentation
The framework includes a built-in argument validator for network contracts. It checks argument count, types, and structure before executing the `onEvent` callback

## Example
```lua
local template = {
	value1 = "number",
	value2 = "string",
	
	value3 = {
		type = "number",
		optional = true
	},
	
	value4 = {
		type = "table",
		child = {
			arg1 = "number",
			arg2 = "number",
			arg3 = "number"
		}
	},
	
	value5 = {
		type = "table",
		child = {
			actionText = "string",
			objectText = {
				type = "string",
				optional = true
			}
		}
	}
}
```

## Explanation
- **optional = true** - The validator skips this field if its value is nil. If the value is not nil, it will still be validated
- **shouldCheckExtraFields = true** - The validator checks for unexpected fields (fields not defined in the template) and rejects them. Default is false
- **shouldUseCache = true** - Saves the result in a cache. If the same input is validated again, the cached result is returned instead of re-validating

## Usage
To use the validator manually (outside of network contracts):
```lua
local Validator = require(ReplicatedStorage.Modules.Global.Validator)

local template = {
    name = "string",
    count = "number"
}

local data = {
    name = "Sword",
    count = 5
}

local success, result = Validator.check(data, template)

if success then
    print("Data is valid!")
else
    warn("Validation failed:", result)
end
```
