<div align="center">
  <img src="https://download.alianblank.com/gameframex/gameframex_logo_320.png" alt="Game Frame X Logo" width="160" />
</div>

# Better Streaming Assets

[![GitHub release](https://img.shields.io/github/v/release/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/releases)
[![License](https://img.shields.io/github/license/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-Online-blue?style=flat-square)](https://gameframex.doc.alianblank.com)

**インディゲーム開発者向けオールインワンソリューション · インディ開発者の夢を支援**

[ドキュメント](https://gameframex.doc.alianblank.com) · [クイックスタート](#クイックスタート) · [QQグループ](https://qm.qq.com/q/5s5e1e6e6e)

**言語**: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | **日本語** | [한국어](README.ko.md)

---

## プロジェクト概要

Better Streaming Assets は、Streaming Assets に統一的かつスレッドセーフな方法で直接アクセスできるプラグインで、オーバーヘッドが最小限です。主に Android プロジェクトで有益で、古くて非効率な WWW の代替や Asset Bundles へのデータ埋め込みを回避できます。API は System.IO.File および System.IO.Directory クラスに基づいています。

## クイックスタート

### インストール

以下のいずれかの方法をお選びください：

1. プロジェクトの `manifest.json` の `dependencies` セクションに以下を追加：
   ```json
   {"com.gameframex.unity.gwiazdorrr.betterstreamingassets": "https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git"}
   ```

2. Unity の Package Manager で `Git URL` を使用：
   ```
   https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git
   ```

3. リポジトリをダウンロードして Unity プロジェクトの `Packages` ディレクトリに配置。自動的にロードされます。

## 使用例

### 初期化

初期化（初回使用前に呼び出す必要があります。メインスレッドで実行する必要があります）：

```csharp
BetterStreamingAssets.Initialize();
```

### ファイルの読み込み

典型的なシナリオ、Xml からの逆シリアル化：

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

ReadFromXml は Foo のコンストラクタが UnityEngine の呼び出しを行わない限り、どのスレッドからでも呼び出すことができます。

### ファイル一覧の取得

```csharp
// すべての xml ファイル
string[] paths = BetterStreamingAssets.GetFiles("\\", "*.xml", SearchOption.AllDirectories);
// Config ディレクトリ（およびネストされたディレクトリ）内の xml ファイル
string[] paths = BetterStreamingAssets.GetFiles("Config", "*.xml", SearchOption.AllDirectories);
```

### ディレクトリの確認

```csharp
Debug.Assert( BetterStreamingAssets.DirectoryExists("Config") );
```

### データの読み込み

```csharp
// 一括読み込み
byte[] data = BetterStreamingAssets.ReadAllBytes("Foo/bar.data");

// ストリームとして最後の 10 バイトを読み込み
byte[] footer = new byte[10];
using (var stream = BetterStreamingAssets.OpenRead("Foo/bar.data"))
{
    stream.Seek(-footer.Length, SeekOrigin.End);
    stream.Read(footer, 0, footer.Length);
}
```

### Asset Bundles

```csharp
// 同期
var bundle = BetterStreamingAssets.LoadAssetBundle(path);
// 非同期
var bundleOp = BetterStreamingAssets.LoadAssetBundleAsync(path);
```

## プラットフォーム対応

### Android と App Bundles

App Bundles (.aab) ビルドは Streaming Assets に関してバグがあります。重要なポイント：

- Streaming Assets 内のすべてのファイル名は小文字にしてください！
- ファイル名に非 ASCII 文字を使用しないでください

### WebGL

現在 WebGL はサポートされていません。異なるアプローチと完全な非同期 API が必要です。

### Android 誤検出圧縮 Streaming Assets メッセージ

Streaming Assets は多くのカスタムプラグインが追加するファイルと同じ APK の部分（`assets` ディレクトリ）に配置されるため、圧縮ファイルが Streaming Asset かどうかを判断できません。このツールは保守的に動作し、`assets` 内で `assets/bin` の外にある圧縮ファイルを発見した場合にエラーをログに記録します。圧縮ファイルが Streaming Asset ではないことが確実な場合、Better Streaming Assets と同じアセンブリに次のようなファイルを追加できます：

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

## ドキュメントとリソース

- ドキュメント: https://gameframex.doc.alianblank.com
- リポジトリ: https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets
- オリジナルプロジェクト: https://github.com/gwiazdorrr/BetterStreamingAssets

## ライセンス

このプロジェクトは MIT ライセンスの下で公開されています。詳細は [LICENSE](https://github.com/gwiazdorrr/BetterStreamingAssets/blob/master/LICENSE) ファイルを参照してください。
