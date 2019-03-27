# KASHacters - ESX Multi Characters

## Introduction

> I've seen a lot of request to have ESX Multi Character, while working on my own framework I've made it multiple times and wanted to try it out with ESX. I tried to edit as less possible to the core of essentialmode or es_extended and it worked well :) PS. just don't mind the name...

## Support

> Because of the many questions I get that are different from the script I've decided to **not support** any not esx_kashacters script. Many errors are caused because of another script which might have something like 'playerJoined' as an event. There are too many different ESX scripts installed for different servers so I can't help everyone.

## Installation
**_How to install instruction._ Created By XxFri3ndlyxX**

**_How to install instruction._**

- Download the resource
- Rename the resource to esx_kashacters
- import the sql file in your database
- Go to *essentialmode\client\main.lua* and edit/comment the code.
```
--[[Citizen.CreateThread(function()
	while true do
		Citizen.Wait(0)

		if NetworkIsSessionStarted() then
			TriggerServerEvent('es:firstJoinProper')
			TriggerEvent('es:allowedToSpawn')
			return
		end
	end
end)]]--
```
- Now we edit the table and add all our identifier to make sure our character loads.
- *Edit the code in esx_kashacters\server\main.lua*
```
local IdentifierTables = {
    {table = "addon_account_data", column = "owner"},
	{table = "addon_inventory_items", column = "owner"},
    {table = "billing", column = "identifier"},
	{table = "characters", column = "identifier"},
	{table = "datastore_data", column = "owner"},
	{table = "owned_vehicles", column = "owner"},
	{table = "rented_vehicles", column = "owner"},
	{table = "society_moneywash", column = "identifier"},
	{table = "users", column = "identifier"},
    {table = "user_accounts", column = "identifier"},
	{table = "user_inventory", column = "identifier"},
	{table = "user_licenses", column = "owner"},
}
```
To get your identifier.
Do this query in your database
```
SELECT * FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME = 'columnNameHere'
``` 
Replace columnNameHere with owner then redo it with identifier
Credit @Xnubil for this query line 


The table list provided is just an example. Yours may differ depending on what you install on your server.

Once You've done all that. add start esx_kashacters to your server.cfg and then start your server.
If it's not working you can try clearing your cache. Now when the ui pops in. You see 4 square. If you click on one then the play or delete show on the left.  Okay hope this helps you guys get this awesome script going on your server.

The Fix for the ambulance is on the kashacter side is already implemented.  
<br>
Now all you have to do is go to your ambulance script and 
comment or delete
```
--[[
AddEventHandler('playerSpawned', function()
	IsDead = false

	if FirstSpawn then
		exports.spawnmanager:setAutoSpawn(false) -- disable respawn
		FirstSpawn = false

		ESX.TriggerServerCallback('esx_ambulancejob:getDeathStatus', function(isDead)
			if isDead and Config.AntiCombatLog then
				while not PlayerLoaded do
					Citizen.Wait(1000)
				end

				ESX.ShowNotification(_U('combatlog_message'))
				RemoveItemsAfterRPDeath()
			end
		end)
	end
end)
]]--
```
<br>
Then add this 
<br>

```
RegisterNetEvent('esx_ambulancejob:multicharacter')
AddEventHandler('esx_ambulancejob:multicharacter', function()
	IsDead = false

		ESX.TriggerServerCallback('esx_ambulancejob:getDeathStatus', function(isDead)
			if isDead and Config.AntiCombatLog then
				ESX.ShowNotification(_U('combatlog_message'))
				RemoveItemsAfterRPDeath()
			end
		end)
end)
```
Then change this 
```
function RespawnPed(ped, coords, heading)
	SetEntityCoordsNoOffset(ped, coords.x, coords.y, coords.z, false, false, false, true)
	NetworkResurrectLocalPlayer(coords.x, coords.y, coords.z, heading, true, false)
	SetPlayerInvincible(ped, false)
	TriggerEvent('playerspawned', coords.x, coords.y, coords.z)
	ClearPedBloodDamage(ped)

	ESX.UI.Menu.CloseAll()
end
```
to
```
function RespawnPed(ped, coords, heading)
	SetEntityCoordsNoOffset(ped, coords.x, coords.y, coords.z, false, false, false, true)
	NetworkResurrectLocalPlayer(coords.x, coords.y, coords.z, heading, true, false)
	SetPlayerInvincible(ped, false)
	TriggerEvent('esx_ambulancejob:multicharacter', coords.x, coords.y, coords.z)
	ClearPedBloodDamage(ped)

	ESX.UI.Menu.CloseAll()
end
```
If you do not do this last part once you repawn after death you will be frozen into place.
<br>

