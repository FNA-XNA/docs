# Appendix B: FNA on Consoles

FNA supports deploying to consoles via NativeAOT. FNA does not have any private branches for each platform; the public master branch of FNA is exactly what is used to ship for these targets. The platform code is contained entirely in SDL and the NativeAOT bootstrap.

***

## General Advice

While the runtimes require a console NDA, there are some things you can do to make your game more robust that just-so-happens to make console support easier, without access to any particular SDK. If you're familiar with consoles, none of these will be surprising:

### NativeAOT

All console builds use NativeAOT as the runtime. If you want a solid head-start, you should read Appendix A. Don't underestimate this step, especially if your game heavily depends on .NET's reflection features!

### Bootstrapping

For non-PC builds it is generally a good idea to assume that platform-specific bootstrapping is needed - in FNA's case we make use of SDL3's `SDL_RunApp` functionality; your code should be able to stay the same except for the Main function:

```
    [STAThread]
    static void Main(string[] args)
#if NET
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

Our GDK support is now 100% public source code! Once you have signed the GDK Agreement with Microsoft and have installed the Xbox GDK you can start with the [NativeAOT repository](https://github.com/FNA-XNA/NativeAOT-Xbox). Additionally, developers can request access to our `#xbox` Discord channel once they join both the FNA and ID@Xbox Discord servers.

## Nintendo Switch

While there is no special code needed for Nintendo Switch support (100% of the platform code is in SDL and NativeAOT, two separate projects), all consulting and documentation is private, per NDA requirements. If you are a licensed developer, please get access to SDL-switch (search the licensee forums) and then get in touch with flibit on Discord. If you are NOT a licensed developer, you're on your own. None of us are able to get you hooked up, so please only get in touch AFTER you have Switch SDK access.

## PlayStation

While there is no special code needed for PS5 support (100% of the platform code is in SDL and NativeAOT, two separate projects), all consulting and documentation is private, per NDA requirements. If you are a licensed developer, please get access to SDL-playstation (contact [Ryan C. Gordon](mailto:icculus@icculus.org) for this) and then get in touch with flibit on Discord. If you are NOT a licensed developer, you're on your own. None of us are able to get you hooked up, so please only get in touch AFTER you have PlayStation SDK access.

One detail we _can_ share is that shaders _must_ be precompiled ahead of time; as a result you will need to preprocess your Effect shaders (but you can continue to use the binaries as-is - no, really!). The best way to do this is to make an [FNA3D trace](https://github.com/FNA-XNA/FNA3D/tree/master/replay) of a complete playthrough of your game, then dump SPIR-V binaries with FNA3D's [dumping tool](https://github.com/FNA-XNA/FNA3D/tree/master/dumpspirv). These SPIR-V modules will come in handy to create a working PlayStation renderer.
