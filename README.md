# kAPI
> A Virtual Machine forked from the powerful xAPI repository.

kAPI is a vanilla Luau executor API built entirely inside Roblox — no injection, no DLL, no external tools required.

## Features
- 90%+ UNC score out of the box
- Pure Luau — runs inside any Roblox game
- Full Instance proxy system with shared coreMeta
- Persistent filesystem per player
- Audio playback (MP3, WAV, OGG) via WavPlayer
- Video playback
- Drawing library (Line, Circle, Square, Triangle, Image, etc.)
- Custom asset loading (PNG, JPG, GIF, MP3, MP3, TXT)
- Actor/thread isolation
- Lua 5.1 runner (`--!Lua`)
- HTML runner (`--!html`)
- WebSocket support
- hookmetamethod, hookfunction, full debug.* library
- scriptspace for getscripthash / getscriptclosure
- FFlag system for granular feature control

## UNC Score
| Configuration | Score |
|---|---|
| Full (all fflags on) | 90%+ |
| No fake compatibility | ~87% |
| No Fiu manipulation | ~84% |
| Minimal (both off) | ~72% |

## Credits
- **xAPI** by [@anon2375517](https://devforum.roblox.com/t/xapi-a-powerful-pentesting-and-debugging-tool/2688148) — original foundation
- **Fiu** by Rerumu — Luau bytecode interpreter
- **LuaCeption** — Luau source compiler
- HashLib, LZ4, Signal, TypeChecking, and more

## License
MIT
