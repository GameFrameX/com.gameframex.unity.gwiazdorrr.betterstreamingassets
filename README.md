<div align="center">
  <img src="https://download.alianblank.com/gameframex/gameframex_logo_320.png" alt="Game Frame X Logo" width="160" />
</div>

# Better Streaming Assets

[![GitHub release](https://img.shields.io/github/v/release/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/releases)
[![License](https://img.shields.io/github/license/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-Online-blue?style=flat-square)](https://gameframex.doc.alianblank.com)

**All-in-One Solution for Indie Game Development · Empowering Indie Developers' Dreams**

[Documentation](https://gameframex.doc.alianblank.com) · [Quick Start](#quick-start) · [QQ Group](https://qm.qq.com/q/5s5e1e6e6e)

**Language**: **English** | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

---

## Project Overview

Better Streaming Assets is a plugin that lets you access Streaming Assets directly in an uniform and thread-safe way, with tiny overhead. Mostly beneficial for Android projects, where the alternatives are to use archaic and hugely inefficient WWW or embed data in Asset Bundles. API is based on System.IO.File and System.IO.Directory classes.

## Quick Start

### Installation

Choose one of the following methods:

1. Add the following to the `dependencies` section in your project's `manifest.json`:
   ```json
   {"com.gameframex.unity.gwiazdorrr.betterstreamingassets": "https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git"}
   ```

2. Use `Git URL` in Unity's Package Manager:
   ```
   https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git
   ```

3. Download the repository and place it in your Unity project's `Packages` directory. It will be loaded automatically.

## Usage Examples

### Initialization

Initialization (before first use, needs to be called on main thread):

```csharp
BetterStreamingAssets.Initialize();
```

### Reading Files

Typical scenario, deserializing from Xml:

```csharp
public static Foo ReadFromXml(string path)
{
    if ( !BetterStreamingAssets.FileExists(path) )
    {
        Debug.LogErrorFormat("Streaming asset not found: {0}", path);
        return null;
    }

    using ( var stream = BetterStreamingAssets.OpenRead(path) )
    {
        var serializer = new System.Xml.Serialization.XmlSerializer(typeof(Foo));
        return (Foo)serializer.Deserialize(stream);
    }
}
```

Note that ReadFromXml can be called from any thread, as long as Foo's constructor doesn't make any UnityEngine calls.

### Listing Files

```csharp
// all the xmls
string[] paths = BetterStreamingAssets.GetFiles("\\", "*.xml", SearchOption.AllDirectories);
// just xmls in Config directory (and nested)
string[] paths = BetterStreamingAssets.GetFiles("Config", "*.xml", SearchOption.AllDirectories);
```

### Checking Directories

```csharp
Debug.Assert( BetterStreamingAssets.DirectoryExists("Config") );
```

### Reading Data

```csharp
// all at once
byte[] data = BetterStreamingAssets.ReadAllBytes("Foo/bar.data");

// as stream, last 10 bytes
byte[] footer = new byte[10];
using (var stream = BetterStreamingAssets.OpenRead("Foo/bar.data"))
{
    stream.Seek(-footer.Length, SeekOrigin.End);
    stream.Read(footer, 0, footer.Length);
}
```

### Asset Bundles

```csharp
// synchronous
var bundle = BetterStreamingAssets.LoadAssetBundle(path);
// async
var bundleOp = BetterStreamingAssets.LoadAssetBundleAsync(path);
```

## Platform Notes

### Android & App Bundles

App Bundles (.aab) builds are bugged when it comes to Streaming Assets. The bottom line is:

- Keep all file names in Streaming Assets lowercase!
- Do not use non-ASCII characters in file names

### WebGL

There is currently no support for WebGL. It would require a different approach and a completely async API.

### Android False-positive Compressed Streaming Assets Messages

Streaming Assets end up in the same part of APK as files added by many custom plugins (`assets` directory), so it is impossible to tell whether a compressed file is a Streaming Asset or not. This tool acts conservatively and logs errors whenever it finds a compressed file inside of `assets`, but outside of `assets/bin`. If you are annoyed by this and are certain a compressed file was not meant to be a Streaming Asset, add a file like this in the same assembly as Better Streaming Assets:

```csharp
partial class BetterStreamingAssets
{
    static partial void AndroidIsCompressedFileStreamingAsset(string path, ref bool result)
    {
        if ( path == "assets/my_custom_plugin_settings.json")
        {
            result = false;
        }
    }
}
```

## Documentation & Resources

- Documentation: https://gameframex.doc.alianblank.com
- Repository: https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets
- Original Project: https://github.com/gwiazdorrr/BetterStreamingAssets

## License

This project is licensed under the MIT License. See [LICENSE](https://github.com/gwiazdorrr/BetterStreamingAssets/blob/master/LICENSE) for details.
