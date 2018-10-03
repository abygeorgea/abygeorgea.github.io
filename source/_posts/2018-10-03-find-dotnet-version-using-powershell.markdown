---
layout: post
title: "Find DotNet Version Using Powershell"
date: 2018-10-03 21:39:55 +1000
comments: true
categories: [codesnippet, powershell]
keywords: codesnipper, powershell
description: How to find DotNet Version using powershell
---

I recently faced an issue where one utility tool created by me was not running properly on another machine which had different dot net version installed. During troubleshooting, I was looking for ways to identify the installed dotnet version. Most of the links in google suggested to look for release value in registry as specified [here](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed). 

Below powershell script will list down installed dotnet version on a machine. This is based on dotnet version listed on https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed. We may have to update below snippet as when new versions are released.  Currently it supports upto dotnet 4.7.2


```
$netRegKey = Get-Childitem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full"  
$release = $netRegKey.GetValue("Release")  
Write-host $release
$releases =@(  
    @{id="378389";value=".NET Framework 4.5"},  
    @{id="378675";value=".NET Framework 4.5.1"},  
    @{id="379893";value=".NET Framework 4.5.2"},  
    @{id="393295";value=".NET Framework 4.6"},   
    @{id="393297";value=".NET Framework 4.6"},  
    @{id="394254";value=".NET Framework 4.6.1"}, 
    @{id="394271";value=".NET Framework 4.6.1"},  
    @{id="394802";value=".NET Framework 4.6.2"},      
    @{id="394806";value=".NET Framework 4.6.2"},  
    @{id="460798";value=".NET Framework 4.7"}, 
    @{id="460805";value=".NET Framework 4.7"},    
    @{id="461308";value=".NET Framework 4.7.1"}, 
    @{id="461310";value=".NET Framework 4.7.1"},
    @{id="461808";value=".NET Framework 4.7.2"},  
    @{id="461814";value=".NET Framework 4.7.2"}
    # Update more if new versions are released
)  
  
  
foreach($framework in $releases)  
{  
    if($framework.id -eq $release){  
        Write-Output $framework.value  
    }  
}  
```

Note: This assumes user can run Powershell with admin access so that it does not go through constraint language Mode . If running above script result in error like "Method Invocation is supported only on core types in this language mode", it means it is on constraint language mode. In that case, we can run just `Get-Childitem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full"` and manually look for release key in above microsoft  link

