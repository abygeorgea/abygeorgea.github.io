---
layout: post
title: "Powershell - Copying Folders and Files"
date: 2016-10-10 06:47:54 +1000
comments: true
categories: Powershell
---


Two options for copying files are below.



- Robocopy - More details can be found at [Robocopy](https://technet.microsoft.com/en-us/library/cc733145.aspx)
- Copy_Item cmdlet - More details can be found at [Copy-Item](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.management/copy-item)


### Copying Folder structure Only ###
``` plain
$source = "C:\tools\DevKit2\octopress-blog\source"
$dest = "D:\delete"
Copy-Item $source $dest -Filter {PSIsContainer} -Recurse -Force

#OR
robocopy $source $dest /e /xf *.*

# /e denotes all folder including empty folders. /xf denotes all files except one of format *.*
# /e can be replaced with /s for ignoring empty folders
```

### Flattening Folder structure - Copy all files from nested folders to a single folder ###

``` plain
$source = "C:\tools\DevKit2\octopress-blog\source"
$dest = "D:\delete"
# Below is required only if we need to create destination folder. Uncomment below line if folder needs to be created
#New-Item $dest -type directory 

Get-ChildItem $source -Recurse | `
    Where-Object { $_.PSIsContainer -eq $False } | `
    ForEach-Object {Copy-Item -Path $_.Fullname -Destination $dest -Force} 
```

### Copy same folder structure ###
``` plain
$source = "C:\tools\DevKit2\octopress-blog\source"
$dest = "D:\delete"
robocopy $source $dest /e
```