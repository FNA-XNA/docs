# Appendix A: NativeAOT on PC

FNA now has support for [NativeAOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/), a new .NET toolchain which allows you to build your game into an ahead-of-time compiled native executable.

## Project Setup

To get started, please read through the [official NativeAOT documentation](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/). Make sure to install the prerequisites listed for your OS.

To make a NativeAOT build, you should make .NET 8 project files for your game - instead of the usual FNA.csproj, you will reference FNA.Core.csproj. The code and content should largely be able to stay the same, with the exception of code that requires a JIT (i.e. you can't emit IL at runtime, as you might expect from ahead-of-time compilation).

To make your .csproj compatible with NativeAOT, add the following:
```
  <PropertyGroup>
    <PublishAot>true</PublishAot>
  </PropertyGroup>

  <ItemGroup>
    <RdXmlFile Include="rd.xml" />
  </ItemGroup>

  <ItemGroup Condition="'$(OS)' != 'Windows_NT'">
    <NativeLibrary Include="-lSDL3" />
    <NativeLibrary Include="-lFNA3D" />
    <NativeLibrary Include="-lFAudio" />
    <NativeLibrary Include="-ltheorafile" />
  </ItemGroup>

  <ItemGroup Condition="'$(OS)' == 'Windows_NT'">
    <NativeLibrary Include="SDL3.lib" />
    <NativeLibrary Include="FNA3D.lib" />
    <NativeLibrary Include="FAudio.lib" />
    <NativeLibrary Include="libtheorafile.lib" />
  </ItemGroup>

  <ItemGroup>
    <DirectPInvoke Include="SDL3" />
    <DirectPInvoke Include="FNA3D" />
    <DirectPInvoke Include="FAudio" />
    <DirectPInvoke Include="libtheorafile" />
  </ItemGroup>
```

You will also need to add an "rd.xml" file to your project directory. This will be explained in the next sections.

## AOT Type Preservation

The rd.xml file informs the compiler of any types it should preserve during the linking stage. This is most often used to preserve types that are only accessed via reflection. You can read [the official doc page](https://github.com/dotnet/runtimelab/blob/feature/NativeAOT/docs/using-nativeaot/rd-xml-format.md) to learn more about it, but here's an example of what rd.xml might look like for a game that uses ContentReader to load a couple of generic types:

```
<Directives>
    <Application>
        <Assembly Name="FNA">
          <Type Name="Microsoft.Xna.Framework.Content.ListReader`1[[System.Char,mscorlib]]" Dynamic="Required All" />
          <Type Name="Microsoft.Xna.Framework.Content.ArrayReader`1[[Microsoft.Xna.Framework.Vector3,FNA]]" Dynamic="Required All" />
        </Assembly>
        <Assembly Name="mscorlib" />
    </Application>
</Directives>
```

## Native Libraries

Even though we're AOT-compiling the project, we still recommend dynamically linking the native libraries rather than statically linking them. Note that this does _not_ mean dynamic loading; the NativeLibrary and DirectPInvoke items in the .csproj ensure that native calls are inlined directly into the executable, rather than requiring a `dlopen` call to load the library at runtime. This is more performant and reliable than the standard .NET PInvoke system, and is only possible with AOT.

Finally, to actually link the fnalibs, we need to set up the build environment appropriately:

### Windows

- Download the MSVC development build of SDL3, then use it to build the other libraries from source.
- Grab the .lib files from SDL3, FNA3D, FAudio, and Theorafile and place them in your app's .csproj directory.
- Build the application.
- Copy the contents of `fnalibs/x64` into the generated output directory.

### Linux

Linux builds are best done in a Steam Linux Runtime container, which can be done both locally and via CI (including GitHub Actions).

```sh
# Set up image for the first time
distrobox create -i registry.gitlab.steamos.cloud/steamrt/sniper/sdk sniper

# Start up the container. It's as easy as that!
distrobox enter sniper
```

Inside the container you will want to install the .NET SDK and the fnalibs:

```sh
wget https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update && sudo apt-get install -y dotnet-sdk-8.0
rm packages-microsoft-prod.deb
```

```sh
git clone --recursive https://github.com/FNA-XNA/FAudio.git
git clone --recursive https://github.com/FNA-XNA/FNA3D.git
git clone --recursive https://github.com/FNA-XNA/Theorafile.git

cd FAudio
cmake -B release -G Ninja . -DCMAKE_BUILD_TYPE=Release
sudo ninja -C release install

cd ../FNA3D
cmake -B release -G Ninja . -DCMAKE_BUILD_TYPE=Release
sudo ninja -C release install

cd ../Theorafile
make
sudo cp libtheorafile.so /usr/local/lib/libtheorafile.so
```

Once the above is complete, `dotnet publish` should build and link a native Linux build. Copy the fnalibs next to the executable and ship!

```sh
cp /usr/local/lib/libFAudio.so.0 bin/Release/net8.0/linux-x64/publish/
cp /usr/local/lib/libFNA3D.so.0 bin/Release/net8.0/linux-x64/publish/
cp /usr/local/lib/libtheorafile.so bin/Release/net8.0/linux-x64/publish/
```

Note that these instructions do not copy SDL3; this is already provided by the Sniper runtime so bundling SDL is optional. If you need to bundle it anyway, be sure to use the binaries provided by fnalibs-dailies instead!

You can see an example of an automated CI build [here](https://github.com/flibitijibibo/RogueLegacy1/blob/main/.github/workflows/ci.yml).
