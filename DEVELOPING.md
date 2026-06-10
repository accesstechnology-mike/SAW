# Developing SAW

This document records the current build assumptions and known blockers for making the SAW fork reproducibly buildable.

The first goal is conservative: **build the existing application reliably on Windows before changing runtime behaviour**.

## Supported development host

SAW is currently a Windows-only .NET Framework desktop application.

Expected development environment:

- Windows 10 or Windows 11.
- Visual Studio 2019 or newer with .NET desktop development workload.
- .NET Framework 4.8 Developer Pack / targeting pack.
- NuGet package restore support.
- NSIS if building installers from `Installer/SAW.nsi`.
- Legacy Managed DirectX assemblies, unless/until references are made repository-local or replaced.

Linux/macOS builds are not expected to work at this stage. The project uses Windows Forms, .NET Framework 4.8, Win32 P/Invoke, and Windows-specific input/audio integration.

## Solution layout

- `SAW.sln` - Visual Studio solution.
- `SAW/SAW.csproj` - main Windows Forms application.
- `Update/Update.csproj` - updater/support application.
- `WebServer/WebServer.csproj` - referenced from the solution, not yet assessed in this fork.
- `Installer/SAW.nsi` - NSIS installer script.
- `Installer/SAW Prototype.nsi` - prototype installer script.
- `Readme.txt` - original project README.
- `MODERNISATION.md` - fork roadmap and safety principles.

## Main application project

`SAW/SAW.csproj` currently uses the legacy MSBuild project format and targets:

```xml
<TargetFrameworkVersion>v4.8</TargetFrameworkVersion>
<OutputType>WinExe</OutputType>
```

The project has x86-specific configurations and should initially be treated as an x86/Windows application until dependencies prove otherwise.

## NuGet packages

Packages are currently managed via `SAW/packages.config` rather than `PackageReference`.

Current packages:

| Package | Version | Notes |
| --- | ---: | --- |
| Microsoft.Bcl.AsyncInterfaces | 7.0.0 | .NET compatibility package |
| NAudio | 1.8.5 | Audio support |
| Newtonsoft.Json | 12.0.2 | JSON support |
| Svg | 3.0.49 | SVG handling |
| System.Buffers | 4.5.1 | .NET compatibility package |
| System.Memory | 4.5.5 | .NET compatibility package |
| System.Numerics.Vectors | 4.5.0 | .NET compatibility package |
| System.Runtime.CompilerServices.Unsafe | 6.0.0 | .NET compatibility package |
| System.Text.Encodings.Web | 7.0.0 | .NET compatibility package |
| System.Text.Json | 7.0.2 | JSON support |
| System.Threading.Tasks.Extensions | 4.5.4 | .NET compatibility package |
| System.ValueTuple | 4.5.0 | .NET compatibility package |

Dependency updates should be done only after a baseline build exists.

## Binary and legacy dependencies

The repository contains several binary dependencies. Their source, licensing, architecture support, and redistribution status need explicit confirmation before any runtime migration.

| Path | Type observed on Linux | Notes |
| --- | --- | --- |
| `SAW/Blade.dll` | PE32 .NET DLL, i386 | Referenced by `SAW/SAW.csproj` via `./Blade.dll` |
| `SAW/RedRat.dll` | PE32 .NET DLL, i386 | Referenced by `SAW/SAW.csproj` via `./RedRat.dll` |
| `RepoB.dll` | PE32 .NET DLL, i386 | Referenced by `SAW/SAW.csproj` and `Update/Update.csproj` via `../RepoB.dll` |
| `SAW/Utilities/Yocto/yapi.dll` | PE32 native DLL, i386 | Copied to output as `yapi.dll`; Yocto API also references `amd64/yapi.dll`, which is not currently present |
| `Installer/Installer files/Blade.dll` | PE32 .NET DLL, i386 | Installer payload copy |
| `Installer/Installer files/Blade.UserEdit.exe` | PE32 .NET EXE, i386 | Installer payload copy |
| `Installer/Update.exe` | PE32 .NET EXE, i386 | Installer payload copy |
| `Installer/SAW 8.00.2.exe` | NSIS installer archive | Historical/bundled installer |
| `Installer/SAW 8.00.3.exe` | NSIS installer archive | Historical/bundled installer |
| `Installer/SAW 8.00.4.exe` | NSIS installer archive | Historical/bundled installer |
| `Installer/SAW 8.00.5.exe` | NSIS installer archive | Historical/bundled installer |

