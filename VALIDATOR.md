## TEMPLATE EXAMPLE
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

## EXPLANATION
- **optional = true** - validator skips a field when its value is nil. If the value is not nil field is still checked
- **shouldCheckExtraFields = true** - validator checks for fields that are not in the template
- **shouldUseCache = true** - saves result in cache and automatically gives it if the result is already in cache
