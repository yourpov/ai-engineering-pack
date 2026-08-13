# Luau Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Read project `AGENTS.md` if present. For reviews use `skills/review/audit/SKILL.md` (scope + mode). For naming/comments use `skills/engineering/craft/SKILL.md` (not detector scoring). Prefer extending an existing repo over scaffolding a parallel tree. Discover verify commands from the project; do not invent a toolchain.

Clean structure for Roblox game development with Luau.

**Roblox Luau only.** This file does not cover other Lua runtimes. If you are on a different Lua platform, follow that project's own resource conventions instead.

---

## Project Structure

```
game/
├── src/
│   ├── server/          # Server-side scripts
│   │   ├── services/    # Game systems
│   │   └── handlers/    # Event handlers
│   ├── client/          # Client-side scripts
│   │   ├── controllers/ # UI and input
│   │   └── effects/     # Visual effects
│   └── shared/          # Code used by both
│       ├── data/        # Game data, configs
│       └── utils/       # Helper functions
└── tests/
```

---

## Principles

**Server authority**
Server validates everything. Client is for display only.

**Separate concerns**
Keep UI, game logic, and data separate.

**Small modules**
Each script does one thing well.

**Type everything**
Use Luau's type system fully.

**No globals**
Use ModuleScripts and require().

---

## Module Structure

### Server Service

**server/services/PlayerData.lua**
```lua
local PlayerData = {}

type PlayerData = {
    coins: number,
    level: number,
    inventory: {string}
}

local data: {[Player]: PlayerData} = {}

function PlayerData.init(player: Player)
    data[player] = {
        coins = 0,
        level = 1,
        inventory = {}
    }
end

function PlayerData.get(player: Player): PlayerData?
    return data[player]
end

function PlayerData.addCoins(player: Player, amount: number)
    local playerData = data[player]
    if not playerData then return end
    
    if amount < 0 then
        warn("Cannot add negative coins")
        return
    end
    
    playerData.coins += amount
end

function PlayerData.canAfford(player: Player, cost: number): boolean
    local playerData = data[player]
    if not playerData then return false end
    
    return playerData.coins >= cost
end

function PlayerData.cleanup(player: Player)
    data[player] = nil
end

return PlayerData
```

### Client Controller

**client/controllers/ShopController.lua**
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ShopRemote = ReplicatedStorage:WaitForChild("ShopRemote")

local ShopController = {}

function ShopController.init()
    setupUI()
    bindEvents()
end

function ShopController.purchase(itemId: string)
    ShopRemote:FireServer("purchase", itemId)
end

function setupUI()
    local shopButton = script.Parent:WaitForChild("ShopButton")
    shopButton.Activated:Connect(function()
        toggleShop()
    end)
end

function bindEvents()
    ShopRemote.OnClientEvent:Connect(function(action, data)
        if action == "purchaseSuccess" then
            showConfirmation(data)
        elseif action == "purchaseFailure" then
            showError(data)
        end
    end)
end

function toggleShop()
    local shopUI = script.Parent:WaitForChild("ShopUI")
    shopUI.Visible = not shopUI.Visible
end

function showConfirmation(message: string)
    print("Purchase successful:", message)
end

function showError(message: string)
    warn("Purchase failed:", message)
end

return ShopController
```

---

## Remote Events

**Server Handler**

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local PlayerData = require(script.Parent.services.PlayerData)
local ShopRemote = ReplicatedStorage:WaitForChild("ShopRemote")

local items = {
    sword = { cost = 100, type = "weapon" },
    shield = { cost = 150, type = "armor" }
}

ShopRemote.OnServerEvent:Connect(function(player, action, itemId)
    if action == "purchase" then
        handlePurchase(player, itemId)
    end
end)

function handlePurchase(player: Player, itemId: string)
    local item = items[itemId]
    if not item then
        ShopRemote:FireClient(player, "purchaseFailure", "Invalid item")
        return
    end
    
    if not PlayerData.canAfford(player, item.cost) then
        ShopRemote:FireClient(player, "purchaseFailure", "Not enough coins")
        return
    end
    
    PlayerData.addCoins(player, -item.cost)
    giveItem(player, itemId)
    
    ShopRemote:FireClient(player, "purchaseSuccess", "Purchased " .. itemId)
end

function giveItem(player: Player, itemId: string)
    local playerData = PlayerData.get(player)
    if not playerData then return end
    
    table.insert(playerData.inventory, itemId)
end
```

