---
title: Ghidra
url: /tips/ghidra
categories: ["Tips"]
tags: ["Ghidra", "Reverse Engineering"]
date: 2023-08-26
summary: "Collection of tips I gathered on Ghidra during my RE journey."
lightgallery: true
---

## GUI tips

### Cursor Text Highlight

By default, you need to use the middle mous button on a variable to highlight all other occurences in the code browser. You can change it to the left mouse button for a more natrual feel.

Go to ***Edit -> Tool Options*** and then in the new window: ***Options -> Listing Fields -> Cursor Text Highlight*** and change the ***Mouse Button To Actovate*** from ***MIDDLE*** to ***LEFT***:

{{< image src="images/tips/ghidra/2023-08-27-213257.png" caption="Cursor Text Highlight" >}}

### Add entropy margin

You can add an entropy margin in the Listing view which may be usefull to identify encrypted/compressed data:

{{< image src="images/tips/ghidra/2023-08-27-214405.png" caption="Add entropy margin" >}}


## Analyzers

### Decompiler Parameter ID Analyzer

You may notice differences in a function parameters between the disassembly and decompiler views. Ghidra decompiler keeps only useful parameters. The ***Decompiler Parameter ID*** analyzer uses decompiler-derived parameter informaion for function's parameters in the disassembly view.

{{< admonition type=warning >}}
This analyzer is enabled by default only for Windows PE files smaller than 2MB.
{{< /admonition >}}

If the analyzer was not enabled during the first analysis, it can be run later with ***Analysis -> One Shot -> Decompiler Parameter ID***.

{{< image src="images/tips/ghidra/2023-08-26-224735.png" caption="Decompiler Parameter ID analyzer description" >}}

## Cross references (XREFS)

### Unwanted XREFS

If the address of a function happens to be at the very beginning of the `.text` section, some PE fields in the Headers section (like **BaseOfCode** for example) indirectly reference the start of the `.text` section, which may create additional XREFS.

## References

- ***The Ghidra Book: The Definitive Guide*** by Chris Eagle and Kara Nance
- https://twitter.com/embee_research/status/1582274169068134400
