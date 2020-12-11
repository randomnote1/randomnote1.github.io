---
date: 2020-12-11 00:00:00 -04:00
layout: post
categories: PowerShell
title: Manually install a module from the PowerShell Gallery
---

There are instances where a computer does not have internet access and therefore cannot use the `Install-Module` cmdlet to install modules from the [PowerShell Gallery](https://www.powershellgallery.com/). This article will walk through how to manually download a module and install it on an offline computer.

### Download a module

1. Navigate to the [PowerShell Gallery](https://www.powershellgallery.com/)1. Search for the desired module
1. Select the **Manual Download** tab
1. Click the **Download the raw nupkg file**
1. After the file finishes downloading, transfer it to the desired computer.

### Install the module

1. Rename the module replacing the `.nupkg` extension with a `.zip`

   ```powershell
    Move-Item -Path .\sqlserver.21.1.18230.nupkg -Destination .\sqlserver.21.1.18230.zip
    ```

1. Extract the ZIP file. The resulting folder will have a name formatted like _<Module Name>.<Module Version>_

    ```powershell
    Expand-Archive -Path .\sqlserver.21.1.18230.zip
    ```
  
1. Determine [where to install the module](https://docs.microsoft.com/powershell/scripting/developer/module/installing-a-powershell-module?view=powershell-5.1#where-to-install-modules). For this example the module will be installed in `$env:ProgramFiles`

    ```powershell
    $env:PSModulePath.Split(';')
    ```

1. Create a new folder in `$Env:ProgramFiles\WindowsPowerShell\Modules` with the name **Module Name**

    ```powershell
    New-Item -Path $env:ProgramFiles\WindowsPowerShell\Modules\SqlServer -ItemType Directory
    ```

1. Rename the module folder to be only the module version.

    ```powershell
    Move-Item -Path .\sqlserver.21.1.18230 -Destination .\21.1.18230
    ```

1. Move the module version folder into the module name folder.

    ```powershell
    Move-Item -Path .\21.1.18230 -Destination $env:ProgramFiles\WindowsPowerShell\Modules\SqlServer
    ```
