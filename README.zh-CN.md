<div align="center">
  <img src="https://download.alianblank.com/gameframex/gameframex_logo_320.png" alt="Game Frame X Logo" width="160" />
</div>

# Better Streaming Assets

[![GitHub release](https://img.shields.io/github/v/release/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/releases)
[![License](https://img.shields.io/github/license/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-Online-blue?style=flat-square)](https://gameframex.doc.alianblank.com)

**独立游戏前后端一体化解决方案 · 独立游戏开发者的圆梦大使**

[文档](https://gameframex.doc.alianblank.com) · [快速开始](#快速开始) · [QQ群](https://qm.qq.com/q/5s5e1e6e6e)

**语言**: [English](README.md) | **简体中文** | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

---

## 项目简介

Better Streaming Assets 是一个插件，允许你以统一且线程安全的方式直接访问 Streaming Assets，开销极小。主要用于 Android 项目，替代陈旧且低效的 WWW 方式或将数据嵌入 Asset Bundles。API 基于 System.IO.File 和 System.IO.Directory 类。

## 快速开始

### 安装

任选以下方式之一：

1. 直接在 `manifest.json` 的文件中的 `dependencies` 节点下添加以下内容：
   ```json
   {"com.gameframex.unity.gwiazdorrr.betterstreamingassets": "https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git"}
   ```

2. 在 Unity 的 `Packages Manager` 中使用 `Git URL` 的方式添加库，地址为：
   ```
   https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git
   ```

3. 直接下载仓库放置到 Unity 项目的 `Packages` 目录下，会自动加载识别。

## 使用示例

### 初始化

初始化（首次使用前需要调用，需要在主线程上执行）：

```csharp
BetterStreamingAssets.Initialize();
```

### 读取文件

典型场景，从 Xml 反序列化：

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

注意 ReadFromXml 可以在任何线程调用，只要 Foo 的构造函数不进行任何 UnityEngine 调用。

### 列出文件

```csharp
// 所有 xml 文件
string[] paths = BetterStreamingAssets.GetFiles("\\", "*.xml", SearchOption.AllDirectories);
// Config 目录（及嵌套目录）中的 xml 文件
string[] paths = BetterStreamingAssets.GetFiles("Config", "*.xml", SearchOption.AllDirectories);
```

### 检查目录

```csharp
Debug.Assert( BetterStreamingAssets.DirectoryExists("Config") );
```

### 读取数据

```csharp
// 一次性读取所有数据
byte[] data = BetterStreamingAssets.ReadAllBytes("Foo/bar.data");

// 以流方式读取最后 10 个字节
byte[] footer = new byte[10];
using (var stream = BetterStreamingAssets.OpenRead("Foo/bar.data"))
{
    stream.Seek(-footer.Length, SeekOrigin.End);
    stream.Read(footer, 0, footer.Length);
}
```

### Asset Bundles

```csharp
// 同步加载
var bundle = BetterStreamingAssets.LoadAssetBundle(path);
// 异步加载
var bundleOp = BetterStreamingAssets.LoadAssetBundleAsync(path);
```

## 平台支持

### Android 和 App Bundles

App Bundles (.aab) 构建在 Streaming Assets 方面存在 bug。要点如下：

- 保持 Streaming Assets 中所有文件名为小写！
- 不要在文件名中使用非 ASCII 字符

### WebGL

目前不支持 WebGL。这需要不同的方法和完全异步的 API。

### Android 误报压缩 Streaming Assets 消息

Streaming Assets 最终位于 APK 中与许多自定义插件添加的文件相同的部分（`assets` 目录），因此无法判断压缩文件是否为 Streaming Asset。此工具采取保守策略，在 `assets` 内但 `assets/bin` 外发现压缩文件时记录错误。如果你确定某个压缩文件不是 Streaming Asset，可以在 Better Streaming Assets 所在同一程序集中添加如下文件：

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

## 文档与资源

- 文档地址: https://gameframex.doc.alianblank.com
- 仓库地址: https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets
- 原始项目: https://github.com/gwiazdorrr/BetterStreamingAssets

## 开源协议

本项目遵循 MIT 许可证。详细信息请查看 [LICENSE](https://github.com/gwiazdorrr/BetterStreamingAssets/blob/master/LICENSE) 文件。
