# DirectInput Replacement Spike

SAW currently builds by installing the legacy DirectX runtime in CI. That is acceptable as a build baseline, but not a good long-term dependency story.

This spike records what would be needed to replace the legacy `Microsoft.DirectX.DirectInput` dependency used by joystick switch access.

## Current usage

DirectInput usage appears isolated to:

```text
SAW/Switches/Switching/JoySwitch.cs
```

`JoySwitch` currently relies on these Managed DirectX behaviours:

- enumerate attached game controllers with `Manager.GetDevices(DeviceClass.GameControl, EnumDevicesFlags.AttachedOnly)`;
- create the first joystick `Device` from the discovered device GUID;
- set cooperative level using `Background | NonExclusive`;
- acquire, poll, unacquire and dispose the device;
- read `JoystickState.GetButtons()`;
- treat button byte values `>= 128` as button-down;
- keep polling on a `System.Windows.Forms.Timer` at a 10ms interval;
- in detection mode, detect rising button transitions between two button snapshots;
- in normal mode, call `OnStateChange(isDown)` for configured joystick switch indices.

The replacement must preserve these behaviours unless deliberately changed and tested.

## Accessibility-critical behaviours

This is not ordinary gamepad input. SAW uses joystick buttons as assistive-technology switches.

Do not casually change:

- 10ms polling cadence;
- button-down threshold/semantics;
- background/non-exclusive input behaviour;
- rising-edge detection in switch-detection mode;
- switch state-change event timing;
- disconnect/error behaviour.

A prettier implementation that loses input while another window has focus is a regression.

## Recommended replacement seam

Introduce an internal joystick source abstraction inside `Switches.Switching`.

Initial shape:

```csharp
internal interface IJoystickSource : IDisposable
{
    void Poll();
    bool[] GetButtonStates();
}
```

The important point is not the exact method names. The important point is to remove `Microsoft.DirectX.DirectInput.Device` and `JoystickState` from `JoySwitch` itself.

Suggested first refactor:

1. Extract current Managed DirectX code into `LegacyDirectInputJoystickSource`.
2. Keep `JoySwitch` responsible for switch semantics, timing and `OnStateChange`.
3. Keep the initial implementation behaviour-identical.
4. Only after that add a second implementation.

This creates a rollback path and prevents a dependency migration from also becoming a behaviour rewrite.

## Candidate replacement APIs

### SharpDX.DirectInput

Recommended first replacement candidate.

Pros:

- close conceptual/API match to DirectInput;
- supports generic joysticks rather than only Xbox controllers;
- available via NuGet;
- likely lowest-risk change for the current .NET Framework 4.8 WinForms app;
- keeps the current polling model intact.

Cons:

- SharpDX is no longer an actively evolving project;
- still leaves SAW on a DirectInput-style model;
- hardware testing remains mandatory.

### Vortice.Windows

Possible longer-term candidate.

Pros:

- more modern maintained Windows interop library;
- better fit for future .NET runtime work.

Cons:

- migration is likely less drop-in than SharpDX;
- more API research needed for DirectInput parity;
- higher risk for a first replacement.

### Windows.Gaming.Input

Not recommended as the first replacement.

Pros:

- modern Windows API;
- clean conceptual model for supported devices.

Cons:

- WinRT integration complexity in a legacy .NET Framework WinForms app;
- may not support older/generic assistive joystick devices as well as DirectInput;
- background/non-exclusive behaviour must be proven.

### XInput

Not suitable as the main replacement.

Pros:

- simple for Xbox-compatible controllers;
- available on modern Windows.

Cons:

- limited device support;
- poor fit for generic/AT switch interfaces.

### SDL2

Probably too heavy for the first replacement.

Pros:

- robust device support;
- well-known input layer.

Cons:

- introduces native dependency payload;
- broadens distribution complexity for a single feature.

## Test and characterization plan

Before changing behaviour, characterize the current implementation.

Minimum checks:

1. **Build baseline**
   - Current `master` CI builds `SAW.sln` as `Debug|x86`.

2. **Button-state semantics**
   - With a known joystick/switch interface, confirm current `GetButtons()` byte values and the `>= 128` down threshold.

3. **Focus behaviour**
   - Confirm joystick/switch input is received while another app, such as Notepad, has focus.

4. **Polling cadence**
   - Confirm replacement `Poll()` does not block or add material jitter to the 10ms timer loop.

5. **Detection mode**
   - Confirm rising-edge detection reports the correct joystick button index.

6. **Disconnect/reconnect behaviour**
   - Confirm unplug/disconnect path does not crash SAW and recovers or reports the same way as current behaviour.

7. **Configured switch mapping**
   - Confirm configured joystick switch numbers map to the same physical buttons before and after replacement.

## First implementation branch

Suggested branch:

```text
refactor/joystick-source-abstraction
```

Initial implementation steps:

1. Add `IJoystickSource`.
2. Add `LegacyDirectInputJoystickSource` wrapping the current Managed DirectX calls.
3. Refactor `JoySwitch` to use the interface while keeping the legacy implementation.
4. Keep CI green.
5. Do not add SharpDX until the abstraction-only branch is green.

Follow-up branch:

```text
spike/sharpdx-joystick-source
```

Steps:

1. Add SharpDX.DirectInput package.
2. Add `SharpDxJoystickSource`.
3. Add a compile-time or config switch if needed for comparison.
4. Run CI.
5. Run manual hardware/focus/timing checks before considering removal of the legacy DirectX runtime dependency.

## Risks

- Some older AT joystick/switch interfaces may behave differently under different DirectInput wrappers.
- Background input behaviour is mandatory and must be tested.
- SAW currently remains x86 due to other binary dependencies, regardless of DirectInput replacement.
- A successful compile is not proof of AT usability.
- Replacing DirectInput before isolating switch semantics would make regressions hard to identify.

## Recommendation

Proceed in two steps:

1. **Refactor only:** introduce the `IJoystickSource` seam while preserving Managed DirectX behaviour.
2. **Replace second:** add a SharpDX implementation and compare behaviour.

Do not remove the legacy DirectX CI install until the replacement has been verified with real hardware and focus/background tests.
