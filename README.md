# kAPI
> A Virtual Machine Environment forked from the powerful xAPI repository.

kAPI is a powerful VME useful for debugging and anti-cheat development, built around the Unified Naming Convention standard (UNC).
Click [here](https://roblox-video-converter.vercel.app/) to convert a video to kAPI's format

## Features
- 90%+ UNC score out of the box
- Full Metatable manipulation
- Function manipulation
- Persistent filesystem per player
- Custom asset loading (PNG, JPG, MP4, MP3, TXT...)
- Actor/thread isolation
- Basic Lua 5.1 runner (`--!Lua`)
- Basic HTML runner (`--!html`)
- WebSocket support
- custom nil instances space (`getnilspace()`)
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
- HashLib, LZ4, Signal, and more

## License
MIT
