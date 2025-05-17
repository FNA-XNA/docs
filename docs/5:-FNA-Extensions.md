# 5: FNA Extensions

FNA, in rare cases, will provide extensions to the XNA 4.0 Refresh specification in order to simplify often-unportable tasks, provide feature parity with other projects such as the MonoGame project, and so on.

***

### GamePad Extensions

The GamePad extensions are unique in that there are a _lot_ of them (our policy is generally to not extend the spec unless it's a life-or-death situation) and that they are pretty much universally recommended to use, unlike the other extensions which generally exist for very specific use cases. FNA currently has the following extensions for controller support:

#### GetGUIDEXT
`public static string GetGUIDEXT(PlayerIndex playerIndex)` is a new method for `GamePad` that allows you to get the hardware GUID for a given controller. Since GUIDs are organized differently depending on the OS, we have abstracted the GUID string into a shortened, unified format that is consistent across operating systems.

To use the GetGUIDEXT extension:
```cs
public MyControllerType GetControllerType(PlayerIndex index)
{
	/* Try to only do this once on initialization! It's slow! */
	string guid = GamePad.GetGUIDEXT(index);
	if (guid.Equals("4c05c405") || guid.Equals("4c05cc09"))
	{
		return MyControllerType.PS4;
	}
	if (guid.Equals("4c05e60c"))
	{
		return MyControllerType.PS5;
	}
	if (guid.Equals("7e050920") || guid.Equals("7e053003"))
	{
		return MyControllerType.Nintendo;
	}
	return MyControllerType.Xbox;
}
```

#### SetLightBarEXT
`public static void SetLightBarEXT(PlayerIndex playerIndex, Color color)` is a new method for `GamePad` that allows you to set the color of the light bar on the DualShock 4 and DualSense controllers. There is a matching `bool HasLightBarEXT` property in `GamePadCapabilities`.

To use the SetLightBarEXT extension:
```cs
public void UpdateLightBar()
{
	if (playerIsDead)
	{
		GamePad.SetLightBarEXT(index, Color.Red);
	}
	else
	{
		GamePad.SetLightBarEXT(index, Color.Green);
	}
}
```

#### SetTriggerVibrationEXT
`public static bool SetTriggerVibrationEXT` is a new method for `GamePad` that allows setting the speed of motors specifically found in the triggers - this is found in the Xbox One controller, for example. There is a matching `bool HasTriggerVibrationMotorsEXT` property in `GamePadCapabilities`.

The function is used in exactly the same way as `SetVibration`, so wherever you have a SetVibration in your game you will probably want a SetTriggerVibrationEXT call as well, should you choose to support trigger-specific haptics.

#### GetGyroEXT/GetAccelerometerEXT
`public static bool GetGyroEXT/GetAccelerometerEXT` are new methods for `GamePad` that poll the state of a gyro in a controller, should they exist (the PS4 and Switch controllers have them, for example). There are matching `bool HasGyroEXT/HasAccelerometerEXT` properties in `GamePadCapabilities`.

To use the extensions:
```cs
public static void DoMotionControls()
{
    Vector3 gyro, accel;
    GamePad.GetGyroEXT(PlayerIndex.One, out gyro);
    GamePad.GetAccelerometerEXT(PlayerIndex.One, out accel);
    // Have fun!
}
```

#### Button Extensions

A number of bitflags have been added to the `Buttons` enum - they are as follows:

```cs
Misc1EXT =   	0x00000400,
Paddle1EXT =	0x00010000,
Paddle2EXT =	0x00020000,
Paddle3EXT =	0x00040000,
Paddle4EXT =	0x00080000,
TouchPadEXT =	0x00100000
```

Polling these works the same way as the other buttons, call `GamePadState.IsButtonDown/IsButtonUp` to check for them. Note that we did NOT add properties to the GamePadButtons struct, nor did we add a new GamePadState struct, but we did add matching properties to `GamePadCapabilities`.

### Content Extensions

Unlike XNA, `Content.Load<>` can also import raw (non-XNB) assets in various file formats. The list of supported formats for each content type is as follows:

- **Effect:** FXB

- **Song:** OGG/OGA, QOA

- **SoundEffect:** WAV

- **Texture2D:** BMP, GIF, JPEG, PNG, TGA, TIFF, DDS, QOI

- **TextureCube:** DDS

- **Video:** OGG/OGV

### SurfaceFormat Extensions

A handful of texture formats have been added for various reasons - they are listed below:

- `ColorBgraEXT` allows the use of BGRA8 texture data, rather than SurfaceFormat.Color's RGBA8 format. This is likely only useful if you are preserving data packed by the XNA 3.1 content pipeline.
- `ColorSrgbEXT` and `Dxt5SrgbEXT` expose sRGB colorspaces to Textures and RenderTargets, where supported.
- `Bc7EXT` and `Bc7SrgbEXT` expose BC7 compression support, where supported.

### FNALoggerEXT
`public static class Microsoft.Xna.Framework.FNALoggerEXT` is a new class that exposes FNA's internal logging system. Simply assign the Log functions inside to use your own logging system, if desired.

To use the FNALoggerEXT extension:

```cs
static void Main(string[] args)
{
    /* We recommend setting this before touching anything XNA-related! */
    FNALoggerEXT.LogInfo = (msg) => MyLogger.Log("FNA", "INFORMATION", msg);
    FNALoggerEXT.LogWarn = (msg) => MyLogger.Log("FNA", "WARNING", msg);
    FNALoggerEXT.LogError = (msg) => MyLogger.Log("FNA", "ERROR", msg);
}
```

### IsBorderlessEXT
`public bool IsBorderlessEXT { get; set; }` is a new property for `GameWindow` that gives you the ability to show/hide the window border without having to perform direct interop on the `Handle` pointer.

To use the IsBorderlessEXT extension:
```cs
public void ApplyVideoSettings()
{
	graphicsDeviceManager.ApplyChanges();

	/* We recommend setting this after calling ApplyChanges! */
	game.Window.IsBorderlessEXT = wantsBorderless;
}
```

### SetStringMarkerEXT
`public void SetStringMarker(string text)` is a new method for `GraphicsDevice` that abstracts access to functions like `glStringMarkerGREMEDY` and `D3DPERF_SetMarker`. It should only be accessed in a debug context!

To use the SetStringMarkerEXT extension:
```cs
[Conditional("DEBUG")]
public void GraphicsDebugString(string text)
{
	graphicsDevice.SetStringMarkerEXT(text);
}

public void DrawStuff()
{
	GraphicsDebugString("First set of polygons");
	graphicsDevice.DrawIndexedPrimitives(...);
	GraphicsDebugString("Second set of polygons");
	graphicsDevice.DrawIndexedPrimitives(...);
}
```

### GetKeyFromScancodeEXT
`public static Keys GetKeyFromScancodeEXT(Keys scancode)` is a new method for `Keyboard` that translates a `Keys` value, interpreted as a scancode, into the actual `Keys` value based on the current keyboard layout. For example, on an AZERTY keyboard, GetKeyFromScancode(Keys.Q) will return Keys.A.

If [FNA_KEYBOARD_USE_SCANCODES](7:-FNA-Environment-Variables.md#fna_keyboard_use_scancodes) is enabled, the return value will always be the same as the input value.

To use the GetKeyFromScancodeEXT extension:
```cs
private Keys PlayerForward;
private Keys PlayerBackward;
private Keys PlayerStrafeLeft;
private Keys PlayerStrafeRight;
private Keys PlayerUse;
public void AssignDefaultKeys()
{
    // Typically you might just assign a Keys value directly. Not here!
    PlayerForward =     Keyboard.GetKeyFromScancodeEXT(Keys.W);
    PlayerBackward =    Keyboard.GetKeyFromScancodeEXT(Keys.S);
    PlayerStrafeLeft =  Keyboard.GetKeyFromScancodeEXT(Keys.A);
    PlayerStrafeRight = Keyboard.GetKeyFromScancodeEXT(Keys.D);
    PlayerUse =         Keyboard.GetKeyFromScancodeEXT(Keys.E);
}
```

### TextInputEXT
`public static class Microsoft.Xna.Framework.TextInputEXT` is a new class that partially abstracts text input event handling.

To use the TextInputEXT extension:
```cs
using Microsoft.Xna.Framework.Input;

private void OnTextInput(char c)
{
	if (c == (char) 22)
	{
		System.Console.WriteLine("PASTED: " + SDL3.SDL.SDL_GetClipboardText());
	}
	System.Console.WriteLine("TEXT ENTERED: " + c.ToString());
}

public void StartTextInput()
{
	TextInputEXT.TextInput += OnTextInput;
	TextInputEXT.StartTextInput();
}

public void StopTextInput()
{
	TextInputEXT.StopTextInput();
	TextInputEXT.TextInput -= OnTextInput;
}
```

In addition to standard text input, the TextInput event can push one of a series of symbols to represent various text input actions:

```text
(char) 2 - Home
(char) 3 - End
(char) 8 - Backspace
(char) 9 - Tab
(char) 13 - Enter
(char) 127 - Delete
(char) 22 - Control+V (Paste operator)
```

### ClickedEXT
`public static Action<int> ClickedEXT` is a new event for Mouse that allows you to receive notifications when a mouse button is clicked. One of the main flaws of the XNA Mouse API is that you are only able to get the "current" state of the mouse buttons, but mouse button input is unique due to the various ways input events can be sent. Consider the following code:

```cs
// Accessible by the rest of the engine
public bool ButtonDown { get; private set; }

// Somewhere in the event loop...
if (evt.type == MOUSEBUTTONDOWN)
{
    ButtonDown = true;
}
else if (evt.type == MOUSEBUTTONUP)
{
    ButtonDown = false;
}

// Somewhere in the game...
if (Input.ButtonDown)
{
    // Stuff!
}
```

For mouse input in particular it is surprisingly common for both a button down and a button up event to show up in a single frame, so when you poll for the "current" state, you miss out on at least one input change, leading to dropped input. This is far more noticeable when using a mouse that uses tap/touch events to send mouse buttons, rather than a physical button (for example, laptop trackpads).

ClickedEXT will send a button index each time a mouse button is clicked:

```cs
// Reset this array at the end of each frame!
private ButtonState[] mouseClicks = new ButtonState[5];
private void OnClicked(int button)
{
    mouseClicks[button] = ButtonState.Pressed;
}
```

This should be combined with the standard XNA Mouse API to get a fully accurate mouse button state:

|     ClickedEXT     |      GetState      |    Final State    |
|:------------------:|:------------------:|:-----------------:|
| ✔️                  | ✔️                  | Clicked, Pressing |
|         ❌         |         ❌         |      Released     |
| ✔️                  |         ❌         | Clicked, Released |
|         ❌         | ✔️                  |      Pressing     |

The following is a basic example of ClickedEXT combined with GetState:

```cs
// Using the code above...

// ButtonState doesn't tell us enough, let's make our own!
public enum InputState
{
    Released,
    Pressing,
    Clicked
}
public InputState LeftMouseButton { get; private set; }

// GetState storage
MouseState mPrevState, mState = new MouseState();

public void UpdateInput()
{
    mPrevState = mState;
    mState = Mouse.GetState();

    // Easiest route is to 'or' the click with the current state
    ButtonState leftButton = mState.LeftButton | mouseClicks[0];

    if (leftButton == ButtonState.Released)
    {
        LeftMouseButton = InputState.Released;
    }
    else if (mPrevState.LeftButton == ButtonState.Released)
    {
        LeftMouseButton = InputState.Clicked;
    }
    else
    {
        LeftMouseButton = InputState.Pressing;
    }

    // Remember to clear your click storage each frame!
    mouseClicks[0] = ButtonState.Released;
}
```

Note that ClickedEXT can send multiple clicks for a single button if it is in fact clicked multiple times (for example, if the game hitches briefly and gives the user time to click repeatedly in one frame).

### IsRelativeMouseModeEXT
`public static bool IsRelativeMouseModeEXT` is a new property for `Mouse` that allows you to change the behavior of `Mouse.GetState()` to get relative (rather than absolute) X/Y coordinates. As part of this, it also automatically locks the mouse to the window. This is particularly helpful for first-person games, where you would otherwise have to constantly call `Mouse.SetPosition()` to keep the mouse in place.

To use the IsRelativeMouseModeEXT extension:
```cs
public static bool EnableMouseCapture()
{
    /* PSA: Please make mouse capture an option
     * in your settings menu if it's not required!
     * Also, maybe disable this when !IsActive.
     */
    Mouse.IsRelativeMouseModeEXT = true;
}

public static void UpdateMouseInput()
{
    Vector2 cursorChange;
    CurrentMouseState = Mouse.GetState();
#if FNA
    // Yup, that's it!
    cursorChange = new Vector2(CurrentMouseState.X, CurrentMouseState.Y);
#else
    // I don't feel so good...
    cursorChange = new Vector2(
        CurrentMouseState.X - screenCenter.X,
        CurrentMouseState.Y - screenCenter.Y
    );
    Mouse.SetPosition(screenCenter.X, screenCenter.Y); // BARF
#endif
}
```

### SubmitFloatBufferEXT
`public void SubmitFloatBufferEXT(float[] buffer)` is a new method for `DynamicSoundEffectInstance` that allows you to directly submit float samples to the source, without having to convert to signed 16-bit PCM data. Typically this is used to directly stream float samples that are often provided by default in audio decoders.

This function is used exactly as the official `SubmitBuffer` method is used in XNA.

### TextureDataFromStreamEXT
`public static void TextureDataFromStreamEXT(Stream stream, out int width, out int height, out byte[] pixels)` is a new method for Texture2D that functions similarly to `Texture2D.FromStream()`, except that it does not generate a Texture2D instance. This is a convenience function that will only decode the image data for you.

### DDSFromStreamEXT
`public static Texture DDSFromStreamEXT(GraphicsDevice graphicsDevice, Stream stream)` is a new method for Texture2D/TextureCube that functions similarly to `Texture2D.FromStream()`, but is specialized for loading DDS texture files. Currently this extension supports the Dxt1, Dxt3, Dxt5, ColorBgraEXT, and Dxt5SrgbEXT formats.

This extension also allows loading DDS image files via the `Content.Load<Texture2D/TextureCube>()` methods.

### GetFormatSizeEXT, GetBlockSizeSquaredEXT
Two new methods for `Texture` are exposed to help with allocating texture staging memory.

The `Texture.GetBlockSizeSquaredEXT` method allows querying the number of pixels in a block for compressed textures. You can use this in order to round sizes up/down to the nearest block, or to calculate how large a buffer needs to be for a compressed texture of a given size. Most compressed texture formats have a block size of `4` (and thus a `BlockSizeSquared` of `16`), but not all do. For formats that are not compressed, this method returns `1`.

The `Texture.GetFormatSizeEXT` method allows querying the number of bytes in a single block (or texel, for formats where the block size is `1`). For the `Color` format, for example, this is `4`.

By combining these two methods you can calculate the appropriate buffer size for any arbitrary GetData call, like so:
```csharp
var elementSize = Texture.GetFormatSizeEXT(texture.Format);
var blockSizeSquared = Texture.GetBlockSizeSquaredEXT(texture.Format);
var buffer = new byte[texture.Width * texture.Height * elementSize / blockSizeSquared];
texture.GetData(buffer);
```

### SetDataPointerEXT
`public void SetDataPointerEXT` is a new method for Buffer and Texture objects that lets you send data directly via IntPtrs rather than arrays. This is ideal for scenarios where you know your input data is valid and want to skip all the managed validation/marshaling that happens with the normal SetData functions.

Be warned: When I say this skips all the validation, I mean it! This will crash if you don't know what you're doing!

### PointListEXT
`PointListEXT` is an additional enum value for PrimitiveType that allows for point rendering.

While FNA supports rendering point vertex data, note that the feature is very strict - we do not expose _any_ other aspect of point rendering that existed in XNA3 and older graphics APIs. Point sprites are always enabled, and you are expected to specify point size and manage min/max sizes either in your vertex data or vertex shader, whichever is more appropriate for your application.

### GetRenderTargetsNoAllocEXT
`public int GetRenderTargetsNoAllocEXT(RenderTargetBinding[] output)` is a new method that acts similarly to GetRenderTargets, but allows for avoiding allocating a new array for each call:

```cs
RenderTargetBinding[] oldTargets = new RenderTargetBinding[0]; // Declared somewhere else, I hope...
int oldTargetCount = currentDevice.GetRenderTargetsNoAllocEXT(null);
Array.Resize(ref oldTargets, oldTargetCount);
currentDevice.GetRenderTargetsNoAllocEXT(oldTargets);
```

### SetAudioTrackEXT, SetVideoTrackEXT
`public void SetAudioTrackEXT(int track)` and `public void SetVideoTrackEXT(int track)` are new methods for Video that enable support for multiple tracks in a single video file. This is useful for supporting multiple languages without having to duplicate video data.

Note that this function may have some delay when set mid-stream, as the decoder may need to complete its current audio packet before moving to the new track. To ensure a 100% clean track, call this before calling `VideoPlayer.Play()`!

Currently, you are only allowed to change audio tracks mid-stream when the channel count and sample rate match the previous track. Changing to a track with a different wave format is unsupported unless done before playback begins.

Video tracks must match in width, height, and YUV format. Mixing image formats in a single file is unsupported.

### FromUriEXT
This is a new method for `Video` that works exactly as `Song` does - see the Song API for details!

### FingerIdEXT
`public int GestureSample.FingerIdEXT` and `public int GestureSample.FingerId2EXT` are new properties for GestureSample that allow you to determine exactly which touch finger(s) were involved in the gesture. 

Note that `FingerId2EXT` is only used for `Pinch` and `PinchComplete` gestures, and it will have a value of -1 for any other gesture type.

The following is a basic example of using these properties:
```cs
// Keep track of which fingers were used for these gestures
private int DragFinger = -1;
private int PinchFinger1 = -1;
private int PinchFinger2 = -1;

// Somewhere later...

while (TouchPanel.IsGestureAvailable)
{
    GestureSample gesture = TouchPanel.ReadGesture();
    if (gesture.GestureType == GestureType.Drag)
    {
        DragFinger = gesture.FingerIdEXT;
    }
    else if (gesture.GestureType == GestureType.Pinch)
    {
        PinchFinger1 = gesture.FingerIdEXT;
        PinchFinger2 = gesture.FingerId2EXT;
    }
}
```
