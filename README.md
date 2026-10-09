# Roblox Framework
Server-client framework to build your games easily

## Table of Contents
- [**The Problem**](#the-problem) - Why this framework exists
- [**Support & Community**](#support--community) - Where to get help
- [**Contributing**](#contributing) - How to help the project
- [**Installation**](#installation) - How to set it up in 5 minutes
- [**Updates**](#updates) - How the auto-updater works
- [**Features**](#features) - What's inside the box
- [**Structure**](#structure) - Folder layout and where to put your modules
- [**Usage**](#usage) - How to write your first module
- [**Quick Example (Server code)**](#quick-example-server-code) - Before vs after
- [**Full API Reference**](#full-api-reference) - All available lifecycle hooks
- [**Network (Remotes)**](#network-remotes) - Client-server communication via Jolt
- [**Dependency Injection**](#dependency-injection) - How modules depend on each other
- [**Credits**](#credits) - Third-party libraries used
- [**License**](#license) - MIT

## The Problem
Every new Roblox project starts with the same boilerplate: wiring up remotes, setting up player data, and managing module loading. Nextinel Framework eliminates that boilerplate so you can focus on your game

## Support & Community
- **Issues:** If you found a bug or have a suggestion, please open an [Issue](../../issues)
- **DevForum:** [Nextinel Framework](https://devforum.roblox.com/t/open-source-nextinel-framework-my-modular-serverclient-setup/4921951) - ask questions, share feedback, or report bugs here

## Contributing
Pull requests are welcome! If you want to add a new feature or fix a bug:
1. Fork the repository
2. Create a new branch (`git checkout -b feature/FeatureName`)
3. Commit your changes
4. Open a Pull Request

## Installation
1. Go to the [Releases](../../releases) tab
2. Download the latest `framework.rbxm`
3. Open the `.rbxm` file in Roblox Studio
4. Expand the `Framework` folder and move its contents into the services with the same name:
   - `ReplicatedStorage`
   - `ServerScriptService`
   - `StarterPlayerScripts`
5. Enable **Allow HTTP Requests** in `Game Settings` -> `Security` (required for the Version Checker)

## Updates
The framework checks for updates automatically on server start. If a new version is available, it will warn in the output
To update: download the new `.rbxm` from the Releases tab and replace the old files in your game

## Features
- **Network Handlers** - Client & server handlers with built-in rate limits and validators. Works over `Jolt` API
- **Loading Screen** - Built-in loading screen
- **DataStoreService** - Module with Migrator, built on ProfileStore
- **Version Checker** - Built-in version checker that checks for updates on server start
- **Dependency Injector** - Modules declare their dependencies, and the framework resolves them in the correct order

## Structure
- ReplicatedStorage
  - Assets
  - Modules
    - Global
    - Initialize
  - HUD
- ServerScriptService
  - Modules
    - Global
    - Initialize
  - Server
- StarterPlayerScripts
  - Client

> Global - modules that are shared or used by other modules (e.g., utilities, handlers)

> Initialize - modules that should be initialized at server/client start (e.g., your gameplay logic)
 
## Usage
The framework is based on **modules**. You don't need to write `while true do` loops or manually connect `PlayerAdded`. Just create a `ModuleScript` inside the specific folders, return a table with lifecycle hooks, and the framework will handle the rest

- **Server modules:** `ServerScriptService.Modules.Initialize`
- **Client modules:** `ReplicatedStorage.Modules.Initialize`

## Quick Example (Server code)
### Without Nextinel:
```lua
local players = game:GetService("Players")

players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        -- ...
    end)
end)
```

### With Nextinel:
```lua
return {
    characterAdded = function(character, player)
        -- ...
    end,

	humanoidAdded = function(humanoid, player)
		-- ...
	end
}
```

## Full API Reference
### Server module Hooks
```lua
return {
    -- Dependencies (optional)
    dependencies = {"BackpackHandler"},
    
    -- Called once on server start. Dependencies are injected here
    init = function(dependingServices) end,
    
    -- Called every Heartbeat (spread across 3 buckets for performance)
    update = function(deltaTime) end,
    
    -- Player lifecycle
    playerAdded = function(player) end,
    playerDataLoaded = function(player, profile) end,
    playerRemoving = function(player) end,
    
    -- Character lifecycle
    characterAdded = function(character, player) end,
    characterLoaded = function(character, humanoid, root, player) end,
    humanoidAdded = function(humanoid, player) end,
    characterRemoved = function(character, player) end,
}
```

### Client module Hooks
```lua
return {
    -- Called synchronously on start
    init = function(dependingServices) end,
    
    -- Called asynchronously (good for yielding tasks)
    asyncInit = function() end,
    
    -- Called every Heartbeat
    update = function(deltaTime) end,
    
    -- Called after the Loading Screen finishes and assets are preloaded
    clientReady = function() end,
    
    -- Character lifecycle (local player only)
    characterAdded = function(character) end,
    characterLoaded = function(character, humanoid, root) end,
    humanoidAdded = function(humanoid) end,
    characterRemoved = function(character) end,
}
```

## Network (Remotes)
You don't need to manually find Remotes or connect `OnServerEvent`/`OnClientEvent`. Just define a `network` table inside your module

Network is built on **Jolt** (`ReplicatedStorage.Modules.Global.Jolt`)

### Listening to a Remote (Server & Client):
```lua
return {
    network = {
        channel = "ChannelName",
        
        -- Optional: Type and argument count validation
        contract = {
            [1] = "string",
            [2] = "number"
        },

		-- Server:
        onEvent = function(player, name, count)
            print(player.Name, "received:", name, count)
        end,

		-- Client:
		onEvent = function(name, count)
            print("Received:", name, count)
        end,
    },
}
```
> **Validator Documentation (for contract):** For advanced validation rules (optional fields, nested tables, caching), please read [VALIDATOR.md](VALIDATOR.md)

### Firing (Server):
Server modules get the channel directly through `Jolt`:

```lua
local replicatedStorage = game:GetService("ReplicatedStorage")

local jolt = require(replicatedStorage.Modules.Global.Jolt)

local channel = jolt.Server("ChannelName")

return {
	playerAdded = function(player)
		channel:Fire(player, "welcome")
	end,
	
	update = function(dt)
		channel:FireAll("tick", dt)
        -- channel:FireExcept(player, "skip") -- skip specific player
	end,
}
```

### Firing (Client):
If your client module needs to send data to the server, use `setupFireServer`. The framework will create channel with `Jolt` and give you a network object
```lua
local channels = {}

return {
    setupFireServer = {
        channelsNames = {"ChannelName"},
        
        setupRemote = function(name, network)
            channels[name] = network
        end,
    },
    
    update = function()
        if channels.ChannelName then
            channels.ChannelName:fireServer("hello", 123)
        end
    end,
}
```
Why use the network object instead of getting the Jolt object directly? Because the network object automatically enforces rate limits, so you dont have to implement them yourself

## Dependency Injection
If Module A needs Module B, just list it in `dependencies`. The framework sorts the initialization order automatically (via `DependencyResolver`) so Module B is initialized before Module A. If Module B fails, Module A will gracefully skip its own `init` with a warning
```lua
return {
    dependencies = {"Inventory", "ShopUI"},
    init = function(dependingServices)
        local Inventory = dependingServices.Inventory
        local ShopUI = dependingServices.ShopUI
    end,
}
```

## Credits
- **[ProfileStore](https://github.com/MadStudioRoblox/ProfileStore)** by **loleris** - Used for DataStore handling in the `DataStoreService` module
- **[Jolt](https://github.com/ToriumSlurs/Jolt)** by **ToriumSlurs** - Used for network handling in the `NetworkHandler`'s modules

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details
