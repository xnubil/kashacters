# KASHacters - ESX Multi Characters

## Introduction

> I've seen a lot of request to have ESX Multi Character, while working on my own framework I've made it multiple times and wanted to try it out with ESX. I tried to edit as less possible to the core of essentialmode or es_extended and it worked well :) PS. just don't mind the name...

## Support

> Because of the many questions I get that are different from the script I've decided to **not support** any not esx_kashacters script. Many errors are caused because of another script which might have something like 'playerJoined' as an event. There are too many different ESX scripts installed for different servers so I can't help everyone.

## Installation
**_How to install instruction._ Created By XxFri3ndlyxX**

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


The table list provided is just an example. Yours may difer depending on what you install on your server.

Once You've done all that. add start esx_kashacters to your server.cfg and then start your server.
If it's not working you can try clearing your cache. Now when the ui pops in. You see 4 square. If you click on one then the play or delete show on the left.  Okay hope this helps you guys get this awesome script going on your server.

@KASH maybe you can add this to the op :)


**Fix for esx_ambulancejob**  Credit to @OfficialRobin for this fix 

Open esx_ambulancejob/client/main.lua
Find 
```
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
```
and just under the last end) 
add
```
RegisterNetEvent('esx_ambulancejob:multicharacter')
AddEventHandler('esx_ambulancejob:multicharacter', function()
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
```

Then go to esx_kashacters/client/main.lua
Find 

```
RegisterNetEvent('kashactersC:SpawnCharacter')
AddEventHandler('kashactersC:SpawnCharacter', function(spawn)
    TriggerServerEvent('es:firstJoinProper')
    TriggerEvent('es:allowedToSpawn')
    SetTimecycleModifier('default')
    local pos = spawn
    SetEntityCoords(GetPlayerPed(-1), pos.x, pos.y, pos.z)
    DoScreenFadeIn(500)
    Citizen.Wait(500)
    cam2 = CreateCamWithParams("DEFAULT_SCRIPTED_CAMERA", -1355.93,-1487.78,520.75, 300.00,0.00,0.00, 100.00, false, 0)
    PointCamAtCoord(cam2, pos.x,pos.y,pos.z+200)
    SetCamActiveWithInterp(cam2, cam, 900, true, true)
    Citizen.Wait(900)
    cam = CreateCamWithParams("DEFAULT_SCRIPTED_CAMERA", pos.x,pos.y,pos.z+200, 300.00,0.00,0.00, 100.00, false, 0)
    PointCamAtCoord(cam, pos.x,pos.y,pos.z+2)
    SetCamActiveWithInterp(cam, cam2, 3700, true, true)
    Citizen.Wait(3700)
    PlaySoundFrontend(-1, "Zoom_Out", "DLC_HEIST_PLANNING_BOARD_SOUNDS", 1)
    RenderScriptCams(false, true, 500, true, true)
    PlaySoundFrontend(-1, "CAR_BIKE_WHOOSH", "MP_LOBBY_SOUNDS", 1)
    FreezeEntityPosition(GetPlayerPed(-1), false)
    Citizen.Wait(500)
    SetCamActive(cam, false)
    DestroyCam(cam, true)
    IsChoosing = false
    DisplayHud(true)
    DisplayRadar(true)
end)
```

and replace with 

```
RegisterNetEvent('kashactersC:SpawnCharacter')
AddEventHandler('kashactersC:SpawnCharacter', function(spawn, isnew)
    TriggerServerEvent('es:firstJoinProper')
    TriggerEvent('es:allowedToSpawn')
    TriggerEvent('esx_ambulancejob:multicharacter')

    SetTimecycleModifier('default')
    local pos = spawn
    SetEntityCoords(GetPlayerPed(-1), pos.x, pos.y, pos.z)
    DoScreenFadeIn(500)
    Citizen.Wait(500)
    cam2 = CreateCamWithParams("DEFAULT_SCRIPTED_CAMERA", -1355.93,-1487.78,520.75, 300.00,0.00,0.00, 100.00, false, 0)
    PointCamAtCoord(cam2, pos.x,pos.y,pos.z+200)
    SetCamActiveWithInterp(cam2, cam, 900, true, true)
    Citizen.Wait(900)
	
 if isnew then
	TriggerEvent('esx_identity:showRegisterIdentity')
 end

    cam = CreateCamWithParams("DEFAULT_SCRIPTED_CAMERA", pos.x,pos.y,pos.z+200, 300.00,0.00,0.00, 100.00, false, 0)
    PointCamAtCoord(cam, pos.x,pos.y,pos.z+2)
    SetCamActiveWithInterp(cam, cam2, 3700, true, true)
    Citizen.Wait(3700)
    PlaySoundFrontend(-1, "Zoom_Out", "DLC_HEIST_PLANNING_BOARD_SOUNDS", 1)
    RenderScriptCams(false, true, 500, true, true)
    PlaySoundFrontend(-1, "CAR_BIKE_WHOOSH", "MP_LOBBY_SOUNDS", 1)
    FreezeEntityPosition(GetPlayerPed(-1), false)
    Citizen.Wait(500)
    SetCamActive(cam, false)
    DestroyCam(cam, true)
    IsChoosing = false
    DisplayHud(true)
    DisplayRadar(true)
end)
```

> **Pay ATTENTION: You have to call the resource 'esx_kashacters' in order for the javascript to work!!!**

## How it works
> What this script does it manipulates ESX for loading characters
So when you are choosing your character it changes your steam id which is normally steam: to Char: this prevents ESX from loading another character because it is looking for you exact steamid. So when you choose your character it will change it from Char: to your normal steam id (steam:). When creating a new character it will spawn you without an exact steamid which creates a new database entry for your steamid

## Credits

> ESX Framework and **KASH** AND **Onno204** for creating the resource. You can do whatever the f with it what you want but it is nice to give the main man credits ;)

- LOVE KASH (Discord: KASH#0205)
