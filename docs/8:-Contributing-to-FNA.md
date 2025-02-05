# 8: Contributing to FNA

This is a rough guide for contributing to FNA. This should NOT be considered the be-all-end-all rulebook for FNA development, but it contains some basic examples of how the project is written and developed. In general, Rule #0 is simply to use your best judgement based on existing traditions in the code and its libraries.

## Forks
GitHub makes forking as easy as clicking a button. Here's when to click that button:

- When you're about to commit a patch

Do NOT clutter up the [network](https://github.com/FNA-XNA/FNA/network) with pointless forks! Either write something or just use our repository. We promise that the clone that you have locally on your machine will never be forcibly updated by us unless you explicitly type `git pull` (that's how Git works!), and we will always have stable releases [archived](https://fna.flibitijibibo.com/archive/). The network graph is critical for tracking developers' changes, if too many forks exist then the feature is forcibly turned off by GitHub!

## Code Style
* Line Endings: Unix newlines ('\n'), NOT Win32 newlines ('\r\n')!
* Tabs: Actual '\t' tabs, NOT SPACES!
* Blank Lines: NO TRAILING SPACE! Blank lines should be, you know, blank.
* Characters per line: ~100. When a line gets too long, start splitting it up into multiple lines. The number should only be considered a maximum value; if you find that you're doing a lot of extensive left-to-right reading rather than top-to-bottom, start splitting the lines up.
* `i++/i--`: Do `i += 1` and `i -= 1` instead unless the increment is genuinely being used to its advantage. If it's that painful to do this, consider `foreach` instead.
* Do NOT use `var`! Use the actual type name!
* `someMethod()` rather than `someMethod ()`, `someArray[x]` rather than `someArray [x]`, etc.
* `(Type) cast` rather than `(Type)cast`.
* Use braces everywhere! I don't care if the if block is one line, braces! Use them!
* Single line comments: `// This code is derp` rather than `//this code is derp`.
* Multi-line comments: Use `/* */` blocks, rather than multiple lines of `//`.

## External Libraries
As noted in the [download documentation](index.md), we use several third-party libraries to support FNA.

A major rule for FNA is that we do NOT fork external libraries! Anything we change must be submitted to upstream, and must be of high enough quality to be merged. FNA's role is solely to reimplement XNA, nothing further. Any issue exhibited by the libraries _must be fixed in the library_ for the benefit of the greater software ecosystem.

The sources can be found here:

- [SDL](https://github.com/libsdl-org/SDL)
- [FNA3D](https://github.com/FNA-XNA/FNA3D)
- [FAudio](https://github.com/FNA-XNA/FAudio)
- [Theorafile](https://github.com/FNA-XNA/Theorafile)

All libraries are built with the default settings.