### Binary fingerprints

These hashes were recorded from the current fork state so future dependency changes are auditable.

| Path | SHA-256 |
| --- | --- |
| `Installer/Installer files/Blade.UserEdit.exe` | `d08ecda61e340e7c666462deebe25f2a10f5d74699bd9b084eb537dd82e4fa00` |
| `Installer/Installer files/Blade.dll` | `ccd71ce4460d64ff18c9910831ba2a97da2b60877dbc4877225b62856b16d1b3` |
| `Installer/SAW 8.00.2.exe` | `712340df7dde342063f4e07d53ebee42cc5c9fd3596f7fc40a576a251e9cd5a7` |
| `Installer/SAW 8.00.3.exe` | `a7ab5e21ca4b6e194f5916bc2afff0be395dd2106b46f0f2a310799d6879b1b2` |
| `Installer/SAW 8.00.4.exe` | `1349ded93f2895563497565fd8b7b2f77ac5fddd31e1233a447f5823c5550cca` |
| `Installer/SAW 8.00.5.exe` | `6393fc37656984ff19b6960ad0049eadedb634107d9cc3630f22303c686dddb0` |
| `Installer/Update.exe` | `eddb7fecd55bf6bd81dd5baf256c5d7b1b5f9b6cbd1a9dba36721741ec89e62d` |
| `RepoB.dll` | `ddba4bfdbd93338e14d1475edc6908b631e6c9d7e28e4007e0ea36e64c9f0e20` |
| `SAW/Blade.dll` | `6a1851175c912141509037c312db44bac2035ace850cea30ba8e6bdfed33dd93` |
| `SAW/RedRat.dll` | `7262eae325b7484a2ae61367f1cd2c2e521f73d0b6a84d368ef19b5ed7450aa3` |
| `SAW/Utilities/Yocto/yapi.dll` | `db8fad6477611ae917e0d9f1de4566b5f83bcc37d6edd2a8272777e2cc7577b4` |

## Known build blockers / questions

### Managed DirectX

`SAW/SAW.csproj` references Managed DirectX assemblies from a hardcoded Windows path:

```xml
<HintPath>C:\Windows\Microsoft.NET\DirectX for Managed Code\1.0.2902.0\Microsoft.DirectX.dll</HintPath>
<HintPath>C:\Windows\Microsoft.NET\DirectX for Managed Code\1.0.2902.0\Microsoft.DirectX.DirectInput.dll</HintPath>
```

The installer scripts expect these DLLs to appear in the x86 release output:

```text
SAW/bin/x86/Release/Microsoft.DirectX.dll
SAW/bin/x86/Release/Microsoft.DirectX.DirectInput.dll
```

This is probably the first external build prerequisite to resolve.

### x86 dependency posture

Several checked-in binaries are i386/PE32. Treat x86 as the baseline until proven otherwise. Moving to AnyCPU or x64 should be a separate compatibility project.

### Yocto native DLL

`SAW/Utilities/Yocto/yapi.dll` is present for Win32. The Yocto API code also has Win64 DLL imports under `amd64/yapi.dll`, but no `amd64/yapi.dll` was found in the current inventory.

### Legacy HTTP/SOAP endpoints

`SAW/App.config` references old HTTP endpoints:

```xml
http://localhost/PrecompiledWeb/SplashServer/tech/repo2.asmx
http://saw-at.co.uk/tech/repo2.asmx
```