To fix The datastore duplicated entry download this https://github.com/XxFri3ndlyxX/esx_datastore   
<br>
Or  
<br>
Add this code to your server/main.lua  
<br>
```-- Fix for kashacters duplication entry --
-- Fix was taken from this link --
-- https://forum.fivem.net/t/release-esx-kashacters-multi-character/251613/448?u=xxfri3ndlyxx --
AddEventHandler('esx:playerLoaded', function(source)

  local result = MySQL.Sync.fetchAll('SELECT * FROM datastore')

	for i=1, #result, 1 do
		local name   = result[i].name
		local label  = result[i].label
		local shared = result[i].shared

		local result2 = MySQL.Sync.fetchAll('SELECT * FROM datastore_data WHERE name = @name', {
			['@name'] = name
		})

		if shared == 0 then

			table.insert(DataStoresIndex, name)
			DataStores[name] = {}

			for j=1, #result2, 1 do
				local storeName  = result2[j].name
				local storeOwner = result2[j].owner
				local storeData  = (result2[j].data == nil and {} or json.decode(result2[j].data))
				local dataStore  = CreateDataStore(storeName, storeOwner, storeData)

				table.insert(DataStores[name], dataStore)
			end
		end
	end

	local _source = source
	local xPlayer = ESX.GetPlayerFromId(_source)
  	local dataStores = {}
  
	for i=1, #DataStoresIndex, 1 do
		local name      = DataStoresIndex[i]
		local dataStore = GetDataStore(name, xPlayer.identifier)

		if dataStore == nil then
			MySQL.Async.execute('INSERT INTO datastore_data (name, owner, data) VALUES (@name, @owner, @data)',
			{
				['@name']  = name,
				['@owner'] = xPlayer.identifier,
				['@data']  = '{}'
			})

			dataStore = CreateDataStore(name, xPlayer.identifier, {})
			table.insert(DataStores[name], dataStore)
		end

		table.insert(dataStores, dataStore)
	end

	xPlayer.set('dataStores', dataStores)
end)
```
<br>
<br>

I do not recommend this as it will mostly lead to errors.
Character Switch Feature. Code posted by @Pattzki [Code Link](https://forum.fivem.net/t/release-esx-kashacters-multi-character/251613/298?u=xxfri3ndlyxx)
 
Put this line of code  in client/main.lua
```
RegisterCommand('switch', function()
TriggerEvent('kashactersC:ReloadCharacters')
end)
```
Then a post related to this to fix issue with skin not loading.
Add the following code in server/main.lua
```
RegisterServerEvent(“kashactersS:CharacterChosen”)
AddEventHandler(‘kashactersS:CharacterChosen’, function(charid, ischar)
local src = source
local spawn = {}
SetLastCharacter(src, tonumber(charid))
SetCharToIdentifier(GetPlayerIdentifiers(src)[1], tonumber(charid))
if ischar == “true” then
spawn = GetSpawnPos(src)
TriggerClientEvent(“kashactersC:Skinchanger”, src)
else
TriggerClientEvent(‘skinchanger:loadDefaultModel’, src, true, cb)
spawn = { x = -1045.31, y = -2750.69, z = 20.36 } – DEFAULT SPAWN POSITION
end
TriggerClientEvent(“kashactersC:SpawnCharacter”, src, spawn)
end)
```
Then add this code in client/main.lua
```
RegisterNetEvent(‘kashactersC:Skinchanger’)
AddEventHandler(‘kashactersC:Skinchanger’, function(source)
local source_ = source
ESX.TriggerServerCallback(‘esx_skin:getPlayerSkin’, function(skin, jobSkin)
TriggerEvent(‘skinchanger:loadSkin’, skin)
end)
end)
```
This guide has been updated with all the posted fix that was posted after i made my post.

> **Pay ATTENTION: You have to call the resource 'esx_kashacters' in order for the javascript to work!!!**

## How it works
> What this script does it manipulates ESX for loading characters
So when you are choosing your character it changes your steam id which is normally steam: to Char: this prevents ESX from loading another character because it is looking for you exact steamid. So when you choose your character it will change it from Char: to your normal steam id (steam:). When creating a new character it will spawn you without an exact steamid which creates a new database entry for your steamid

## Credits

> ESX Framework and **KASH** AND **Onno204** for creating the resource. You can do whatever the f with it what you want but it is nice to give the main man credits ;)

- LOVE KASH (Discord: KASH#0205)
