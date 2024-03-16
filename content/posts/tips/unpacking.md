---
title: Unpacking tips
url: /tips/unpacking
categories: ["Tips"]
tags: ["Reverse Engineering", "x64dbg"]
date: 2024-03-05
summary: "Tips for unpacking."
lightgallery: true
---

## Identify packed samples

### Review sections

#### High entropy

Entropy measures the randomness and complexity of the data. Encrypted and compressed data will resut in a high entropy, which helps to identify packed data.

If we take a simple example by packing `calc.exe` with the famous [UPX](https://upx.github.io) packer, we can compare entropy between both samples.

With [Detect It Easy (DIE)](https://github.com/horsicq/Detect-It-Easy), we can see it recognizes the usage of UPX:

{{< image src="images/tips/unpacking/2024-03-12-125606.png" caption="DIE analyses" >}}

If we compare entropy, a section has a very high entropy (above 7.0) in the packed sample:

{{< image src="images/tips/unpacking/2024-03-12-125424.png" caption="High entropy section" >}}

You can also check section's entropy with [Pestudio](https://www.winitor.com):

{{< image src="images/tips/unpacking/2024-03-16-125926.png" caption="Section's entropy in PeStudio" >}}

#### Absence of standard sections and usage of custom one

In the example above we can also see some standard sections are missing and custom ones are used, this can also be useful to identify packer.

#### Differences between raw and virtual sizes

To compare sections sizes between raw and virtual, one very useful tool for this is [PE-bear](https://github.com/hasherezade/pe-bear), which also provides a visualization:

{{< image src="images/tips/unpacking/2024-03-16-130752.png" caption="View section's raw and virtual sizes with PE-bear" >}}

Here the fact that a section has a raw size of zero but a consequent virtual size is very suspicious for the presence of a packer.   

### Review Imports

- No imports at all  
- Small number of imports and presence of both `LoadLibrary` and `GetProcAddress` (common APIs used to resolve imports)  
Example:
{{< image src="images/tips/unpacking/2024-03-16-144522.png" caption="Imports differences between packed and unpacked sample" >}}

- Absence of imports related to executable functionalities based on dynamic analysis or expected behavior (like networking, encryption related functions)  

## References

- Book - Practical Malware Analysis: The Hands-On Guide to Dissecting Malicious Software by Michael Sikorski  
- [Zero 2 Automated reverse engineering course](https://courses.zero2auto.com)
- [Unveiling custom packers: A comprehensive guide](https://estr3llas.github.io/unveiling-custom-packers-a-comprehensive-guide)  
