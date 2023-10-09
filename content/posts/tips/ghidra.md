---
title: Ghidra tips
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

You can add an entropy margin in the Listing view which may be useful to identify encrypted/compressed data:

{{< image src="images/tips/ghidra/2023-08-27-214405.png" caption="Add entropy margin" >}}

### Better route edges in graph view

If in the graphical view you have difficulties to read properly routes edges, you can try the graph option to route edges around vertices (which is disabled by default): ***Edit -> Tool Options -> Options -> Graph -> Nested Code Layout -> Route Edges Around Vertices***:

{{< image src="images/tips/ghidra/2023-08-28-201830.png" caption="Route edges around vertices option" >}}

## Analyzers

### Decompiler Parameter ID Analyzer

You may notice differences in a function parameters between the disassembly and decompiler views. Ghidra decompiler keeps only useful parameters. The ***Decompiler Parameter ID*** analyzer uses decompiler-derived parameter informaion for function's parameters in the disassembly view.

{{< admonition type=warning >}}
This analyzer is enabled by default only for Windows PE files smaller than 2MB.
{{< /admonition >}}

If the analyzer was not enabled during the first analysis, it can be run later with ***Analysis -> One Shot -> Decompiler Parameter ID***.

{{< image src="images/tips/ghidra/2023-08-26-224735.png" caption="Decompiler Parameter ID analyzer description" >}}

### Disable PDB Analyzer
If analyzing executables that you compiled yourself, you may not want to load related `PDB` files as it is very rare to have them when analysing malware samples. This is enabled by default during the analysis step and you can disable it at that point :


{{< image src="images/tips/ghidra/2023-09-04-123732.png" caption="Disable PDB analyzer" >}}

{{< admonition type=note >}}
There is no prompt asking to load or not the PDB when first importing a file like in IDA.
{{< /admonition >}}

## Cross references (XREFS)

### Unwanted XREFS

If the address of a function happens to be at the very beginning of the `.text` section, some PE fields in the Headers section (like **BaseOfCode** for example) indirectly reference the start of the `.text` section, which may create additional XREFS.

## Namespaces

To better organize functions through your analysis, you can create namespaces and drag an drop the function into it. When adding a function into a namespace, it will no longer be available in the ***Functions*** directory. You can also create sub-namespaces:
{{< image src="images/tips/ghidra/2023-10-09-165641.png" caption="Namespaces created for better organization" >}}

If a function is part of a namespace, it will be appended at the beginning of its name, as well as all sub-namespaces it is a part of:
{{< image src="images/tips/ghidra/2023-10-09-170333.png" caption="Namespaces and sub-namespaces appended at the beginning of the functioname" >}}


## References

- ***The Ghidra Book: The Definitive Guide*** by Chris Eagle and Kara Nance
- https://twitter.com/embee_research/status/1582274169068134400
