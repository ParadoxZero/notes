Types of breakpoints - 
- Bp - break point
- Bu - unresolved breakpoint
- Bm - sets a new breakpoint on symbols that match a specified pattern. This command can create more than one breakpoint. By default, after the pattern is matched, bm breakpoints are the same as bu breakpoints
- Memory breakpoints - TODO

# Kernel Debugger ( KD )

## Configure KD

```
bcdedit [/store "E:\EFI\Microsoft\Boot\BCD"] /dbgsettings NET hostip:10.171.69.197 port:50001 key:1.2.3.4
```

## Enable KD

If via mounted VHD

```
bcdedit [/store "E:\EFI\Microsoft\Boot\BCD"] /debug {default} on
```

Else

```
bcdedit  /debug on
```

# Commands

## Set BA

```
ba e 1 xxxx
```

## Search process-

```
!process /m ppibars.dll 0 0
```

## Change context

```
.process /r /p 00ffff............
```

## Search code and set breakpoints

```
x *ppibars.dll*!*SmallBottomBar::OnSkypeButtonClick*
Bp oofffasdfs // add breakpoint
Bl // to check if breakpoint is added
```

## pspinsertprocess

```
ad /q ${/v:NewImageName}

bu nt!PspInsertProcess "as /ma ${/v:NewImageName} @@c++(NewProcess->ImageFileName);.block {.echo Creating process: NewImageName;.if ($spat(\"${NewImageName}\",\"ppishell.exe\") == 0) {gc} .else { }}"

kd> dv
   NewProcess = 0xffff9e89`111d2080

2: kd> .process /p /r 0xffff9e89`111d2080

7: kd> dt nt!_PEB

	+0x002 BeingDebugged : UChar
	if (Peb->BeingDebugged) {
	LdrpDoDebuggerBreak ();
	}

2: kd> * Mark the process as being debugged.

2: kd> eb @$peb+2 1
```

## List thread-

```
!process -1 7
```

## Switch thread

```
.thread 9d546b80
```

## Find Process by name

```
!process 0 0 werfault.exe
```

## Print all Stacks

```
~*k
```

## Break on Dll Load

```
 !gflag +ksl  
 sxeld mymodule.dll
```

## Debug WINAPI

```
x  advapi32!InitiateShutdownW

ba e 1 xxxx <- BA
```

# References

1. [Debugging 106 - Kernel Debugging - OSG Wiki](https://www.osgwiki.com/wiki/Debugging_106_-_Kernel_Debugging)
2. [.process (Set Process Context) - Windows drivers | Microsoft Docs](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-process--set-process-context-)
3. [process - Windows drivers | Microsoft Docs](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-process)