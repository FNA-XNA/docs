# Appendix B: FNA on Consoles

FNA supports deploying to Xbox and Nintendo Switch via NativeAOT. FNA does not have any private branches for each platform; the public master branch of FNA is exactly what is used to ship for these targets. The platform code is contained entirely in SDL and the NativeAOT bootstrap.

***

## General Advice

While the runtimes require a console NDA, there are some things you can do to make your game more robust that just-so-happens to make console support easier, without access to any particular SDK. If you're familiar with consoles, none of these will be surprising:

### NativeAOT

All console builds use NativeAOT as the runtime. If you want a solid head-start, you should read Appendix A. Don't underestimate this step, especially if your game heavily depends on .NET's reflection features!

### Window Size Changes
Even if your window is not resizable, operating systems (including Windows!) may forcibly change the window size for a multitude of reasons, and so the graphics device will reset.

Consider the following code:

```
void ApplyVideoSettings(int width, int height, bool fullscreen, bool vsync);
{
    // Update GraphicsDeviceManager...
    graphics.PreferredBackBufferWidth = width;
    graphics.PreferredBackBufferHeight = height;
    graphics.SynchronizeWithVerticalRetrace = vsync;
    graphics.IsFullScreen = fullscreen;

    // Apply!
    graphics.ApplyChanges();

    // A bunch of engine stuff
    menu.ResizeScreen();
    renderer.RecreateRenderTargets();
}
```

This is actually kind of wrong; even in fullscreen mode it is possible for the operating system to affect your window size and break your game. But you don't need to go through a bunch of trouble to support `AllowUserResizing` or anything like that, as XNA internally hooks up `ClientSizeChanged` to `GraphicsDeviceManager.ApplyChanges()`, which should lead to a reset that you can catch with `GraphicsDeviceManager.DeviceReset` events:

```
public MyGame() : base()
{
    graphics = new GraphicsDeviceManager(this);
    graphics.DeviceCreated += OnDeviceCreated;
    graphics.DeviceReset += OnDeviceReset;
}

private void OnDeviceCreated(object sender, EventArgs e)
{
    // A bunch of engine size stuff before Initialize()
}

private void OnDeviceReset(object sender, EventArgs e)
{
    // A bunch of engine stuff
    menu.ResizeScreen();
    renderer.RecreateRenderTargets();
}

void ApplyVideoSettings(int width, int height, bool fullscreen, bool vsync);
{
    // Update GraphicsDeviceManager...
    graphics.PreferredBackBufferWidth = width;
    graphics.PreferredBackBufferHeight = height;
    graphics.SynchronizeWithVerticalRetrace = vsync;
    graphics.IsFullScreen = fullscreen;

    // Apply!
    graphics.ApplyChanges();

    // A bunch of engine stuff used to be here. It's gone now.
}
```

Even if you do not support resizable windows, you need to be prepared for resizes to happen at absolutely any time in your program.

### Filesystem Portability (Again)

Unless you're working with savedata, using `System.IO.File` is highly discouraged. (And even then, savedata was supposed to be done with `Microsoft.Xna.Framework.Storage` in the XBLIG world). If you're loading files with `File.Open` specifically, your game may not even work on PC because the player may have the game installed in a location without write permissions!

To load files, use `TitleContainer.OpenStream` instead. Save data should be handled with `Microsoft.Xna.Framework.Storage`, but if you already have established savedata out in the wild, isolate your filesystem calls as much as possible. Lord knows how many times I've done [this](../4:-FNA-and-Windows-API.md#environmentspecialfolder) to make Linux savedata not go directly in `$HOME`...

## Xbox GDK
GDK support is now available to ID@Xbox licensees. SDL supports GDK on both PC and Xbox, and FAudio/Theorafile work as-is. FNA3D is currently targeting GDKX via a port of GLon12, which we upstreamed for release in Mesa 23.1.

Developers can request NativeAOT-GDKX access via Discord once they have signed the GDK agreements with Microsoft. 

### Building fnalibs
[SDL](https://github.com/libsdl-org/SDL/), FNA3D, FAudio, and Theorafile all have VisualC-GDK folders with pre-made project files. Compile, grab the DLLs, add said DLLs to your project.

### Code Differences
Your code should be able to stay the same except for the Main function:

```
    [STAThread]
    static void Main(string[] args)
#if GDK
    {
        Environment.SetEnvironmentVariable("FNA_PLATFORM_BACKEND", "SDL3");
        realArgs = args;
        SDL3.SDL.SDL_main_func mainFunction = FakeMain;
        SDL3.SDL.SDL_RunApp(0, IntPtr.Zero, mainFunction, IntPtr.Zero);
    }

    static string[] realArgs;
    static int FakeMain(int argc, IntPtr argv)
    {
        RealMain(realArgs);
        return 0;
    }

    static void RealMain(string[] args)
#endif
    {
        // blah blah blah
    }
```

## Nintendo Switch

While there is no special code needed for Nintendo Switch support (100% of the platform code is in SDL and NativeAOT, two separate projects), all consulting and documentation is private, per NDA requirements. If you are a licensed developer, please get access to SDL-switch (search the licensee forums) and then get in touch with flibit on Discord. If you are NOT a licensed developer, you're on your own. None of us are able to get you hooked up, so please only get in touch AFTER you have Switch SDK access.

## PlayStation

FNA for PlayStation 4 and 5 is now in progress - the first draft of SDL-playstation was recently finished, with FNA3D support coming up next. FAudio and Theorafile are already working on PlayStation targets! For runtimes we are currently using NativeAOT, with Mono as our fallback plan.

If you are a licensee, please get in touch with [Ryan](mailto:icculus@icculus.org) for SDL access, then once you have access to that, let us know!
