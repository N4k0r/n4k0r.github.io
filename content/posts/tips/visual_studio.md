---
title: Visual Studio tips
url: /tips/visual_studio
categories: ["Tips"]
tags: ["Programming"]
date: 2024-03-24
summary: "Collection of tips I gathered on Visual Studio during my programming journey."
lightgallery: true
---

## Disable optimization

If you disassemble a program you write yourself, you may want at first to disable optimization and enable it later on and compare differences to better understand how the optimization modifies the assembly code.

To disable it (it should be enabled by default): 
```{lineNos=false}
Project
└── <Project_name> Properties
    └── Configuration Properties
        └── C/C++
            └── Optimization
                └── Optimization = Disabled (/Od)
```

{{< image src="images/tips/visual_studio/2024-03-24-144922.png" caption="Access project properties" >}}

{{< image src="images/tips/visual_studio/2024-03-24-145122.png" caption="Disable optimization" >}}

## Display inline hints

When working with Windows APIs, it can be very helpful to display function's signature directly in visual studio. 

You can configure it with: 
```{lineNos=false}
Tools
└── Options
    └── Text Editor
        └── C/C++
            └── IntelliSense
                └── Display inline hints 
```

{{< image src="images/tips/visual_studio/2024-03-09-134515.png" caption="Access Options settings" >}}

{{< image src="images/tips/visual_studio/2024-03-09-134530.png" caption="Enable and display inline hints" >}}

Once enabled, it will look like it:

{{< image src="images/tips/visual_studio/2024-03-24-150308.png" caption="VirtualAlloc call with inline hints" >}}
