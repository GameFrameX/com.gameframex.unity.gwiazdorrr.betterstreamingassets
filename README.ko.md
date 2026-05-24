<div align="center">
  <img src="https://download.alianblank.com/gameframex/gameframex_logo_320.png" alt="Game Frame X Logo" width="160" />
</div>

# Better Streaming Assets

[![GitHub release](https://img.shields.io/github/v/release/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/releases)
[![License](https://img.shields.io/github/license/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets?style=flat-square)](https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-Online-blue?style=flat-square)](https://gameframex.doc.alianblank.com)

**인디 게임 개발자를 위한 올인원 솔루션 · 인디 개발자의 꿈을 실현**

[문서](https://gameframex.doc.alianblank.com) · [빠른 시작](#빠른-시작) · [QQ 그룹](https://qm.qq.com/q/5s5e1e6e6e)

**언어**: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | **한국어**

---

## 프로젝트 개요

Better Streaming Assets는 최소한의 오버헤드로 Streaming Assets에 통일되고 스레드 안전한 방식으로 직접 액세스할 수 있는 플러그인입니다. 주로 Android 프로젝트에 유용하며, 구식이고 매우 비효율적인 WWW 대안이나 Asset Bundles에 데이터를 임베드하는 것을 피할 수 있습니다. API는 System.IO.File 및 System.IO.Directory 클래스를 기반으로 합니다.

## 빠른 시작

### 설치

다음 방법 중 하나를 선택하세요:

1. 프로젝트의 `manifest.json` 파일의 `dependencies` 섹션에 다음을 추가:
   ```json
   {"com.gameframex.unity.gwiazdorrr.betterstreamingassets": "https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git"}
   ```

2. Unity의 Package Manager에서 `Git URL` 사용:
   ```
   https://github.com/AlianBlank/com.gameframex.unity.gwiazdorrr.betterstreamingassets.git
   ```

3. 저장소를 다운로드하여 Unity 프로젝트의 `Packages` 디렉토리에 배치. 자동으로 로드됩니다.

## 사용 예시

### 초기화

초기화 (첫 사용 전에 호출해야 하며, 메인 스레드에서 실행해야 함):

```csharp
BetterStreamingAssets.Initialize();
```

### 파일 읽기

일반적인 시나리오, Xml에서 역직렬화:

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

Foo의 생성자가 UnityEngine 호출을 수행하지 않는 한 ReadFromXml은 모든 스레드에서 호출할 수 있습니다.

### 파일 나열

```csharp
// 모든 xml 파일
string[] paths = BetterStreamingAssets.GetFiles("\\", "*.xml", SearchOption.AllDirectories);
// Config 디렉토리 (및 중첩된 디렉토리)의 xml 파일
string[] paths = BetterStreamingAssets.GetFiles("Config", "*.xml", SearchOption.AllDirectories);
```

### 디렉토리 확인

```csharp
Debug.Assert( BetterStreamingAssets.DirectoryExists("Config") );
```

### 데이터 읽기

```csharp
// 한 번에 모두 읽기
byte[] data = BetterStreamingAssets.ReadAllBytes("Foo/bar.data");

// 스트림으로 마지막 10바이트 읽기
byte[] footer = new byte[10];
using (var stream = BetterStreamingAssets.OpenRead("Foo/bar.data"))
{
    stream.Seek(-footer.Length, SeekOrigin.End);
    stream.Read(footer, 0, footer.Length);
}
```

### Asset Bundles

```csharp
// 동기
var bundle = BetterStreamingAssets.LoadAssetBundle(path);
// 비동기
var bundleOp = BetterStreamingAssets.LoadAssetBundleAsync(path);
```

## 플랫폼 지원

### Android 및 App Bundles

App Bundles (.aab) 빌드는 Streaming Assets와 관련된 버그가 있습니다. 핵심 사항:

- Streaming Assets의 모든 파일 이름은 소문자로 유지하세요!
- 파일 이름에 비 ASCII 문자를 사용하지 마세요

### WebGL

현재 WebGL은 지원되지 않습니다. 다른 접근 방식과 완전히 비동기적인 API가 필요합니다.

### Android 거짓 양성 압축 Streaming Assets 메시지

Streaming Assets는 많은 커스텀 플러그인이 추가하는 파일과 동일한 APK 부분 (`assets` 디렉토리)에 배치되므로, 압축 파일이 Streaming Asset인지 확인할 수 없습니다. 이 도구는 보수적으로 작동하며 `assets` 내부에서 `assets/bin` 외부에 있는 압축 파일을 발견하면 오류를 기록합니다. 압축 파일이 Streaming Asset이 아닌 것이 확실한 경우, Better Streaming Assets와 동일한 어셈블리에 다음과 같은 파일을 추가할 수 있습니다:

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

## 문서 및 자료

- 문서: https://gameframex.doc.alianblank.com
- 저장소: https://github.com/GameFrameX/com.gameframex.unity.gwiazdorrr.betterstreamingassets
- 원본 프로젝트: https://github.com/gwiazdorrr/BetterStreamingAssets

## 라이선스

이 프로젝트는 MIT 라이선스에 따라 배포됩니다. 자세한 내용은 [LICENSE](https://github.com/gwiazdorrr/BetterStreamingAssets/blob/master/LICENSE) 파일을 참조하세요.
