# Roblox Framework
Server-client framework to build your games easily

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
- **Network Handlers** - Client & server handlers with built-in rate limits and validators
- **Loading Screen** - Built-in loading screen
- **DataStoreService** - Module with Migrator, built on ProfileStore
- **Version Checker** - Built-in version checker that checks for updates on server start

## Structure
- ReplicatedStorage
  - Assets
  - Modules
  - Remotes
  - HUD
- ServerScriptService
  - Modules
  - Server
- StarterPlayerScripts
  - Client
 
## Usage

The framework is based on **modules**. You don't need to write `while true do` loops or manually connect `PlayerAdded`. Just create a `ModuleScript` inside the specific folders, return a table with lifecycle hooks, and the framework will handle the rest

- **Server modules:** `ServerScriptService.Modules.Initialize`
- **Client modules:** `ReplicatedStorage.Modules.Initialize`

### Basic Module Example (Server)
```lua
return {
    -- Dependencies (optional)
    dependencies = {"BackpackHandler"},
    
    -- Called once on server start. Dependencies are injected here
    init = function(dependingServices)
        local backpackHandler = dependingServices.BackpackHandler
        print("Server module initialized!")
        -- // ...
    end,
    
    -- Called every Heartbeat (spread across 3 buckets for performance)
    update = function(deltaTime)
        -- // ...
    end,
    
    -- Player lifecycle
    playerAdded = function(player)
        print(player.Name .. " joined!")
    end,
    
    playerDataLoaded = function(player, profile)
        print(player.Name .. "'s data loaded:", profile)
    end,
    
    playerRemoving = function(player)
        print(player.Name .. " is leaving.")
    end,
    
    -- Character lifecycle
    characterAdded = function(character, player) end,
    characterLoaded = function(character, humanoid, root, player) end,
    humanoidAdded = function(humanoid, player) end,
    characterRemoved = function(character, player) end,
}
```

### Basic Module Example (Client)
```lua
return {
    -- Called synchronously on start
    init = function(dependingServices) end,
    
    -- Called asynchronously (good for yielding tasks)
    asyncInit = function() end,
    
    -- Called every Heartbeat
    update = function(deltaTime) end,
    
    -- Called after the Loading Screen finishes and assets are preloaded
    clientReady = function()
        print("Client is fully loaded!")
    end,
    
    -- Character lifecycle (local player only)
    characterAdded = function(character) end,
    characterLoaded = function(character, humanoid, root) end,
    humanoidAdded = function(humanoid) end,
    characterRemoved = function(character) end,
}
```

## Network (Remotes)
You don't need to manually find Remotes or connect `OnServerEvent`/`OnClientEvent`. Just define a `network` table inside your module

### Listening to a Remote (Server & Client):
```lua
return {
    network = {
        remote = "RemoteName",
        
        -- Optional: Type and argument count validation
        contract = {
            [1] = "string",
            [2] = "number"
        },
        
        onEvent = function(name, count)
            print("Received:", name, count)
        end,
    },
}
```
**Validator Documentation (for contract):** For advanced validation rules (optional fields, nested tables, caching), please read [VALIDATOR.md](VALIDATOR.md)

### Firing to Server (Client only):
If your client module needs to send data to the server, use `setupFireServer`. The framework will find the remote and give you a network object
```lua
local remotes = {}

return {
    setupFireServer = {
        remoteNames = {"RemoteName"},
        
        setupRemote = function(name, network)
            remotes[name] = network
        end,
    },
    
    update = function()
        if remotes.RemoteName then
            remotes.RemoteName:fireServer("hello", 123)
        end
    end,
}
```

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

## License
This project is licensed under the MIT License
