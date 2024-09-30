---
sidebar_position: 4
---
 
# Tutorial

Imagine you're making a game, the player can get Coins and XP. Wouldn't want the Player Data from simply vanishing when the player closes the game, would you?

## Setting Up the PlayerData Module

We'll need to start with getting services and the required modules
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

local DocumentService = require(ReplicatedStorage.Shared.ThirdParty.DocumentService)
local Guard = require(ReplicatedStorage.Shared.Utils.Guard)
```
Here, we're fetching two Modules:
- `DocumentService`: To simply get access to the DocumentService module
- `Guard`: This module essentially acts as a type-checker. If you don't have it, check the `Resources` page.

## Defining the Player's Data Structure

And then, we'll need to focus on the `DataSchema`, `DataInterface`, a function to validate data, and a `DocumentStore`.

**DataSchema**:
The DataSchema defines the Data that will be stored for each Player. For this, each player would have `Coins` and `XP`
```lua
type DataSchema = {
    Coins: number,
    XP: number,
}
```
It could be extended further, but to keep things simple, we'll only be keeping `Coins` and `XP`

**DataInterface**:
The DataInterface helps validate that the data types are correct. The `Guard` Module is used for this
```lua
local DataInterface = {
    Coins = Guard.Integer,
    XP = Guard.Integer,
}
```

**DataCheck**:
Of course, we'd need a function to actualyl validate the player's data. This is where `dataCheck` comes in. It ensures the data we're working with matches the format defined in `DataSchema`
```lua
local function dataCheck(value: unknown): DataSchema
	assert(type(value) == "table", "Data must be a table")
	local Value: any = value

	return {
		Coins = DataInterface.Coins(Value.Coins),
		XP = DataInterface.XP(Value.XP),
	}
end
```

**DocumentStore**:
This is where we create a new `DocumentStore` and specify the **default** values for each property, and how it should be managed.
```lua
local PlayerDataStore = DocumentService.DocumentStore.new({
	dataStore = DataStoreService:GetDataStore("PlayerData"),
	check = Guard.Check(dataCheck),
	default = {
		Coins = 0,
		XP = 0,
	},
	migrations = {
		backwardsCompatible = false, 
	},
	lockSessions = true,
})
```
- `BackwardsCompatible`: if perhaps you update your DataSchema, players will only be able to join the up-to-date servers if this option is `False`. Otherwise, players can join older servers without hinderance.
- `LockSessions`: prevents multiple or the same device from accessing a player's data simultaneously.

## Managing Player Data with Functions

Let's define two functions: `GetDocument` and `CloseDocument`

**GetDocument**:
This functions retrieves the player's document, allowing us to access their data.
```lua
function PlayerData:GetDocument(player: Player)
	return PlayerDataStore:GetDocument(`{player.UserId}`)
end
```

CloseDocument:
If a player were to leave the game, we can run this function to prevent it from being lost.
```lua
function PlayerData:CloseDocument(player: Player)
	local document = PlayerDataStore:GetDocument(`{player.Name}_{player.UserId}`)

	if not document:IsOpen() and document:IsOpenAvailable() then
		document:Open()
	end

	if document and document:IsOpen() then
		local successfulClose, closeResult = pcall(function()
			return document:Close()
		end)

		if not successfulClose then
			warn("Failed to close document for player:", player.Name, "Error:", closeResult)
		end
	else
		warn("Failed to retrieve document for player:", player.Name)
	end
end
```

## Best Practices

If you want to edit player's data, you'd do:
```lua
-- Server
local document = YourPlayerDataModule:GetDocument(player)

if not document:IsOpen() and document:IsOpenAvailable() then
	document:Open()
end

local documentClone = table.clone(document:GetCache())

documentClone.Coins = 99
documentClone.XP = 99
document:SetCache(documentClone)
```

- `document:SetCache()`: This allows us to set the Cache of the player data. And once the player leaves, we can simply save their data afterwards with `document:Close()` or `PlayerDataModule:CloseDocument(player)`

Perhaps you want to get the player's data from the Client:
```lua
-- Server
GetPlayerData:Connect(function(player)
	local playerDocument = PlayerData:GetDocument(player)

	if playerDocument then
		return playerDocument:Read()
	end

	return {}
end)

-- Client
local playerDocument = GetPlayerData:Invoke()

print(playerDocument.XP) -- 99
print(playerDocument.Coins) -- 99
```
- `playerDocument:Read()`: This simply returns the current Data of the player. Why not `playerDocument:Open()`? well, the client wouldn't be editing Data. So, we simply call `:Read()`