---

## Data Validation

**shared/Validator.lua**
```lua
local Validator = {}

function Validator.isValidUsername(name: string): boolean
    if #name < 3 or #name > 20 then
        return false
    end
    
    return name:match("^[%w_]+$") ~= nil
end

function Validator.isValidAmount(amount: number): boolean
    return amount > 0 and amount <= 1000000
end

function Validator.isInRange(value: number, min: number, max: number): boolean
    return value >= min and value <= max
end

return Validator
```

---

## Configuration

**shared/data/GameConfig.lua**
```lua
local GameConfig = {
    maxPlayers = 50,
    roundDuration = 300,
    
    coins = {
        startAmount = 100,
        maxAmount = 999999
    },
    
    items = {
        sword = { cost = 100, damage = 10 },
        shield = { cost = 150, defense = 5 }
    }
}

return GameConfig
```

---

## State Management

**shared/State.lua**
```lua
local Signal = require(script.Parent.Signal)

local State = {}
State.__index = State

type State<T> = {
    value: T,
    changed: any
}

function State.new<T>(initial: T): State<T>
    local self = setmetatable({}, State)
    self.value = initial
    self.changed = Signal.new()
    return self
end

function State:get()
    return self.value
end

function State:set(newValue)
    if self.value == newValue then return end
    
    self.value = newValue
    self.changed:fire(newValue)
end

return State
```

**Usage**
```lua
local State = require(ReplicatedStorage.shared.State)

local coins = State.new(0)

coins.changed:connect(function(amount)
    print("Coins changed:", amount)
end)

coins:set(100)  -- Fires signal
```

---

## UI Components

**client/ui/Button.lua**
```lua
local Button = {}

function Button.create(parent: Instance, text: string, onClick: () -> ())
    local button = Instance.new("TextButton")
    button.Text = text
    button.Size = UDim2.new(0, 200, 0, 50)
    button.Parent = parent
    
    button.Activated:Connect(onClick)
    
    return button
end

return Button
```

**Usage**
```lua
local Button = require(script.Parent.Button)

local screenGui = script.Parent
Button.create(screenGui, "Click Me", function()
    print("Button clicked!")
end)
```

---

## Tween Utilities

**shared/utils/Tween.lua**
```lua
local TweenService = game:GetService("TweenService")

local Tween = {}

function Tween.scale(instance: Instance, scale: number, duration: number)
    local info = TweenInfo.new(
        duration,
        Enum.EasingStyle.Quad,
        Enum.EasingDirection.Out
    )
    
    local goal = { Size = instance.Size * scale }
    local tween = TweenService:Create(instance, info, goal)
    
    tween:Play()
    return tween
end

function Tween.fade(instance: GuiObject, targetTransparency: number, duration: number)
    local info = TweenInfo.new(duration)
    local goal = { BackgroundTransparency = targetTransparency }
    
    local tween = TweenService:Create(instance, info, goal)
    tween:Play()
    
    return tween
end

return Tween
```

---

## Error Handling

**Validate input**

```lua
function purchaseItem(player: Player, itemId: string)
    if typeof(player) ~= "Instance" or not player:IsA("Player") then
        warn("Invalid player")
        return
    end
    
    if typeof(itemId) ~= "string" then
        warn("Invalid item ID")
        return
    end
    
    local item = items[itemId]
    if not item then
        warn("Item not found:", itemId)
        return
    end
    
    -- Process purchase
end
```

**Use pcall for risky operations**

```lua
function loadPlayerData(player: Player)
    local success, result = pcall(function()
        return dataStore:GetAsync("Player_" .. player.UserId)
    end)
    
    if not success then
        warn("Failed to load data for", player.Name, ":", result)
        return getDefaultData()
    end
    
    return result
end
```

---

## Best Practices

**Use types everywhere**

```lua
type Vector = {
    x: number,
    y: number,
    z: number
}

function add(a: Vector, b: Vector): Vector
    return {
        x = a.x + b.x,
        y = a.y + b.y,
        z = a.z + b.z
    }
end
```

