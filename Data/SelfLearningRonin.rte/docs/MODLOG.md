# MODLOG

## Facts
- Game version the player runs: 6.2.2 (console: "this game version (v6.2.2)"). The repository the mod was first written against is 7.0.0-dev (`meson.build`): the two Lua APIs differ.
- `MusicMan` exists only in the development version. 6.2.2 plays music with `AudioMan:ClearMusicQueue()`, `AudioMan:PlayMusic(path, loops, volume)`, `AudioMan:QueueSilence(s)`, `AudioMan:QueueMusicStream(path)`.
- `LuaMan` file API (`FileOpen/FileReadLine/FileWriteLine/FileEOF/FileExists/FileRemove/FileRename`) is the same in 6.2.2 and 7.0.0-dev. Only paths containing `.rte` are allowed; `FileWriteLine` does not add a newline, `FileReadLine` keeps it.
- `DataModule::CheckSupportedGameVersion`: no `SupportedGameVersion` = assertion; mismatch on a development build = assertion; the module still loaded in the player's run.
- `AddInventoryItem(nil)` crashes the game; every item goes through `AddItem()` in the script.
- Lua `AddActor` gives ownership to the engine: read `UniqueID` before calling it and never use the object afterwards.
- Groups in Ronin.rte (identical in 6.2.2 and 7.0.0-dev): actors `Actors`, `Actors - Light`, `Actors - Heavy`, `Brains`; weapons `Weapons - Primary/Secondary/Light/Heavy/CQB/Explosive/Sniper/Melee`, `Bombs - Grenades`.

## What failed and why
1. Mod absent from the activity list: `SupportedGameVersion` was missing from `Index.ini`. Fixed.
2. "Activity was NOT started due to errors": `attempt to index global 'MusicMan'` (script line 341). Reproduced on the test rig by removing `MusicMan`; fixed with a helper that uses whichever music API exists.
3. Red crash dialog (EXCEPTION_ACCESS_VIOLATION): cause not identified, no `AbortLog.txt` available. Hypothesis: follow-up of the failed start. Not reproduced.

## Checks done
- Every engine method and property the script uses was looked up in the 6.2.2 Lua bindings by script: all present (only `PlayDynamicSong`, which is skipped when `MusicMan` is absent).
- Test rig: LuaJIT built from the repo's own sources plus a fake engine and a toy battle; learner unit tests; save/load round trip including a corrupted file.

## Diagnostics added
- Every important step prints a line starting with `Ronin AI:` to the in-game console (`~`). The console is also dumped into `AbortLog.txt` when the game crashes, so the last `Ronin AI:` line shows how far the mod got.
- An error inside the AI's once-per-second thinking no longer pauses the whole activity: it is printed (up to 5 times) and then the AI switches itself off while the game keeps running.
- If Ronin.rte is not loaded the activity says so instead of failing later.

## Next step
Waiting for the first real run on 6.2.2: what the AI did, console lines "Ronin AI sortie ...", any error text.
