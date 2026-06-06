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
Each stick has a configurable response curve — chosen from `Linear`,
`Power`, `Expo`, or `Sigmoid` — for tuning how small deflections map to
output magnitude.

Unlike the rest of the project's settings, the controller keys below
live in the `[DXController]` section of `DeusEx.ini`, not `[DeusExe]`.

| Key | Type | Default | Notes |
| --- | ---- | ------- | ----- |
| `StickDeadzoneLeft` | int (0..32767) | `2500` | Radial deadzone for the left stick, in raw SHORT units. |
| `StickDeadzoneRight` | int (0..32767) | `2500` | Radial deadzone for the right stick. |
| `StickCurveLeft` | string | `Power` | Response curve for the left (movement) stick. One of `Linear` (no shaping), `Power` (`u^k`), `Expo` (`(1-e)u + e u^3`), or `Sigmoid` (S-shaped). Unknown values fall back to `Linear`. |
| `StickCurveRight` | string | `Power` | Response curve for the right (look) stick. Same options as `StickCurveLeft`. |
| `StickCurvePowerLeft` | float | `2.0` | Power exponent `k` for the left stick when `StickCurveLeft = Power`. `1.0` is linear; `>1.0` gives finer control at small deflections while still reaching full speed at full deflection; `<1.0` is snappier near centre. Clamped to `[0.1, 10.0]`. |
| `StickCurvePowerRight` | float | `2.0` | Power exponent `k` for the right stick when `StickCurveRight = Power`. Same range and behaviour as `StickCurvePowerLeft`. |
| `StickCurveExpoLeft` | float | `0.60` | Expo amount `e` for the left stick when `StickCurveLeft = Expo`. `0.0` is linear; `1.0` is full cubic. Clamped to `[0.0, 1.0]`. |
| `StickCurveExpoRight` | float | `0.60` | Expo amount `e` for the right stick. Same range and behaviour as `StickCurveExpoLeft`. |
| `StickCurveSigmoidSteepnessLeft` | float | `6.0` | Sigmoid steepness `k` for the left stick when `StickCurveLeft = Sigmoid`. Higher values sharpen the S. Clamped to `[1.0, 12.0]`. |
| `StickCurveSigmoidSteepnessRight` | float | `6.0` | Sigmoid steepness `k` for the right stick. Same range and behaviour as `StickCurveSigmoidSteepnessLeft`. |
| `StickCurveSigmoidMidpointLeft` | float | `0.60` | Sigmoid midpoint `c` for the left stick. Lower values move the steep region earlier (more sensitive overall). Clamped to `[0.15, 0.85]`. |
| `StickCurveSigmoidMidpointRight` | float | `0.60` | Sigmoid midpoint `c` for the right stick. Same range and behaviour as `StickCurveSigmoidMidpointLeft`. |
| `StickCurveSigmoidStrengthLeft` | float | `0.60` | Sigmoid strength `w` for the left stick — dry/wet blend with linear. `0.0` is linear; `1.0` is pure S. Clamped to `[0.0, 1.0]`. |
| `StickCurveSigmoidStrengthRight` | float | `0.60` | Sigmoid strength `w` for the right stick. Same range and behaviour as `StickCurveSigmoidStrengthLeft`. |
| `TriggerThreshold` | int (0..255) | `30` | Trigger pull below this is treated as zero. Microsoft's default is `30`. |
| `MouseActivityPx` | int | `4` | Mouse motion of this many pixels marks the mouse as the active input source. |
| `PadActiveGraceMs` | int | `500` | Time after the last pad input during which the pad still counts as "active". |
| `HotplugScanMs` | int | `1000` | How often to poll disconnected slots for newly-attached pads. |

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
| `-localdata`   | Stores configuration and saves in the game directory instead of `Documents\Deus Ex`. |

