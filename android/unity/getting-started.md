# Android SDK 시작하기 - Unity (beta)

Unity 프로젝트에서 nap mx Android SDK를 연동하는 방법입니다.

> 🧪 **beta — 연동 전 문의해 주세요.**
> Unity 플러그인(`AdMixer.cs`, `AdMixerAdListener.cs`, `AdMixerUnityBridge.java` 등)은 별도 배포됩니다.
> 플러그인 배포본과 최신 연동 방법은 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요.
>
> 네이티브(Android) 연동은 정식 지원됩니다 — [Android 네이티브 시작하기](/android/native/getting-started)

## 지원 버전

| 항목 | 버전 |
|---|---|
| nap mx Android SDK | `admixer-ssp` **2.2.2** (BOM `2026.09.02`) |
| 최소 Android API | **21** (코어 기준) |
| 어댑터별 최소 API 상향 | Google AdManager · Naver Ad Manager **23**, GMA NextGen · AppLovin **24** |

> ℹ️ 어댑터를 추가하면 앱 전체 `minSdkVersion`이 그에 맞춰 올라갑니다. 자세한 표는 [네이티브 시작하기](/android/native/getting-started)를 참고하세요.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. 플러그인 구성

플러그인은 다음 파일로 구성됩니다.

| 파일 | 역할 |
|---|---|
| `Assets/Scripts/AdMixer.cs` | 단일 진입점 (싱글톤 `MonoBehaviour`). Inspector에서 Media Key / Adunit ID 설정, `Awake()`에서 SDK 자동 초기화 |
| `Assets/Scripts/AdMixerAdListener.cs` | 광고 이벤트 수신 `MonoBehaviour`. GameObject 이름이 **`AdMixerAdListener`** 로 고정 |
| `Assets/Plugins/Android/AdMixerUnityBridge.java` | 네이티브 브릿지 |
| `Assets/Plugins/Android/nativeadlayout-release.aar` | 네이티브 광고 레이아웃 (`item_320x100`, `item_320x480`) |
| `Assets/Plugins/Android/mainTemplate.gradle`, `settingsTemplate.gradle` | SDK 의존성 · Maven 저장소 |
| `Assets/Plugins/Android/AndroidManifest.xml` | 권한 · 네트워크 메타데이터 |

---

## 2. 권한 설정 (AndroidManifest.xml)

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

> ℹ️ **AppLovin은 매니페스트 `applovin.sdk.key`를 사용하지 않습니다.** SDK Key는 서버(파트너 사이트)에서 전달됩니다. 자세한 내용은 [Q&A](/android/native/qna)를 참고하세요.

---

## 3. Gradle 설정 (mainTemplate.gradle)

**네트워크 SDK를 직접 추가하지 마세요.** nap mx는 **어댑터 아티팩트**로 연동하며, 각 네트워크 SDK는 어댑터의 전이 의존으로 자동 포함됩니다.

