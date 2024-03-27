---
title: x64dbg tips
url: /tips/x64dbg
categories: ["Tips"]
tags: ["Reverse Engineering", "x64dbg"]
date: 2024-03-23
summary: "Collection of tips I gathered on x64dbg during my RE journey."
lightgallery: true
---

## Conditional breakpoint

Let's say for example we want to break every time `VirtualProtect` is called but only when the value of the argument related to the protection parameter is one that will add execution permission.

If we check the [VirtualProtect documentation](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect), the argument we are interested in is the third one, which will be translated to `[esp+c]` in the x64dbg window related to stack memory.

We can find several [memory protection constants](https://learn.microsoft.com/en-us/windows/win32/Memory/memory-protection-constants) that will add execution permission. Moreover, there are modifiers that can also be used, wich increases the number of possibility.

One first approach would be to break when one of those value is encountered as a third argument to `VirtualProtect` calls. This would give us, including modifiers, the following list:

```C
[esp+C]==0x10 || [esp+C]==0x20 || [esp+C]==0x40 || [esp+C]==0x80
// Values with PAGE_GUARD(0x100) modifier
|| [esp+C]==0x110 || [esp+C]==0x120 || [esp+C]==0x140 || [esp+C]==0x180
// Values with PAGE_NOCACHE(0x200) modifier
|| [esp+C]==0x210 || [esp+C]==0x220 || [esp+C]==0x240 || [esp+C]==0x280
// Values with PAGE_WRITECOMBINE(0x400) modifier
|| [esp+C]==0x410 || [esp+C]==0x420 || [esp+C]==0x440 || [esp+C]==0x480
```

To add this condition to our breakpoint on `VirtualProtect`:
- Go to the `Breakpoints` tab
- Edit the wanted breakpoint (`Shift+F2`)
- Add the condition in the `Break condition` field

In our case this will give us the following:

{{< image src="images/tips/x64dbg/2024-03-23-190833.png" caption="Add a conditional breakpoint" >}}

{{< admonition type=warning >}}
The field's size limit is reached, so the first approach to list all possible values is not viable.
{{< /admonition >}}

As we are also considering modifiers, we could instead check only the first byte of the argument as this is the one that will contain the execution permission regardless of modifiers. As per the [official x64dbg documentation](https://help.x64dbg.com/en/latest/introduction/Values.html#memory-locations), it is possible to read `n` bytes from an address with the `n:[addr]` syntax.

If we take the example from before and we only check the first byte of the 3rd argument, we would then have:
```
1:[esp+C]==0x10 || 1:[esp+C]==0x20 || 1:[esp+C]==0x40 || 1:[esp+C]==0x80
```

Once added, the condition can be seen in the `Summary` column:
{{< image src="images/tips/x64dbg/2024-03-23-193130.png" caption="Conditional breakpoint checking first byte of 3rd argument to VirtualProtect" >}}

To test our condition, we try it with the following `C` code that will do several calls to `VirtualProtect` with different permission values:
```C {hl_lines=["28-31"]}
#include <stdio.h>
#include <windows.h>

void ChangeProtection(PVOID pVirtualAddress, SIZE_T size, DWORD flNewProtect, PDWORD pflOldProtect);

int main()
{
    PVOID pVirtualAddress;
    SIZE_T size = 1024;
    DWORD pflOldProtect = 0;
    CHAR* cString = "Test";

    pVirtualAddress = VirtualAlloc(NULL,
       size,
        MEM_COMMIT,
        PAGE_READWRITE);

    if (pVirtualAddress == NULL)
    {
        printf("Failed to allocate memory with VirtualAlloc. Error: %d\n", GetLastError());
    }
    else {
        printf("The allocated memory is at address: 0x%p\n", pVirtualAddress);
    }

    memcpy(pVirtualAddress, cString, strlen(cString));

    ChangeProtection(pVirtualAddress, size, 0x04, &pflOldProtect); //PAGE_READWRITE
    ChangeProtection(pVirtualAddress, size, 0x10, &pflOldProtect); //PAGE_EXECUTE
    ChangeProtection(pVirtualAddress, size, 0x02, &pflOldProtect); //PAGE_READONLY
    ChangeProtection(pVirtualAddress, size, 0x240, &pflOldProtect); //PAGE_NOCACHE + PAGE_EXECUTE_READWRITE

    VirtualFree(pVirtualAddress, 
        size,
        MEM_DECOMMIT);

    return 0;
}

void ChangeProtection(PVOID pVirtualAddress, SIZE_T size, DWORD flNewProtect, PDWORD pflOldProtect) {
    BOOL changedProtection = VirtualProtect(pVirtualAddress, size, flNewProtect, pflOldProtect);
    if (changedProtection == TRUE) {
        printf("Successfully changed protection. New value: %#04x\n", flNewProtect);
    }
    else {
        printf("Failed to change protection for value: %#04x\n", flNewProtect);
    }
}
```

We only get two breaks, on value `0x10` and `0x240`, which is what we wanted:

{{< image src="images/tips/x64dbg/2024-03-23-193645.png" caption="First break on PAGE_EXECUTE(0x10)" >}}

{{< image src="images/tips/x64dbg/2024-03-23-193734.png" caption="Second break on PAGE_NOCACHE (0x200)+ PAGE_EXECUTE_READWRITE (0x40)" >}}
