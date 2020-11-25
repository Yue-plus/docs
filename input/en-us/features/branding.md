---
Order: 150
xref: branding
Title: Branding Chocolatey Applications (C4B)
Description: Brand Chocolatey applications with your own organisational icons
RedirectFrom: docs/features-branding
---

We are aware that some of customers want to be able to brand Chocolatey GUI so
that is "looks" like their own internal applications.  Out of the box, Chocolatey
GUI's main screen looks like this:

![Chocolatey GUI Main Screen](/assets/images/gui/main-screen.png)

However, with branding applied, Chocolatey GUI's main screen can instead look like
this:

![Chocolatey GUI Main Screen with Branding](/assets/images/gui/main-screen-with-branding.png)

Here, the logo of a company called A Squared Software Ltd is being used, instead
of the main Chocolatey logo.  In addition, a _Powered By Chocolatey_ logo is added
to the bottom left corner of the application.

## Requirements for branding

> :memo: **NOTE:** Branding of Chocolatey GUI is only available to our Business
License customers, and requires the **Chocolatey GUI licensed extension**
(chocolateygui.extension) to be installed, alongside Chocolatey GUI.  Currently,
the chocolateygui.extension package is only available in a beta version, please
reach out to support via the normal channels to get access to the package.

In order for branding to work, there are a number of image files that are required.
These have to be named exactly as the following:

* icon_256x256.ico
* logo_150x250.png
* splash_700x302.png
* splash_975x421.png
* splash_1250x540.png


> :memo: **NOTE:** The reason that there are multiple splash screen images is because
Chocolatey GUI makes a decision, based on the resolution of the screen, which
splash screen image to display to the user.

The numbers in the file names are a suggestion as to the width and height of each
image.  While the images don't have to _exactly_ match these dimensions, it is
recommended that they are as close as possible to these dimensions.

## Location of branding files

In order for your custom branding files to be applied, you must place the above
files into one of two specific places.

### Default Location

By default, Chocolatey GUI will look for custom branding files in the Chocolatey
installation directory (normally `c:/programdata/chocolatey`, and then in a folder
called `branding/gui`.  i.e. it will look in the following folder:

`c:/programdata/chocolatey/branding/gui`

### Custom Location

It is possible to use a custom location by first settings an environment variable
called `ChocolateyBrandingLocation` to a new location.  For example, if you created
this environment variable with a value of `c:/temp/branding`, then Chocolatey GUI
would then expect to find the above asset files in this location:

`c:/temp/branding/gui`

## ChocolateyGuiBranding.dll

The first time Chocolatey GUI, with the Chocolatey GUI licensed extension installed,
is executed, and the above asset files are in one of the defined locations, a new
file will be generated in the same location called `ChocolateyGuiBranding.dll`.
The new file actually contains all the image files that were created, as they have
been embedded as resources within this assembly file.  This approach is used in
order to optimize the loading of the assets.  Once this ChocolateyGuiBranding.dll
has been created, Chocolatey GUI will use it each time the application runs.  The
original image asset files are actually no longer required, and can be removed.
If at any point you need to re-generate the branding that is being used, simply
delete the following two files:

* ChocolateyGuiBranding.dll
* ChocolateyGuiBranding.resources

Then update, your image asset files, and re-run Chocolatey GUI and the branding
assembly will be re-generated.

Using a single ChocolateyGuiBranding.dll as the source of branding makes it very
simple to generate and distribute this assembly to apply branding across your
entire organisation.

## Branding in action

The below GIF shows the default opening of the Chocolatey GUI application when
there is no branding applied.

![Chocolatey GUI in action](/assets/images/gui/in-action.gif)

In this GIF, we see branding being applied to the Chocolatey GUI application.

![Chocolatey GUI in action with branding](/assets/images/gui/in-action-with-branding.gif)

Notice that the splash screen image has been replaced, as well as the logo at the
top left of the application, and the icon in the taskbar.

> :memo: **NOTE:** There is an open issue regarding the icon in the taskbar not
being correctly replaced, visit https://github.com/chocolatey/chocolatey-licensed-issues/issues/157
for more information.

> :memo: **NOTE**: To see all feature videos for Chocolatey for Business, please visit https://chocolatey.org/resources/features#c4b.

## Deploying Branding

What follows is a suggestion on how a branded version of Chocolatey GUI can be
deployed out to your environment.

> :memo: **NOTE:** In order for the below to work, you must have the Chocolatey GUI licensed
extension (chocolateygui.extension) installed.

1. Follow the steps above to place the branding image assets into the correct location.
1. Run the Chocolatey GUI application to generate the ChocolateyGuiBranding.dll
1. Create a new folder on your file system to house the branding Chocolatey Package
1. In this folder, create a `tools` folder
1. Copy the generated `ChocolateyGuiBranding.dll` into this folder
1. Copy the following xml into a file called `chocolateygui-branding.nuspec`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
  <metadata>
    <id>chocolateygui-branding</id>
    <version>0.1.0</version>
    <title>chocolateygui-branding (Install)</title>
    <authors>__REPLACE_AUTHORS_OF_SOFTWARE_COMMA_SEPARATED__</authors>
    <projectUrl>https://_Software_Location_REMOVE_OR_FILL_OUT_</projectUrl>
    <tags>chocolateygui-branding SPACE_SEPARATED</tags>
    <summary>__REPLACE__</summary>
    <description>__REPLACE__MarkDown_Okay </description>
    <dependencies>
      <dependency id="chocolateygui" />
    </dependencies>
  </metadata>
  <files>
    <file src="tools\**" target="tools" />
  </files>
</package>
```

`7.` Copy the following PowerShell into a file called `tools\chocolateyInstall.ps1`

```powershell
  $toolsDir                      = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)"
  $helpersFile                   = Join-Path -Path $toolsDir -ChildPath 'helpers.ps1'

  . $helpersFile

  $chocolateyGuiBrandingAssembly = Join-Path -Path $toolsDir -ChildPath 'ChocolateyGuiBranding.dll'
  $chocolateyGuiBrandingLocation = Join-Path -Path (Get-BrandingLocation) -ChildPath 'gui'

  if(!(Test-Path -Path $chocolateyGuiBrandingLocation)){
    New-Item $chocolateyGuiBrandingLocation -ItemType Directory -Force | Out-Null
  }

  Copy-Item -Path $chocolateyGuiBrandingAssembly -Destination $chocolateyGuiBrandingLocation
