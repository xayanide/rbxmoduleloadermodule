# rbxmoduleloader
A module that loads and executes ModuleScripts from either ServerScriptService or ReplicatedStorage and stores it into global variable "shared" with its contents depend on runtime context (Client or Server).

Features:
- Loads modules dynamically, only when needed.
- Loaded ModuleScripts implement lifecycle methods (hooks): `onModuleSetup` (synchronous) and `onModuleStart` (asynchronous).
- Follows a specific design pattern when making ModuleScripts for the Client or Server but doesn't dictate the way you script.
- Allows for both shared and non-shared (ModuleContainer) module storage, depending on whether the module should be available globally or locally.
- Allows only loading module scripts from specific Instance's descendants.
- Uses `xpcall` and `pcall` for safe module requiring, prevents the module loader from failing, with error handling that logs any failures to load modules.

## Prerequisites

- Roblox Studio

## Usage

To use the `rbxmoduleloader` **stock**, you will need to call it with no options. This module can be loaded either as a server or client-side script, and that'll dictate what module scripts it will load.

**Client**

Create a `Script` in `ReplicatedStorage` and set its `RunContext` to `Client` so that it'll run in `ReplicatedStorage`.
Copy and paste, enable this script and update require path if needed.

>[!NOTE]
> You can rewrite this below as a one-liner since the Script only exists for one purpose: Loading modules.
```lua
--!strict
local ReplicatedStorageService = game:GetService("ReplicatedStorage")
require(ReplicatedStorageService.MainModule)()
```
Take note of its RunContext: `Client`. This will load the modules found in `ReplicatedStorage` and store them in the `shared` global variable, making them globally accessible on other client scripts.

**Server**

Similarly, create a `Script` in `ReplicatedStorage` and set its `RunContext` to `Server` so that it'll run in `ReplicatedStorage`.
Copy and paste, enable this script and update require path if needed.

>[!NOTE]
> You can rewrite this below as a one-liner since the Script only exists for one purpose: Loading modules.
```lua
--!strict
local ReplicatedStorageService = game:GetService("ReplicatedStorage")
require(ReplicatedStorageService.MainModule)()
```
Take note of its RunContext: `Server`. This will load the modules found in `ServerScriptService` and store them in the `shared` global variable, making them globally accessible on other server scripts.


## Configuration

The `ModuleLoaderOptions` table allows you to configure the loader's behavior. This is passed as an argument to the `rbxmoduleloader` function when you require it.

`isShared` (optional)
  - `Type`: boolean?
  - `Default`: true
  - `Description`: Determines if the loaded modules should be stored in the shared table (accessible globally) or in an isolated container (child of rbxmoduleloader "ModuleContainer"). When `isShared` is true, the modules are stored in the global shared table, meaning they can be accessed anywhere in the game depending on the runtime context.

**Example Usage:**
This will load all ModuleScripts inside `ReplicatedStorage`.

> [!NOTE] For this example, write this as a `LocalScript` inside `StarterPlayer -> StarterPlayerScripts`
```lua
local ReplicatedStorageService = game:GetService("ReplicatedStorage")
local loadModules = require(ReplicatedStorageService.MainModule)
loadModules({ isShared = true })
```

`targetInstances` (optional)
  - `Type`: { Instance }?
  - `Default`: nil
  - `Description`: Specifies the list of instances whose descendants should be processed for module loading. If not provided, the loader will scan all descendants of the appropriate service based on the runtime environment (server or client):
  - Server
    - ServerScriptService
  - Client
    - ReplicatedStorage

**Example Usage:**
This will load all ModuleScripts inside `ServerScriptService` and `ServerStorage`.

> [!NOTE] For this example, write this as a `Server Script` in `ServerScriptService`
```lua
local ReplicatedStorageService = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local ServerStorageService = game:GetService("ServerStorage")

local loadModules = require(ReplicatedStorageService.MainModule)
loadModules({
    targetInstances = { ServerScriptService, ServerStorageService }
})
```

## Lifecycle Methods

The loader supports two lifecycle methods that will be invoked on the modules after they are loaded (required):
1. `onModuleSetup`
    - `Description`: This method is called synchronously for each module after it is loaded.
    - `Purpose`: Use this to perform setup tasks on the module, like initializing variables or setting properties.

2. `onModuleStart`
    - `Description`: This method is called asynchronously for each module after the setup phase.
    - `Purpose`: Use this to trigger the start of the module, such as initiating processes that may involve waiting or other asynchronous tasks.
