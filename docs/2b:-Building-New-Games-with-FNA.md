# 2b: Building New Games with FNA

## Before You Start

This is strictly a tutorial about using FNA. It is NOT a C# tutorial! If you are learning C# for the first time, use Microsoft's official [Introduction to C#](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/intro-to-csharp/) on MSDN first before continuing on.

Also, be sure to finish [Page 1](1:-Setting-Up-FNA.md) before starting this page!

## What is XNA?

XNA was, at its core, the software equivalent of an 80's Saturday morning cartoon based on a toy line: A massive advertisement masquerading as a real product. It was built to advertise many new (at the time) products in development at Microsoft:

- C# 2.0
- Direct3D Effects Framework
- XACT Audio Creation Tool
- XInput and the Xbox 360 Controller
- Xbox 360 + Windows Media Center

From 2006 to 2010, Microsoft maintained XNA as a means of allowing independent game developers to ship small games written in C# on Xbox 360, via the "Xbox Live Indie Games" marketplace. The final XNA release also supported building for Windows Phone 7 devices.

As for the XNA API, it was largely a C# wrapper for various DirectX components, but not quite all of them - many features are unavailable in favor of the aforementioned new-fangled DirectX products. For example, while there is a `GraphicsDevice` class that effectively acts as a 1:1 map of `ID3D10Device`, notably missing is support for low-level shaders and constant buffers; instead you are expected to use Effects for shader support.

XNA was officially discontinued in 2012, and the Xbox Live Indie Games marketplace was shut down on November 2017.

## What is FNA?

