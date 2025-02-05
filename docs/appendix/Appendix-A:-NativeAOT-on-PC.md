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
    <DirectPInvokeList Include="SDLApis.txt" />
    <DirectPInvoke Include="FNA3D" />
    <DirectPInvoke Include="FAudio" />
    <DirectPInvoke Include="libtheorafile" />
  </ItemGroup>
```

You will also need to add two more files to your project directory: rd.xml and SDLApis.txt. These will be explained in the next sections.

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

Finally, to actually link the fnalibs, follow these platform-specific instructions:

* **Windows:**
    * Download the MSVC development build of SDL3, then use it to build the other libraries from source.
    * Grab the .lib files from SDL3, FNA3D, FAudio, and Theorafile and place them in your app's .csproj directory.
    * Build the application.
    * Copy the contents of `fnalibs/x64` into the generated output directory.
* **Linux:**
    * NOTE: For maximum compatibility, we recommend you build using a distro with a low glibc version, like Rocky Linux 8.
    * Build SDL3 from source or install the SDL3 development package from a package manager, then use it to build FNA3D, FAudio, and Theorafile from source.
    * Copy all the resulting \*.so files into your LD_LIBRARY_PATH (e.g. `/usr/local/lib64`). Make sure the symversioning is preserved during the copy!
    * Build the application.
    * Copy the contents of `fnalibs/lib64` into the generated output directory.
