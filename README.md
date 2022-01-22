# Vero's Save Game
## Full documentation
#### This readme serves as the documentation for my small code bit for GameMaker studio 2 that helps developers implement a very simple save and load function into their games with just a few lines of code.
##### If you need any help, don't hesitate to reach out to me at dev.predationwebsite@outlook.com and I'll get back to you as soon as possible!

### Quickstart
If you don't care for any of the advanced features, and just want a quick and easy save and load system, here's what you need to do.
1. In the very first room of your game, on any layer, please create the ```VeroSaveSystem``` object. This object is persistent, you don't have to worry about it any longer.
2. From now on, the system will be saving the state of all of your instances, if at any point you'd like to save the game to a file, simply do the following:
```
if (instance_exists(VeroSaveSystem))
{
	VeroSaveSystem.SaveGame();
}
```
3. And similarly, if you'd like to load the previously saved game, use the following code:
```
if (instance_exists(VeroSaveSystem))
{
	VeroSaveSystem.LoadGame();
}
```
4. And last, you need to include TWO functions, a ```save()``` and ```load()``` function, in each object's Create event you have which includes variables you'd like to save. In an example, if you'd like to save
the player object's X and Y position, you'd implement this in such a way:
```
function save()
{
	var saveData = ds_map_create();
	ds_map_add(saveData, "PlayerX", x);
	ds_map_add(saveData, "PlayerY", y);
	return saveData;
}

function load(dsMap)
{
	x = dsMap[? "PlayerX"];
	y = dsMap[? "PlayerY"];
}
```

> IMPORTANT! Both the ```save()``` and ```load()``` functions MUST be lowercase.

Which game the player was last in will also be saved when the ```SaveGame()``` function is called.
And that's it! You now have a very simple and working Save / Load routine implemented, however, there are some other things you might want to configure if your
game gets any more complex than a quick 1 day project.

First and foremost...
### Configuring ignored rooms
In almost every case, we don't want rooms like our main menu, cutscenes or anything in between to be included in the save and load routine.
There's an easy way to add rooms that we want to ignore.
Inside the ```VeroSaveSystem``` object, it's create event, we'll see a ```RestrictedSaveRooms``` array.
This array already contains the demo placeholder, but we can easily remove this and add more. All we need to do is expand
the array_push function, and add our own rooms.
As an example, if we had a room called "rm_Cutscene" and "rm_MainMenu" and we wanted these to be ignored, we'd do the following:
```
array_push(RestrictedSaveRooms,
	string(rm_Cutscene),
	string(rm_MainMenu)
);
```
Make sure you convert each room ID to a string with the ```string()``` function, as shown above, and include a comma
in between each.   

