<div align="center">
  <img src="https://download.alianblank.com/gameframex/gameframex_logo_320.png" alt="Game Frame X Logo" width="160" />
</div>

# Better Streaming Assets

[![GitHub release](https://img.shields.io/github/v/release/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/releases)
[![License](https://img.shields.io/github/license/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-Online-blue?style=flat-square)](https://gameframex.doc.alianblank.com)

**獨立遊戲前後端一體化解決方案 · 獨立遊戲開發者的圓夢大使**

[文檔](https://gameframex.doc.alianblank.com) · [快速開始](#快速開始) · [QQ群](https://qm.qq.com/q/5s5e1e6e6e)

**語言**: [English](README.md) | [简体中文](README.zh-CN.md) | **繁體中文** | [日本語](README.ja.md) | [한국어](README.ko.md)

---

## 項目簡介

Better Streaming Assets 是一個插件，允許你以統一且線程安全的方式直接訪問 Streaming Assets，開銷極小。主要用於 Android 項目，替代陳舊且低效的 WWW 方式或將數據嵌入 Asset Bundles。API 基於 System.IO.File 和 System.IO.Directory 類。

## 快速開始

### 安裝

任選以下方式之一：

1. 直接在 `manifest.json` 的文件中的 `dependencies` 節點下添加以下內容：
   ```json
   {"com.gameframex.unity.gwiazdorrr.betterstreamingassets": "https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git"}
   ```

2. 在 Unity 的 `Packages Manager` 中使用 `Git URL` 的方式添加庫，地址為：
   ```
   https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git
   ```

3. 直接下載倉庫放置到 Unity 項目的 `Packages` 目錄下，會自動加載識別。

## 使用範例

### 初始化

初始化（首次使用前需要調用，需要在主線程上執行）：

```csharp
BetterStreamingAssets.Initialize();
```

### 讀取文件

典型場景，從 Xml 反序列化：

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

注意 ReadFromXml 可以在任何線程調用，只要 Foo 的構造函數不進行任何 UnityEngine 調用。

### 列出文件

```csharp
// 所有 xml 文件
string[] paths = BetterStreamingAssets.GetFiles("\\", "*.xml", SearchOption.AllDirectories);
// Config 目錄（及嵌套目錄）中的 xml 文件
string[] paths = BetterStreamingAssets.GetFiles("Config", "*.xml", SearchOption.AllDirectories);
```

### 檢查目錄

```csharp
Debug.Assert( BetterStreamingAssets.DirectoryExists("Config") );
```

### 讀取數據

```csharp
// 一次性讀取所有數據
byte[] data = BetterStreamingAssets.ReadAllBytes("Foo/bar.data");

// 以流方式讀取最後 10 個字節
byte[] footer = new byte[10];
using (var stream = BetterStreamingAssets.OpenRead("Foo/bar.data"))
{
    stream.Seek(-footer.Length, SeekOrigin.End);
    stream.Read(footer, 0, footer.Length);
}
```

### Asset Bundles

```csharp
// 同步加載
var bundle = BetterStreamingAssets.LoadAssetBundle(path);
// 異步加載
var bundleOp = BetterStreamingAssets.LoadAssetBundleAsync(path);
```

## 平台支援

### Android 和 App Bundles

App Bundles (.aab) 構建在 Streaming Assets 方面存在 bug。要點如下：

- 保持 Streaming Assets 中所有文件名為小寫！
- 不要在文件名中使用非 ASCII 字符

### WebGL

目前不支援 WebGL。這需要不同的方法和完全異步的 API。

### Android 誤報壓縮 Streaming Assets 消息

Streaming Assets 最終位於 APK 中與許多自定義插件添加的文件相同的部分（`assets` 目錄），因此無法判斷壓縮文件是否為 Streaming Asset。此工具採取保守策略，在 `assets` 內但 `assets/bin` 外發現壓縮文件時記錄錯誤。如果你確定某個壓縮文件不是 Streaming Asset，可以在 Better Streaming Assets 所在同一程序集中添加如下文件：

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

## 文檔與資源

- 文檔地址: https://gameframex.doc.alianblank.com
- 倉庫地址: https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets
- 原始項目: https://github.com/gwiazdorrr/BetterStreamingAssets

## 開源協議

本項目遵循 MIT 許可證。詳細信息請查看 [LICENSE](https://github.com/gwiazdorrr/BetterStreamingAssets/blob/master/LICENSE) 文件。
