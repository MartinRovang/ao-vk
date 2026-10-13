# The Witched fork

This is `MartinRovang/ao-vk`, a fork of [wannabuh/ao-vk](https://github.com/wannabuh/ao-vk) for the
Anarchy Online - Witched server. Our work is on the `witched` branch; `main` follows the original.
This file is the running record: what we changed, what we know is wrong, and how a change gets from
here into the game.

## Where things are

| What | Where |
| --- | --- |
| This fork | `~/Desktop/ao-vk`, remotes `origin` (ours) and `upstream` (the original; pushing to it is disabled) |
| The client it goes into | built by `~/Desktop/ao-client/apply`; the renderer is `overlay/randy31.dll` there |
| Scratch client for trying a build | `~/Games/AO-62-vk` (start with its `play`) |
| The client that gets published | `~/Games/AO-Witched/game` |

Our client is the 18.8.62 game build. The original project is developed on 18.8.50. `Gamecode.dll`
and `GUI.dll` differ between the two; the game's own renderer (`randy31_orig.dll`),
`DisplaySystem.dll`, `N3.dll` and `Utils.dll` are the same code.

## Building

```
docker/build
```

Needs only Docker. The first run builds the toolchain image (about a gigabyte of downloads: Clang 19,
the Windows SDK through xwin, the Vulkan SDK). The result is `build/linux-release/randy31.dll`, plus
the test programs the README describes.

Three things the image does that the README's instructions do not mention, each because the build
failed without it on Ubuntu 24.04:

- Clang 19, not 18: the MSVC standard library xwin fetches refuses anything older.
- `clang-cl` is a link to `clang`: Ubuntu's package has no `clang-cl`.
- `/clang:-msse2`: Ubuntu's Clang builds 32-bit code without SSE2, even with `/arch:SSE2`, and
  `stb_image.h` needs it.

## Trying a build in the game

1. Close the game.
2. Copy `build/linux-release/randy31.dll` to `~/Games/AO-62-vk/game/randy31.dll`. The release we ship
   is kept beside it as `randy31.dll.release-0.4.1`.
3. Run `~/Games/AO-62-vk/play`. It plays on the shared server.
4. Read `~/Games/AO-62-vk/game/randy-vk.log` afterwards. Lines with "not hooked", "particles off" or
   "unknown client" say what the renderer skipped.

Ctrl+Shift+F9 in the game writes a frame dump next to the log (`rvk-frame-*.txt` and `.bmp`): a
screenshot, every draw and every light of that frame.

## Shipping a build

1. Copy the DLL to `~/Desktop/ao-client/overlay/randy31.dll` and commit it there.
2. Rebuild the client with `apply`, or copy the DLL into `~/Games/AO-Witched/game` as well.
3. Publish from the launcher repo with `./deploy client "<news line>"`.

`ao-client/apply` still runs `vk-particles` on the DLL. Once a build with the first change below is
shipped, that step refuses the file ("not the ao-vk this was written against") and must be removed
from `apply`.

## Changes

Newest first. "Tested" says what was actually checked.

### Particle hook finds its class by name (2026-10-13)

`proxy/ddraw/rvk_particles.cpp`. The hook on `_GfxControlStars_t::Update` used two fixed addresses
that are only right for the 18.8.50 `Gamecode.dll`; on 18.8.62 the log said "unknown client version,
particles off". It now looks the class's vtable up through its type information when the hook is
installed (`FindVtable`), so it works on either build with no patching of the DLL.

Tested: builds. Not yet seen in the game.

### Build in Docker (2026-10-13)

`docker/Dockerfile`, `docker/build`. See Building above.

Tested: a build from this, with the particle addresses patched by `ao-client/vk-particles`, was
played on the 18.8.62 client and looked the same as the 0.4.1 release.

## Known problems

Found while moving the server to 18.8.62 on 2026-10-13. None is fixed unless it is listed under
Changes.

| Problem | What we know |
| --- | --- |
| Breeds almost black on the character creation screen | Happens with light override on (`RVK_LightOver=1`, the default); off, they are lit. The scene has three point lights, one of them black; a frame dump with it off is `rvk-frame-095319` in `~/Downloads/Anarchy Online/game`. |
| Close-up blur on character selection and creation | Depth of field. With "blur behind the focus (always)" off, the blur starts when the focus is nearer than `RVK_DofMacro` (3), and those screens hold the camera closer than that. The launcher switches depth of field off once per install as a workaround. |
| Hotkeys register several times, or flip other switches | Seen on Proton: one Ctrl+Shift+D set `RVK_Enhance=0` as well as `RVK_Dof`, and the log shows a setting going off and on in the same instant. |
| Window stays 119x34 when the game starts in fullscreen | A fresh install starts at 1024x768 fullscreen; the device is created but the window never grows. Windowed works. The launcher writes the display settings, so players do not meet it. |
| Interface redraw limit off on 18.8.62 | Log: `interface: WindowController_c::Render - unexpected code (another GUI.dll build?), not hooked`, the same for `View::_CallRender`. `proxy/native/gui.cpp` checks code that differs in the 18.8.62 `GUI.dll`. Costs performance only. |
| Edits to `randy-vk.ini` while the game runs did not apply | Two edits were back to the old values moments later while hotkeys were being pressed; the README says such edits apply within half a second. Not looked into. |

## Following the original

```
git fetch upstream
git switch main && git merge --ff-only upstream/main && git push origin main
git switch witched && git merge main
```

Changes that would help everyone (the particle lookup, the known problems above) are worth offering
to the original as issues or pull requests; that is Martin's call each time.