[FNA](http://fna-xna.github.io) is a preservation project designed to accurately reimplement the XNA runtime libraries. When you have an XNA game, you should be able to take the source, compile it against FNA, and have a fully-functioning port. At its core, FNA is a portability library, but many continue to develop new games with FNA. This tutorial will help you make your own FNA games, without needing XNA as a prerequisite. If you are bringing an existing XNA game to FNA, follow [this wiki page](2a:-Building-XNA-Games-with-FNA.md) instead.

## Your First Game

See [Page 1](1:-Setting-Up-FNA.md#chapter-5-creating-new-projects) for a quick refresher on making new projects. Once you have a project made, you can then proceed:

## The First Program

This is the smallest possible program using the framework portion of XNA:
```cs
using System;
using Microsoft.Xna.Framework;

static class Program
{
	[STAThread]
	static void Main(string[] args)
	{
		using (Game g = new Game())
		{
			new GraphicsDeviceManager(g);
			g.Run();
		}
	}
}
```

This should compile into a folder like `bin/Debug/`. Next to your executable, you will put the native libraries you downloaded earlier into this folder. You only need to worry about the libraries for your development platform; the rest will be for when you [distribute your game](3:-Distributing-FNA-Games.md). For example, if you're building an AnyCPU program on Windows x64, you would take the contents of the native library archive's `x64` folder and put them next to your exe.

When using a developer environment on macOS, you will want to add an environment variable that sets `DYLD_LIBRARY_PATH=./osx/` (or wherever your dylib files are), so that the IDE's runtime environment will find the fnalibs binaries.

When running this program, you might see some random trash in the window; that is most likely old graphics memory from another program you were running. Aside from that, the game is fully functional; it is reading input, running updates, and presenting frames to the window. But if this is the whole program, where do we put the rest of the game?

## The First Game Object

The trick is that you're not going to create a `Game` directly. Instead, you're going to inherit it!

```cs
using System;
using Microsoft.Xna.Framework;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private FNAGame()
	{
		// This gets assigned to something internally, don't worry...
		new GraphicsDeviceManager(this);
	}
}
```

But again, there's still no place to put the game. That's because `Game` has several `protected` methods that you are meant to implement. Here's what it looks like with the most commonly-used methods:

```cs
using System;
using Microsoft.Xna.Framework;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private FNAGame()
	{
		GraphicsDeviceManager gdm = new GraphicsDeviceManager(this);

		// Typically you would load a config here...
		gdm.PreferredBackBufferWidth = 1280;
		gdm.PreferredBackBufferHeight = 720;
		gdm.IsFullScreen = false;
		gdm.SynchronizeWithVerticalRetrace = true;
	}

	protected override void Initialize()
	{
		/* This is a nice place to start up the engine, after
		 * loading configuration stuff in the constructor
		 */
		base.Initialize();
	}

	protected override void LoadContent()
	{
		// Load textures, sounds, and so on in here...
		base.LoadContent();
	}

	protected override void UnloadContent()
	{
		// Clean up after yourself!
		base.UnloadContent();
	}

	protected override void Update(GameTime gameTime)
	{
		// Run game logic in here. Do NOT render anything here!
		base.Update(gameTime);
	}

	protected override void Draw(GameTime gameTime)
	{
		// Render stuff in here. Do NOT run game logic in here!
		GraphicsDevice.Clear(Color.CornflowerBlue);
		base.Draw(gameTime);
	}
}
```

## The First Input

It's not a game without input, right? FNA exposes `GamePad`, `Keyboard`, and `Mouse` for user input. There is also a `Microsoft.Xna.Framework.Input.Touch` namespace for touch screen support.

Input isn't terribly complicated; you store two copies of input state, one for current input and another for previous input. This lets you detect presses and releases, in addition to just checking for a button being down:

```cs
using System;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Input;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private KeyboardState keyboardPrev = new KeyboardState();
	private MouseState mousePrev = new MouseState();
	private GamePadState gpPrev = new GamePadState();

	private FNAGame()
	{
		new GraphicsDeviceManager(this);
	}

	protected override void Update(GameTime gameTime)
	{
		// Poll input
		KeyboardState keyboardCur = Keyboard.GetState();
		MouseState mouseCur = Mouse.GetState();
		GamePadState gpCur = GamePad.GetState(PlayerIndex.One);

		// Check for presses
		if (keyboardCur.IsKeyDown(Keys.Space) && keyboardPrev.IsKeyUp(Keys.Space))
		{
			System.Console.WriteLine("Space bar was pressed!");
		}
		if (mouseCur.RightButton == ButtonState.Released && mousePrev.RightButton == ButtonState.Pressed)
		{
			System.Console.WriteLine("Right mouse button was released!");
		}
		if (gpCur.Buttons.A == ButtonState.Pressed && gpPrev.Buttons.A == ButtonState.Pressed)
		{
			System.Console.WriteLine("A button is being held!");
		}

		// Current is now previous!
		keyboardPrev = keyboardCur;
		mousePrev = mouseCur;
		gpPrev = gpCur;

		base.Update(gameTime);
	}

	protected override void Draw(GameTime gameTime)
	{
		GraphicsDevice.Clear(Color.CornflowerBlue);
		base.Draw(gameTime);
	}
}
```

Be sure to read all of the input APIs for more details! You may also be interested in some [extensions to the XNA spec](5:-FNA-Extensions.md) that improve input support in FNA.

## The First Sprite

Finally, some graphics! In XNA, there is a class called `SpriteBatch` that makes sprite drawing relatively easy. Combine that with your own textures and you have the foundation of a 2D renderer.

This sample loads a PNG named "FNATexture", located in a "Content" folder, and renders it with a SpriteBatch:

```cs
using System;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private SpriteBatch batch;
	private Texture2D texture;

	private FNAGame()
	{
		new GraphicsDeviceManager(this);

		// All content loaded will be in a "Content" folder
		Content.RootDirectory = "Content";
	}

	protected override void LoadContent()
	{
		// Create the batch...
		batch = new SpriteBatch(GraphicsDevice);

		// ... then load a texture from ./Content/FNATexture.png
		texture = Content.Load<Texture2D>("FNATexture");
	}

	protected override void UnloadContent()
	{
		batch.Dispose();
		texture.Dispose();
	}

	protected override void Draw(GameTime gameTime)
	{
		GraphicsDevice.Clear(Color.CornflowerBlue);

		// Draw the texture to the corner of the screen
		batch.Begin();
		batch.Draw(texture, Vector2.Zero, Color.White);
		batch.End();

		base.Draw(gameTime);
	}
}
```

If all went well, the PNG you chose should now be displayed! When drawing sprites, be absolutely sure that you draw as many as you possibly can before calling `End()`; batches are meant to be large, singular groups rather than lots of small, fragmented groups. The more you put in a single batch, the better your program will perform.

And whatever you do, do NOT use `SpriteSortMode.Immediate`!

## The First Sound

In addition to XACT, there is also a `SoundEffect` API available. It's as simple as loading a .wav file and mashing `Play()`:


```cs
using System;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Audio;
using Microsoft.Xna.Framework.Input;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private SoundEffect sound;
	private KeyboardState keyboardPrev = new KeyboardState();

	private FNAGame()
	{
		new GraphicsDeviceManager(this);

		// All content loaded will be in a "Content" folder
		Content.RootDirectory = "Content";
	}

	protected override void LoadContent()
	{
		// Sound is ./Content/FNASound.wav
		sound = Content.Load<SoundEffect>("FNASound");
	}

	protected override void UnloadContent()
	{
		sound.Dispose();
	}

	protected override void Update(GameTime gameTime)
	{
		KeyboardState keyboardCur = Keyboard.GetState();

		if (keyboardCur.IsKeyDown(Keys.Space) && keyboardPrev.IsKeyUp(Keys.Space))
		{
			sound.Play();
		}

		keyboardPrev = keyboardCur;
	}

	protected override void Draw(GameTime gameTime)
	{
		GraphicsDevice.Clear(Color.CornflowerBlue);
		base.Draw(gameTime);
	}
}
```

There is lots of deeper functionality, including instance management, 3D audio APIs, and even a streaming sound object, useful for streaming from larger files (for example, sending decoded data from an Ogg Vorbis music file).

## The First Song

XNA includes a `Media` namespace, which includes support for basic playback of music and video files. FNA supports Ogg Vorbis and QOA for the `Song` implementation:

```cs
using System;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Media;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private Song song;

	private FNAGame()
	{
		new GraphicsDeviceManager(this);

		// All content loaded will be in a "Content" folder
		Content.RootDirectory = "Content";
	}

	protected override void LoadContent()
	{
		// Song is ./Content/FNASong.ogg
		song = Content.Load<Song>("FNASong");
	}

	protected override void UnloadContent()
	{
		song.Dispose();
	}

	protected override void Update(GameTime gameTime)
	{
		// Just keep playing the song over and over
		if (MediaPlayer.State == MediaState.Stopped)
		{
			MediaPlayer.Play(song);
		}
		base.Update(gameTime);
	}

	protected override void Draw(GameTime gameTime)
	{
		GraphicsDevice.Clear(Color.CornflowerBlue);
		base.Draw(gameTime);
	}
}
```

## The First Video

`Video` objects are a fair bit more involved than `Song`. In addition to playing the sound, a `VideoPlayer` will provide the frames in the form of a texture, which you can then render however you like:


```cs
using System;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Graphics;

class FNAGame : Game
{
	[STAThread]
	static void Main(string[] args)
	{
		using (FNAGame g = new FNAGame())
		{
			g.Run();
		}
	}

	private GraphicsDeviceManager gdm;
	private Video video;
	private VideoPlayer videoPlayer;
	private SpriteBatch batch;

	private FNAGame()
	{
		gdm = new GraphicsDeviceManager(this);

		// All content loaded will be in a "Content" folder
		Content.RootDirectory = "Content";
	}

	protected override void LoadContent()
	{
		// Video is ./Content/FNAVideo.ogv
		video = Content.Load<Video>("FNAVideo");
		batch = new SpriteBatch(GraphicsDevice);

		gdm.PreferredBackBufferWidth = video.Width;
		gdm.PreferredBackBufferHeight = video.Height;
		gdm.ApplyChanges();

		// Just loop the video over and over
		videoPlayer = new VideoPlayer();
		videoPlayer.IsLooped = true;
		videoPlayer.Play(video);
	}

	protected override void UnloadContent()
	{
		batch.Dispose();
		videoPlayer.Dispose();
		video = null;
	}

	protected override void Draw(GameTime gameTime)
	{
		// Draw the video frame to the window, which should be the same size
		batch.Begin(SpriteSortMode.Deferred, BlendState.Opaque);
		batch.Draw(videoPlayer.GetTexture(), Vector2.Zero, Color.White);
		batch.End();
		base.Draw(gameTime);
	}
}
```

## The First Save

XNA provides two filesystem APIs: `TitleContainer` and `StorageContainer`. `TitleContainer` is how you should open files in the game folder (provided `Content.Load()` does not do what you want), and is pretty much exactly the same as `File.OpenRead`. `StorageContainer` is a lot more involved, however:

```cs
using System;
using Microsoft.Xna.Framework.Storage;

void DoStorageContainerThing()
{
	IAsyncResult result;

	result = StorageDevice.BeginShowSelector(null, null);
	while (!result.IsCompleted)
	{
		// Just hang out for a bit...
		System.Threading.Thread.Sleep(1);
	}
	StorageDevice device = StorageDevice.EndShowSelector(result);

	result = device.BeginOpenContainer("SaveData", null, null);
	while (!result.IsCompleted)
	{
		// Just hang out for a bit...
		System.Threading.Thread.Sleep(1);
	}
	StorageContainer container = device.EndOpenContainer(result);

	// Do stuff!

	// Clean up after yourself! Maybe keep `device` from getting collected.
	container.Dispose();
}
```

From there, `container`'s API is self-explanatory. There are Create/Delete/Exists/Open/GetNames APIs for directories and files. Pretty much what you'd expect!

The container's path is `$SAVELOC/$CONTAINERNAME/$PLAYERINDEX`:

- `$SAVELOC` looks something like this... and before you ask, yes, XNA really based the save folder on the EXE name:
```cs
string platform = SDL.SDL_GetPlatform();
string exeName = Path.GetFileNameWithoutExtension(
	AppDomain.CurrentDomain.FriendlyName
).Replace(".vshost", "");
if (platform.Equals("Windows"))
{
	return Path.Combine(
		Environment.GetFolderPath(
			Environment.SpecialFolder.MyDocuments
		),
		"SavedGames",
		exeName
	);
}
if (platform.Equals("Mac OS X"))
{
	string osConfigDir = Environment.GetEnvironmentVariable("HOME");
	if (String.IsNullOrEmpty(osConfigDir))
	{
		return "."; // Oh well.
	}
	return Path.Combine(
		osConfigDir,
		"Library/Application Support",
		exeName
	);
}
if (	platform.Equals("Linux") ||
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
	return Path.Combine(osConfigDir, exeName);
}
return SDL.SDL_GetPrefPath(null, exeName);
```
- `$CONTAINERNAME` is the name you passed to `BeginOpenContainer`.
- `$PLAYERINDEX` is either `AllPlayers` if you didn't pass a PlayerIndex, or `Player1` through `Player4`.

## The First Effect

NOTE: This is an advanced subject! You may want to read the [official Effects documentation](https://docs.microsoft.com/en-us/windows/win32/direct3d9/dx9-graphics-reference-effects) first.

XNA and FNA use Direct3D Effects for shader support. Effects are groups of HLSL shaders bundled together into one file, which can be executed in separate subgroups called "techniques" and "passes". The XNA API is slightly dumbed down compared to the stock Effects API:

```cs
using Microsoft.Xna.Framework.Graphics;

// Effects can be loaded as content!
Effect effect = Content.Load<Effect>("FNAEffect");

// You can set parameters...
effect.Parameters["MadeUpParameter"].SetValue(0.0f);

// Set techniques...
effect.CurrentTechnique = effect.Techniques["MadeUpTechnique"];

// ... and then render each pass in the technique!
foreach (EffectPass p in effect.CurrentTechnique.Passes)
{
	p.Apply(); // Sets the shaders, passes the parameters
	GraphicsDevice.DrawIndexedPrimitives(...);
}

// Clean up after yourself!
effect.Dispose();
```

XNA has multiple effects built in, for those who just want basic rendering without having to write shaders. Examples include `BasicEffect`, `AlphaTestEffect`, and `EnvironmentMapEffect`.

When writing your own Effects, you must precompile them first. This is done with `FXC`, the Microsoft DirectX Shader Compiler, which you can find in the [DirectX SDK](https://www.microsoft.com/en-us/download/details.aspx?id=6812). To compile .fx files:

```
fxc.exe /T fx_2_0 FNAEffect.fx /Fo FNAEffect.fxb
```

Note that FXC works with Wine, so on Linux and macOS you can still develop shaders by calling `wine fxc.exe`.

To see some examples of .fx files, look at the [stock effects](https://github.com/FNA-XNA/FNA/tree/master/src/Graphics/Effect/StockEffects/HLSL)!