```groovy
dependencies {
    // BOM 으로 버전을 묶어 관리 (권장) — 멤버는 버전 생략
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.09.02')

    // nap mx 코어 (필수)
    implementation 'io.github.nasmedia-tech:admixer-ssp'
    // Google Advertising ID (필수)
    implementation 'com.google.android.gms:play-services-ads-identifier:18.2.0'

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

## 4. SDK 초기화

초기화는 `AdMixer` 컴포넌트가 `Awake()`에서 자동으로 수행합니다. 코드로 초기화 함수를 호출할 필요가 없습니다.

1. 씬에 GameObject를 만들고 `AdMixer.cs` 컴포넌트를 추가합니다. (씬에 **하나만** 존재해야 하며, `DontDestroyOnLoad` 처리됩니다)
2. Inspector에서 `Android MediaKey`와 사용할 광고 타입의 `Android ... AdUnitId`를 입력합니다.
3. 이벤트 수신용으로 이름이 **`AdMixerAdListener`** 인 GameObject를 만들고 `AdMixerAdListener.cs` 컴포넌트를 추가합니다.

`Awake()` 시점에 Inspector에 입력된 Adunit ID 전체를 모아 네이티브 `AdMixer.getInstance().initialize(context, mediaKey, adUnits)`를 **1회** 호출합니다.

| Inspector 필드 | 설명 |
|---|---|
| `Android MediaKey` | 발급받은 Media Key |
| `Android Banner AdUnitId` | 배너 Adunit ID |
| `Android Interstitial AdUnitId` | 전면 Adunit ID |
| `Android Reward AdUnitId` | 리워드 동영상 Adunit ID |
| `Android Video AdUnitId` | 동영상 Adunit ID |
| `Android Native AdUnitId` | 네이티브 Adunit ID |
| `Android Native Use Large Layout` | 네이티브 레이아웃 (`true`: 320×480, `false`: 320×100) |

> ⚠️ 코드로 GameObject를 생성하는 경우, `AdMixer.Awake()`가 필드를 읽으므로 **비활성 상태로 생성 → 필드 설정 → 활성화** 순서로 진행하세요.

---

## 5. 광고 API 호출

모든 API는 `AdMixer.Instance`를 통해 호출합니다. Adunit ID는 Inspector 값을 사용하므로 인자로 넘기지 않습니다.

| 광고 타입 | 로드 | 표시 | 제거 |
|-----------|------|------|------|
| 배너 | `LoadBanner()` | `ShowBanner()` | `DestroyBanner()` |
| 전면 | `LoadInterstitial()` | `ShowInterstitial()` | `DestroyInterstitial()` |
| 네이티브 | `LoadNativeAd()` | (로드 시 자동 표시) | `DestroyNativeAd()` |
| 리워드 동영상 | `LoadRewardVideo()` | `ShowRewardVideo()` | `DestroyRewardVideo()` |
| 동영상 | `LoadVideoAd()` | (로드 시 자동 표시) | `DestroyVideoAd()` |

배너는 `BannerOnPause()` / `BannerOnResume()`으로 갱신 타이머를 수동 제어할 수 있습니다.

---

## 6. 이벤트 수신

네이티브 브릿지는 모든 광고 타입의 이벤트를 `AdMixerAdListener` GameObject의 세 메서드로 전달합니다. 광고 타입은 구분되지 않으므로, 앱에서 어떤 광고를 요청했는지 상태를 관리해야 합니다.

| 메서드 | 파라미터 | 설명 |
|---|---|---|
| `OnReceivedAd(string param)` | 응답한 네트워크 어댑터명 (예: `"admixer"`) | 광고 로드 성공 |
| `OnFailedToReceiveAd(string param)` | `"message\|code"` | 광고 로드 · 표시 실패 |
| `OnEventAd(string param)` | 이벤트 이름 | 아래 표 참고 |

**`OnEventAd` 이벤트 이름**

| 이름 | 발생 광고 | 설명 |
|---|---|---|
| `DISPLAYED` | 전체 | 광고 노출 |
| `CLICK` | 전체 | 광고 클릭 |
| `CLOSE` | 전면 · 리워드 | 광고 닫힘 |
| `COMPLETION` | 전면 · 리워드 · 동영상 | 동영상 재생 완료 (네트워크에 따라 미발화) |
| `SKIPPED` | 동영상 | 사용자가 Skip 클릭 |
| `EARNEDREWARD` | 리워드 | **보상 적립 — 보상 지급 기준** |
| `BANNER_DESTROYED` · `INTERSTITIAL_DESTROYED` · `REWARD_DESTROYED` · `VIDEO_DESTROYED` · `NATIVE_DESTROYED` | 해당 광고 | 제거 완료 |

```csharp
using UnityEngine;
using UnityEngine.Scripting;

public class AdMixerAdListener : MonoBehaviour
{
    [Preserve]
    private void Awake()
    {
        gameObject.name = "AdMixerAdListener";
        DontDestroyOnLoad(gameObject);
    }

    [Preserve]
    public void OnReceivedAd(string param)
    {
        Debug.Log("[AdMixer] OnReceivedAd: " + param);
    }

    [Preserve]
    public void OnFailedToReceiveAd(string param)
    {
        // 형식: "message|code"
        string[] tokens = (param ?? "").Split('|');
        string message = tokens.Length > 0 ? tokens[0] : "";
        string code    = tokens.Length > 1 ? tokens[1] : "";
        Debug.Log($"[AdMixer] OnFailedToReceiveAd: message={message}, code={code}");
    }

    [Preserve]
    public void OnEventAd(string param)
    {
        Debug.Log("[AdMixer] OnEventAd: " + param);
    }
}
```

---

## 다음 단계

- [배너 - Unity](/android/unity/banner)
- [네이티브 - Unity](/android/unity/native-ad)
- [리워드 동영상 - Unity](/android/unity/rewarded-video)
- [동영상 - Unity](/android/unity/video)
