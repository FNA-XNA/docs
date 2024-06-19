# 0: FAQ

***

## How can I financially contribute to FNA's development?
Currently the only way is through [GitHub Sponsors](https://github.com/sponsors/flibitijibibo). 100% of your contribution goes directly to the lead maintainer (unlike competing crowdfunding services) and your payment information is securely stored and not shared with the FNA team in any way (i.e. "you don't have to give Some Random Guy your credit card information").

## What is the difference between FNA and MonoGame?
Please read the documentation regarding [compatibility between the two](appendix/Appendix-D:-MonoGame.md). For any deeper comparison, the only way to know the difference is to run both of them yourself.

## I'm new to programming and-
Hi there! Glad you've taken an interest in programming. Please do not use FNA yet.

While FNA is a programming tool, it is targeted specifically at experienced programmers. If you are not familiar with C# and msbuild (via Visual Studio, MonoDevelop, or maybe just the XML format?) you will not have a good experience.

It is required that you learn at least the basics of the above, and also the difference between managed and native code, before using FNA.

## I was following a tutorial from Website X and-
Unless it came directly from this wiki, do NOT use any third party documentation! Most tutorials are very poorly written and completely unmaintained; if there was a good tutorial we would have endorsed it on the front page of our wiki. It's rare to see a tutorial that even gets the development environment right (see below).

Our documentation is minimal, but straightforward for experienced C# programmers (see above).

## What development environments are supported?
The only officially supported environments are the following:

- The environment described on [Page 1](1:-Setting-Up-FNA.md)
- MonoDevelop on Fedora Workstation using the mono-project.com repository, targeting .NET Framework with FNA.csproj
- Visual Studio 2010 or newer on Windows, targeting .NET Framework with FNA.csproj
- Visual Studio 2019 or newer on Windows, targeting consoles with NativeAOT and FNA.Core.csproj
- Visual Studio for Mac, targeting iOS/tvOS with FNA.Core.csproj

_All_ other environments and runtimes are unsupported and should not be used.

## Where is the NuGet package?
FNA and its sublibraries do not use NuGet in any capacity. We _strongly_ recommend avoiding NuGet in general, and for FNA we recommend adding FNA.csproj (or FNA.Core.csproj) directly to your solution. FNA itself compiles almost instantly, and debug builds are incredibly valuable to have during development.

Any and all NuGet packages for any of our code (FNA, SDL2#, etc.) are unauthorized and should be avoided. If the package claims that we're the authors, please report the package as it is misrepresenting its authors and potentially violating the copyright license (i.e. "... must not be misrepresented as being the original software").

## What is the FNA content system?
FNA supports the XNA content pipeline for preservation reasons, but we _strongly_ recommend against using it on new projects. FNA supports loading common data formats like PNG, WAV and OGG directly, and the community maintains a few libraries for font loading and rendering. For anything more specialized you can bring in an external library or write your own processing and importing tools. Your content system does not have to be complex and there is nothing wrong with simple approaches like loading textures from PNG files.

## How do I use shaders with FNA?
FNA uses Direct3D 9 Effects, to match XNA's shader content format. Using fxc.exe, ideally from the DirectX SDK:

```
fxc.exe /T fx_2_0 MyEffect.fx /Fo MyEffect.fxb
```

Other shader formats, languages, etc are not supported. While FXC is a Windows binary, the native d3dcompiler used to build these binaries is known to work with Wine.

## Where is PlayStation support?
If a project comes along and there's money behind it, we'll do it. There are a couple pending projects already, but you should not hesitate to get in touch if you have SDK access and are willing to develop and/or fund the completion of FNA PS4/PS5.

## Where is Android support?
Android is not and cannot be supported. The solution you just thought of does not work. No, not that other one either. Or even the [one someone got booting, almost](https://github.com/0x0ade/FNADroid).

Internally, we have what's called a "body count" for anyone that tries to add support. It was pretty funny, at least for the first dozen increments.

We intend to look at [Fuchsia](https://en.wikipedia.org/wiki/Fuchsia_(operating_system)) once production devices are available. In the meantime, FNA is known to work on [PinePhone](https://www.pine64.org/pinephone/) with various Free mobile operating systems.

## I get a BadImageFormatException, and-

You mixed up 32- and 64-bit binaries. As it turns out, Microsoft broke their CPU arch configuration in two ways:

1. They added a flag to AnyCPU to make it default to 32-bit even on 64-bit Windows. It's a checkbox in your project settings, uncheck it.
2. They made it so new projects don't actually have x86 and x64 configs, only AnyCPU, so you may have added FNA to your solution only to get an x64 entry in your platform dropdown that isn't actually x64! Feel free to keep using AnyCPU and ignore x86/x64 once you've unchecked Prefer 32-bit, but if it's any consolation you can probably just make an x64 config for your project and always use that, so there's no ambiguity as to what arch you're targeting.

(Also, once you've done all this, delete bin/ and obj/. No, we don't know why you need to do this.)

## I have a bug when running on VirtualBox, and-
The bug is that you are using VirtualBox. Please use [VMware Player](https://www.vmware.com/products/workstation-player.html) instead.

## I have a bug when running on system Mono, and-
Your LD_LIBRARY_PATH is busted. You can do one of three things:

- Preserve the lib64 folder (like you're supposed to anyway) and set LD_LIBRARY_PATH to include that folder
- Delete your output's copy of libSDL2-2.0.so.0, keeping the rest of the libs next to your exe, and be sure your distribution provides the latest stable SDL release (maybe don't do this one since it's not the official build)
- Throw the fnalibs into /usr/local/lib (definitely don't do this one)

For shipping builds, [use MonoKickstart](3:-Distributing-FNA-Games.md#gnulinux), do NOT use system dependencies!

## Where can I get fnalibs for CPU architecture X?
If it is not included in the standard fnalibs.tar.bz2 you will need to build the libraries from source. The instructions for each library can be found in their respective README files.

## What happens if I ask a question that's answered in this wiki?
\**Bubs voice*\* That'll be [five dollars](https://github.com/sponsors/flibitijibibo/). No, this isn't a joke. Expect a link to the sponsors page for each of your requests.
