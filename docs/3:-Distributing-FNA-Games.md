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

macOS:
OS: 10.9 Mavericks and newer
Graphics (Minimum): OpenGL 3.0+ support (2.1 with ARB extensions acceptable)
Graphics (Recommended): Metal support

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

### macOS
Download the latest version of MonoKickstart:

https://github.com/flibitijibibo/MonoKickstart

You only need the latest revision; it is actually _not_ recommended to download using Git, as the repository is mostly binary blobs, so the download time will be much longer.

In the precompiled/ folder you will notice the following:

* `kick.bin.osx`, `kick.bin.x86_64`, `monoconfig`, `monomachineconfig`
	* For macOS, you care about all of these except `kick.bin.x86_64`
* Lots and lots of DLL files.
	* If you don't know which ones you need, just use them all.

What follows is really convoluted and annoying, because that's the Apple Way:

You'll start by making a series of seemingly-arbitrary folders that will look something like this:

```
flibitGame.app/
	Contents/
		MacOS/
		Resources/
```

Next, you will put `kick.bin.osx` into the `MacOS/` folder and rename it to the name of your main EXE. For example, for `flibitGame.exe` you will name it `flibitGame`, no extension. Next to that you will put the `osx/` folder (NOT ITS CONTENTS) from the [fnalibs.tar.bz2 package](1:-Setting-Up-FNA.md#step-2-download-native-libraries). The `vulkan/` folder from fnalibs.tar.bz2 will go in the `Resources/` folder. Any other native libraries you have will also go in the `osx/` folder (for example, if you're using Steamworks.NET, you would put libsteam_api.dylib in the `osx/` folder). Lastly, if you're shipping on Steam, you will put your `steam_appid.txt` file in `Resources/`.

So now your bundle should look like this:

```
flibitGame.app/
	Contents/
		MacOS/
			flibitGame
			osx/
		Resources/
			vulkan/
			steam_appid.txt
```

Now, onto the `Resources/` folder. You will put `monoconfig`, `monomachineconfig`, and the DLL files into this folder.

Those DLL files, config files, and `kick.bin.osx` are actually a highly compacted Mono runtime that will be executing the C# assemblies, just as .NET would on Windows. The upside is, there are no system dependencies - the whole runtime is in this one folder, and all the native dependencies are in the lib folder. Convenient!

However, note that not every single DLL in the C# standard library exists in this folder. Libs like System.Web.Services are not provided by default to save disk space, but if you need these you can just grab these from any Mono runtime and we'll recognize it. These libs are typically found in the lib/mono/4.x/ folder (the precompiled folder uses 4.5).

Finally, you will put _your whole game_ into the `Resources/` folder. After that, the bundle should look like this:

```
flibitGame.app/
	Contents/
		MacOS/
			flibitGame
			osx/
			steam_appid.txt
		Resources/
			vulkan/
			monoconfig
			monomachineconfig
			mscorlib.dll, System.dll, blah blah
			flibitGame.exe
			Content/
			etc...
```

At this point, we're now ready for the Mac-specific data. No, seriously, we haven't even gotten to that part yet.

First, you will need to put a `.icns` file in the `Resources/` folder. An icns file can be generated with any image using this website:

https://cloudconvert.com/png-to-icns

It is _strongly_ recommended that you use an image that is at least 512x512 in size. For certification, Apple actually requires a 4096x4096 image for your icon!

The very last file that will be made before your bundle is done is an `Info.plist` file, which will go in the `Contents/` folder. Here's an example Info.plist file:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleDevelopmentRegion</key>
	<string>en</string>
	<key>CFBundleExecutable</key>
	<string>EscapeGoat2</string>
	<key>CFBundleIconFile</key>
	<string>EscapeGoat2</string>
	<key>CFBundleIdentifier</key>
	<string>com.magicaltimebean.Bastille2</string>
	<key>CFBundleInfoDictionaryVersion</key>
	<string>6.0</string>
	<key>CFBundleName</key>
	<string>Escape Goat 2</string>
	<key>CFBundlePackageType</key>
	<string>APPL</string>
	<key>CFBundleShortVersionString</key>
	<string>1.0</string>
	<key>CFBundleSignature</key>
	<string>GOAT</string>
	<key>CFBundleVersion</key>
	<string>1</string>
	<key>LSApplicationCategoryType</key>
	<string>public.app-category.games</string>
	<key>LSMinimumSystemVersion</key>
	<string>10.9</string>
	<key>NSHumanReadableCopyright</key>
	<string>Copyright © 2014 MagicalTimeBean. All rights reserved.</string>
	<key>NSPrincipalClass</key>
	<string>NSApplication</string>
	<key>NSHighResolutionCapable</key>
	<string>True</string>
</dict>
</plist>
```

With that, the final look of the bundle:

```
flibitGame.app/
	Contents/
		Info.plist
		MacOS/
			flibitGame
			osx/
		Resources/
			vulkan/
			steam_appid.txt
			monoconfig
			monomachineconfig
			mscorlib.dll, System.dll, blah blah
			flibitGame.exe
			flibitGame.icns
			Content/
			etc...
```

Once you've compiled all of this together, you should have a working app bundle! FINALLY! Place your app bundle and any other items you want to include with your game into a folder, then you are ready to upload via [SteamPipe](https://partner.steamgames.com/doc/sdk/uploading), [butler](https://itch.io/docs/butler/), or the [GOG Galaxy builder](https://docs.gog.com/bc-build-game/).

One extra note: If for some reason you want to codesign your app (**this is optional**), you will want to have this in an `entitlements.plist` file when signing:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>com.apple.security.app-sandbox</key>
	<true/>
	<key>com.apple.security.automation.apple-events</key>
	<true/>
	<key>com.apple.security.cs.allow-dyld-environment-variables</key>
	<true/>
	<key>com.apple.security.cs.allow-jit</key>
	<true/>
	<key>com.apple.security.cs.allow-unsigned-executable-memory</key>
	<true/>
	<key>com.apple.security.cs.disable-executable-page-protection</key>
	<true/>
	<key>com.apple.security.cs.disable-library-validation</key>
	<true/>
	<key>com.apple.security.device.usb</key>
	<true/>
</dict>
</plist>
```

When signing for Steam, the first key should be `false`, otherwise it won't be able to detect when Steam is running.

### .NET Core

The above guide works for .NET Framework and Mono applications, but does not work with .NET 8. The publishing system for modern .NET has completely changed and is described below.

`dotnet publish -r <win-x64/linux-x64/osx-x64> -c Release --self-contained` will produce the executable package, but each platform has different requirements for where the fnalibs must be placed.

* **Windows:** Place the x64 fnalibs in the `publish` directory alongside your executable.
* **MacOS:** Place the osx fnalibs in the `publish` directory alongside your executable. Then use `install_name_tool -add_rpath @executable_path <your_app_executable_name>` to force the application to first look in the executable directory for the fnalibs, instead of `/usr/local/lib`.
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
