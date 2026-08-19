# Android SDK 시작하기 - Unity (beta)

Unity 프로젝트에서 nap mx Android SDK를 연동하는 방법입니다.

> 🧪 **beta — 연동 전 문의해 주세요.**
> Unity 연동에 필요한 **C# 플러그인은 별도 제공**되며 이 문서에는 포함되어 있지 않습니다.
> 아래 C# 예제는 **흐름 참고용**이고 실제 클래스·메서드명은 제공되는 플러그인 기준을 따릅니다.
> 플러그인 배포본과 최신 연동 방법은 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요.
>
> 네이티브(Android) 연동은 정식 지원됩니다 — [Android 네이티브 시작하기](/android/native/getting-started)

## 지원 버전

| 항목 | 버전 |
|---|---|
| nap mx Android SDK | `admixer-ssp` **2.2.0** (BOM `2026.08.01`) |
| 최소 Android API | **21** (코어 기준) |
| 어댑터별 최소 API 상향 | Google AdManager · Naver Ad Manager **23**, GMA NextGen · AppLovin **24** |

> ℹ️ 어댑터를 추가하면 앱 전체 `minSdkVersion`이 그에 맞춰 올라갑니다. 자세한 표는 [네이티브 시작하기](/android/native/getting-started)를 참고하세요.

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

> ℹ️ **AppLovin은 매니페스트 `applovin.sdk.key`를 사용하지 않습니다.** SDK Key는 서버(파트너 사이트)에서 전달되며, 서버에 없을 때만 앱에서 `AdInfo.Builder.setAdapterConfig(...)`로 주입합니다. 자세한 내용은 [Q&A](/android/native/qna)를 참고하세요.

---

## 2. Gradle 설정 (mainTemplate.gradle)

**네트워크 SDK를 직접 추가하지 마세요.** nap mx는 **어댑터 아티팩트**로 연동하며, 각 네트워크 SDK는 어댑터의 전이 의존으로 자동 포함됩니다.

```groovy
dependencies {
    // BOM 으로 버전을 묶어 관리 (권장) — 멤버는 버전 생략
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.08.01')

    // nap mx 코어 (필수)
    implementation 'io.github.nasmedia-tech:admixer-ssp'

    // --- 선택적 어댑터 (필요한 것만) ---
    implementation 'io.github.nasmedia-tech:admixer-admanager'      // Google AdManager
    implementation 'io.github.nasmedia-tech:admixer-adfit'          // Kakao Adfit
    implementation 'io.github.nasmedia-tech:admixer-pangle'         // Pangle
    implementation 'io.github.nasmedia-tech:admixer-applovin'       // AppLovin
    implementation 'io.github.nasmedia-tech:admixer-unity'          // Unity Ads
    implementation 'io.github.nasmedia-tech:admixer-naveradmanager' // Naver Ad Manager
    implementation 'io.github.nasmedia-tech:admixer-teads'          // Teads
}
```

> ⚠️ **`com.google.android.gms:play-services-ads` 같은 벤더 SDK를 직접 추가하면 안 됩니다.** 어댑터 클래스가 없어 해당 네트워크가 워터폴에서 동작하지 않습니다.

네트워크별 추가 Maven 저장소가 필요한 경우 `settingsTemplate.gradle`에 추가합니다.

```groovy
maven { url "https://devrepo.kakao.com/nexus/content/groups/public/" }  // Kakao Adfit
maven { url "https://artifact.bytedance.com/repository/pangle/" }       // Pangle
maven { url "https://sdk.teads.tv/android/repo" }                       // Teads
maven { url "https://teads.jfrog.io/artifactory/SDKAndroid-maven-prod" } // Teads
maven { url "https://developer.huawei.com/repo/" }                      // Teads (Huawei 호환 — 공식 설치 가이드 필수)
```

> ℹ️ 개별 버전 지정, ProGuard, 네트워크 SDK 중복 예외 처리 등 상세 설정은 [네이티브 시작하기](/android/native/getting-started)와 동일합니다.

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

> 🧪 **클래스·메서드명은 제공되는 플러그인 기준을 따릅니다.** 위 예제는 초기화 시점(앱 시작 시 1회)을 보여주기 위한 흐름 참고용입니다.
>
> 네이티브 SDK의 초기화는 **Media Key와 함께 사용할 Adunit ID 목록**을 받습니다(`AdMixer.getInstance().initialize(context, mediaKey, adUnits)`). 플러그인도 동일하게 Adunit 목록 전달이 필요하므로, 실제 시그니처는 배포본 문서를 확인하세요.

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

> 🧪 표의 메서드명 역시 플러그인 배포본 기준을 따릅니다. **전면(Interstitial) 전용 문서는 아직 제공되지 않습니다.**

---

## 다음 단계

- [배너 - Unity](/android/unity/banner)
- [네이티브 - Unity](/android/unity/native-ad)
- [리워드 동영상 - Unity](/android/unity/rewarded-video)
- [동영상 - Unity](/android/unity/video)
