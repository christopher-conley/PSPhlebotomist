---
external help file: PSPhlebotomist.dll-Help.xml
Module Name: PSPhlebotomist
online version: https://github.com/christopher-conley/PSPhlebotomist
schema: 2.0.0
---

# New-Injection

## SYNOPSIS
Injects one or more DLLs into a target process.

Aliases: New-Syringe, New-Needle, New-Patient, Insert-Needle

## SYNTAX

### default (Default)
```
New-Injection [-Inject <String[]>] [-Id <IntPtr>] [-ProcessName <String>] [-Wait] [-Timeout <UIntPtr>] [-Admin]
 [-ProgressAction <ActionPreference>] [<CommonParameters>]
```

### pid
```
New-Injection [-Inject <String[]>] -Id <IntPtr> [-ProcessName <String>] [-Wait] [-Timeout <UIntPtr>] [-Admin]
 [-ProgressAction <ActionPreference>] [<CommonParameters>]
```

### pname
```
New-Injection [-Inject <String[]>] [-Id <IntPtr>] -ProcessName <String> [-Wait] [-Timeout <UIntPtr>] [-Admin]
 [-ProgressAction <ActionPreference>] [<CommonParameters>]
```

## DESCRIPTION
A PowerShell cmdlet for injecting one or more DLLs into a target process, supporting both process ID and process name selection, optional elevation, and interactive or automated operation.

This cmdlet can be invoked using several aliases, including 'New-Syringe', 'New-Injection', 'New-Patient', and 'Insert-Needle', because at one point I decided that I'm all-in on the bit.

It supports both automated and interactive modes: if no parameters are provided, or only the -Admin parameter is provided, the cmdlet will prompt for input interactively. The cmdlet will attempt to elevate privileges and relaunch with the original commandline args if the 'Admin' switch is specified and it is not already running within an Administrator security context. If already running within an Administrator security context, the -Admin switch is ignored and the normal injection workflow and logic continues. This Cmdlet prefers to use an implmentation of sudo to elevate privileges if available, otherwise it will use the standard Windows process launch and UAC prompt/runas flow.

Output is a PatientDiagnosis object representing the result of the injection attempt(s). For best results, ensure that the target process is accessible and that DLL paths are valid. Thread safety is not guaranteed; concurrent invocations on the same process should probably be avoided, although multiple DLL injections *within the same Cmdlet call* are a native and supported feature. Use the 'Wait' switch to block and wait for the process to launch (only valid when using Process Name, not PID), and 'Timeout' to specify the maximum process launch wait duration in seconds.

## EXAMPLES

### Example 1: Injection by Process ID
```powershell
PS C:\> New-Injection -Id 98472 -Inject "C:\path\to\my.dll"
```

Injects the specified DLL into the process with ID 98472.

### Example 2: Injection by Process Name
```powershell
PS C:\> New-Injection -ProcessName "notepad" -Inject "C:\path\to\my.dll"
```

Injects the specified DLL into a process named "notepad". The .exe extension is optional.

### Example 3: Multiple DLL injection
```powershell
PS C:\> New-Injection -Id 9970 -Inject "C:\First.dll", "C:\Second.dll", "C:\Third.dll"
```

Injects three DLLs into the process with ID 9970 in the specified order.

### Example 4: Multiple DLL injection, explicit PowerShell array syntax
```powershell
PS C:\> New-Injection -Id 9970 -Inject @("C:\First.dll", "C:\Second.dll", "C:\Third.dll")
```

Injects three DLLs into the process with ID 9970 in the specified order, using explicit PowerShell array syntax.

### Example 5: Wait for process launch
```powershell
PS C:\> New-Injection -ProcessName "target.exe" -Inject "C:\Your.dll" -Wait
```

Waits indefinitely for a process named "target.exe" to start, then injects the DLL "C:\Your.dll" immediately upon launch.

### Example 6: Wait with timeout
```powershell
PS C:\> New-Injection -ProcessName "application.exe" -Inject "C:\intercept.dll" -Wait -Timeout 60
```

Waits up to 60 seconds for a process named "application.exe" to start, then injects the DLL "C:\intercept.dll" immediately upon launch. If the target process is not detected after 60 seconds have elapsed, the command times out and the injection attempt is abandoned.

### Example 7: Elevate privileges first, inject by Process ID
```powershell
PS C:\> New-Injection -Id 9924 -Inject "C:\Your.dll" -Admin
```

Relaunches the cmdlet with Administrator privileges, then injects the DLL "C:\Your.dll" into the process. If already running within an Administrator security context, this parameter is ignored.

### Example 8: Interactive mode
```powershell
PS C:\> New-Injection
```

Enters interactive mode. You will be prompted step-by-step for all required information.

### Example 9: Pipeline input (SEE NOTES SECTION)
```powershell
PS C:\> Get-Process notepad | New-Injection -Inject "C:\logger.dll"
```

Accepts process information from the pipeline (specifically the `ProcessName` and `Id` attributes) and injects the DLL "C:\logger.dll" into ALL matching notepad processes. If you want to target a single instance of a process with multiple running threads, ensure your filter criteria in `Get-Process` (or other cmdlet) returns only a single object.

### Example 10: Using aliases
```powershell
PS C:\> New-Syringe -ProcessName "calc" -Inject "C:\hook.dll"
```

Uses the 'New-Syringe' alias to inject a DLL into the calculator process.

