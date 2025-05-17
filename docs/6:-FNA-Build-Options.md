# 6: FNA Build Options

FNA's implementation of certain features may be configurable at build time - that is, parts of the implementation may be temporarily modified to allow for easier debugging. To enable these options, create a file next to your .sln file called `FNA.Settings.props`, which will look something like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <PropertyGroup>
                <DefineConstants>VERBOSE_PIPELINECACHE;$(DefineConstants)</DefineConstants>
        </PropertyGroup>
</Project>
```

You can also use this file to sneak in other bits and pieces, such as adding SDL2_image to the build (as an aside, be sure to update FNA.dll.config with dllmaps if you actually do this):

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <ItemGroup>
                <Compile Include="lib\SDL2-CS\src\SDL2_image.cs" />
        </ItemGroup>
</Project>
```

The supported build options are listed below!

- [VERBOSE_PIPELINECACHE](#verbose_pipelinecache)
- [CASE_SENSITIVITY_HACK](#case_sensitivity_hack)

***

### VERBOSE_PIPELINECACHE
**Affected file: src/Graphics/PipelineCache.cs**

If you want to debug the PipelineCache to make sure it's interpreting your Effects' render state changes properly, you can enable this and get a bunch of messages logged to FNALoggerEXT.

### CASE_SENSITIVITY_HACK
**Affected file: src/TitleContainer.cs**

On Linux, the file system is case sensitive. This means that unless you really focused on it, there's a good chance that your filenames are not actually accurate! The result: File/DirectoryNotFound.

This is a quick alternative to MONO_IOMAP=all, but the point is that you should NOT depend on either of these two things. PLEASE fix your paths!
