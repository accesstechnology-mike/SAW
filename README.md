# SAW - Special Access to Windows

SAW is a programmable on-screen keyboard and access environment for Windows. It provides switch, pointer and joystick access to the Windows operating system.

This repository is a fork for careful modernisation work.

## Original project description

From the original `Readme.txt`:

> SAW - Special Access to Windows is a programable on-screen keyboard to provide
> Switch, Pointer and Joystick acess to the Windows Operating System.

Copyright and licensing details from the original project are preserved in `Readme.txt` and installer/license files.

## Modernisation approach

This fork will modernise SAW incrementally rather than begin with a full rewrite.

SAW is assistive-technology software. Switch scanning, dwell selection, joystick input, speech/audio feedback and Windows focus handling are accessibility-critical behaviours. They should be documented and protected before major runtime or UI changes.

See:

- [`MODERNISATION.md`](MODERNISATION.md) - roadmap, safety principles and first branches.

## Initial priorities

1. Make the project reproducibly buildable on a clean Windows development machine.
2. Document build prerequisites and legacy/binary dependencies.
3. Add Windows-based CI.
4. Modernise package/project structure conservatively.
5. Characterise and test switch/input behaviour before deeper refactoring.

## Current technical shape

- Windows-only desktop application.
- .NET Framework 4.8.
- Windows Forms.
- Legacy Visual Studio/MSBuild project format.
- `packages.config` NuGet restore style.
- Direct Windows API integration.
- Specialist switch, dwell, pointer and joystick input engines.

## Non-goals for the first phase

- Full rewrite from scratch.
- Cross-platform port.
- Replacing WinForms immediately.
- Behavioural changes to input/scanning timing.
- Broad formatting-only rewrites that obscure meaningful changes.
