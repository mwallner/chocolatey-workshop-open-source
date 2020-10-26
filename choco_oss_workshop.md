# Chocolatey (OSS) Workshop

This workshop aims to provide a basic understanding of the Chocolatey package manager.
A series of exercises will guide you through the installation of Chocolatey itself, package installation and creation as well as some additional material on how to get started using Chocolatey OSS in an organization.

Chocolatey Workshops on GitHub

* [this workshop](https://github.com/chocolatey-community/chocolatey-workshop-open-source)
* [original workshop by Rob Reynolds](https://github.com/chocolatey/chocolatey-workshop)
* [Chocolatey 4 Business by Paul Broadwidth and Gary Park](https://github.com/chocolatey/chocolatey-workshop-organizational-use)

## Prerequisites

* Computer with network connection and RDP client
  * on Windows, you are probably all set
  * on macOS, get Microsoft Remote Desktop from the App Store
  * on Linux, get [remmina](https://wiki.ubuntuusers.de/remmina/)

## What is Chocolatey

> Modern Software Automation

Chocolatey Software Inc.

> Like apt-get, but for Windows

Rob Reynolds, a.k.a. Willy Wonka - CEO and founder of Chocolatey

> It's the solution to automating the Windows OS

Manfred Wallner, Linux Advocate turned Chocolatey enthusiast

## Workshop Exercises

The following exercises are meant to be guided tasks after which workshop participants should be able to repeat the required steps on their own without further assistance.

* Exercises
  * [1: Install Chocolatey](#exercise-1-Install-Chocolatey)
  * [2: Install a Package from the Community Repository](exercise-2-Install-Visual-Studio-Code)
  * [3: Install a Package using package parameters](exercise-3-Install-Git-using-package-parameters)
  * [4: Create a package the old fashioned way](exercise-4-Create-a-package-the-old-fashioned-way)
  * [5: Manually repackage something from the Community Repository](#exercise-5-manually-repackage-something-from-the-community-repository)
  * [6: Setup a local Chocolatey Repository](#exercise-6-setup-proget-as-chocolatey-repository)
  * [7: Automatically Test Chocolatey Packages](#exercise-7-automatically-test-chocolatey-packages)
  * [8: Use Boxstarter to deploy Packages requiring a reboot](#exercise-8-use-boxstarter-to-deploy-packages-requiring-a-reboot)

## What is a Package

A fancy zip file containing:

* a **.nuspec**, containing the metadata for the Package (e.g. name, version, description or dependencies)
* _OPTIONAL_: a couple of PowerShell files which are run on certain events
  * chocolateyInstall.ps1
  * chocolateyUninstall.ps1
  * chocolateyBeforeModify.ps1
* _OPTIONAL_: arbitrary additional files
  * MSI/exe files
  * ZIP files
  * ...

## Chocolatey Installation

### Online Installation

The PowerShell way
```PowerShell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

The cmd.exe way
```cmd
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command " [System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

:warning: Both methods really break down to do the same thing - calling the `install.ps1` of chocolatey.org. You **DO NOT** want to do this in an organizational context (for many reasons including security) - see [Offline Installation](#Offline-Installation).

### Exercise 1: Install Chocolatey

TODO: task description

### Offline Installation

It's perfectly fine to utilize the online installer script of Chocolatey in small/private scenarios, but you'll definitely want to take a good look at **Security** and **Availability** in organizational use cases.

* Security
  * do not run arbitrary code
  * do not take any dependency on a resource that's not checked by an Anti Virus tool
* Availability
  * do not rely on remote resources to be available - have a internal copy of everything you need

See [completely offline install](https://chocolatey.org/docs/installation#completely-offline-install) on [chocolatey.org](https://chocolatey.org).

TODO: Prepare offline installer (7z SFX installer)[https://florianwinkelbauer.com/posts/2019-09-27-standalone-chocolatey-installer/]

## Installing Packages from the Community Repository

### A quick word on Fair-Use limits (a.k.a. "Rate Limiting")

To improve stability and availability of the Community Repository, installs and upgrades are limited per IP address.
To put it simple: be nice, if you're using Chocolatey on more than a handful of machines, setup a local cache and download the packages from there. (see [Local Chocolatey Package Repository](#local-chocolatey-package-repository))

:warning: purchasing a pro/enterprise license has no effect on rate-limiting!

> NOTE: If you do find you have been blocked / rate-limited, having commercial licenses will not have any effect the policies with the community package repository. These policies were put into place to ensure stability and availability for the entire community, not to try to get folks to pay for licensing.
> Blocks are meant to be temporary bans, but require you to act to remedy the situation. If you have been blocked, please see the next sections for corrective actions.

source: [chocolatey.org](https://chocolatey.org/docs/community-packages-disclaimer#excessive-use)

### Exercise 2: Install Visual Studio Code

1. Open an Administrator terminal (`powershell.exe` or `cmd.exe`)
2. Call `choco install vscode -y`
3. Note the message "Environment Vars have changed".
4. Type `code`. Notice that it errors.
5. Type `refreshenv`.
6. Type `code`. Note that it opens Visual Studio Code.

### Exercise 3: Install Git using package parameters

1. Call `choco install git.install --params "/GitAndUnixToolsOnPath /NoGitLfs /SChannel /NoAutoCrlf"`
2. Inspect the content of `chocolateyInstall.ps1` and approve the installation
3. Type `refreshenv`.
4. Type `git --version`

### List installed packages

```PowerShell
choco list -lo
```

### Check for outdated packages

```PowerShell
choco outdated
```

Another alternative:

```PowerShell
choco upgrade all -noop
```

**Try adding the `-r` switch to the commands above!**

## Creating and installing your own packages

See also:

* [comprehensive howto on chocolatey.org](https://chocolatey.org/docs/create-packages)
* [example package for simple .exe setup](https://chocolatey.org/blog/create-chocolatey-packages)

```PowerShell
choco new <packageName>
```

### Exercise 4: Create a package the old fashioned way

This is meant to be an exploratory exercise and intentionally doesn't provide much direction.

1. Download Google Chrome from https://dl.google.com/tag/s/dl/chrome/install/googlechromestandaloneenterprise64.msi and https://dl.google.com/tag/s/dl/chrome/install/googlechromestandaloneenterprise.msi.
2. From a command line, call `choco new googlechrome`
3. Go into the googlechrome folder and read the readme, look through the files that were created and try to create a package just using the 64 bit Google Chrome MSI.
4. Run `choco pack`
5. Install the package using Chocolatey - `choco install googlechrome -y -s .`

### Exercise 5: Manually repackage something from the Community Repository

1. Go to https://chocolatey.org/packages/GoogleChrome
2. Find the "Download" button on the left hand side and download the nupkg
3. Extract the nupkg (install `7zip` via Chocolatey if not already present)
4. Take a look at the nuspec and see if there are any dependencies
5. Inspect all .ps1 files and download contained URLs
6. Replace external URLs with the corresponding internal/offline file
7. Run `choco pack`
8. Install the package using Chocolatey - `choco install googlechrome -y -s .`

### Best Practice vs. Good Enough

DO NOT:

* duplicate packages that already exist (check the community repo!)
* create packages fore more than one piece of software (one package - one responsibility!)

DO:

* keep packages as simple as possible
* use dependencies to break-up larger packages
* use meta-packages where applicable
* fail package if installation is not successful
* use existing extension packages where possible

## Publishing Packages

TODO: a quick note on distribution rights

### Self-Contained vs. Included Installers vs. Just In Time Downloads

When creating packages, try to prefer self-contained over included installers over jit-downloads.

1. "self-contained": ditch all automation scripts, just include binaries/executable, Chocolatey & shimgen will take care of the rest.

2. "include installers": distribute application installers or zips with the nupkg, use automation scripts to execute/unpack them.

3. "jit downloads": use automation scripts to download installation files from remote locations and execute/unpack them.

## Setting up an internal Chocolatey repository

### UNC-Share-Repository

The structure should just be a flat folder or share (no subfolders) with nupkgs inside that folder. You get that when you choco push to that location.

```PowerShell
New-Item -Type Directory C:/mychoco -Force
choco source add -n localfeed -s C:/mychoco
```

check the repository is set up:

```PowerShell
choco source list
```

now we can disable the community repository:

```PowerShell
choco source disable -n chocolatey
```

### Pushing Packages to UNC shares

#### Advantages

* Speed of setup (can be setup almost immediately).
* Package store is a simple file system.
* Can be easily upgrade to Simple Server.
* Permissions are based on file system/share permissions.
* Can manage PowerShell gallery type packages.
* There is no limitation on package sizes (or rather, it can likely handle 100MB+ file sizes, maybe even GB sized packages). Don't create multiple GB sized packages, what is wrong with you?! 😉

#### Disadvantages

* Anyone with permission can push and overwrite packages.
* No HTTP/HTTPS pushing. Must be able to access the folder/share to push to it.
* Starts to affect choco performance once the source has over 500 packages (maybe?).
* No tracking on number of downloads / no package statistics
* Big disadvantage: For a file share there is not a guarantee of package version immutability. Does not do anything to keep from package versions being overwritten. This provides no immutability of a package version and no guarantee that a version of a package installed is the same as the version in the source.

### Exercise 6: Setup ProGet as Chocolatey Repository

```PowerShell
choco install ProGet -y --package-parameters="/LicenseKey:XXX"
```

**Note:** ProGet is a commercial product which also offers a free edition.

## Packaging -> Test -> Push Pipeline

TODO: why?

### Exercise 7: Automatically Test Chocolatey Packages

1. install `psake`

```PowerShell
choco install psake -y
refreshenv
```

2. create a `psakefile`

```PowerShell
code psakefile.ps1
```

```PowerShell

task Init {
  $script:PackageFile = Get-Item "*.nuspec" | Select-Object -First 1
  $script:PackageNuspec = [xml](Get-Content $PackageFile)
  $script:PackageId = $script:PackageNuspec.package.metadata.id
}

task Pack -Depends Init {
  Exec {
    choco pack $PackageFile
  }
}

task Test -Depends Pack {
  Exec {
    choco install -noop $PackageId -source . -y
  }
}

task Publish -Depends Test {
  Exec {
    Get-Item "*.nupkg" | ForEach-Object {
      choco push $_ --source myfeed
    }
  }
}
```

3. test the psake script

```PowerShell
psake Test
```

## Updates, Reports & Monitoring

Blog Post: [getting started with Chocolatey & Jenkins](https://blog.pauby.com/post/getting-started-with-chocolatey-and-jenkins/)

### Chocolatey OSS

There's no "automatic" reporting in the OSS version of Chocolatey, you can however create your own monitoring by watching these directories:

* `$env:ChocolateyInstall\lib\` includes every installed package
* `$env:ChocolateyInstall\lib-bad\` contains failed installations

This PowerShell sample script could be run on user login to push "installed packages" and "failed packages" to a "log-server":

```PowerShell
param(
  $UNCPath = "\\rogue-one\hostreports\"
)
$ErrorActionPreference = "Stop"

$hostInfo = @{
  HOST        = $env:COMPUTERNAME
  USER        = $env:USERNAME
  DOMAIN      = $env:USERDOMAIN
  packages    = @()
  bad_packges = @()
  has_choco   = $false
}

if (Test-Path $env:ChocolateyInstall) {
  $hostInfo.has_choco = $true
  $exPkgs = {
    param($base)
    gci $base | % {
      $spec = ([xml](gc (gci $_.FullName "*.nuspec").FullName)).package.metadata
    }
  }
  $hostInfo.packages = & $exPkgs $env:ChocolateyInstall/lib
  $hostInfo.bad_packges = & $exPkgs $env:ChocolateyInstall/lib-bad
}

$targetFile = Join-Path $UNCPath "$($hostInfo.HOST)_stats.json"
$hostInfo | ConvertTo-Json -Depth 3 | Out-File -FilePath $targetFile -Encoding utf8
```

### Chocolatey Management UI (C4B)

TODO: screenshot, feature overview

## 1-Click Deployments with Boxstarter

[Boxstarter](https://boxstarter.org/) - [view it on GitHub](https://github.com/chocolatey/boxstarter)

```PowerShell
choco install Boxstarter -y
```

### Portable Boxstarter Package

```PowerShell
BoxstarterShell
Copy-Item $Boxstarter.BaseDir C:\BoxstarterPortable -Recurse
C:\BoxstarterPortable\Boxstarter example
```

Create a PowerShell script that looks like this (make sure that the `cinst powershell` is at the very top. Replace the following lines with whatever fits your scenario)

```PowerShell
cinst powershell
cinst my-package
cinst another-package
```

### Exercise 8: Use Boxstarter to deploy Packages requiring a reboot

TODO

## Extended application possibilities

### [Chocolatey Automatic Package Updater Module](https://github.com/majkinetor/au)

> Automatic packaging is a process that package maintainers can run on their own to keep the packages they maintain up to date. It is not a required step for maintaining packages on the community feed (https://chocolatey.org/packages), but it is recommended you find a way to automate the delivery of packages to the community feed when there are updates if you are going to maintain more than 5 packages and you are not the software vendor for the packages you maintain.

### Using Chocolatey as Transport & Execution Framework for arbitrary PowerShell scripts

TODO: examples

* configuration packages
* use Chocolatey to package & run unit tests
