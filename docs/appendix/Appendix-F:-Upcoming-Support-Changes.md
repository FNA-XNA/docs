# Appendix F: Upcoming Support Changes

While very rare, FNA does occasionally make changes to its support matrix, usually to migrate existing platforms to new system requirements or development environments. In extremely rare cases, certain features may even be removed from FNA. This page is meant to track the more extreme cases, so that developers can plan their own products' development accordingly.

## glibc Support Calendar

As of April 1, 2023, FNA requires glibc 2.28 or newer. The fnalibs build OS is RHEL8-based.

On April 1, 2025, glibc 2.34 will be required. The fnalibs build OS will be RHEL9-based.

"RHEL-based" OSes include [RHEL for Developers](https://developers.redhat.com/products/rhel/download), [Rocky Linux](https://rockylinux.org/), and [AlmaLinux](https://almalinux.org/).

## macOS Support (Single-Assembly)

Apple has distanced themselves (and the Mac) significantly from the traditional PC - as of today "PC" is defined as desktop computers running Windows and Linux, as well as Intel Macs running Mojave or older. As part of this, Apple have put significant work into unifying macOS, iOS, and tvOS as part of the Mac's transition to Apple Silicon CPUs. Additionally, the codesign/notarization requirements have increased substantially, meaning the development model used for Linux and Windows will no longer be practical for macOS.

Upon the release of SDL 3.0 (or April 1, 2025, whichever is sooner), fnalibs and MonoKickstart will no longer provide binaries for macOS, and support/development will be migrated to the same process as iOS and tvOS. The downside is that single-assembly portability support will likely no longer include macOS, but the upside is that if you can manage to target one Apple OS, you will likely be able to target them all at once (provided you can navigate your way through Apple's usual quirks and demands).

## FNA3D and SDL_gpu

Ryan Gordon is, with the aid of an Epic Megagrant, developing a [next-gen GPU abstraction layer in SDL](https://www.patreon.com/posts/new-project-top-58563886). SDL_gpu will at minimum be targeting Vulkan, Metal, Direct3D 12, and GNM, covering all of the platforms that FNA supports!

The FNA team is fully committed to supporting and co-developing SDL_gpu with Ryan, and we want it to be how we handle platform graphics in FNA3D going forward. With this in mind, it's worth looking at our existing renderers and where they fit in this plan:

### Vulkan

Our Vulkan renderer is now stable, and it performs very well on the hardware that supports it. That said, it is an _immense_ support burden and requires a team of very experienced developers to maintain.

We are using FNA3D Vulkan as the basis for our input in SDL_gpu's design - we now have a very good picture of what is useful to us in explicit graphics APIs, and we know what a good app implementation of Vulkan looks like. The short version: It's a _lot_ of work, but there's very little that can't be done in a shared library like SDL.

The goal is to make it so that we can write an SDL_gpu renderer that is as good (or better!) than the direct Vulkan implementation we have now, so that Vulkan development can be done within SDL instead. With this in mind, it is likely we will remove the Vulkan renderer when SDL_gpu is done, but in exchange SDL_gpu will be made the default renderer on all platforms immediately.

In addition to being easier to maintain (there are a lot more SDL developers than FNA3D developers!), this also allows us to target Metal directly, so MoltenVK will no longer be necessary on Apple platforms.

### D3D11

This is less likely to be removed than Vulkan, but it is still worth looking at where D3D11 fits into FNA's support matrix:

- Windows 11 and GDKX _require_ Direct3D 12 hardware
- Windows 8.1 and 10 will be fully EOL'd by the time SDL 3.0 ships
- Windows 7 will have been EOL'd for almost half a decade!
- Starting January 1, 2024, Windows 8.1 and older will no longer be supported by the Steam client (and likely other CEF-based game launchers)

And that's just the software side - on the hardware side, hardware that only supports the D3D11 runtime has become exceedingly rare, and it will only get rarer as the years go on. To put it in perspective: All hardware released in the last 10 years has Vulkan support of some kind!

Lastly, it has been proposed that SDL_gpu will have a D3D11 backend. This isn't set in stone like D3D12 is, but if it does happen it will add to the numerous reasons to remove the D3D11 renderer in FNA3D.

### OpenGL

This is by _far_ the least likely to be removed; it has the broadest support, is highly stable, continues to be competitive in performance even compared to Vulkan, and the team is extremely familiar with the implementation in FNA3D. Additionally, it is extremely unlikely that SDL_gpu will have OpenGL support, as the feature set required to make a good backend overlaps completely with Vulkan support, making the backend entirely redundant.

Most importantly, OpenGL is the only renderer that supports full XNA preservation as of writing. It is the only renderer that has full support for the pipeline linkage that XNA allowed: You can have vertex input layouts that do not match the vertex shader input, and the pixel shader input layout does not have to match the vertex shader output layout. Our renderers handle the latter category, but only OpenGL supports the former. That's not to say that it's _impossible_ for the other renderers to support this, it is just very difficult to do so. (You ever written a shader linker before? It's no fun.)

However, it _could_ be possible to add such support into SDL_gpu. If we are able to support flexible vertex/pixel shader input layouts in the SDL_gpu shader linker, OpenGL will no longer have any benefits other than broad support, which like D3D11 will only become less true over time.

If SDL_gpu ends up being the only renderer left, we will remove the FNA3D_Driver system and optimize FNA3D's architecture to be an XNA-friendly frontend to SDL_gpu, and _all_ platform graphics work will happen in SDL (and MojoShader, for optimized shader compilation) from that point onward.