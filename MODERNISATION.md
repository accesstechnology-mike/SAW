# SAW Modernisation Roadmap

This fork is intended to modernise **SAW - Special Access to Windows** incrementally, without discarding the domain knowledge embedded in the existing application.

SAW is assistive-technology software. Changes that look harmless in ordinary desktop software can be user-breaking here, especially around switch timing, dwell selection, speech/audio feedback, joystick input, and Windows focus/control integration.

## Current position

SAW is currently a Windows-only .NET Framework application.

Observed repository state:

- Visual Studio solution: `SAW.sln`
- Main application project: `SAW/SAW.csproj`
- Target framework: .NET Framework 4.8
- UI framework: Windows Forms
- Package management: `packages.config`
- Installer: NSIS script under `Installer/`
- Update/supporting projects: `Update/`, `WebServer/`
- Windows integration: direct Win32/PInvoke usage, including `user32.dll`
- Input/access components: switch, pointer, joystick, dwell and scanning engines under `SAW/Switches/`
- Speech/audio components under `SAW/Utilities/`

The codebase should be treated as a mature legacy AT application, not a greenfield UI project.

## Guiding decision: modernise, do not rewrite first

The initial strategy is **incremental modernisation of the existing codebase**, not a clean-room rewrite.

A rewrite would only be justified if the product goal changes substantially, for example:

- cross-platform support becomes a hard requirement;
- the project becomes a new SAW-like product rather than SAW itself;
- compatibility with existing SAW behaviours, files, hardware assumptions, or workflows is no longer required;
- the core input/scanning behaviours have first been documented and covered by tests.

Until then, the existing code is valuable because it encodes years of accessibility-specific behaviour and edge cases.

## Safety principles

1. **Do not break switch access.** Scanning timing, repeat behaviour, dwell logic, and input mapping are core user affordances.
2. **Preserve behaviour before refactoring.** Where behaviour is unclear, document and test it before changing it.
3. **Prefer small reversible branches.** Avoid broad rewrites that mix formatting, dependency changes, and behaviour changes.
4. **Build reproducibility comes first.** If a clean Windows machine cannot build the fork, modernisation work is not yet under control.
5. **Treat binary dependencies explicitly.** Unknown DLLs and legacy DirectX references must be inventoried before runtime/platform changes.
6. **Windows-first is acceptable.** SAW is deeply Windows-integrated; cross-platform work should be considered a separate product track.

## Phase 1: Repository onboarding and build reproducibility

Goal: make it possible for a developer to understand and build the project predictably.

Tasks:

- Create clear `README.md` and/or `DEVELOPING.md` documentation.
- Document required Windows, Visual Studio, .NET Framework, NuGet, DirectX and installer prerequisites.
- Inventory binary dependencies such as `Blade.dll`, `RedRat.dll`, `RepoB.dll`, and Yocto components.
- Replace hardcoded machine-specific references where possible with repository-relative paths or documented installation steps.
- Confirm a clean clone can restore packages and build on Windows.

Suggested branch:

```text
fix/build-reproducibility
```

## Phase 2: Continuous integration

Goal: add automated verification before code changes accelerate.

Tasks:

- Add GitHub Actions using a Windows runner.
- Restore NuGet packages.
- Build the solution with MSBuild.
- Publish build logs/artifacts where useful.
- Keep CI initially compile-focused; add tests after seams exist.

Suggested branch:

```text
infra/windows-ci
```

## Phase 3: Dependency and project-file modernisation

Goal: improve maintainability without changing runtime behaviour.

Tasks:

- Migrate from `packages.config` to `PackageReference` where practical.
- Update dependencies conservatively.
- Keep .NET Framework 4.8 initially unless a separate migration branch proves safe.
- Consider SDK-style project conversion only after baseline builds are repeatable.

Suggested branch:

```text
infra/package-reference
```

## Phase 4: Behaviour characterization and tests

Goal: protect the accessibility-critical core before deeper refactoring.

Tasks:

- Identify seams around scanning engines, logical/physical switch mapping, dwell timing and repeat behaviour.
- Add characterization tests for timing/state-machine behaviour where possible.
- Document expected behaviour for one-switch, two-switch, pointer and dwell modes.
- Avoid changing timers/threading until tests exist.

Candidate areas:

- `SAW/Switches/Engine.cs`
- `SAW/Switches/Engines/`
- `SAW/Switches/Switching/`

## Phase 5: Runtime/UI modernisation experiments

Goal: explore modern runtime options without committing the mainline too early.

Possible tracks:

- .NET 8 WinForms migration spike.
- Replacement for legacy Managed DirectX/DirectInput assumptions.
- High-DPI improvements.
- Separation of core scanning/input logic into testable libraries.
- Longer-term UI replacement only after behavioural compatibility is understood.

Suggested branch:

```text
spike/dotnet-8-winforms
```

A .NET 8 migration should be treated as a spike first. It should produce a list of blockers, not a half-migrated main branch.

## Immediate next branches

1. `fix/build-reproducibility`
   - Establish a clean Windows build and document required dependencies.

2. `infra/windows-ci`
   - Add a Windows GitHub Actions build once prerequisites are clear.

3. `infra/package-reference`
   - Modernise package management after the baseline build is stable.

## Explicit non-goals for the first phase

- Full rewrite from scratch.
- Cross-platform port.
- Replacing WinForms.
- Behavioural changes to scanning/input timing.
- Large formatting-only rewrites that obscure future diffs.
- Removing binary dependencies before their purpose and licensing are understood.

## Open questions

- Are the binary DLLs redistributable, and do source versions exist?
- Which SAW behaviours/files must remain compatible with existing users?
- What hardware configurations must be supported for any release candidate?
- Is high-DPI support a priority for current users?
- Is the update server still operational or should update functionality be retired/replaced?

## Working rule

Modernise the maintenance surface first. Protect the AT behaviour always.
