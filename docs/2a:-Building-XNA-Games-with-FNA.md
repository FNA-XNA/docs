# 2a: Building XNA Games with FNA
This is a guide to getting your existing XNA game running on FNA. If you are creating a new game, please see [this wiki page](2b:-Building-New-Games-with-FNA.md).

Note that this entire guide can be done on Windows - you do not need a Mac or a Linux box to perform anything described on this page.

Also note that this is NOT strictly a guide on having your game fully ported to non-XNA platforms! FNA can only make XNA itself portable; whether or not your game is in any way portable is out of our hands. If this is a concern, look at "[FNA and Windows API](4:-FNA-and-Windows-API.md)".

### 1. Making the FNA Solution
We strongly recommend making a brand new solution for the FNA version. It's possible to fiddle with your XNA solution to link your code against FNA, but it's probably just easier to make `YourGame.FNA.sln` and keep `YourGame.sln` as it is. See [Page 1](1:-Setting-Up-FNA.md#chapter-5-creating-new-projects) for a quick refresher on how to do this!

One thing you will NOT need to recreate is the content projects. When deploying to FNA, your content should already be complete and built. You can keep this in your solution if you like, but you will need to reference the XNA content pipeline here. You should not reference FNA in content projects!

(As an aside, if you are creating _new_ content instead of porting existing content, you should not feel pressured to use processing tools like the XNA/MonoGame content pipeline tools! It is perfectly reasonable to develop your own content system that can be designed and optimized to work well with you and your development team, and in fact that is what we recommend doing for new projects.)

Once you've remade your solution with all of your game's subprojects, add FNA.csproj into your solution. FNA, despite using many different C# wrappers, is just a single project file. This simplifies project generation and quickly gives you access to, for example, SDL2# if you need it in your game code.

Your projects' references are going to be the same as they were in XNA4, except now you will reference FNA instead of the XNA libraries.

Once all of the projects have been made and are linking to FNA, your solution should now compile. Simple as that!

### 2. About Content Support
While we do our best to support 100% of the XNA content available, there are a few notable exceptions due to both technical and legal obstacles.

#### 2a. About Effect Support
FNA uses MojoShader to parse and rebuild Effect shaders to non-D3D graphics APIs. This allows us to use the original XNB-packed Effects built by XNA and `fx_2_0` effect binaries built by [FXC](https://msdn.microsoft.com/en-us/library/windows/desktop/bb232919.aspx), the Effect compiler from the [June 2010 DirectX SDK](https://www.microsoft.com/en-us/download/details.aspx?id=6812). However, there are exactly three caveats to this:

- We do not attempt to undo half-pixel offsets applied by the game's shaders. If you don't know what this is, don't worry. If you do know what this is and have applied them in your engine, simply remove them from the FNA version of your game and it should visually match D3D without any problems.
- Another half-pixel-like issue present is in the VPOS attribute; D3D9 puts the VPOS at integer values while all other graphics APIs put it at half-pixel values (in contrast to the half-pixel offset described above, where it's the exact opposite). If you find that VPOS isn't acting as expected, take the input VPOS value and `floor()` it before doing anything else in the shader.
- For maximum compatibility, vertex input layouts must match the vertex shader input parameters _exactly_. D3D9 and OpenGL would quietly drop input streams that were unused in the vertex shader, but newer graphics APIs like D3D11 and Vulkan are far more strict about this. For example, if your vertex shader expects a `TEXCOORD0` input, but your VertexDeclaration does not include a TextureCoordinate with index 0, this is a hard error unless you forcibly use the OpenGL renderer exclusively.

#### 2b. Rebuilding WMA/WMV Files
XNA uses Windows Media Player for the `Song` and the `VideoPlayer` implementations. Obviously we cannot use this in a multiplatform reimplementation, so currently we do not support WMA/WMV files. Instead, we support [Ogg Vorbis](http://www.vorbis.com/) and [QOA](https://qoaformat.org/) for audio and [Ogg Theora](http://www.theora.org/) for video.

#### 2c. Rebuilding XACT WaveBanks
XACT for Windows supports a codec called [xWMA](http://wiki.multimedia.cx/index.php?title=Microsoft_xWMA) for highly-compressed audio streams, rather than the XMA codec typically used on the 360. Like WMA support we do not have support for either of these, but you can still compress your WaveBanks with ADPCM. The file size will likely be larger, but the functionality of your XACT code/content should be exactly the same, and no changes are required if you do not use xWMA or XMA.

### 3. Window Icons
Typically a Windows application will use the embedded .ico image for both the window icon and the icon used in other parts of the OS, such as the taskbar. While this does work on Windows, it actually turns out to be unusable on other platforms, so we instead use a separate bitmap file to set the process icon.

For macOS you don't have to worry about this; the icns file that is discussed in the "[Distributing](3:-Distributing-FNA-Games.md#macos)" section will cover everything there.

For Linux (and optionally Windows, if you want a higher-res icon), simply place a bitmap file in the game's root directory. The bitmap's filename is the same string as the window title, minus the characters that are not allowed for filenames. The recommended image size is at least 512x512, as many desktops will use this icon for more than just the 16x16 image next to the window title.

### 4. Running the FNA Output
Once the FNA version has been built, copy over your Content folder and native libraries (which you should have downloaded along with FNA itself) into the output folder. From there, the game should be able to run!

This exact output will be what you run on Linux and macOS. The only difference will be the native libraries in addition to `FNA.dll.config`, which should be in your output folder even if you're on Windows. `FNA.dll.config` is what remaps the native DLL names to the proper names of the native libraries on non-Windows operating systems. Aside from this, everything else should work - the C# assemblies, the Content, everything. [Push this output to a Linux/macOS box and try them out!](3:-Distributing-FNA-Games.md)

When using a developer environment on macOS, you will want to add an environment variable that sets `DYLD_LIBRARY_PATH=./osx/`, so that the IDE's runtime environment will find the fnalibs binaries.