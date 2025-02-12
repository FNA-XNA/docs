# 1: Setting Up FNA

This page has two parts: The first sets up the FNA team's recommended development environment, and the second prepares FNA itself. If you already have a development environment you like, you can skip to the [Download FNA section](#chapter-3-download-and-update-fna) if you want, but note that our environment prepares you for [remote debugging on Steam Deck](3:-Distributing-FNA-Games.md#steam-deck-remote-debugging).

## Chapter 1a: Linux Setup

The Linux development environment for FNA is supported on all distributions with Flatpak support, including SteamOS! (But, if you absolutely _must_ know, the FNA team uses [Fedora Workstation](https://fedoraproject.org/workstation/).)

You may be able to find VSCode and the .NET SDKs via apps like KDE Discover, but it's easier to get everything at once with a single portable terminal command:

```
flatpak install com.visualstudio.code org.freedesktop.Sdk.Extension.mono6 org.freedesktop.Sdk.Extension.dotnet8
```

This installs VSCode, Mono, and .NET 8 all at once! If it asks which version of the SDKs to install, select 24.08.

All that's left is to expose the .NET and Mono SDKs to VSCode's sandbox:

```
flatpak --user override --env=FLATPAK_ENABLE_SDK_EXT=mono6,dotnet8 com.visualstudio.code
```

## Chapter 1b: Windows Setup

At minimum you will need to install the following software:

- [Visual Studio Code](https://code.visualstudio.com/download)
- [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet)
- [.NET 4.6.2 Developer Pack](https://dotnet.microsoft.com/en-us/download/dotnet-framework/thank-you/net462-developer-pack-offline-installer)
- [Mono for 64-bit Windows](https://www.mono-project.com/download/stable/#download-win)

When setting up projects, Windows has an additional setup step. An important part of multiplatform development is filesystem case sensitivity - for example, on Linux and console platforms, "filename" is NOT the same as "FileName"! To ensure that developers follow this rule, it is recommended to make your project folders case sensitive on Windows.

To do this, open a Command Prompt as Administrator, then (carefully!) enter the following command:

```
fsutil.exe file SetCaseSensitiveInfo "C:\path\to\your\project" enable
```

(While we're on the subject, remember [not to use `\\` for file paths](4:-FNA-and-Windows-API.md#filesystem-portability)!)

## Chapter 2: Visual Studio Code Extensions

After starting VSCode, hit Ctrl+Shift+X to go to the extensions view, then search for "C# Dev Kit". Install the kit from the marketplace and let it download all of the components. You should be able to clear the search results and see the various new installed extensions!

Lastly, search for and install the "Mono Debug" extension.

## Chapter 3: Download and Update FNA

We strongly recommend using Git to download and update FNA. This tutorial will guide you through this process.

If you are using an official zipped release of FNA, you only need to worry about step 2.

### Step 1: Clone FNA
FNA uses several Git submodules to access the source to additional libraries, such as SDL3# and FAudio. To fully download FNA, add the `--recursive` parameter to your `git clone` command:

```
git clone --recursive https://github.com/FNA-XNA/FNA
```

This will clone FNA, then clone all of the submodules into the appropriate locations.

### Step 2: Download Native Libraries
FNA uses several native libraries for various pieces of functionality, such as window management, input, and audio output.

Here's what we use and why:

* SDL: Used for window management, input, image I/O, etc.
* FNA3D: Only required if you use the Graphics namespace.
* FAudio: Only required if you use the Audio or Media namespaces.
* Theorafile: Only required if you use VideoPlayer.

You can find the libraries precompiled at our [fnalibs-dailies](https://github.com/FNA-XNA/fnalibs-dailies/actions) repository. The "fnalibs" archive contains all of the native libraries for Windows and Linux.

### Step 3: Update FNA
It is _strongly_ recommended that you update at least once a month. FNA releases are always on the first of every month, so you may simply want to make a calendar reminder for yourself to redownload FNA and fnalibs.zip at the beginning of each month.

To update FNA, simply enter the FNA directory and run `git pull`. This will update to the latest FNA version, assuming you have not made local changes that conflict with the upstream changes. If you do have local changes, store them elsewhere and update, or revert your changes. (By the way, if you really do have local changes, please let us know! We want working code in upstream, and it will make your life easier, we promise!)

Sometimes, FNA will update one of its submodules. When this occurs, run `git submodule update --init --recursive` and the submodules will fully update. Again, this assumes that you have not made local changes to the submodules.

### Step 4: Join the FNA Discord
Aside from the commit log, a good place to keep an eye on major FNA changes is to join the [Discord server](https://discord.gg/2Gg8zju). This is where announcements and development discussion occur; in particular, there are channels allocated for general development, XNA preservation research, private console development, and experimental platform development.

## Chapter 4: Building Old Visual Studio Projects

For those using old pre-.NET Core solutions, you will want to make these changes to allow building your solution:

1. Right click the C# Dev Kit extension and disable it, leaving all other extensions alone
2. Right click the C# extension and select Extension Settings
3. Search for useModernDotNet, uncheck Use Modern .NET
4. Search for useOmnisharp, check Use Omnisharp
5. Create a `.vscode` folder next to your solution, then add a `tasks.json` file containing something like this:

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "msbuild",
            "args": [
                // Ask msbuild to generate full paths for file names.
                "/property:GenerateFullPaths=true",
                "/t:build",
                // Do not generate summary otherwise it leads to duplicate errors in Problems panel
                "/consoleloggerparameters:NoSummary",
                "THIS_IS_YOUR_GAME.sln",
                "/p:Configuration=Debug"
            ],
            "group": "build",
            "presentation": {
                // Reveal the output only if unrecognized errors occur.
                "reveal": "silent"
            },
            // Use the standard MS compiler pattern to detect errors, warnings and infos
            "problemMatcher": "$msCompile"
        }
    ]
}
```

With this in place, you should now be able to build (*Terminal -> Run Build Task*)!

## Chapter 5: Creating New Projects

Making an FNA project is relatively simple for basically every C# IDE, though the process has changed in recent years:

For Visual Studio Code, you can use the built-in terminal to navigate to your project folder, then run `dotnet new sln --name YourProjectName` to create a new solution, `dotnet new console --name YourProjectName` to create an empty project, and `dotnet sln add YourProjectName/YourProjectName.csproj` to add it to the solution. Visual Studio and MonoDevelop have New Solution wizards for creating empty C# projects.

Once the empty project is created, you can go to the Solution Explorer and right click the solution, click "Add Existing Project", then add FNA (FNA.NetFramework.csproj for .NET Framework and Mono, FNA.Core.csproj for .NET 8, or FNA.csproj for old Visual Studio projects). You can then right click the empty project and add FNA as a Project Reference.

For existing XNA projects, we recommend targeting .NET Framework and Mono for better compatibility. For new projects, we recommend .NET 8.

### .NET Framework Fixes

When targeting .NET Framework via Visual Studio Code, be sure to open `YourProjectName/YourProjectName.csproj` and change the target framework to `net4.0`! 

You'll also find that running/debugging takes a few more steps - we're lobbying to streamline this process, but if you want to get debugging ASAP, you can open the folder where your .sln file is, make a `.vscode/` folder, then rip off this `launch.json` file and place it in the `.vscode/` folder:

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "mono",
            "request": "launch",
            "program": "${workspaceRoot}/path/to/your/bin/Debug/net4.0/Game.exe",
            "cwd": "${workspaceRoot}",
            "env": {
                "LD_LIBRARY_PATH": "${workspaceRoot}/path/to/your/fnalibs/lib64/"
            }
        },
        {
            "name": "Attach",
            "type": "mono",
            "request": "attach",
            "address": "localhost",
            "port": 55555
        }
    ]
}
```

With this in place, you should now be able to launch the program after it's built (*Run -> Start Debugging*, or F5!)!

### .NET Core Fixes

DllMap is a critical component of .NET portability that maps native library names to appropriate equivalents on multiple platforms. For example, while Windows might look for `fmod_studio.dll`, Linux will instead look for `libfmodstudio.so.XX`, which is nontrivial for the runtime to figure out on its own. By adding a config file like [this one](https://github.com/FNA-XNA/FNA/blob/master/app.config), we're able to automatically map DLL names for all platforms without resorting to per-platform builds with customized DLL names.

Shamefully, this is currently absent from .NET Core. We will continue to lobby for adding this feature back to modern .NET, but for now we have developed a workaround called [FNADllMap](https://github.com/FNA-XNA/FNA/blob/master/src/Utilities/FNADllMap.cs). We strongly encourage everyone to copy this file directly and add it to every single project, particularly those that use DllImport, so that all managed EXE/DLL files have a rough equivalent of real DllMap support.

### Visual Studio AnyCPU Fix

For Visual Studio 2019 users, follow these additional steps to allow VS to build your project properly for 64-bit:

1. In the Visual Studio toolbar, click on the Solution Platforms dropdown menu (where it says 'Any CPU'), and click on 'Configuration Manager...'
2. In the Configuration Manager window that appears, change the 'Active solution platform' to x64. Notice that the referenced FNA project changes to x64, but your project remains 'Any CPU'
3. Address this discrepancy by clicking on the platform dropdown for your project and clicking New
4. In the New Project Platform dialog that appears, create a new platform using x64 from the dropdown. Copying settings from Any CPU is fine, and make sure the 'Create new solution platforms' is unchecked

You can now build your project for x64!