**Avoid global variables**

```lua
-- Bad
_G.playerData = {}

-- Good
local PlayerData = require(script.Parent.PlayerData)
```

**Clean up connections**

```lua
local connection

function init()
    connection = workspace.ChildAdded:Connect(function(child)
        print("Child added:", child.Name)
    end)
end

function cleanup()
    if connection then
        connection:Disconnect()
    end
end
```

**Use constants**

```lua
local MAX_HEALTH = 100
local WALK_SPEED = 16
local RESPAWN_TIME = 5

function resetPlayer(player: Player)
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid.Health = MAX_HEALTH
        humanoid.WalkSpeed = WALK_SPEED
    end
end
```

---

## Testing

**Unit tests with TestEZ**

```lua
return function()
    local PlayerData = require(script.Parent.PlayerData)
    
    describe("PlayerData", function()
        it("should initialize with default values", function()
            local player = game.Players:CreateLocalPlayer()
            PlayerData.init(player)
            
            local data = PlayerData.get(player)
            expect(data.coins).to.equal(0)
            expect(data.level).to.equal(1)
        end)
        
        it("should add coins", function()
            local player = game.Players:CreateLocalPlayer()
            PlayerData.init(player)
            
            PlayerData.addCoins(player, 100)
            
            local data = PlayerData.get(player)
            expect(data.coins).to.equal(100)
        end)
        
        it("should not add negative coins", function()
            local player = game.Players:CreateLocalPlayer()
            PlayerData.init(player)
            
            PlayerData.addCoins(player, -50)
            
            local data = PlayerData.get(player)
            expect(data.coins).to.equal(0)
        end)
    end)
end
```

---

## Summary

Server validates everything.
Use ModuleScripts, avoid globals.
Type all functions and data.
Keep modules small and focused.
Clean up connections and data.
Use RemoteEvents for client-server communication.

---

## Project Prompt

Write Luau for Roblox against the structure and rules above. Where they disagree with
your defaults, this file wins.

Read `../standards/Principles.md` alongside this file before starting.

**Server Authority**
- Server validates all actions
- Never trust client input
- Server owns game state
- Rate limiting on RemoteEvents

**Networking**
- RemoteEvents for client → server actions
- RemoteFunctions for client requests
- BindableEvents for server-to-server
- Minimal network traffic

**Performance**
- No .Touched in loops
- Use GetPartBoundsInBox for area checks
- Cache frequently accessed services
- Debounce expensive operations

**Error Handling**
- pcall for risky operations
- Validate remote arguments
- Graceful degradation

**Resource Management**
- Disconnect connections on cleanup
- Clear tables when done
- Destroy unused instances

### Setup

```bash
# Rojo syncs filesystem code into Roblox Studio
rojo init                     # creates default.project.json
# map src/server, src/client, src/shared to the right services in the project file
rojo serve                    # connect from the Rojo plugin in Studio
```

Tooling: `selene` for linting, `stylua` for formatting, `luau-lsp` for types. No npm, the toolchain installs via `aftman`/`rokit`.

### Deliverables

1. Complete project following architecture structure above
2. Server scripts with validation
3. Client scripts for UI/input
4. Shared modules for types
5. RemoteEvents/Functions properly secured
6. Data persistence system
7. Player management
8. Game logic modules

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] Server validates all client actions
- [ ] Type annotations on all functions
- [ ] No globals (use ModuleScripts)
- [ ] Connections cleaned up properly
- [ ] No trust in client data
- [ ] Names match domain and local convention (skills/engineering/craft/SKILL.md)
- [ ] Exploits patched (RemoteEvents)

### Security Checklist

- [ ] Server validates all RemoteEvent arguments
- [ ] Rate limiting on player actions
- [ ] Sanity checks on numerical values
- [ ] Item existence checks before granting
- [ ] Client cannot set own stats
- [ ] Exploit testing completed

### Pre-Delivery

```bash
selene src/                   # lint: no warnings
stylua --check src/           # formatting is clean
rojo build -o game.rbxlx      # builds a place file with no errors
# In Studio: playtest, then exploit-test every RemoteEvent for unvalidated input
```
