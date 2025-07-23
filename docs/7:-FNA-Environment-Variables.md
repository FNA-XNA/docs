# 7: FNA Environment Variables

FNA will check for certain environment variables to perform certain tasks in a non-default manner. For the most part these are meant to be used by both developers and customers, but there are also cases where we use environment variables as a means of working around a specific problem without requiring a build-time option/configuration.

**NOTE:** .NET Core intentionally broke native library compatibility with `System.Environment.SetEnvironmentVariable`, so for any FNA3D environment variables, we recommend using `SDL_SetHint` instead, even though this will have to be updated when SDL3 is released. The documentation below uses this function, if you need an example. (You are of course welcome to continue using .NET Framework and Mono instead, which does not have this issue.)

FNA:

- [FNA_GRAPHICS_ENABLE_HIGHDPI](#fna_graphics_enable_highdpi)
- [FNA_GRAPHICS_JPEG_SAVE_QUALITY](#fna_graphics_jpeg_save_quality)
- [FNA_KEYBOARD_USE_SCANCODES](#fna_keyboard_use_scancodes)
- [FNA_GAMEPAD_NUM_GAMEPADS](#fna_gamepad_num_gamepads)
- [FNA_SDL_FORCE_BASE_PATH](#fna_sdl_force_base_path)

FNA3D:

- [FNA3D_FORCE_DRIVER](#fna3d_force_driver)
- [FNA3D_ENABLE_HDR_COLORSPACE](#fna3d_enable_hdr_colorspace)
- [FNA3D_ENABLE_LATESWAPTEAR](#fna3d_enable_lateswaptear)
- [FNA3D_MOJOSHADER_PROFILE](#fna3d_mojoshader_profile)
- [FNA3D_BACKBUFFER_SCALE_NEAREST](#fna3d_backbuffer_scale_nearest)
- [FNA3D_OPENGL_WINDOW_DEPTHSTENCILFORMAT](#fna3d_opengl_window_depthstencilformat)
- [FNA3D_OPENGL_FORCE_ES3](#fna3d_opengl_force_es3)
- [FNA3D_OPENGL_FORCE_CORE_PROFILE](#fna3d_opengl_force_core_profile)
- [FNA3D_OPENGL_FORCE_COMPATIBILITY_PROFILE](#fna3d_opengl_force_compatibility_profile)

***

### FNA_GRAPHICS_ENABLE_HIGHDPI
While many modern displays are capable of much higher DPIs (such as "Retina" displays on macOS and iOS), XNA does not have any mechanism for supporting high-DPI rendering.

Enabling this feature is as simple as setting this before creating your Game:

```
Environment.SetEnvironmentVariable("FNA_GRAPHICS_ENABLE_HIGHDPI", "1");
```

After calling your Game constructor, you can then check to see if high-DPI creation was successful:

`Settings.HighDPI = Environment.GetEnvironmentVariable("FNA_GRAPHICS_ENABLE_HIGHDPI") == "1";`

While it is possible for us to create windows with high-DPI drawing space, we cannot guarantee that it will be available at all times. Worse, we can only acquire this feature on startup, so toggling high-DPI mode will require restarting your game.

When this is enabled, the drawable size will take priority over the window size. So, when running at 1920x1080 in windowed mode, the _drawable_ size will be 1920x1080, but the _window_ size will be 960x540. This allows users to set the game resolution to the maximum resolution allowed by the OS while having the appropriate window size.

In fullscreen mode we will continue to run at the desktop resolution with a faux-backbuffer for non-native resolutions, but in high-DPI mode the native resolution will now be the _actual_ native resolution rather than the _simulated_ resolution.

As always, be sure that your RenderTargets and Viewports match the backbuffer size and NOT the window size!

Lastly, when packaging for macOS, be sure this is in your app bundle's Info.plist:
```plist
	<key>NSHighResolutionCapable</key>
	<string>True</string>
```

This variable is accessible to users by passing `/enablehighdpi:1` as a launch option.

### FNA_GRAPHICS_JPEG_SAVE_QUALITY
For some reason they didn't expose a quality parameter to SaveAsJpeg, so we added one ourselves! This variable is checked each time you call the function, so feel free to customize this based on the context for each time you save an image. The value is expected to be between 1 and 100.

### FNA_KEYBOARD_USE_SCANCODES
XNA keys are based on keycodes, rather than scancodes.

With SDL, for example, you can actually pick between SDL_Keycode and SDL_Scancode, but scancodes will not be accurate to XNA4. The benefit is that scancodes will ignore "foreign" keyboard layouts, making default keyboard layouts work out of the box everywhere (unless the actual symbol for the keys matters in your game).

While FNA provides the [GetKeyFromScancodeEXT](5:-FNA-Extensions.md#getkeyfromscancodeext) extension and developers are encouraged to use it, this is not a required function and users may benefit from this environment variable in the event that layouts are not checked in-game.

In either case, [TextInputEXT](5:-FNA-Extensions.md#textinputext) will still read the actual chars correctly, so you can (mostly) have your cake and eat it too if you don't care about your bindings menu not making a lot of sense on foreign layouts.

To use scancodes instead of keycodes, set this variable to "1" before starting the game.

This variable is accessible to users by passing `/usescancodes:1` as a launch option.

### FNA_GAMEPAD_NUM_GAMEPADS
XNA4 supports four controllers, per XInput's limitations. However, SDL gives us the ability to support more controllers when available. You can set this environment variable on/before program startup to set a controller count without modifying FNA:

```cs
Environment.SetEnvironmentVariable("FNA_GAMEPAD_NUM_GAMEPADS", "8");
```

### FNA_SDL_FORCE_BASE_PATH
For the desktop, we use `AppDomain.CurrentDomain.BaseDirectory` as the root path, as this is more accurate on platforms where the "real" EXE path is the path of the CLR. However, there may be some scenarios even on desktop where SDL_GetBasePath() is more appropriate. If you want to override our defaults, this variable can be set on startup.

### FNA3D_FORCE_DRIVER
FNA3D supports multiple graphics backends with a single binary. Sometimes the default may not produce the optimal result for specific hardware, or a backend may exhibit graphics driver bugs that are not present in other backends. The list of available driver strings for this environment variable is as follows, in order of the current default priority:

- SDL_GPU (Vulkan, Metal, D3D12)
- D3D11
- OpenGL

This variable is accessible to users by passing `/gldevice:%s` as a launch option, where `%s` can be one of the above strings. Note that the string must match an available driver _exactly_, or device creation will fail!

### FNA3D_ENABLE_HDR_COLORSPACE
The XNA specification includes multiple surface formats capable of using an HDR colorspace - however, all rendering is assumed to be standard sRGB. Applications can set this variable to "1" to enable support for creating swapchains with the HDR10 (Rgba1010102) and extended sRGB (HalfVector4/HdrBlendable) colorspaces.

Note that this only exposes the ability to set the colorspace of the swapchain and nothing else - any and all colorspace conversion still has to be done by the application. Additionally, be careful when adding user-facing support for HDR; it should be treated as the equivalent of a full kernel modeset, so it is very expensive and fragile to attempt anywhere except on startup!

This feature is only supported on SDL_GPU and D3D11.

### FNA3D_ENABLE_LATESWAPTEAR
For many years FNA would default to using `FIFO_RELAXED`/[EXT_swap_control_tear](https://www.khronos.org/registry/OpenGL/extensions/EXT/GLX_EXT_swap_control_tear.txt) VSync to improve performance when games temporarily performed below the display's refresh rate. However, in recent years this has fallen out of fashion; most platforms don't support this feature at all and the platforms that _did_ support it have opted to avoid presentation systems that involve tearing at all (in favor of other systems like "mailbox" presentation). Additionally, refresh rates have rapidly become unstandardized within the last hardware generation, so fixed-rate games would see tearing if the game rate was below the refresh rate of the monitor. Some may still want to try using the late swap tear feature with their OS/driver/display combo, however, so users can set this to "1" if they'd like.

This variable is also accessible to users by passing `/enablelateswaptear:1` as a launch option.

### FNA3D_MOJOSHADER_PROFILE
FNA3D uses [MojoShader](https://icculus.org/mojoshader/) to support XNA4 (D3D9) Effect shader binaries. One major benefit of MojoShader is that it supports multiple shader profiles, including GLSL and SPIR-V. By default, each renderer picks the most appropriate shader profile by itself, but you can set this variable to forcibly use a specific profile.

Unless you're a developer working on a new shader profile, this is probably useless to you.

This variable is accessible to users by passing `/mojoshaderprofile:%s` as a launch option, where `%s` can be `glsl120` or `glspirv`. Currently this is only useful for OpenGL.

### FNA3D_BACKBUFFER_SCALE_NEAREST
When using the faux-backbuffer, we use a linear filter to blit to the window's backbuffer. However, some games may prefer to use a point/nearest-neighbor filter to scale in specific scenarios (such as pixel art games running in multiples of the desktop resolution). To use that filter instead, set this variable to "1" before running the game.

This variable is accessible to users by passing `/backbufferscalenearest:1` as a launch option.

### FNA3D_OPENGL_WINDOW_DEPTHSTENCILFORMAT
OpenGL contexts are very clunky and require the RGB/Depth/Stencil sizes at window creation time rather than context creation time, and cannot be reset without destroying the window and GL context. By default we play it safe and create a window with a `Depth24Stencil8` backbuffer, but if you want to optimize on this you can set this environment variable to a `DepthFormat` enum value at program startup to override our settings. For example:

```cs
SDL3.SDL.SDL_SetHintWithPriority(
    "FNA3D_OPENGL_WINDOW_DEPTHSTENCILFORMAT",
    "None",
    SDL3.SDL.SDL_HintPriority.SDL_HINT_OVERRIDE
);
```

Note, however, that the OS may completely ignore your settings and pick whatever it wants for its window backbuffer format. (This is rare, but it's still a nonzero chance...)

### FNA3D_OPENGL_FORCE_ES3
FNA3D has support for OpenGL ES 3.0. Formerly used for iOS/tvOS, it is also known to work with Google's ANGLE project (D3D11 only) and, to some extent, some Android hardware. This may also be useful to those that would like to try to port to new ES platforms (WebAssembly, Android, etc.). Simply set this to "1" on startup and an ES context will be used instead of a desktop context.

This variable is accessible to users by passing `/glprofile:es3` as a launch option.

### FNA3D_OPENGL_FORCE_CORE_PROFILE
FNA3D currently aims for OpenGL 2.1 with a handful of ARB extensions for features like multisampling, hardware instancing, and so on. However, we are also capable of targeting the OpenGL 4.6 Core Profile, with a small handful of mostly-harmless changes. [RenderDoc](https://renderdoc.org/) requires this to capture frames; set this variable to "1" in the capture GUI before launching to successfully capture frames.

While this does enable Core Profile features, note that we still do what your engine tells us to do. For example, even though client-side vertex/index data is forbidden, we still still attempt to render as such, and ARB_debug_output as well as the RenderDoc capturing system will throw errors (and possibly crash) because you are not using buffer objects.

Note that this is probably useless in every situation other than accommodating graphics debuggers.

This variable is accessible to users by passing `/glprofile:core` as a launch option.

### FNA3D_OPENGL_FORCE_COMPATIBILITY_PROFILE
Some devices (i.e. NVIDIA Tegra) will default to either an OpenGL ES context or a modern OpenGL context that is incompatible with certain legacy OpenGL features. However, sometimes those devices will support the context version that FNA3D tries to aim for, so this variable exists to let developers attempt a 2.1 Compatibility context for those situations. That said, if your device wants a context version that your data isn't compatible with, try fixing the data at some point!

This variable is accessible to users by passing `/glprofile:compatibility` as a launch option.
