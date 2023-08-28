# DebugAmsi
## TL;DR

Just run the file from Release and get access to powershell without AMSI!



https://github.com/MzHmO/DebugAmsi/assets/92790655/7417215b-ed0d-47f5-b879-c1f0debc110a



## How It Works
One day I've discovered an interesting function [DebugActiveProcess](https://learn.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-debugactiveprocess) which allows us to become a debugger for a process. Full-fledged debugging will be available if our process has the SeDebug privilege or the ability to call `OpenProcess()` with the `PROCESS_ALL_ACCESS` mask. 

Once our process becomes a debugger, it can handle the `LOAD_DLL_DEBUG_EVENT` event, which is generated by the Windows system when any DLL is loaded into the process address space.

Thus, we can start powershell.exe, then become a debugger for it and intercept an attempt to load amsi.dll . And then patch it at the moment of loading.

The problem is that we may miss the point of loading amsi.dll into the powershell.exe process, so in my code, the process starts in a suspended state (at that point, it will only have ntdll.dll in its address space). Then installs the debugger and resumes the main thread of the process, which causes powershell.exe to resume and load the necessary libraries.
![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/028494f2-01de-47de-967f-7b54728acbb7)


![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/8ccbb250-67c7-40a1-b08a-c7b304af34a8)


After generating the LOAD_DLL_DEBUG_EVENT event, I find amsi.dll and parsing its EAT to find the AmsiOpenSession and AmsiScanBuffer functions.
![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/ce608515-0518-4a25-b0fb-9a0216bd591b)


EAT parsing is done by reading the memory of the amsi.dll library loaded in powershell.exe . This is done so as not to load amsi.dll into our own process.
![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/27d24ee7-1d5c-4885-a2e4-4c0e4aa74736)


After that, a simple patch is applied that renders AMSI useless. This results in a running powershell.exe process with amsi disabled. Also added hiding strings by encrypting them at compile time using XOR with a dynamic key (macroc `h()` for ASCII and `hW()` for Unicode)