## PARAMETERS

### -Admin
Indicates whether the cmdlet should attempt to elevate privileges and relaunch as administrator. When this switch is specified and the cmdlet is not already running with administrator privileges, it will attempt to relaunch itself in an elevated context, preserving the original command-line arguments. The cmdlet prefers using a sudo implementation if available, otherwise it falls back to the standard UAC prompt. If already running within an Administrator security context, this parameter is ignored.

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: AsAdmin, Administrator, AsAdministrator, Root

Required: False
Position: Named
Default value: False
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -Id
Specifies the Process ID (PID) of the target process which will receive the injected DLLs/PE images. This parameter is mandatory when using the 'pid' parameter set. The process must be accessible and running for injection to succeed when using this parameter. This parameter cannot be used simultaneously with the `ProcessName` parameter in the same parameter set, unless the cmdlet is receiving input from the pipeline, such as output from the `Get-Process` cmdlet.

```yaml
Type: IntPtr
Parameter Sets: default, pname
Aliases: ProcessId, PID

Required: False
Position: Named
Default value: None
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

```yaml
Type: IntPtr
Parameter Sets: pid
Aliases: ProcessId, PID

Required: True
Position: Named
Default value: None
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -Inject
Specifies an array of file paths (or a single string) to DLL files that will be injected into the target process. This parameter accepts multiple DLL paths and can be provided by property name. If no DLLs are specified, the cmdlet will enter interactive mode to prompt for input. Paths should be valid and accessible to the current user context. Providing a single string to this parameter instead of an array will result in the input being treated as an array with a single element.

```yaml
Type: String[]
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -ProcessName
Specifies the name of the target process to inject DLLs into. This parameter is mandatory when using the 'pname' parameter set. When combined with the Wait switch, the cmdlet will wait for a process with this name to launch before attempting injection. This parameter cannot be used simultaneously with the `Id` parameter in the same parameter set, unless the cmdlet is receiving input from the pipeline, such as output from the `Get-Process` cmdlet.

```yaml
Type: String
Parameter Sets: default, pid
Aliases: Name, PName

Required: False
Position: Named
Default value: None
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

```yaml
Type: String
Parameter Sets: pname
Aliases: Name, PName

Required: True
Position: Named
Default value: None
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -Timeout
Specifies the maximum time (in seconds) to wait for a process to launch when using the `Wait` switch. This parameter only applies when the `Wait` switch and a process name is specified, and is ignored otherwise. If the target process does not launch within the specified timeout period, the cmdlet will terminate with an error. The default value is the maximum unsigned integer value for the current architecture.

```yaml
Type: UIntPtr
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Maximum UInt value (platform-dependent)
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -Wait
Indicates whether the cmdlet should wait for the target process to launch before attempting injection. This switch is primarily useful when using the ProcessName parameter to specify a process by name. When enabled, the cmdlet will block and monitor for a process with the specified name to start, up to the timeout duration. This is useful for injecting into processes that are about to be launched.

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -ProgressAction
Controls how progress messages are displayed. This is a common parameter available in PowerShell 7+.

```yaml
Type: ActionPreference
Parameter Sets: (All)
Aliases: proga

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### CommonParameters
This cmdlet supports the common parameters: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction, and -WarningVariable. For more information, see [about_CommonParameters](http://go.microsoft.com/fwlink/?LinkID=113216).

## INPUTS

### System.String[]
You can pipe an array of DLL file paths to this cmdlet.

### System.IntPtr
You can pipe a process ID to this cmdlet.

### System.String
You can pipe a process name to this cmdlet.

### System.Management.Automation.SwitchParameter
You can pipe switch parameter values to this cmdlet.

### System.UIntPtr
You can pipe an unsigned integer timeout value to this cmdlet.

## OUTPUTS

### PSPhlebotomist.Core.PatientDiagnosis
Returns a PatientDiagnosis object containing the results of the injection attempt(s), including injection success/failure status, target process information, error details (if any), and diagnostic information.

## NOTES
- Thread safety is not guaranteed. Avoid concurrent invocations on the same process, although multiple DLL injections within the same cmdlet call are supported.
- When privilege elevation is required, the cmdlet will attempt to use an available sudo implementation before falling back to the standard Windows UAC prompt.
- If no parameters are provided (or only -Admin is specified), the cmdlet enters interactive mode.
- This cmdlet uses the classic CreateRemoteThread approach via Windows API P/Invoke for DLL injection.
- All handles are tracked and properly released regardless of success/failure to avoid memory leaks.
- IMPORTANT: When piping input to this cmdlet as part of a pipeline, ensure that the filtering criteria in the cmdlet immediately prior to this cmdlet in the pipeline matches your intent. PowerShell will invoke this cmdlet FOR EACH element in the pipeline, so if you call `Get-Process "notepad.exe"` and there are 4 instances of `notepad.exe` running, this cmdlet will be called 4 TIMES and attempt to inject your DLLs into each running instance. If the intent is to inject into only one of the running processes, adjust your filtering criteria in or after `Get-Process` or other cmdlet to ensure only one object is returned. This is a behavior inherent to PowerShell's pipeline processing model and cannot be changed; it is not specific to this cmdlet.

## RELATED LINKS

[Project Repository:](https://github.com/christopher-conley/PSPhlebotomist)

[Get-Process:](https://docs.microsoft.com/powershell/module/microsoft.powershell.management/get-process)