```

`8.` Copy the following PowerShell into a file called `tools\chocolateyuninstall.ps1`

```powershell
  $toolsDir                      = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)"
  $helpersFile                   = Join-Path -Path $toolsDir -ChildPath 'helpers.ps1'

  . $helpersFile

  $chocolateyGuiBrandingLocation = Join-Path -Path (Get-BrandingLocation) -ChildPath 'gui'
  $chocolateyGuiBrandingAssembly = Join-Path -Path $chocolateyGuiBrandingLocation -ChildPath 'ChocolateyGuiBranding.dll'

  Remove-Item -Path $chocolateyGuiBrandingAssembly -Force -ErrorAction Continue | Out-Null
```

`9.` Copy the following PowerShell into a file called `tools\helpers.ps1`

```powershell
  function Get-BrandingLocation {
    <#
    .SYNOPSIS
    Gets the top level location for branding files used by Chocolatey
    applications
    .DESCRIPTION
    Creates or uses an environment variable that a user can control to
    communicate with packages about where they would like branding files to
    be located.
    .NOTES
    Sets an environment variable called `ChocolateyBrandingLocation`.
    .INPUTS
    None
    .OUTPUTS
    None
    #>

      $brandingLocation = $env:ChocolateyBrandingLocation

      if ($null -eq $brandingLocation) {
        $chocoInstallLocation = $env:ChocolateyInstall
        $brandingLocation = Join-Path -Path $chocoInstallLocation -ChildPath 'branding'
      }

      # Add a drive letter if one doesn't exist
      if (-not($brandingLocation -imatch "^\w:")) {
        $brandingLocation = Join-Path $env:systemdrive $brandingLocation
      }

      if (-not($env:ChocolateyBrandingLocation -eq $brandingLocation)) {
        try {
          Set-EnvironmentVariable -Name "ChocolateyBrandingLocation" -Value $brandingLocation -Scope User
        } catch {
          if (Test-ProcessAdminRights) {
            # sometimes User scope may not exist (such as with core)
            Set-EnvironmentVariable -Name "ChocolateyBrandingLocation" -Value $brandingLocation -Scope Machine
          } else {
            throw $_.Exception
          }
        }
      }

      return $brandingLocation
      }
```

`10.` Run the command `choco pack`

`11.` Deploy the generated `chocolateygui-branding.nupkg` to the repository that you are using

`12.` Install the Chocolatey GUI Branding package