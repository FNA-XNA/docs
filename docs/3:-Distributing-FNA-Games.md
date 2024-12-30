# 3: Distributing FNA Games

At some point you will likely want to _ship_ your FNA game to customers, so here's a guide to shipping your game in a way that is hassle-free for players.

Note that this guide is for .NET Framework and Mono applications - for those using .NET 8, refer to the .NET Core section of this page.

***

### Overall System Requirements
While we can't automatically determine CPU/RAM/Storage requirements (that's up to you!), we can provide a reasonably accurate requirement list for the following specs:

```
Windows:
OS: Windows 7, fully updated
Graphics (Minimum): Direct3D 11 support (feature level 10_0)
Graphics (Recommended): Vulkan support

Linux:
OS: glibc 2.28+, 64-bit only
Graphics (Minimum): OpenGL 3.0+ support (2.1 with ARB extensions acceptable)
Graphics (Recommended): Vulkan support

Other: SDL_GameController devices fully supported
```

***

### Windows
Because you're no longer shipping with XNA, you no longer need to provide the XNA4 redist package. Additionally, as of the latest SDL revision, we no longer require the DirectX redist to run correctly.

However, as you might expect, you still need to include the appropriate .NET Framework installer. Match the version that you're targeting in VS and you're good to go.

The native libraries needed by FNA are in the [fnalibs.tar.bz2 package](1:-Setting-Up-FNA.md#step-2-download-native-libraries) under the `x86/` folder. The `x64/` folder will only apply if you add [64-bit support](4:-FNA-and-Windows-API.md#64-bit-support) to your Windows version.

***

### GNU/Linux

Download the latest version of MonoKickstart:

https://github.com/flibitijibibo/MonoKickstart

You only need the latest revision; it is actually _not_ recommended to download using Git, as the repository is mostly binary blobs, so the download time will be much longer.

In the precompiled/ folder you will notice the following:

* `kick.bin.x86_64`, `kick.bin.osx`, `monoconfig`, `monomachineconfig`
	* For Linux, you care about all of these except `kick.bin.osx`
* Lots and lots of DLL files.
	* If you don't know which ones you need, just use them all.

What you're going to do is place the game itself into the same folder as these Kick/DLL files, and you're also going to put the `lib64/` folder (NOT ITS CONTENTS) from the [fnalibs.tar.bz2 package](1:-Setting-Up-FNA.md#step-2-download-native-libraries) next to your game files. Any other native libaries you have will also go into the `lib64/` folder (for example, if you're using Steamworks.NET, you would put libsteam_api.so in that folder).

These files you're looking at are a highly compacted Mono runtime that will be executing the C# assemblies, just as .NET would on Windows. The upside is, there are no system dependencies - the whole runtime is in this one folder, and all the native dependencies are in the lib folder. Convenient!

However, note that not every single DLL in the C# standard library exists in this folder. Libs like System.Web.Services are not provided by default to save disk space, but if you need these you can just grab these from any Mono runtime and we'll recognize it. These libs are typically found in the lib/mono/4.x/ folder (the precompiled folder uses 4.5).

`kick.bin.x86_64` is going to be renamed to the name of your main EXE file. For example:

```
flibitGame.exe
flibitGame.bin.x86_64
```

You can optionally name it just `flibitGame` with no extension, if you prefer that for whatever reason.

Once this is finished, you are ready to upload via [SteamPipe](https://partner.steamgames.com/doc/sdk/uploading), [butler](https://itch.io/docs/butler/), or the [GOG Galaxy builder](https://docs.gog.com/bc-build-game/).

#### Steam Deck Remote Debugging

Using Visual Studio Code and the above packaging process, it's actually possible to attach to a Steam Deck for remote C# debugging! The process is as follows:

1. Set up the Steam Deck devkit and upload your game, complete with debug symbols (see [Valve's documentation](https://partner.steamgames.com/doc/steamdeck/loadgames), you can ignore anything involving Proton as it's not used here!)
2. On the Deck, go to your new devkit game and click *⚙️ -> Properties*
3. Under *Shortcut -> Launch Options*, enter the following excruciatingly long text:

```
MONO_BUNDLED_OPTIONS='--debugger-agent=address=0.0.0.0:55555,transport=dt_socket,server=y --debug=mdb-optimizations' %command%
```

We're hoping to streamline this step and have [sent a patch to Valve for review](https://gitlab.steamos.cloud/devkit/steamos-devkit/-/issues/10).

4. Add a task in launch.json to connect to the Deck debug server:

```json

        {
            "name": "Attach to Deck",
            "type": "mono",
            "request": "attach",
            "address": "192.168.1.yoursteamdeckIP",
            "port": 55555
        }
```

5. Launch the game and start the debugger, enjoy!

#### Single-Assembly Portability and Steam

When leveraging FNA's single-assembly portability, you can run a single binary on both Windows and Linux. For distribution, you still have to make two separate packages of roughly the same game.

With Steam, however, there is a way to optimize this. If you architect your depots in the following manner...

```
Depot 4206901 - Shared Content, Windows + SteamOS + Linux
Depot 4206902 - Windows Depot, Windows
Depot 4206903 - Linux Depot, SteamOS + Linux
```

The idea is that you upload the Windows fnalibs to the second depot, the Linux fnalibs and MonoKickstart environment to the third depot, then upload _the entire rest of the game_ to the first depot. You can also put OS-specific binaries in their appropriate folders, if applicable (C# Steamworks wrappers like Steamworks.NET and Facepunch.Steamworks are the only example these days, but who knows).

Because the folder layout is identical between the two, this means you can limit your upload process to a single depot, only updating the other two depots when updating FNA specifically. This dramatically reduces the workload and also reduces the chance for version sync issues! Sadly this only applies to Steam; itch and GOG do not have this feature.

A good publicly-available example of this layout is the PC version of [I MAED A GAM3 W1TH Z0MB1ES 1NIT!!!1](https://steamdb.info/app/1800730/depots/).

***

### .NET Core

The above guide works for .NET Framework and Mono applications, but does not work with .NET 8. The publishing system for modern .NET has completely changed and is described below.

`dotnet publish -r <win-x64/linux-x64> -c Release --self-contained` will produce the executable package, but each platform has different requirements for where the fnalibs must be placed.

* **Windows:** Place the x64 fnalibs in the `publish` directory alongside your executable.
* **Linux:** Place the lib64 fnalibs in the `publish` directory, in a sub-directory called `netcoredeps`.

#### Single-File Applications

The above steps for publishing will produce a `publish` directory with an absolutely enormous amount of DLLs. If you want to build a single-file executable instead, just add this to a property group in your .csproj:
```
<PublishSingleFile>true</PublishSingleFile>
```
However, if you do this, we request that you make an exception for FNA.dll so that it is not bundled into the exe like the rest of the app. This is not required, but it is beneficial for both end users and FNA developers, since it allows for dynamically swapping out the FNA.dll in the game's files (for debugging, modding, etc.).

You can prevent FNA.dll from being bundled by changing the FNA ProjectReference in your game's .csproj to the following:
```
  <ItemGroup>
    <ProjectReference Include="path/to/FNA.Core.csproj">
      <ExcludeFromSingleFile>true</ExcludeFromSingleFile>
    </ProjectReference>
  </ItemGroup>
```

On a similar note, please do not bundle the native fnalibs into the single-file executable, for the same reasons. (Don't worry, this will not happen unless you go out of your way to explicitly include them in the project.)
