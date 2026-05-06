# Appendix G: WSL Support

FNA games can be run via the [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/about), easing the development and testing process for native Linux games on Windows.

## Installing

The installation process for an FNA-compatible environment is as follows:

1. Open PowerShell
2. Enter `wsl --install FedoraLinux-44`. (If it says something about rebooting, reboot then enter this command again.)
3. Create a username for the Linux container, it can be anything you want
4. Install the minimum requirements for GUI/audio support by entering `sudo dnf install SDL3-devel`
5. You should now be able to copy Linux builds to your container's home directory, copy and run!

After these steps are complete, you only need to enter `wsl` in PowerShell to enter your container.

## Caveats

There is only one remaining caveat for testing, which is that the D3D12 swapchain on Linux is passed to windows via host memory - essentially, it's going from GPU -> CPU -> GPU. This is incredibly slow, especially at high resolutions, so for testing purposes you may want to use a smaller window size (i.e. 1280x720) to avoid Windows-bound performance issues.

As for _building_, note that this environment is _not_ suitable for compiling native binaries as it uses a glibc version higher than our standard OS minimum requirement. Instead, you are strongly encouraged to use the build OS listed in Appendix F.

This issue is being tracked here: https://github.com/microsoft/wslg/issues/387