See [array_push()](https://manual.yoyogames.com/#t=GameMaker_Language%2FGML_Reference%2FVariable_Functions%2Farray_push.htm) for more information.
Rooms included in this array won't be tracked, however, it is still possible to call the LoadGame function from here.

### Setting up multiple save slots
If you want multiple save slots to be available, you need to create your own system to keep track of how many you'd like to include, and you'd
loop through each file in the game's appdata directory, looking for files matching the PREFIX and EXT you set. [More info](https://github.com/VeroSquirrel/VerosSaveSystemDocumentation/blob/main/README.md#setting-a-custom-prefix-and-extension-for-the-save-file)


To set a different slot where the system will save or load from, you may use the ```SetCurrentSlot(int)``` function.
Example:
```
VeroSaveSystem.SetCurrentSlot(2);
```
This would set the current save slot to 2. PLEASE NOTE: Save slots start from 0, but a "Human friendly" display option is available, which saves
the slot name inside the save file as a more human friendly number. In this variable, Save Slot 0 would become Slot 1, and etc...
After setting the slot, every subsequent ```SaveGame``` and ```LoadGame``` function will be executed with this slot number.
It's also possible to call ```SaveGame``` and ```LoadGame``` functions with a slot number independent of what's currently set, the function
will be executed on the slot passed in, without the active save slot being changed.
As an example, having save slot 0 set in the object, and calling ```LoadGame(2)``` will load the save file that was saved with slot 2 (if it exists),
and after loading, the active slot will still be 0. Same can be done with ```SaveGame``` in case the player wants to copy a save slot, for example.

### Setting a custom Prefix and Extension for the save file
In many cases we want our save file to be unique, both in terms of the filename and the extension. In truth, this system doesn't actually care
what extension a file is, as it saves everything from a buffer in binary data. For this very reason, there's a very simple way of configuring
our prefix and extension inside the main object's create event.
```
#macro SAVE_FILE_PREFIX "data"
#macro SAVE_FILE_EXT ".sav"
```
A save file will be constructed in the following way: ```SAVE_FILE_PREFIX + Slot + SAVE_FILE_EXT``` which in this case would be ```data0.sav```
Based on this, we can easily loop through our game's data folder to look for already existing save files and list them, then simply use the
```VeroSaveSystem.SetCurrentSlot()``` function to set the currently selected save slot to the desired one.

Please be mindful of Windows filename character limit which doesn't accept filenames
larger than 255 characters, as well as illegal characters or reserved names, such as CON, PRN, AUX and more, [MS docs for more info](https://docs.microsoft.com/en-us/windows/win32/fileio/naming-a-file).

Same goes for Linux users, with the maximum character limit being 255 bytes, [more info here](https://askubuntu.com/questions/859945/what-is-the-maximum-length-of-a-file-path-in-ubuntu)

### Save and load explained further, and saving more complex data structures
As seen in the quickstart guide, this save system will look for objects which implement two functions, a ```save()``` and ```load()``` functions.
Here we'll examine further on how these two work and how they should be implemented.
First, the ```save``` function. This function should:
1. Create a DS map, name is completely up to the developer.
2. Fill the DS map with the variables that need to be saved. Example: Player's X and Y position, Player health points, activated state of a switch, etc...
3. Return this DS map to the caller.

When it's time to load the game, the ```load``` function gets called. When writing this function, we can safely assume that the system will
pass the same structured DS map that we created before using the ```save``` function, hence if we wanted to "load" an object, all we have to do
is reverse what we did in our save function. This function should:
1. Accept a dsMap object as it's argument
2. Set this object's variables to the value contained within the dsMap it got from the save system.

Saving a more complex structure, such as a DS map, can be achieved by first using [json_encode()](https://manual.yoyogames.com/#t=GameMaker_Language%2FGML_Reference%2FFile_Handling%2FEncoding_And_Hashing%2Fjson_encode.htm)
on it inside the ```save``` function, then when setting this same map back in the ```load``` function, we use [json_decode()](https://manual.yoyogames.com/#t=GameMaker_Language%2FGML_Reference%2FFile_Handling%2FEncoding_And_Hashing%2Fjson_decode.htm).
Please also note: If you use an incremental counter for your DS maps, make sure you convert them to a string when saving and loading, otherwise you won't be able to properly retrieve the information from within.
There is an example of this inside the demo code, please refer to that!

An example for the Demo player's save function:
```
function save()
{
	var saveData = ds_map_create();
	ds_map_add(saveData, "VeryImportantData", VeryImportantData);
	ds_map_add(saveData, "AnotherVeryImportantData", AnotherVeryImportantData);
	ds_map_add(saveData, "MoreImportantData", json_encode(MoreImportantData));
	ds_map_add(saveData, "CurIndex", CurIndex);
	ds_map_add(saveData, "PlayerX", x);
	ds_map_add(saveData, "PlayerY", y);
	ds_map_add(saveData, "SaveOnExit", SaveOnExit);
	return saveData;
}
```
And it's load function:
```
function load(dsMap)
{
	VeryImportantData = dsMap[? "VeryImportantData"];
	AnotherVeryImportantData = dsMap[? "AnotherVeryImportantData"];
	MoreImportantData = json_decode(dsMap[? "MoreImportantData"]);
	CurIndex = dsMap[? "CurIndex"];
	x = dsMap[? "PlayerX"];
	y = dsMap[? "PlayerY"];
	SaveOnExit = dsMap[? "SaveOnExit"];
}
```

### Custom error handling and handling save file overwrites
Things will go wrong, no matter how much we try to prevent it, but we can prepare for it!
The system by itself will handle a few simple exceptions that may occur, namely those related to saving or loading a savegame file.
These sensitive routines are already put inside a try catch block, inside the ```VeroSaveSystem```'s CREATE event, but all they do
is log the exception's long message to the debug console. If you'd like to do something special on each different exception, for example, when
a given file doesn't exist, you may do so by calling your own functions inside those catch events.
Example:
```
catch (Exception)
{
  show_debug_message("Save failed: " + Exception.longMessage);
  game_end(1);
}
```
```
var filename = SAVE_FILE_PREFIX + string(slot) + SAVE_FILE_EXT;
if (!file_exists(filename))
{
  show_debug_message("Load failed: Save file doesn't exist.");
  /* Put your own code here to handle what happens
     if the save file doesn't exist. */
  return;
}
```

Overwriting a save file happens automatically in the included demo, but the system WILL catch this event, and we can handle it by calling our own function
inside the ```VeroSaveSystem```'s ```SaveGame()``` function:
```
var filename = SAVE_FILE_PREFIX + string(slot) + SAVE_FILE_EXT;
if (file_exists(filename))
{
  /* Put your own code here to ask the user if they want to
  overwrite the current save file.
  In this demo, this happens automatically. */
  show_debug_message("Overwriting file: " + filename);
}
```

### Setting which layer we'd like to use
Once our game gets big enough, we might not want to iterate through every single object in the entire game when saving or loading, and for that, we can tell the save system which layer to look for specifically. Currently, you may only set one. This means that, if you create an instance layer, and only put objects which have a ```save()``` and ```load()``` function there, the game will ONLY iterate through those objects, ignoring all the other instance layers completely, potentially speeding up save and load times.
All we have to do is edit the ```InstanceLayerToUse``` variable in the ```VeroSaveSystem``` object's create event, and set it to the string of the layer we'd like to check. Later on, it'll be possible to add more than one layer.
Example:
```
InstanceLayerToUse = "Instances";
```
