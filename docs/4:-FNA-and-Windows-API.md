# 4: FNA and Windows API

While we have set out to make our XNA reimplementation as portable as possible, this work can only ever make XNA itself portable. We cannot make your game portable automatically!

Here are some common things we've seen that will get in the way of making your game 100% portable:

- [64-bit Support](#64-bit-support)
- [DirectInput Support](#directinput-support)
- [Filesystem Portability](#filesystem-portability)
- [Environment.SpecialFolder](#environmentspecialfolder)
- [User32, Kernel32, etc.](#user32-kernel32-etc)
- [System.Drawing](#systemdrawing)
- [System.Windows.Forms](#systemwindowsforms)
- [Using the FNA GameWindow in System.Windows.Forms](#using-the-fna-gamewindow-in-systemwindowsforms)
- [C++/CLI Assemblies](#ccli-assemblies)

***

### 64-bit Support
Unlike XNA, FNA supports both 64-bit and AnyCPU configurations. On Linux and macOS this does not really matter, as the Mono CLR does not care what the architecture is for managed binaries and MonoKickstart automatically picks the right library folder, but on Windows each target architecture needs its own version. These days you can safely assume 64-bit only, but if you absolutely require AnyCPU (be careful, modern Visual Studio releases will still prefer 32-bit for AnyCPU builds anyway!) you should add this code to the start of your Main function:

```
using System;
using System.IO;
using System.Runtime.InteropServices;

...

[DllImport("kernel32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
static extern bool SetDefaultDllDirectories(int directoryFlags);

[DllImport("kernel32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
static extern void AddDllDirectory(string lpPathName);

[DllImport("kernel32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
static extern bool SetDllDirectory(string lpPathName);

const int LOAD_LIBRARY_SEARCH_DEFAULT_DIRS = 0x00001000;

...

static void Main(string[] args)
{
	if (Environment.OSVersion.Platform == PlatformID.Win32NT)
	{
		try
		{
			SetDefaultDllDirectories(LOAD_LIBRARY_SEARCH_DEFAULT_DIRS);
			AddDllDirectory(Path.Combine(
				AppDomain.CurrentDomain.BaseDirectory,
				Environment.Is64BitProcess ? "x64" : "x86"
			));
		}
		catch
		{
			// Pre-Windows 7, KB2533623 
			SetDllDirectory(Path.Combine(
				AppDomain.CurrentDomain.BaseDirectory,
				Environment.Is64BitProcess ? "x64" : "x86"
			));
		}
	}

	...
}
```
With this you can keep the C# binaries at the top level, and the native libraries can be in `x86` and `x64` separately (we do this for you already in fnalibs.tar.bz2).

### DirectInput Support
If you have a specialized DirectInput path for your XNA game, go ahead and take it out of your FNA build. FNA uses SDL_GameController for its GamePad implementation, which supports both XInput and DirectInput on Windows (and for Linux/macOS, we support the standard joystick input interfaces). Even for non-XInput controllers you can expect them to map to the 360 layout provided by GamePad; SDL_GameController pulls in configurations from its own internal database as well as the Steam Big Picture Mode database to automap any known joystick to the 360 layout. Additionally, FNA checks for [gamecontrollerdb.txt](https://github.com/gabomdq/SDL_GameControllerDB/) in the game's root folder so that developers and customers without access to the other databases may add their own configurations as well. Lastly, our implementation is far more flexible - we support numerous extensions and environment variables to allow modernized input support, including vendor/product detection, higher player counts, motion controls, etc.

For more information, see the [SDL_GameController documentation](https://wiki.libsdl.org/CategoryGameController).

### Filesystem Portability
Here are three things you may get hit by when running an FNA title on Linux and macOS:

* Paths are separated with `/`, NOT with `\`! We do what we can to accommodate this in FNA, but _you should not depend on us saving you here!_
* On Linux, filenames are case-sensitive! For example, `fileName` is NOT the same as `Filename`!
* There are no drive letters on Unix-like operating systems. For example, while a user folder on Windows is typically `C:\Users\flibitijibibo\`, on Linux it's typically `/home/flibitijibibo/`, and on macOS it's `/Users/flibitijibibo/`.

There exists an environment variable called MONO_IOMAP that can attempt to resolve these for you, but you should not depend on this feature, as it is neither guaranteed to work nor is it performant! Your I/O performance can get hit by this!

The solution is to just check your code to be sure that you do not depend on any Windows-specific behavior in your code. We recommend changing your code rather than your content's filenames, as that will retain consistency across all platforms without, say, having to worry about platform content folders X and Y when rebuilding your content. It's confusing and it probably won't work if you deploy from Windows anyway.

The best thing you can do to make your file reading portable is to use `TitleContainer.OpenStream` instead of `File.OpenRead`, as this deals with both directory separators as well as special path requirements for platforms with unique filesystems (but this does NOT deal with case sensitivity!). Other helpful features in C# are `Path.DirectorySeparatorChar` and `Path.Combine()`, found in `System.IO`.

### Environment.SpecialFolder
For the most part, these simply don't work correctly on Linux or macOS.

If you use StorageDevice for all your file I/O, don't worry - we take care of this for you.

* For Linux/*BSD, save directories are meant to go in the location specified by the XDG specification.
	* Config files should go in $XDG_CONFIG_HOME, or `~/.config/YourApp/`.
	* Save data should go in $XDG_DATA_HOME, or `~/.local/share/YourApp/`.
* For macOS, save directories are meant to go in the location required by Apple for certification.
	* All user data should go in `~/Library/Application Support/YourApp/`.
* Other platforms may have their own specific needs for savedata.
	* For unspecified platforms, consider using `SDL_GetPrefPath`.

It is _strongly_ recommended that you try to determine these locations at runtime, rather than with platform defs. You want to be able to leverage our single-assembly portability!

Here's an example for determining the save location at runtime:

```
using System;
using System.IO;
using SDL3;

public const string GameName = "flibitGame";
public static readonly string SaveDirectory = GetSaveDirectory();

private static string GetSaveDirectory()
{
	string platform = SDL.SDL_GetPlatform();
	if (platform.Equals("Windows"))
	{
		return Path.Combine(
			Environment.GetFolderPath(
				Environment.SpecialFolder.MyDocuments
			),
			"SavedGames",
			GameName
		);
	}
	else if (platform.Equals("macOS"))
	{
		string osConfigDir = Environment.GetEnvironmentVariable("HOME");
		if (String.IsNullOrEmpty(osConfigDir))
		{
			return "."; // Oh well.
		}
		osConfigDir += "/Library/Application Support";
		return Path.Combine(osConfigDir, GameName);
	}
	else if (	platform.Equals("Linux") ||
			platform.Equals("FreeBSD") ||
			platform.Equals("OpenBSD") ||
			platform.Equals("NetBSD")	)
	{
		string osConfigDir = Environment.GetEnvironmentVariable("XDG_DATA_HOME");
		if (String.IsNullOrEmpty(osConfigDir))
		{
			osConfigDir = Environment.GetEnvironmentVariable("HOME");
			if (String.IsNullOrEmpty(osConfigDir))
			{
				return "."; // Oh well.
			}
			osConfigDir += "/.local/share";
		}
		return Path.Combine(osConfigDir, GameName);
	}
	else
	{
		return SDL.SDL_GetPrefPath(CompanyName, GameName);
	}
}

```

### User32, Kernel32, etc.
Some XNA games use the Windows API directly for features such as the event loop.

FNA actually uses the SDL_Event loop internally, so you cannot directly replace the event loop as you would want to do with SDL_PollEvent. Instead, use SDL_AddEventWatch or SDL_SetEventFilter:

https://wiki.libsdl.org/SDL_AddEventWatch
https://wiki.libsdl.org/SDL_SetEventFilter

When SDL pumps events, these callbacks will be called with the relevant events.

For a more complete guide to the SDL API, see the SDL wiki:

https://wiki.libsdl.org/APIByCategory

Most (if not all) Win32 functions can be replaced with an equivalent SDL call.

### System.Drawing
For the most part this is a harmless namespace, unless you make a call that depends on GDI+. `System.Drawing.Imaging` is particularly painful in this respect.

The problem is that on Linux and macOS, you will need libgdiplus, which has an _insane_ amount of dependencies, absolutely none of which you want.

Sometimes you can simply replace this code with values or constants that replace the GDI-dependent values, but sometimes you may need a real replacement for things like software surface manipulation. In this case, you can use the SDL_Surface API:

https://wiki.libsdl.org/CategorySurface

### System.Windows.Forms
In many XNA games, there is frequent use of the `System.Windows.Forms` namespace for various operations, mostly related to window management.

We strongly recommend replacing this with SDL when moving to FNA. The SDL documentation can be found here:

https://wiki.libsdl.org/APIByCategory

Subsystems like SDL_Video, SDL_Cursor, and SDL_Clipboard should be able to provide you with the same functionality found in System.Windows.Forms.

For example, if you want to use a messagebox:

```
#if SDL3
	SDL3.SDL.SDL_ShowSimpleMessageBox(
		SDL3.SDL.SDL_MessageBoxFlags.SDL_MESSAGEBOX_ERROR,
		title,
		message,
		game.Window.Handle
	);
);
#else
	System.Windows.Forms.MessageBox.Show(
		message,
		title
	);
#endif
```

#### Using the FNA GameWindow in System.Windows.Forms
As mentioned in the previous section, you should NOT be using System.Windows.Forms when shipping an FNA game. However, if you simply want to use it for something like a developer-internal editor that's only used on Windows, there is still a way to hook the FNA window to a Form.

Because the `GameWindow.Handle` no longer directly refers to a Win32 HWND, System.Windows.Forms code that depends on this handle will no longer work. But, there is still a way to keep most of your code here.

For an example on how to use FNA's `GameWindow.Handle` IntPtr for System.Windows.Forms, see this example:

https://gist.github.com/flibitijibibo/cf282bfccc1eaeb47550

Note that you want the window to be borderless when attaching to a Panel. To do this, you can use the `GameWindow.IsBorderlessEXT` extension to remove the border.

### C++/CLI Assemblies
C++/CLI is not available in Mono, so these binaries cannot run anywhere except on Windows with .NET.

The solution is to separate the C# half from the native C/C++ half, and access the native half with P/Invoke calls and native C entry points. For example:

```
/* somelib.h */
#ifndef SOMELIB_H
#define SOMELIB_H

#include <stdint.h>

#ifdef __cplusplus
extern "C" {
#endif

#ifdef _WIN32
	#define EXPORTFN __declspec(dllexport)
	#define DELEGATECALL __stdcall
#else
	#define EXPORTFN
	#define DELEGATECALL
#endif

typedef void (DELEGATECALL *SomeCallback)(int32_t);

EXPORTFN void SomeFunction(SomeCallback callback);

#undef EXPORTFN
#undef DELEGATECALL

#ifdef __cplusplus
}
#endif

#endif /* SOMELIB_H */

/* End somelib.h */

/* somelib.c */
#include "somelib.h"

void SomeFunction(SomeCallback callback)
{
	callback(1337);
}
/* End somelib.c */

/* SomeLib.cs */
using System.Runtime.InteropServices;

public static class SomeLib
{
	public delegate void SomeCallback(int result);

	[DllImport("somelib.dll", CallingConvention = CallingConvenction.Cdecl)]
	public static extern void SomeFunction(SomeCallback callback);
}
/* End SomeLib.cs */
```
