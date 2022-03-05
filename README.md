# mach/system-sdk, more libraries for cross-compilation with Zig <a href="https://hexops.com"><img align="right" alt="Hexops logo" src="https://raw.githubusercontent.com/hexops/media/master/readme.svg"></img></a>

`mach/system-sdk` is something that [Mach engine](https://github.com/hexops/mach) uses today to enable cross compilation of DirectX/Metal/OpenGL/Vulkan graphics (and a few other bits in the future) with Zig.

Although it was intended for Mach specifically (I wouldn't be surprised if one day Zig provides a better option out of the box), I realize others might benefit from this and so I've made it easy for anyone to use.

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

## "Eww I don't like it"

To be clear, I don't think this is the perfect way or 100% ideal way to handle this. I am positive that the Zig community will eventually land on better solutions here. For example, even producing the updated DirectX 12 headers for MinGW/Zig was not an easy or straightforward task. Storing a few binaries in Git is not ideal, either, even if we do clone with `--depth 1`.

I care about Mach engine being incredibly easy to use and cross compile, just `zig` and `git`, no reliance on what is installed on your system otherwise. And so this is more of a workaround until better options present themselves.

Luckily, with Mach, we're really only depending on a select few system libs. We even build e.g. GLFW and the DirectX shader compiler from source - so it's not like we're just distributing tons of binaries via Git. These are exceptions.

Please file issues/complaints in the [main Mach repository](https://github.com/hexops/mach/issues).
