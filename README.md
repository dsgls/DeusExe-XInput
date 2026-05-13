# Deus Exe with XInput support

This is a fork of the [Deus Exe](https://kentie.net/article/dxguide/) to add XInput support so Xbox controllers can be used with Deus Ex.

By itself this project just enables XInput and hands the events to the Unreal engine. You'll be able to bind buttons to actions, but that's not particularly useful since the game is not designed for controllers at all. The sister project [DXController](https://git.dsg.is/dsg/DXController) is a mod to overhaul the UI and input handling in the game's scripts, with the goal of providing a first-class controller experience.

You'll also need a modern renderer like kentie's d3d10drv. The game will crash if you launch it with the default d3ddrv.
https://www.kentie.net/article/d3d10drv/

## Building

Open `DeusExe.sln` in Visual Studio 2022 with the v145 toolset, or run
`build.sh` from a developer shell. Building requires a copy of the
Unreal Engine 1 / Deus Ex SDK headers under `games/`; patch with
`patches/sdk-v145-compat.patch` after dropping a fresh SDK in place.

## Installation

Build with Visual Studio, then copy the contents of `Release/` into the game's `System` directory.

## Configuration

Settings live in the `[DeusExe]` section of `DeusEx.ini` (or whichever
`.ini` is selected via the standard `INI=` command-line option).

### XInput controller

The active controller is whichever connected slot most recently produced
input. Stick/trigger ranges are read raw and rescaled past the configured
deadzone or threshold, so axes hit full magnitude near the physical edge.
Each stick also has a configurable response curve for tuning how small
deflections map to output magnitude.

| Key | Type | Default | Notes |
| --- | ---- | ------- | ----- |
| `XInputLeftStickDeadzone` | int (0..32767) | `2500` | Radial deadzone for the left stick, in raw SHORT units. |
| `XInputRightStickDeadzone` | int (0..32767) | `2500` | Radial deadzone for the right stick. |
| `XInputLeftStickExponent` | float | `2.0` | Response curve applied to the left (movement) stick magnitude. `1.0` is linear; `>1.0` gives finer control at small deflections while still reaching full speed at full deflection; `<1.0` is snappier near centre. |
| `XInputRightStickExponent` | float | `2.0` | Response curve applied to the right (look) stick magnitude. `1.0` is linear; `>1.0` gives finer aim at small deflections while still reaching full turn speed at full deflection; `<1.0` is snappier near centre. |
| `XInputTriggerThreshold` | int (0..255) | `30` | Trigger pull below this is treated as zero. Microsoft's default is `30`. |
| `XInputMouseActivityPx` | int | `4` | Mouse motion of this many pixels marks the mouse as the active input source. |
| `XInputPadActiveGraceMs` | int | `500` | Time after the last pad input during which the pad still counts as "active". |
| `XInputHotplugScanMs` | int | `1000` | How often to poll disconnected slots for newly-attached pads. |

Sticks emit `IK_JoyX`/`Y` (left) and `IK_JoyU`/`V` (right) axis events;
triggers emit `IK_JoyZ` (left) and `IK_JoyR` (right); buttons and the
D-pad emit `IK_Joy1`..`16` / `IK_JoyPov*` press/release events. These can
be bound in `DeusEx.ini` or via the in-game key binding screen.

### General

| Key | Type | Default | Notes |
| --- | ---- | ------- | ----- |
| `FPSLimit` | int | `120` | Frame rate cap. `0` disables. |
| `RawInput` | bool | `True` | Raw mouse input, eliminates acceleration. Also enables mouse buttons 4/5. |
| `UseAutoFOV` | bool | `True` | Compute FOV from the current resolution. |
| `BorderlessFullscreenWindow` | bool | `True` | Borderless windowed full-screen instead of exclusive full-screen. |
| `BorderlessFullscreenWindowAllMonitors` | bool | `False` | When borderless, span all monitors. |
| `UseSingleCPU` | bool | `False` | Pin the game to a single CPU core. |
| `SubtitleFix` | bool | `True` | Correct cinematic subtitle placement for wider-than-16:9 resolutions. |
| `GUIScalingFix` | int | `0` | UI scale factor; `0` disables, `1`+ selects scale levels. |

## Command-line options

| Flag           | Effect |
| -------------- | ------ |
| `-skipdialog`  | Skips the launcher dialog and starts the game immediately. |
| `-userprofile` | Stores configuration and saves in `Documents\Deus Ex` instead of the game directory. |

