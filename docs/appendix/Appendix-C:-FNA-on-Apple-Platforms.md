# Appendix C: FNA on Apple Platforms

FNA supports deploying to macOS/iOS/tvOS via the .NET SDK. FNA does not have special branches or configurations for each platform; the public master branch of FNA and the configuration you're already used to is exactly what is used to ship for these targets. The platform code is contained entirely in SDL.

## Getting Started

This is the basic guide to getting your game running on macOS/iOS/tvOS. Once these steps are followed, your game should be able to boot on real hardware.

### Prerequisites

In order to build and deploy FNA apps for iOS/tvOS, you must have the following:

1. Mac hardware (required by Apple)
2. The latest [.NET SDK](https://dotnet.microsoft.com/en-us/download/dotnet) for macOS
3. The latest version of Xcode, downloaded from the macOS App Store or from the Apple Developer site
4. The iOS/tvOS workloads for the .NET SDK. Once you have installed .NET on your Mac, run the following commands:

```zsh
sudo dotnet workload install ios
sudo dotnet workload install tvos
```

### Building fnalibs

macOS, iOS, and tvOS libraries are now built automatically and can be downloaded from [fnalibs-dailies](https://github.com/FNA-XNA/fnalibs-dailies/actions). You may need to be logged into GitHub to download these. Simulator libraries are not currently provided, so you will need to build those yourself if desired.

Note: On newer macOS versions, you may need to run `xattr -c *.dylib` or `xattr -c *.a` on these libraries before using them in your projects to avoid security errors.

### Creating/Publishing a macOS Project

A macOS project is essentially the same as any other .NET Core build; the difference is in the publishing process:

`dotnet publish -r <osx-x64> -c Release --self-contained` will produce the executable package. Place the osx version of your fnalibs in the `publish` directory alongside your executable. Then use `install_name_tool -add_rpath @executable_path <your_app_executable_name>` to force the application to first look in the executable directory for the fnalibs, instead of `/usr/local/lib`.

For NativeAOT, the directions in Appendix A largely apply here, with these added steps:

* Build SDL3 from source or install the SDL3 development package from a package manager, then use it to build the other libraries from source.
* Copy the resulting \*.dylib files from SDL3, FNA3D, FAudio, and Theorafile into `/usr/local/lib`.
* Build the application.
* Copy the contents of `fnalibs/osx` into the generated output directory.
* Finally, to ensure your application uses the correct search path for SDL3, use `install_name_tool -change /usr/local/lib/libSDL3.0.dylib @rpath/libSDL3.0.dylib <my-app-name>`.

That about covers macOS - iOS/tvOS are a whole different story:

### Creating an iOS Project

Creating a .NET iOS project is almost identical to the process of [creating a desktop .NET project](https://github.com/FNA-XNA/FNA/wiki/1%3A-Setting-Up-FNA#chapter-5-creating-new-projects).
The only difference is that instead of `dotnet new console`, you must run `dotnet new ios`.
This will generate an "iOS Application" template project we can then modify like so:

1. Delete the `SceneDelegate.cs` and `AppDelegate.cs` files
2. Update the `Info.plist` file as follows:
  - Set your application metadata (display name, bundle identifier, etc.)
  - Specify which device orientations you want to support (landscape, portrait, etc.)
  - Change the `UIRequiredDeviceCapabilities` value from `armv7` to `arm64`
  - If your game supports game controllers (MFi, Xbox, PlayStation, etc.) add the following:
```plist
   <key>GCSupportedGameControllers</key>  
   <array>
       <dict>
           <key>ProfileName</key>
           <string>ExtendedGamepad</string>
       </dict>
   </array>  
   <key>GCSupportsControllerUserInteraction</key>  
   <true/>
```
See [this Apple documentation](https://developer.apple.com/documentation/bundleresources/information_property_list/gcsupportedgamecontrollers) for more information about how to support different kinds of controllers.

3. Add the following to your game's .csproj:

```xml
<PropertyGroup>
  <MtouchNoSymbolStrip>true</MtouchNoSymbolStrip>
  <MtouchExtraArgs>--cxx --gcc_flags "-L$(MSBuildProjectDirectory) -force_load $(MSBuildProjectDirectory)/libSDL3.a -force_load $(MSBuildProjectDirectory)/libFAudio.a -force_load $(MSBuildProjectDirectory)/libFNA3D.a -force_load $(MSBuildProjectDirectory)/libtheorafile.a -force_load $(MSBuildProjectDirectory)/libmojoshader.a -framework AudioToolbox -framework CoreBluetooth -framework CoreHaptics -framework CoreMotion -framework OpenGLES -framework Metal"</MtouchExtraArgs>
  <_ExportSymbolsExplicitly>false</_ExportSymbolsExplicitly>
  <OptimizePNGs>false</OptimizePNGs>
</PropertyGroup>
```
4. Place all the native libraries in your project directory. They will be linked on build, as specified by the `MtouchExtraArgs` property you just added to the project.
5. Add a project reference to FNA.Core.csproj, as per usual. And that's it for the project setup!

### Creating a tvOS Project

Creating a .NET tvOS project is almost identical to the process of [creating a desktop .NET project](https://github.com/FNA-XNA/FNA/wiki/1%3A-Setting-Up-FNA#chapter-5-creating-new-projects).
The only difference is that instead of `dotnet new console`, you must run `dotnet new tvos`.
This will generate a "tvOS Application" template project we can then modify like so:

1. Delete the `ViewController.cs`, `ViewController.designer.cs`, and `AppDelegate.cs` files
2. Rename `Main.storyboard` to `LaunchScreen.storyboard` and delete the following from line 10 of the file:
`customClass="ViewController"`
3. Update the `Info.plist` file as follows:
  - Set your application metadata (display name, bundle identifier, etc.)
  - If your game supports game controllers (MFi, Xbox, PlayStation, etc.) add the following:
```xml
   <key>GCSupportedGameControllers</key>  
   <array>
       <dict>
           <key>ProfileName</key>
           <string>ExtendedGamepad</string>
       </dict>
   </array>  
   <key>GCSupportsControllerUserInteraction</key>  
   <true/>
```
   See [this Apple documentation](https://developer.apple.com/documentation/bundleresources/information_property_list/gcsupportedgamecontrollers) for more information about how to support different kinds of controllers.
   - Point the app to the correct storyboard file, like so:
```xml
    <!-- Replace this... -->
    <key>UIMainStoryboardFile</key>
    <string>Main</string>

    <!-- ...with this! -->
    <key>UILaunchStoryboardName</key>
    <string>LaunchScreen</string>
```

4. Add the following to your game's .csproj:

```xml
<PropertyGroup>
  <MtouchNoSymbolStrip>true</MtouchNoSymbolStrip>
  <MtouchExtraArgs>--cxx --gcc_flags "-L$(MSBuildProjectDirectory) -force_load $(MSBuildProjectDirectory)/libSDL3.a -force_load $(MSBuildProjectDirectory)/libFAudio.a -force_load $(MSBuildProjectDirectory)/libFNA3D.a -force_load $(MSBuildProjectDirectory)/libtheorafile.a -force_load $(MSBuildProjectDirectory)/libmojoshader.a $(MSBuildProjectDirectory)/libtvStubs.a -framework AudioToolbox -framework CoreBluetooth -framework CoreHaptics -framework OpenGLES -framework Metal"</MtouchExtraArgs>
  <_ExportSymbolsExplicitly>false</_ExportSymbolsExplicitly>
  <OptimizePNGs>false</OptimizePNGs>
</PropertyGroup>
```
5. Place all the native libraries in your project directory. They will be linked on build, as specified by the `MtouchExtraArgs` property you just added to the project.
6. Add a project reference to FNA.Core.csproj, as per usual. And that's it for the project setup!

### Code Changes

#### Initialization
To get your game booting on iOS/tvOS, update your `Main.cs` as follows:

```cs
	static void Main(string[] args)
#if __IOS__ || __TVOS__
	{
		// Enable high DPI "Retina" support. Trust us, you'll want this.
		SDL3.SDL.SDL_SetHint("FNA_GRAPHICS_ENABLE_HIGHDPI", "1");

		// Keep mouse and touch input separate.
		SDL3.SDL.SDL_SetHint(SDL3.SDL.SDL_HINT_MOUSE_TOUCH_EVENTS, "0");
		SDL3.SDL.SDL_SetHint(SDL3.SDL.SDL_HINT_TOUCH_MOUSE_EVENTS, "0");
		SDL3.SDL.SDL_SetHint(SDL3.SDL.SDL_HINT_PEN_TOUCH_EVENTS, "0");

		// Don't let the accelerometer take over controller slot 0.
		SDL3.SDL.SDL_SetHint(SDL3.SDL.SDL_HINT_ACCELEROMETER_AS_JOYSTICK, "0");

		realArgs = args;
		SDL3.SDL.SDL_RunApp(0, IntPtr.Zero, FakeMain, IntPtr.Zero);
	}
	
	static string[] realArgs;

	[ObjCRuntime.MonoPInvokeCallback(typeof(SDL3.SDL.SDL_main_func))]
	static int FakeMain(int argc, IntPtr argv)
	{
		RealMain(realArgs);
		return 0;
	}

	static void RealMain(string[] args)
#endif
	{
		// your stuff
	}
```

#### Background Safety

To avoid crashes when your app enters into a background state, add the following to your Game.Update method:

```cs
if (!IsActive)
{
	SuppressDraw();
}
```

#### Retina Displays

For your app to take full advantage of iPhone and iPad retina displays, you need to set your preferred backbuffer size to the full extent of the display. This should happen at the start of your game and in the `Window.OrientationChanged` event callback (if your app supports both portrait and landscape orientations).

```cs
graphics.PreferredBackBufferWidth = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Width;
graphics.PreferredBackBufferHeight = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Height;
```

On iOS, you may need to set fullscreen true if you run into issues with touches where the statusbar might normally be.

#### tvOS User Storage

Note that tvOS does NOT have local storage available. There is temporary storage but it can be wiped by the OS at any time, so standard filesystem code will NOT work past a single instance of the program (this unfortunately includes `Microsoft.Xna.Framework.Storage`). To support save data you will need to implement some form of iCloud support, but on the bright side, this means iOS/tvOS can share that data. Our recommended solution for games with a guaranteed total save data size of <1MB is to use [TinyKVS](https://github.com/TheSpydog/TinyKVS).

If you're using iCloud and receive an error about iCloudContainerEnvironment when submitting to the App Store, add this to your Entitlements.plist for that submission:
```xml
<key>com.apple.developer.icloud-container-environment</key>
<array>
  <string>Production</string>
</array>
```

### Content Changes

All your game assets (e.g. your Content/ folder) must be placed inside the Resources/ folder in your project directory. (On tvOS you will need to add this folder manually.) This will ensure they are copied into the app bundle.

If you load any resources yourself via a FileStream, the stream must have a file access parameter of `FileAccess.Read` or you'll get a System.UnauthorizedAccessException.

#### Texture Compression

iOS/tvOS do NOT support DXT compression, so you will want to build uncompressed textures at compile time to avoid our fallback decompressor, which wastes lots of time and memory. We are open to considering other texture formats (namely, those supported by [Basis](https://github.com/BinomialLLC/basis_universal)) once we have active use cases to look at.

### Running Your App

Deploying your app to a simulator or physical device can be a bit tricky, but thankfully the .NET documentation can walk you through the process. [Please see this page for instructions](https://learn.microsoft.com/en-us/dotnet/maui/ios/cli?view=net-maui-8.0), but **skip to step 5 of the introduction**, as the preceding steps are only applicable to MAUI apps.

As you might expect, you can substitute `ios` for `tvos` in the build line when running a tvOS app.

Note that to run your app in the iOS/tvOS simulators on an Intel-based Mac, you must include x86_64 builds of your native libraries. For the fnalibs, these are automatically generated by the build scripts when run with "ios-sim" or "tvos-sim".

## Debugging

Debugging an FNA game running on a physical device is not yet supported due to bugs with Visual Studio Code's MAUI extension.

## Profiling

See [this wiki page](https://github.com/xamarin/xamarin-macios/wiki/Profiling) from the official repository for instructions on how to profile your app.

## Troubleshooting

* If you get a bunch of this spam in your terminal during the build process: `Requested but did not find extension point with identifier ... for extension ...` don't be alarmed. These messages, while annoying, are harmless and can be ignored.

* If you don't see any `Console.WriteLine` messages or exception logs in the Terminal, try checking the `Console` app on macOS. Select your iOS/tvOS device from the menu on the left, then add your app's name as a Process filter in the top-right. This should show you all the logs relevant to your app. (We're fairly certain the lack of messages is a .NET tooling bug, so hopefully this will be resolved in the future.)

* If .NET whines that it "Could not find any provisioning profiles for iOS", you need to create a dummy Xcode iOS project with the same bundle identifier, which will create a provisioning profile. Then you shouldn't need to touch the Xcode project ever again.

* If you're not getting any sound, flip the ringer switch on the side of the phone to be unmuted. When you set your phone to silent, it mutes all sound from games, unlike other app types (such as media players).

* If you get a DllNotFoundException from a native library at runtime, make sure to include the native library in the `MtouchExtraArgs` property, alongside the fnalibs, like so: `-force_load $(MSBuildProjectDirectory)/{your_library}.a`

* If a tvOS build fails because the linker complains about an "undefined symbol", that could mean one of two things...
	1. You forgot to add a native library to the `MtouchExtraArgs` property, as described earlier.
	2. There is an `extern` method somewhere in your code (or in the fnalibs) that refers to a function not defined for the tvOS platform. If it's a problem with one of the fnalibs, file an issue and we can take a look. If it's in your code, consider removing the extern declaration or wrapping it inside `#if !__TVOS__`.
