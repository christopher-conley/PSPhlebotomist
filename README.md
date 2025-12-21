# PSPhlebotomist

> *"It's just a little prick"*

A PowerShell module enabling DLL/arbitrary code injection into a target process, wrapped in a dumb, corny, medically-themed API, because at some point while writing this I fell too deep into the hole of the bit to dig myself back out again.

## Overview

PSPhlebotomist provides a PowerShell cmdlet for injecting one or more DLLs/PE images into a target process on Windows. It supports fully automated runs via commandline arguments, an interactive operation mode, optional privilege elevation including relaunching itself afterward with the same args, and flexible process targeting by PID or process name.

## Features

- **Multiple Injection Methods**: Target processes by Process ID (PID) or Process Name
- **Automated Mode**: 100% configurable/launchable via commandline arguments for a one-line command injection with no prompts or further interaction necessary
- **Interactive Mode**: User-friendly(ish) prompts when parameters aren't provided
- **Privilege Elevation**: Elevation to an Administrator security context when requested, which leverages a `sudo` implementation if available, regular `runas` process launch if not
- **Process Launch Waiting**: Configure options, loiter, and actively monitor for a launch of the target process, and then inject at the first opportunity
- **Multiple DLL Support**: Inject `N`-number of DLLs/PE images in a single operation, in the exact order that you specify
- **Comprehensive Logging**: Built-in Serilog integration for detailed operation logging
- **Medical-Themed API**: Because injecting DLLs should feel professional *(the bit is real and I can't stop now <sup>oh god please help me</sup>)*

## Installation

### From Source
### STOP AND READ THIS IF YOU WANT TO BUILD FROM SOURCE
#### This project uses **Paket** for ease of use with dependency/package management. Don't just autopilot nuget/dotnet restore, just follow these steps:

1. Clone the repository:
```powershell
git clone https://github.com/christopher-conley/PSPhlebotomist.git
cd PSPhlebotomist
```

2. Install Paket for dependency management and restore packages:

```powershell
dotnet tool install --global Paket
dotnet tool restore
dotnet paket install
```

3. Build the project:
```powershell
dotnet build PSPhlebotomist.sln
```

1. Import the module (psd1 or the DLL directly):
```powershell
Import-Module .\PSPhlebotomist\bin\Debug\netstandard2.0\PSPhlebotomist.psd1
```

## Requirements for use

- **.NET Standard 2.0** or higher (you have this already)
- **PowerShell 7+**
- **Administrator privileges are not necessary** (but are helpful for most injection scenarios)
- **Windows** operating system
  - This is actually a little bit of a lie. It ***will*** build ***and*** successfully import ***and*** run under PowerShell in Linux, I've tested it. What I *have not* tested is attempting to inject a `.so` library into a native Linux process, and I *seriously* doubt that it would come anywhere close to functioning because of the reliance on Windows APIs to actually do the injection work. So yes, it will *technically* import and look pretty and be available for "use" in Linux, but it's probably not going to do anything useful for you, at best. I actually tried to inject via PowerShell running in a WSL distro for shits and giggles, but the WSL kernel doesn't appear to have access to the Windows process table. It **might** actually work for a process being ran via WINE/Proton though, but I haven't tested it. I should do that, that's interesting.

This project will build and import on PowerShell 5.1, but it ***will not*** run. There is a tangled yarn ball of dependency issues involved with that, which, frankly, I can think of several other unpleasant things I'd rather do than figure that bullshit out. Building to netstandard2.0 is already enough of a pain in the ass dependency-wise, and I'm not going to rework that entire dependency tree to support a legacy technology which is no longer actively developed, and hasn't been for a number of years. If this paragraph made you mad, channel that anger into [downloading the latest release of PowerShell](https://github.com/PowerShell/PowerShell/releases).

## Usage

### Basic Usage

Inject a DLL by Process ID:
```powershell
New-Injection -PID 48257 -Inject "C:\Path\To\Your.dll"
```

Inject a DLL by Process Name (extension is optional):
```powershell
New-Injection -Name "notepad" -Inject "C:\Path\To\Your.dll"
```

### Multiple DLL Injection

```powershell
New-Injection -PID 9970 -Inject "C:\First.dll", "C:\Second.dll", "C:\Third.dll"
```

### Multiple DLL Injection with explicit PowerShell array syntax

```powershell
New-Injection -PID 62470 -Inject @("C:\First.dll", "C:\Second.dll", "C:\Third.dll")
```

### Wait for Process Launch

Wait indefinitely for a process to start before injecting:
```powershell
New-Injection -Name "target.exe" -Inject "C:\Your.dll" -Wait
```

### Wait for Process Launch, with 30 Second Timeout

Wait for a process to start before injecting, abanadon attempt after 30 seconds:
```powershell
New-Injection -Name "target.exe" -Inject "C:\Your.dll" -Wait -Timeout 30
```

### Elevated Privileges

Automatically relaunch with Administrator privileges, and rerun with the same args:
```powershell
New-Injection -PID 9924 -Inject "C:\Your.dll" -Admin
```

### Interactive Mode

Simply run the cmdlet without parameters for a step-by-step interactive mode:
```powershell
New-Injection
```

### Interactive Mode as Admin

Interactive mode will also trigger if the only paramter is `-Admin` or an alias of it:
```powershell
New-Injection -Admin
```


The module will guide you through the injection process with prompts.

### Cmdlet Aliases

Because we're all-in on the medical theme:
```powershell
New-Syringe -PID 1842 -Inject "C:\Your.dll"
New-Needle -Name "notepad" -Inject "C:\Your.dll"
New-Patient -PID 10789 -Inject "C:\Your.dll"
Insert-Needle -Name "calc" -Inject "C:\Your.dll"
```

## Parameters

### `-Inject`
Array of file paths to DLLs/PE images to inject. A single string argument to this parameter will be treated as an array containing one element.
- **Type**: `Array`
- **Required**: No (triggers interactive mode if omitted)
- **Pipeline**: Yes

### `-PID` / `-ProcessId` / `-Id`
Process ID of the target process.
- **Type**: `Int`
- **Required**: Yes (when using PID to target)
- **Aliases**: `ProcessId`, `Id`

### `-Name` / `-ProcessName` / `-PName`
Name of the target process.
- **Type**: `String`
- **Required**: Yes (when using Process Name to target)
- **Aliases**: `ProcessName`, `PName`

### `-Wait`
Wait and actively monitor for the target process to launch before injecting.
- **Type**: `SwitchParameter`
- **Required**: No
- **Note**: Only valid with `-Name` parameter

### `-Timeout`
Maximum time in seconds to wait for process launch.
- **Type**: `Unsigned Int`
- **Required**: No
- **Default**: Maximum unsigned integer value, platform-dependent

### `-Admin` / `-AsAdmin` / `-Administrator` / `-Root`
Attempt to elevate privileges and relaunch as Administrator before injecting. Leverages a `sudo` implementation if available, regular process launch with `runas` verb if not
- **Type**: `SwitchParameter`
- **Required**: No
- **Aliases**: `AsAdmin`, `Administrator`, `AsAdministrator`, `Root`

## Output

Returns a `PatientDiagnosis` object containing the results of the injection attempt(s), including:
- Injection success/failure status
- Target process information
- Error details (if any)
- Diagnostic information

## Examples

### Example 1: Simple Injection
```powershell
New-Needle -PID 5678 -Inject "C:\MyTools\hook.dll"
```

### Example 2: Multiple DLLs with Elevation
```powershell
New-Needle -Name "game.exe" -Inject "C:\mod1.dll", "C:\mod2.dll" -Admin
```

### Example 3: Wait for Process to Launch and Inject, Timeout and Abandon attempt after 60 seconds
```powershell
New-Needle -Name "application.exe" -Inject "C:\interceptor.dll" -Wait -Timeout 60
```

### Example 4: Wait for Process to Launch and Inject, Wait Indefinitely with no Timeout
```powershell
New-Needle -Name "application.exe" -Inject "C:\interceptor.dll" -Wait
```

### Example 5: Interactive Mode
```powershell
New-Needle
```

### Example 6: Pipeline Input
```powershell
Get-Process notepad | New-Needle -Inject "C:\logger.dll"
```

## Architecture

- **Cmdlet Layer**: PowerShell cmdlet implementation with parameter validation
- **Core Layer**: Native Windows API interop for process manipulation
- **Helpers**: Interactive mode, process validation, and utility functions
- **Logging**: Serilog-based structured logging throughout
- **Reusable DI Container**: Microsoft.Extensions.DependencyInjection for service management

## Technical Details

### Threading Model
The module handles PowerShell's threading limitations by maintaining references to the active cmdlet instance, allowing for proper use of PowerShell output streams across threads.

### Privilege Elevation
The module prefers using a `sudo` implementation if available, such as the official `sudo` imlementation in Windows 11, [gsudo](https://github.com/gerardog/gsudo), or any other `sudo.exe` which is in the `PATH`. Otherwise, it will fall back to standard Windows UAC prompts via `runas`.

### Process Injection Method
Uses the classic `CreateRemoteThread` approach via Windows API P/Invoke for DLL injection. All handles are tracked and properly released regardless of success/failure to avoid leaking memory.

## Important Notes

⚠️ **Security Warning**: This tool performs process injection and requires careful, responsible use. Injecting arbitrary code into processes can:
- Destabilize target applications
- Trigger anti-virus/anti-cheat software
- Violate terms of service

**Use only on processes you own or have explicit permission to modify.**

⚠️ **Thread Safety**: Separate, concurrent invocations on the same process should be avoided. Multiple sequential DLL injections performed within the same cmdlet call are supported and safe.

## Troubleshooting

### "Access Denied" Errors
- Try running the Cmdlet again wih the same args, but with the `-Admin` parameter added
- **Temporarily** disable antivirus software and test the injection again. If successful, re-enable the antivirus software and whitelist the path to the module DLL
- Check that the target process isn't protected by anti-injection mechanisms. This is not common, but is a possibility

### DLL Fails to Load
- Verify that the DLL path is correct, accessible, and that you have the appropriate filesystem access rights to read it
- Ensure the DLL architecture (x86/x64) matches the target process
- Check that all DLL dependencies are available

### Process Not Found
- Verify the process name or PID is correct
- Use `-Wait` with `-Name` if the process hasn't started yet
- Check that the process hasn't already exited

## Contributing

Feel free to submit pull requests, report bugs, or suggest features.

### Development Setup
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This hideous monstrosity that no one asked for is available under the MIT license, as detailed in the `LICENSE` file.

## Disclaimer

This software is provided "as is" without warranty of any kind. Use at your own risk. The authors are not responsible for any damage or legal consequences resulting from the use of this software. Don't be an idiot.

## Author

Christopher Conley

## Acknowledgments

- Built with love for the bit 💉🩸
- Styled with [Spectre.Console](https://github.com/spectreconsole/spectre.console) (and manual ANSI codes where Serilog wants to be an asshole)
- Logging with [Serilog](https://github.com/serilog/serilog)
- This project would not exist if [Advanced DLL Injector](https://github.com/s4yr3x/advanced-DLLInjector) supported commandline arguments. It's a great tool, but good God, I got so sick of having to fully interact with it every time I wanted to use it. But it's still a great tool.

---

*Yes, I know that Phlebotomists are typically extracting blood/something from blood and not introducing something into it, and no, I don't care, I'm committed to the bit at this point.*
