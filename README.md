# kAPI

> A Roblox executor **emulator** written entirely in Luau. It runs inside your own Roblox Studio session and reproduces the behavior of real executor APIs (UNC / sUNC) so scripts, tools, and anti-cheats can be built and tested without a real executor.

The successor of [xAPI](https://devforum.roblox.com/t/xapi-a-powerful-pentesting-and-debugging-tool/2688148).

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
4. **Identity emulation.** Thread identity is tracked per thread (0–13, with Roblox's capability table) and inherited by new threads. When the plugin, command bar or RbxStu bridge is running, elevated code runs there for real.
5. **Actors / parallel Luau.** `run_on_actor`, `getactors`, `getluastate`, and comm channels emulate executor actor APIs using real Actors.
6. **Server-backed features.** A server-side component handles things a client script can't do alone: HTTP requests, a persistent per-player filesystem, `game:GetObjects`, WebSockets, audio, images, and video.

---

## Features

- **High UNC compatibility** — 98% (82/83) on the UNC Environment Check with all FFlags on.
- **Closures & functions** — `hookfunction`, `newcclosure`, `iscclosure`, `islclosure`, `clonefunction`, `checkcaller`, `getcallingscript`.
- **Debug library** — `getupvalue(s)`, `setupvalue`, `getconstant(s)`, `setconstant`, `getproto(s)`, `getstack`, `setstack`, `getinfo`.
- **Metatables** — `getrawmetatable`, `setrawmetatable`, `hookmetamethod`, `setreadonly`, `isreadonly`, `getnamecallmethod`.
- **Environment** — `getgenv`, `getrenv`, `getgc`, `getreg`, `getsenv`, `getloadedmodules`, `getrunningscripts`.
- **Instances & signals** — `getconnections`, `firesignal`, `fireclickdetector`, `fireproximityprompt`, `firetouchinterest`, `gethiddenproperty`, `sethiddenproperty`, `cloneref`, `compareinstances`, `getnilinstances`, `getnilspace`.
- **Filesystem** — `readfile`, `writefile`, `appendfile`, `listfiles`, `isfile`, `isfolder`, `makefolder`, `delfile`, `delfolder`, `loadfile`. Persistent per player, and none of them yield.
- **Network** — `request` / `http_request`, `game:HttpGet`, `WebSocket.connect`.
- **Assets** — `getcustomasset` for images, audio, video, fonts and text (PNG, JPG, MP4, WebM, MP3, TTF/OTF, TXT …), and `game:GetObjects` (`.rbxm` / `.rbxmx`, with model scripts that actually run).
- **UI** — `gethui`, the `Drawing` library, console functions.
- **Crypt** — `crypt.base64encode/decode`, `crypt.encrypt/decrypt`, `crypt.hash`, `crypt.generatekey`, `crypt.generatebytes`, plus LZ4 compression.
- **Actors** — `run_on_actor`, `getactors`, `getluastate`, comm channels.
- **Extras** — Lua 5.1 runner (`--!Lua`), HTML runner (`--!html`), an in-game IDE with autocomplete, and an FFlag system to switch subsystems on or off.

---

## Installation

1. Download `kAPI.rbxm` from this repository.
2. Insert it into your place in Roblox Studio and keep its structure intact (kAPI has both client and server parts).
3. From a **LocalScript** (or a plugin Script, the command bar, or RbxStu), run:

```lua
-- kAPI Require: paste this anywhere (plugin Script, command bar, RbxStu) — kAPI figures out the rest for identity elevation.

local RunService = game:GetService("RunService")
if RunService:IsRunning() and RunService:IsClient() then
	local function find() return game:GetService("ReplicatedStorage"):FindFirstChild("_ENV") or game:FindFirstChild("_ENV", true) end
	local env = find()
	while not env do task.wait() env = find() end
	require(env)()
end
```

The environment is injected into the calling script automatically. kAPI is most capable when code runs through its `loadstring`, since that code executes on kAPI's own interpreter:

```lua
loadstring([[
    print(identifyexecutor())
    print(getgenv())
]])()
```

Video files play straight from the workspace through `getcustomasset`: MP4 (H.264 + AAC) and WebM (VP8 / VP9 + Opus / Vorbis) are decoded by kAPI itself, no converter needed.

### Identity elevation

The same snippet starts an elevation bridge when it runs somewhere with more access. kAPI detects where it is:

| Where you paste it | Identity |
|---|---|
| Studio command bar (client view, during Play) | 4 |
| A plugin Script | 5 / 6 |
| RbxStu | Any (0–13) |

When a bridge is running, `setthreadidentity(n)` at the top level of a `loadstring` chunk runs the rest of that chunk through the bridge. Without one, identities are emulated.

---

## FFlags

kAPI's subsystems can be switched on or off through its FFlags module (filesystem, custom globals, fake compatibility shims, Fiu manipulation, drawing, websockets, and more). Turning flags off makes kAPI lighter but lowers compatibility:

| Configuration | UNC score |
|---|---|
| Full (all FFlags on) | 98% |
| No fake compatibility | ~87% |
| No Fiu manipulation | ~84% |
| Minimal (both off) | ~72% |

Flags can also be read and set at runtime with `getfflag` / `setfflag`.

---

## Known limitations

- **Emulation, not engine access.** Scripts that use the real `game` (not kAPI's proxy) see the world normally, so a few behaviors that depend on engine internals are approximated. For example, `getscriptbytecode` on a Roblox script kAPI can't read returns a labelled stub, and `getscriptclosure` can't reach `CoreGui.RobloxGui` modules.
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
| `getthreadidentity()` / `setthreadidentity(n)` | The current thread's identity (0–13). New threads inherit it. |
| `getthreadcapabilities(n?)` / `hascapability(name, n?)` | Capabilities of an identity (`Plugin`, `LocalUser`, `RobloxScript`, …). |
| `getidentityname(n?)` | The identity's name (`8` → `WebService`). |
| `withidentity(n, f, ...)` | Run `f` at identity `n`, then restore the old one. |
| `isidentitybridged(n?)` / `getrealidentity()` | Whether a bridge really carries identity `n` / the engine's real identity. |
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

- **xAPI** by SQLanguage — the original foundation kAPI was built on
- **Fiu** by TheGreatSageEqualToHeaven (rce incorporated) — Luau bytecode interpreter
- **LuauCeption** by RealEthanPlayzDev — Luau compiler in Luau
- **lxm** and **LZ4** by metatablecat
- **HashLib** by Egor Skriptunoff, boatbomber and howmanysmall
- **WavPlayer** by Adiont, **ImageRunner** by Seph
- **RoProxy** by okfalse, **WebhookProxy** by lewisakura
- **PNGService**, **MeshService** by Seraph1nity
- **AES** by RobloxGamerPro200007, **html** by pat-dill, **Remote Image Library** by plainenglish, **FiO** by Rerumu, **Yueliang** by Kein-Hong Man

The full list is in `_ENV/README` inside the model.

## License

GPL-3.0. See [LICENSE](LICENSE).
