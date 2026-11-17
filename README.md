# kAPI

> A Roblox executor **emulator** written entirely in Luau. It runs inside your own Roblox Studio session and reproduces the behavior of real executor APIs (UNC / sUNC) so scripts, tools, and anti-cheats can be built and tested without a real executor.

kAPI is a fork of [xAPI](https://devforum.roblox.com/t/xapi-a-powerful-pentesting-and-debugging-tool/2688148), heavily expanded by [@kumori-kuno](https://github.com/kumori-kuno).

---

## What kAPI is (and isn't)

**kAPI is:**
- A virtual machine environment written in plain Luau, loaded like any other ModuleScript.
- An emulation layer that recreates executor globals (`getgenv`, `hookfunction`, `hookmetamethod`, `getgc`, `readfile`, `Drawing`, `gethui`, …) with the same names, signatures, and behavior described by the UNC/sUNC standards.
- A tool for **debugging, script testing, and anti-cheat development** — you can see exactly how executor scripts behave and build detections against them, all inside your own place.

**kAPI is not:**
- A real executor. It doesn't inject anything, doesn't attach to the Roblox client, and doesn't run in live games you don't own.
- Able to bypass or interact with Roblox's real anti-cheat. It only uses normal Roblox engine features, so it can only affect the experience it's installed in (typically your own Studio session).

---

## How it works

kAPI's core idea is to **own the VM that user code runs in**:

1. **Interpreted execution.** `loadstring` compiles source with **LuauCeption** (a Luau compiler written in Luau) and runs the bytecode on **Fiu** (a Luau bytecode interpreter written in Luau). Because kAPI controls the interpreter, it can see every closure, upvalue, constant, proto, stack frame, and call — which is what makes globals like `getgc`, `getupvalues`, `getconstants`, `getprotos`, `getstack`, and `hookfunction` real instead of faked.
2. **Instance proxies.** Scripts receive proxied versions of `game` and its Instances. That enables `hookmetamethod` (`__index`, `__newindex`, `__namecall`), `getnamecallmethod`, `getrawmetatable`, `cloneref`, `getconnections`, `firesignal`, and hidden/spoofed properties.
3. **Environment injection.** Executor globals are injected into the script's environment and exposed through `getgenv()`, kept separate from the game's own `getrenv()`.
4. **Identity emulation.** `getidentity` / `setidentity` emulate thread identity levels, with bridges for elevated contexts in Studio.
5. **Actors / parallel Luau.** `run_on_actor`, `getactors`, `getluastate`, and comm channels emulate executor actor APIs using real Actors.
6. **Server-backed features.** A server-side component handles things a client script can't do alone: HTTP requests, a persistent per-player filesystem, `game:GetObjects`, WebSockets, audio, images, and video.

---

## Features

- **High UNC compatibility** — 90%+ on the UNC test with all FFlags on.
- **Closures & functions** — `hookfunction`, `newcclosure`, `iscclosure`, `islclosure`, `clonefunction`, `checkcaller`, `getcallingscript`.
- **Debug library** — `getupvalue(s)`, `setupvalue`, `getconstant(s)`, `setconstant`, `getproto(s)`, `getstack`, `setstack`, `getinfo`.
- **Metatables** — `getrawmetatable`, `setrawmetatable`, `hookmetamethod`, `setreadonly`, `isreadonly`, `getnamecallmethod`.
- **Environment** — `getgenv`, `getrenv`, `getgc`, `getreg`, `getsenv`, `getloadedmodules`, `getrunningscripts`.
- **Instances & signals** — `getconnections`, `firesignal`, `fireclickdetector`, `fireproximityprompt`, `firetouchinterest`, `gethiddenproperty`, `sethiddenproperty`, `cloneref`, `compareinstances`, `getnilinstances`, `getnilspace`.
- **Filesystem** — `readfile`, `writefile`, `appendfile`, `listfiles`, `isfile`, `isfolder`, `makefolder`, `delfile`, `delfolder`, `loadfile`. Persistent per player.
- **Network** — `request` / `http_request`, `game:HttpGet`, `WebSocket.connect`.
- **Assets** — `getcustomasset` for images, audio, video, fonts, and text (PNG, JPG, MP4, MP3, TXT …), and `game:GetObjects`.
- **UI** — `gethui`, the `Drawing` library, console functions.
- **Crypt** — `crypt.base64encode/decode`, `crypt.encrypt/decrypt`, `crypt.hash`, `crypt.generatekey`, `crypt.generatebytes`, plus LZ4 compression.
- **Actors** — `run_on_actor`, `getactors`, `getluastate`, comm channels.
- **Extras** — Lua 5.1 runner (`--!Lua`), HTML runner (`--!html`), an in-game IDE with autocomplete, and an FFlag system to switch subsystems on or off.

---

## Installation

1. Download `kAPI.rbxm` from this repository.
2. Insert it into your place in Roblox Studio and keep its structure intact (kAPI has both client and server parts).
3. From a **LocalScript**, run:

```lua
require(path_to_kAPI)()
```

The environment is injected into the calling script automatically. kAPI is most capable when code runs through its `loadstring`, since that code executes on kAPI's own interpreter:

```lua
loadstring([[
    print(identifyexecutor())
    print(getgenv())
]])()
```

Video files for kAPI's video player can be converted with the [kAPI video converter](https://roblox-video-converter.vercel.app/).

---

## FFlags

kAPI's subsystems can be switched on or off through its FFlags module (filesystem, custom globals, fake compatibility shims, Fiu manipulation, drawing, websockets, and more). Turning flags off makes kAPI lighter but lowers compatibility:

| Configuration | UNC score |
|---|---|
| Full (all FFlags on) | 90%+ |
| No fake compatibility | ~87% |
| No Fiu manipulation | ~84% |
| Minimal (both off) | ~72% |

Flags can also be read and set at runtime with `getfflag` / `setfflag`.

---

## Known limitations

- **Emulation, not engine access.** Scripts that use the real `game` (not kAPI's proxy) see the world normally, so a few behaviors that depend on engine internals (for example `getscriptbytecode` on the game's own scripts) are approximated or unavailable.
- **Compiler version.** LuauCeption targets an older Luau release, so very new syntax or heavily obfuscated scripts may fail to compile.
- **Performance.** Code run through `loadstring` executes on an interpreter written in Luau, so it is slower than native Luau.
- **Unions.** `game:GetObjects` can't recover real geometry for `PartOperation` (unions).

---

## Docs

kAPI follows the [UNC](https://github.com/unified-naming-convention/NamingStandard) / sUNC naming standards, with [Madium's docs](https://getmadium.net/docs) as the behavior reference. If a global below exists in those standards, it's meant to behave exactly as documented there. Most globals also answer to their common aliases (`get_hidden_gui`, `http_request`, `syn.request`, …).

### Directives (first line of a `loadstring` chunk)

| Directive | Effect |
|---|---|
| `--!Lua` | Runs the chunk as Lua 5.1 instead of Luau |
| `--!html` | Renders the chunk as an HTML window GUI |
| `--!NoEnv` | Runs the chunk without kAPI's environment (plain sandbox) |

### Environment

| Global | Description |
|---|---|
| `getgenv()` | The executor's global table, shared by every kAPI script. Same table every call. |
| `getrenv()` | The game's environment, kept separate from executor globals. |
| `getsenv(script)` | A script's environment table (same table per script). |
| `getreg()` / `debug.getregistry()` | kAPI's registry table. |
| `getgc(includeTables?)` | Every function (and optionally table) kAPI's VM has created. |
| `filtergc(type, options, returnOne?)` | Search `getgc` by type and fields. |
| `getloadedmodules()` / `getrunningscripts()` | ModuleScripts that were required / scripts currently running. |
| `identifyexecutor()` | Returns kAPI's name and version. |
| `getidentity()` / `setidentity(n)` | Read or emulate the thread identity level. |
| `getfflag(name)` / `setfflag(name, value)` | Read or set kAPI's emulated engine FFlags. |

### Closures

| Global | Description |
|---|---|
| `hookfunction(target, hook)` | Replaces `target` everywhere it's referenced; returns a callable original. |
| `restorefunction(target)` / `isfunctionhooked(f)` | Undo or check a hook. |
| `newcclosure(f)` | Wraps `f` as a real C closure. |
| `iscclosure(f)` / `islclosure(f)` | Check whether a function is a C or Luau closure. |
| `isexecutorclosure(f)` | `true` if kAPI (not the game) created the function. |
| `clonefunction(f)` | A copy of `f` that shares its prototype. |
| `checkcaller()` | `true` when the current thread belongs to kAPI. |
| `getcallingscript()` | The script that called the current function. |
| `oth.hook` / `oth.unhook` | Hooks that run on a separate thread. |

### Debug

| Global | Description |
|---|---|
| `debug.getupvalue(s)` / `debug.setupvalue` | Read or write a function's upvalues. |
| `debug.getconstant(s)` / `debug.setconstant` | Read or write a function's constants. |
| `debug.getproto(s)` | A function's inner functions. |
| `debug.getstack` / `debug.setstack` | Read or write a stack frame's registers. |
| `debug.getinfo(f)` | Function info (name, source, line, params). |
| `getlocal` / `getlocals` / `setlocal` | Read or write local variables by stack level. |
| `decompile(f)` | Source for a function compiled by kAPI. |

### Metatables

| Global | Description |
|---|---|
| `getrawmetatable(obj)` / `setrawmetatable(obj, mt)` | Bypass `__metatable` locks. |
| `hookmetamethod(obj, name, hook)` | Hook `__index`, `__newindex`, `__namecall` and others on instances. |
| `getnamecallmethod()` / `setnamecallmethod(name)` | The method name inside a `__namecall` hook. |
| `setreadonly(t, bool)` / `isreadonly(t)` | Lock or unlock a table. |

### Instances & signals

| Global | Description |
|---|---|
| `cloneref(instance)` | A second, distinct reference to the same instance. |
| `compareinstances(a, b)` | `true` if two references point to the same instance. |
| `gethiddenproperty` / `sethiddenproperty` | Read or write non-scriptable properties. |
| `setscriptable(obj, prop, bool)` | Make a hidden property scriptable. |
| `getnilinstances()` / `getnilspace()` | Instances parented to `nil` / kAPI's nil-instance space. |
| `getconnections(signal)` | Connection objects (`Fire`, `Disable`, `Enable`, `Function`, …). |
| `firesignal(signal, ...)` | Fire a signal's connections with your arguments. |
| `replicatesignal(signal, ...)` | Fire a signal as if it came from the server/engine. |
| `fireclickdetector` / `fireproximityprompt` / `firetouchinterest` | Trigger interactions directly. |
| `gethui()` | A hidden UI container for your ScreenGuis, invisible to the game's own scripts. |

### Filesystem (persistent, per player)

| Global | Description |
|---|---|
| `readfile` / `writefile` / `appendfile` | Read, overwrite, or append to a file. |
| `listfiles(path)` | Files and folders inside `path` (`""` is the workspace root). |
| `isfile` / `isfolder` | Existence checks. |
| `makefolder` / `delfile` / `delfolder` | Create or delete. |
| `loadfile(path)` | Compile a file into a function. |
| `getcustomasset(path)` | An asset id for an image, audio, video, font, or text file. |

### Network

| Global | Description |
|---|---|
| `request(options)` | HTTP request (`Url`, `Method`, `Headers`, `Body`). |
| `game:HttpGet(url)` | GET a URL and return the body. |
| `WebSocket.connect(url)` | A WebSocket with `Send`, `Close`, `OnMessage`, `OnClose`. |
| `game:GetObjects(assetId)` | Load a model as a table of real, built Instances. |

### Input

| Global | Description |
|---|---|
| `mouse1click` / `mouse1press` / `mouse1release` (and `mouse2…`) | Simulate mouse buttons. |
| `mousemoverel` / `mousemoveabs` / `mousescroll` | Simulate mouse movement. |
| `keypress` / `keyrelease` | Simulate keys. |

### Crypt

| Global | Description |
|---|---|
| `crypt.base64encode` / `crypt.base64decode` | Base64. |
| `crypt.encrypt` / `crypt.decrypt` | AES with a key and IV (several modes). |
| `crypt.hash(data, algorithm)` | MD5, SHA-1, SHA-2, SHA-3 and more. |
| `crypt.generatekey()` / `crypt.generatebytes(n)` | Random keys and bytes. |
| `lz4compress` / `lz4decompress` | LZ4 compression. |

### Drawing & console

| Global | Description |
|---|---|
| `Drawing.new(type)` | `Line`, `Text`, `Image`, `Circle`, `Square`, `Quad`, `Triangle`. |
| `cleardrawcache()` / `isrenderobj(obj)` | Manage drawings. |
| `rconsoleprint` / `rconsoleclear` / `rconsolecreate` | kAPI's console window. |

### Actors

| Global | Description |
|---|---|
| `run_on_actor(actor, code)` | Run code inside an Actor's own Lua state. |
| `return_on_actor(actor, code)` | Same, but waits and returns the result. |
| `getactors()` | Real Actors in the game. |
| `getluastate(actor)` | A proxy for an Actor's Lua state (`:Execute`). |
| `create_comm_channel()` / `get_comm_channel(id)` | Pass data between Actor VMs. |

### kAPI extras

| Global | Description |
|---|---|
| `iy` | Infinite Yield, preloaded (needs the HTTP flag). |
| `queue_on_teleport(code)` | Code to run after a teleport. |
| `setfpscap(n)` / `getfpscap()` | Emulated FPS cap. |
| `setclipboard` / `getclipboard` | kAPI's clipboard. |

---

## Credits

- **xAPI** by [@anon2375517](https://devforum.roblox.com/t/xapi-a-powerful-pentesting-and-debugging-tool/2688148) — original foundation
- **Fiu** by Rerumu — Luau bytecode interpreter
- **LuauCeption** — Luau compiler in Luau
- HashLib, LZ4, Signal, and other bundled libraries

## License

MIT
