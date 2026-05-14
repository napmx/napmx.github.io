# Android SDK 시작하기 - Unity

Unity 프로젝트에서 nap SSP Android SDK를 연동하는 방법입니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. 권한 설정 (AndroidManifest.xml)

```xml
<manifest>
    <uses-permission android:name="android.permission.INTERNET" />
    <application>
        ...
    </application>
</manifest>
```

Google AdManager 사용 시 추가:

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="발급받은 App ID" />
```

AppLovin 사용 시 추가:

```xml
<meta-data
    android:name="applovin.sdk.key"
    android:value="발급받은 키" />
```

---

## 2. Gradle 설정 (mainTemplate.gradle)

```groovy
dependencies {
    // nap SSP 메인 SDK (필수)
    implementation 'io.github.nasmedia-tech:admixer-ssp:1.0.23'
    implementation 'com.google.android.gms:play-services-ads-identifier:18.9.0'

    // --- 선택적 어댑터 ---
    // Google AdManager
    implementation 'com.google.android.gms:play-services-ads:23.x.x'

    // Pangle
    implementation 'com.pangle.global:pag-sdk:7.1.0.4'

    // AppLovin
    implementation 'com.applovin:applovin-sdk:12.x.x'

    // Unity Ads
    implementation 'com.unity3d.ads:unity-ads:4.x.x'
}
```

Pangle 사용 시 `settingsTemplate.gradle`에 저장소 추가:

```groovy
maven { url "https://artifact.bytedance.com/repository/pangle/" }
```

---

## 3. SDK 초기화 (C#)

Unity에서 `Awake()` 또는 `Start()`에서 SDK를 초기화합니다.

```csharp
using UnityEngine;

public class AdManager : MonoBehaviour
{
    void Awake()
    {
        NapSspAndroid.InitSdk("발급받은_MEDIA_KEY");
    }
}
```

---

## 4. 광고 타입별 흐름

| 광고 타입 | 로드 | 표시 | 제거 |
|-----------|------|------|------|
| 배너 | `LoadBanner()` | `ShowBanner()` | `DestroyBanner()` |
| 전면 | `LoadInterstitial()` | `ShowInterstitial()` | `DestroyInterstitial()` |
| 네이티브 | `LoadNativeAd()` | — | `DestroyNativeAd()` |
| 리워드 동영상 | `LoadRewardVideo()` | `ShowRewardVideo()` | — |
| 동영상 | `LoadVideoAd()` | — | `DestroyVideoAd()` |

---

## 다음 단계

- [배너 - Unity](/android/unity/banner)
- [네이티브 - Unity](/android/unity/native-ad)
- [리워드 동영상 - Unity](/android/unity/rewarded-video)
