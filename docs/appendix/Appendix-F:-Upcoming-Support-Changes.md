# Appendix F: Upcoming Support Changes

While very rare, FNA does occasionally make changes to its support matrix, usually to migrate existing platforms to new system requirements or development environments. In extremely rare cases, certain features may even be removed from FNA. This page is meant to track the more extreme cases, so that developers can plan their own products' development accordingly.

## glibc Support Calendar

As of April 1, 2023, FNA requires glibc 2.28 or newer. The fnalibs build OS is Rocky Linux 8.

Upon the release of SDL 3.0 (or April 1, 2025, whichever is sooner), glibc 2.31 will be required. The fnalibs build OS will be [Steam Linux Runtime 3.0 (Sniper)](https://gitlab.steamos.cloud/steamrt/sniper/sdk).

## macOS Support (Single-Assembly)

Apple has distanced themselves (and the Mac) significantly from the traditional PC - as of today "PC" is defined as desktop computers running Windows and Linux, as well as Intel Macs running Mojave or older. As part of this, Apple have put significant work into unifying macOS, iOS, and tvOS as part of the Mac's transition to Apple Silicon CPUs. Additionally, the codesign/notarization requirements have increased substantially, meaning the development model used for Linux and Windows will no longer be practical for macOS.

Upon the release of SDL 3.0 (or April 1, 2025, whichever is sooner), fnalibs and MonoKickstart will no longer provide binaries for macOS, and support/development will be migrated to the same process as iOS and tvOS. The downside is that single-assembly portability support will likely no longer include macOS, but the upside is that if you can manage to target one Apple OS, you will likely be able to target them all at once (provided you can navigate your way through Apple's usual quirks and demands).

## Vulkan Support

The FNA team has completed its work on SDL_GPU, which is now included in the upcoming SDL 3.0 release. As a result, the Vulkan renderer has been removed in favor of a new SDL_GPU renderer for FNA3D, which allows supporting Vulkan/D3D12/Metal with a single FNA3D driver. This change will go into effect for FNA 25.01.