`SAW/Utilities/Server.cs` also references:

```text
http://www.saw-at.co.uk/tech/
```

Before changing these, determine whether the update/repository features are still live, optional, or should be retired/replaced.

## Baseline build procedure to verify on Windows

The first GitHub Actions probe was run on branch `probe/windows-ci-build`.

Observed result from Actions run `27286484594`:

- NuGet package restore reached NuGet successfully.
- Building `SAW.sln` failed before reaching main application compile issues.
- `SAW.sln` references `WebServer/WebServer.csproj`, but that project file was not present in the fork checkout.
- `Update/Update.csproj` targets .NET Framework 4.6.1, and the GitHub `windows-latest` runner did not have the 4.6.1 reference assemblies installed.

Relevant errors:

```text
SAW.sln.metaproj : error MSB3202: The project file "...\WebServer\WebServer.csproj" was not found.
SAW\SAW.csproj.metaproj : error MSB3202: The project file "...\WebServer\WebServer.csproj" was not found.
Microsoft.Common.CurrentVersion.targets(1259,5): error MSB3644: The reference assemblies for .NETFramework,Version=v4.6.1 were not found. [Update\Update.csproj]
```

The build probe now targets `SAW/SAW.csproj` directly so the main application blockers can be identified before repairing solution-level/update-project issues.

Observed result from the direct main-app build probe, Actions run `27286584674`:

- `SAW/SAW.csproj` package restore succeeded.
- Main application compilation reached C# compilation.
- Build failed because Managed DirectX assemblies were not available on the runner.

Relevant errors:

```text
warning MSB3245: Could not resolve this reference. Could not locate the assembly "Microsoft.DirectX".
warning MSB3245: Could not resolve this reference. Could not locate the assembly "Microsoft.DirectX.DirectInput".
SAW\Switches\Switching\JoySwitch.cs(5,17): error CS0234: The type or namespace name 'DirectX' does not exist in the namespace 'Microsoft'
SAW\Switches\Switching\JoySwitch.cs(90,18): error CS0246: The type or namespace name 'Device' could not be found
```

A follow-up probe installed the legacy DirectX runtime with Chocolatey before building. Actions run `27286782316` then completed successfully for the direct main application build.

Current CI behaviour:

- installs the legacy DirectX runtime using `choco install directx`;
- restores packages for `SAW/SAW.csproj`;
- builds `SAW/SAW.csproj` as `Debug|x86`;
- uploads an MSBuild binary log if present.

This gives the fork a reproducible CI baseline for the main SAW application. Full-solution build was then repaired on branch `fix/full-solution-build` by removing the dead `WebServer` project reference from `SAW.sln`, retargeting `Update/Update.csproj` to .NET Framework 4.8, and changing CI to build the solution. Actions run `27287367859` completed successfully for `SAW.sln` as `Debug|x86`.

A full Windows build pass should eventually do the following:

1. Clone the fork.
2. Open `SAW.sln` in Visual Studio.
3. Restore NuGet packages.
4. Confirm .NET Framework 4.8 targeting pack is installed.
5. Confirm Managed DirectX assemblies are available at the expected path or update the project to use a repository-local documented path.
6. Build `Debug|x86`.
7. Build `Release|x86`.
8. Record any missing references or compile errors in this document.
9. Only after a successful application build, test installer generation via NSIS.

## Modernisation boundaries for this branch

This branch should not change runtime behaviour.

Allowed:

- Documentation.
- Dependency inventory.
- Build prerequisite notes.
- Non-behavioural build metadata if verified.

Not allowed yet:

- Switching target framework.
- Converting project format.
- Updating package versions.
- Changing input/scanning/timer behaviour.
- Removing or replacing binary dependencies.
- Changing installer contents.

## Next action

Run a real Windows build and update this document with observed failures. The likely first failure is missing Managed DirectX assemblies or opaque binary dependency resolution.
