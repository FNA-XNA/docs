# Appendix C: FNA on Google Stadia

**UPDATE (September 29, 2022):** Google has [announced](https://blog.google/products/stadia/message-on-stadia-streaming-strategy/) that they are transitioning Stadia to a white-label product, so this platform's support is now only for licensees of the Stadia technology. Stadia as a standalone platform will be discontinued in 2023.

***

As of FNA 19.12, if you can make a Linux version, you can make a Stadia version. No, really! FNA for Stadia, like all the other platforms, uses SDL for all platform code; the master branch you see on GitHub is exactly what ships on Stadia. In fact, aside from SDL, all of the fnalibs binaries are byte-for-byte identical between Linux and Stadia.

## Graphics Support

FNA uses ANGLE's Vulkan renderer for graphics support on Stadia. This means that you will want your graphics data to be compliant with OpenGL ES 3.0 for the best result (for example, don't use ColorBgraEXT). When our Vulkan renderer is made the default this will no longer be necessary.

## Filesystem Portability (Again)

As usual, be sure your loading/saving code is portable, [as described earlier](../4:-FNA-and-Windows-API.md#filesystem-portability) in this documentation!

## Anything Else?

Aside from Stadia being Linux with Vulkan, PulseAudio, glibc, and libc++, there's not much else that can be discussed here. If you are a licensed Stadia developer, please get in touch with the admins in the [FNA Discord server](https://discord.gg/2Gg8zju).