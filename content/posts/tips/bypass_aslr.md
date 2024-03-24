---
title: Bypass Address Space Layout Randomization (ASLR)
url: /tips/bypass_aslr
categories: ["Tips"]
tags: ["Ghidra", "Reverse Engineering", "x64dbg"]
date: 2023-10-08
summary: "Different techniques to bypass/disable ASLR."
lightgallery: true
---

## DLL Characteristics

To know if a PE file will use ASLR, you can check the ***DLL Characteristics*** in the ***Optional Header*** PE headers. If the field contains the value `0x0040`( cf [Microsoft Documentation - DLL characteristics](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#dll-characteristics)), it means that the image base address can be relocated at load time. 

For example, if we check with `PE-bear`:

{{< image src="images/tips/bypass_aslr/2023-10-08-143417.png" caption="PE Bear DLL characteritics" >}}

{{< admonition type=warning >}}
You can modify this value in a hex editor, however this is not recommended as you will modify the file itself and it may behave differently.
{{< /admonition >}}

## Rebase in Ghidra

If you are working on a sample in both a debugger and a disassembler, you can modify the image base address of the binary in the disassembler to have the same alignment than the one from memory found in the debugger.

First, find the virtual address in the debugger. In `x64dbg`, go to ***Memory Map***, and then find the address related to the name of the sample:

{{< image src="images/tips/bypass_aslr/2023-10-08-144705.png" caption="x64dbg base image address" >}}


In Ghidra, go to ***Window -> Memory Map***:
{{< image src="images/tips/bypass_aslr/2023-10-08-155126.png" caption="Memory map in Ghidra" >}}

Then click on the ***Home*** icon and specify the address in the new window:
{{< image src="images/tips/bypass_aslr/2023-10-08-155310.png" caption="Modify base address in Ghidra" >}}

{{< admonition type=warning >}}
This would need to be done every time the address changes (reboot of the system). This may not be the case when the program is restarted, as mentioned [here](https://www.mandiant.com/resources/blog/six-facts-about-address-space-layout-randomization-on-windows):
`each DLL or EXE image gets assigned a random load address by the kernel the first time it is used, and as additional instances of the DLL or EXE are loaded, they receive the same load address. If all instances of an image are unloaded and that image is subsequently loaded again, the image may or may not receive the same base address. Only rebooting can guarantee fresh base addresses for all images systemwide.`
{{< /admonition >}}


## Disable ASLR at Operating System level

In Windows, you can disable ALSR by going into the ***Exploit Protection*** security settings windows and disable all ASLR related settings:
{{< image src="images/tips/bypass_aslr/2023-10-08-160742.png" caption="Disable ASLR in Exploit Protection Windows Security settings" >}}

{{< admonition type=note >}}
Modifying those settings will require a system reboot for them to be applied.
{{< /admonition >}}


## References

- [Microsoft Documentation - DLL characteristics](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#dll-characteristics)
- [ OALabs - Disable ASLR For Easier Malware Debugging With x64dbg and IDA Pro](https://www.youtube.com/watch?v=DGX7oZvdmT0)
- [Six Facts about Address Space Layout Randomization on Windows](https://www.mandiant.com/resources/blog/six-facts-about-address-space-layout-randomization-on-windows)
- [Windows Malware Analysis for Hedgehogs - Beginner Training](https://www.udemy.com/course/windows-malware-analysis-for-hedgehogs-beginner-training/)

