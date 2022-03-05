# mach/system-sdk, more libraries for cross-compilation with Zig <a href="https://hexops.com"><img align="right" alt="Hexops logo" src="https://raw.githubusercontent.com/hexops/media/master/readme.svg"></img></a>

* Updated DirectX 12 headers for use with MinGW/Zig
* Cross-compile DirectX apps targetting Windows
* Cross compile Metal apps targetting macOS/iOS Intel or Apple Silicon.
* Cross compile OpenGL/Vulkan apps targetting Linux (not other OSs, sorry)

## What is this?

One thing I care about extremely with [Mach engine](https://github.com/hexops/mach) is that you're able to cross compile for any OS with nothing more than `zig` and `git`. And while we can build most things from source (GLFW, and even the DirectX Shader Compiler!) there are a few system headers / libraries where, really, we just need a copy of them.

`mach/system-sdk` is how Mach engine gets a copy of them via the Zig build system.

Although it was intended for Mach specifically, and I wouldn't be surprised if one day Zig provides better options out of the box, I realize others might benefit from this and so I've made it easy for anyone to use!

## What does it provide?

Depending on the target OS, `system_sdk.zig` will automatically `git clone` the relevant system SDK for you and include it:

* `sdk-windows-x86_64` (~7MB):
  * Updated DirectX 12 headers (and prior versions) for use with Zig/MinGW when specifying a `-Dtarget=x86_64-windows-gnu` target.
  * DirectX libraries such as `dxgi.lib` and `dxguid.lib`
* `sdk-linux-x86_64` (~40MB):
  * X11/xcb/Wayland libraries/headers (as static as possible)
  * OpenGL and Vulkan headers
  * Generated Wayland protocol sources
* `sdk-macos-11.3` (~198MB) and `sdk-macos-12.0` (~149MB)
  * A nearly full copy of the macOS 11.3 and 12.0 XCode SDKs, with just a few particularly large frameworks excluded.
  * Pretty much every framework, header, etc. that you need to develop macOS and iOS applications for Intel and Apple Silicon.
  * Symlinks mostly eliminated for ease of use on Windows

The build script will `git clone` these SDKs for you, pin them to a specific Git revision denoted in the `system_sdk.zig` file so they are versioned, and even prompt for XCode license agreement in the case of macOS SDKs.

into the appdata directory 

```
/Users/slimsag/Library/Application Support/hexops
```

## Single file & one-liner to use

Get [system_sdk.zig](https://github.com/hexops/mach/blob/main/glfw/system_sdk.zig) into your codebase however you prefer. I suggest just copying it for now, it's a single file.

In your `build.zig`:

```
const system_sdk = @import("system_sdk.zig");
...
system_sdk.include(b, step, .{});
```

Where `step` is the exe / lib you're building.

## Shared between Zig projects

The SDKs are cloned into your `<appdata>/hexops` directory to ensure they are shared across projects and you don't end up with multiple copies. Prior to use the build script will `git reset` to the target revision.

## Customization

If you don't like what is provided in the SDK repositories, I've tried to make it as easy as possible to switch to your own SDK repos. Just pass `Options`:

```zig
system_sdk.include(b, step, .{
    .github_org = "myorg",
    .linux_x86_64_revision = "ab7fa8f3a05b06e0b06f4277b484e27004bfb20f",
    .windows_x86_64_revision = "5acba990efd112ea0ced364f0428e6ef6e7a5541",
});
```

And set the revisions to the Git revisions of your forks of [the SDK repositories.](https://github.com/hexops?q=mach-sdk&type=all&language=&sort=)

## Is this the right way to do this? Should I be using this?

To be clear, I don't think this is the perfect way or 100% ideal way to handle this. I am positive that the Zig community will eventually land on better solutions here.

For example, generating updated DirectX 12 headers for MinGW/Zig requires patching Microsoft's IDL files, running them through the Wine WIDL compiler, and then some. Storing even a few binaries in Git is not ideal, even if we do clone with `--depth 1`.

But, if you care about that developer experience as immensely as I do I hope you'll see a bit of reason behind the madness.

Use at your own peril!

## Issues

Please file issues/complaints in the [main Mach repository](https://github.com/hexops/mach/issues).
