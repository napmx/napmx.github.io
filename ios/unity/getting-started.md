# iOS SDK 시작하기 - Unity

Unity 프로젝트에서 nap mx iOS SDK를 연동하기 위한 가이드 문서이며, nap mx Mediation을 지원합니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. 플러그인 구성

플러그인은 다음 파일로 구성됩니다.

| 파일 | 역할 |
|---|---|
| `Assets/Scripts/AdMixer.cs` | 단일 진입점 (싱글톤 `MonoBehaviour`). Inspector에서 Media Key / Adunit ID 설정, `Awake()`에서 SDK 자동 초기화 |
| `Assets/Scripts/NAPSSPPluginIOS.cs` | iOS 네이티브 함수 바인딩 및 이벤트(`static event`) 노출. `AdMixer`가 자동으로 컴포넌트 추가 |
| `Assets/Plugins/iOS/NAPSSPBridge.swift` | 네이티브 브릿지 |
| `Assets/Plugins/iOS/iOSATTRequest.mm` | ATT 권한 요청 (앱 활성화 후 자동 호출) |
| `Assets/Plugins/iOS/NativeTemplates/NapSSPNativeTemplateView.xib` | 네이티브 광고 템플릿 |
| `Assets/Editor/iOSPostProcess.cs` | 빌드 후처리 — `Info.plist` 항목 · `AppTrackingTransparency.framework` 추가 |

---

## 2. Xcode 설정

nap mx SDK는 **Xcode 16 이상**, **iOS deployment target 13.0 이상** 환경에서 동작합니다.

### 2-1. CocoaPods를 통한 설치

Unity에서 iOS 빌드를 생성하면 플러그인의 빌드 후처리(`iOSPostProcess.cs`)가 Xcode 프로젝트 디렉토리에 **`Podfile`을 자동 생성**합니다. (이미 `Podfile`이 있으면 건드리지 않습니다)

생성되는 `Podfile`은 다음과 같습니다. 사용하지 않는 어댑터는 지우세요.

```ruby
platform :ios, '13.0'

target 'Unity-iPhone' do
  use_frameworks!
end

target 'UnityFramework' do
  use_frameworks!

  pod 'AdMixerMediation', '2.4.6'

  # 미디에이션 네트워크 (사용하는 것만 남기세요)
  pod 'AdMixerMediationGAM'       # Google AdManager
  pod 'AdMixerMediationAdFit'     # Kakao AdFit
  pod 'AdMixerMediationPangle'    # Pangle
  pod 'AdMixerMediationAppLovin'  # AppLovin
  pod 'AdMixerMediationUnityAds'  # UnityAds
end
```

Xcode 프로젝트 디렉토리에서 pod를 설치합니다.

```bash
pod install --repo-update
```

이후 `.xcworkspace`를 열어 남은 설정을 진행해주세요.

> SDK는 `UnityFramework` 타겟에 추가합니다. 플러그인의 Swift 브릿지가 `UnityFramework`에서 컴파일되기 때문입니다.
>
> ⚠️ `AdMixerMediation` 버전을 플러그인이 대응하는 버전(현재 **2.4.6**)과 다르게 바꾸면 빌드가 실패할 수 있습니다.

### Google 네트워크 - SDK 입찰 광고 소스 설정

Google 네트워크를 사용하시는 경우, SDK 입찰 광고 소스 사용을 위해 아래 광고 소스 라이브러리를 모두 추가해주세요.

- Pangle / AppLovin / DT Exchange / InMobi / Liftoff Monetize / Meta / Moloco / Unity Ads / Mintegral

---

## 3. SDK 연동

### 3-1. IDFA 설정

ATT(App Tracking Transparency) 권한 요청은 플러그인의 `iOSATTRequest.mm`이 앱 활성화(`UIApplicationDidBecomeActive`) 1초 후 자동으로 수행합니다. 별도 코드가 필요 없습니다.

`Info.plist`의 `NSUserTrackingUsageDescription` 문구는 `Assets/Editor/iOSPostProcess.cs`가 빌드 시 자동으로 추가합니다. 문구를 변경하려면 해당 파일에서 수정하세요.

```csharp
plist.root.SetString("NSUserTrackingUsageDescription", "맞춤형 광고 제공을 위해 광고 추적 권한이 필요합니다.");
```

Google AdManager 사용 시, `AdMixer` Inspector의 `iOS GAD Application Id`에 **발급받은 App ID**(`ca-app-pub-xxx~yyy`)를 입력합니다. 빌드 시 `iOSPostProcess.cs`가 이 값을 `Info.plist`의 `GADApplicationIdentifier`로 기록합니다. 비워 두면 기록하지 않습니다.

### 3-2. 미디에이션 네트워크 초기화 (선택)

일부 네트워크는 앱 시작 시 초기화 함수를 호출해야 합니다. `NAPSSPPluginIOS`의 정적 메서드를 SDK 초기화 이후에 호출하세요.

```csharp
NAPSSPPluginIOS.initGAM();
NAPSSPPluginIOS.initPangle("발급받은_PANGLE_APP_ID");
NAPSSPPluginIOS.initAppLovin("발급받은_APPLOVIN_SDK_KEY");
NAPSSPPluginIOS.initUnityAds("발급받은_UNITYADS_GAME_ID");
```

> Pangle, UnityAds App ID 및 AppLovin SDK Key 발급은 nap mx 운영팀([nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr))으로 문의해주세요.

---

## 4. SDK 초기화

초기화는 `AdMixer` 컴포넌트가 `Awake()`에서 자동으로 수행합니다. 코드로 초기화 함수를 호출할 필요가 없습니다.

1. 씬에 GameObject를 만들고 `AdMixer.cs` 컴포넌트를 추가합니다. (씬에 **하나만** 존재해야 하며, `DontDestroyOnLoad` 처리됩니다)
2. Inspector에서 `iOS MediaKey`와 사용할 광고 타입의 `iOS ... AdUnitId`를 입력합니다. **iOS의 Media Key와 Adunit ID는 정수(int)** 입니다.

`Awake()` 시점에 다음이 자동으로 수행됩니다.

- 이벤트 수신 오브젝트 등록 (`AdMixer` GameObject 자신)
- Inspector의 Adunit ID 전체로 SDK 초기화 (`AMMediation.initialize`)
- 배너 · 동영상 · 네이티브 뷰 인스턴스 생성 (`*ViewInit`) 및 델리게이트 등록

| Inspector 필드 | 타입 | 설명 |
|---|---|---|
| `iOS MediaKey` | int | 발급받은 Media Key |
| `iOS Banner AdUnitId` | int | 배너 Adunit ID |
| `iOS Interstitial AdUnitId` | int | 전면 Adunit ID |
| `iOS Reward AdUnitId` | int | 리워드 동영상 Adunit ID |
| `iOS Video View AdUnitId` | int | 동영상 Adunit ID |
| `iOS Native AdUnitId` | int | 네이티브 Adunit ID |
| `iOS Banner Top Position` | bool | 배너 위치 (`true`: 상단, `false`: 하단) |
| `iOS Native Top Position` | bool | 네이티브 위치 (`true`: 상단, `false`: 하단) |
| `iOS GAD Application Id` | string | Google AdManager App ID (GAM 사용 시) |

> ⚠️ 이벤트는 `AdMixer` GameObject 이름으로 전달되므로, **Inspector에서 `AdMixer` GameObject의 이름을 변경하면 이벤트가 수신되지 않습니다.**

---

## 5. 광고 API 호출

공통 API는 `AdMixer.Instance`를 통해 호출합니다. Adunit ID는 Inspector 값을 사용하므로 인자로 넘기지 않습니다.

| 광고 타입 | 로드 | 표시 | 제거 |
|-----------|------|------|------|
| 배너 | `LoadBanner()` | (로드 시 자동 표시) | `DestroyBanner()` |
| 전면 | `LoadInterstitial()` | `ShowInterstitial()` | — |
| 네이티브 | `LoadNativeAd()` | (로드 시 자동 표시) | `DestroyNativeAd()` |
| 리워드 동영상 | `LoadRewardVideo()` | `ShowRewardVideo()` | — |
| 동영상 | `LoadVideoAd()` | (로드 시 자동 표시) | `DestroyVideoAd()` |

전면 동영상은 `AdMixer`에 래퍼가 없으며 `NAPSSPPluginIOS`의 정적 메서드를 직접 호출합니다. ([동영상 - Unity](/ios/unity/video) 참고)

---

## 6. 이벤트 수신

iOS 이벤트는 `NAPSSPPluginIOS`의 **C# `static event`** 로 제공됩니다. 원하는 이벤트에 핸들러를 구독하세요. 이벤트 목록은 광고 타입별 페이지를 참고하세요.

```csharp
using UnityEngine;

public class AdEventHandler : MonoBehaviour
{
    void OnEnable()
    {
        NAPSSPPluginIOS.OnSuccessBanner += HandleBannerLoaded;
        NAPSSPPluginIOS.OnFailBanner   += HandleBannerFailed;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessBanner -= HandleBannerLoaded;
        NAPSSPPluginIOS.OnFailBanner   -= HandleBannerFailed;
    }

    void HandleBannerLoaded() { Debug.Log("배너 로드 성공"); }
    void HandleBannerFailed() { Debug.Log("배너 로드 실패"); }
}
```

---

## 다음 단계

- [배너 - Unity](/ios/unity/banner)
- [네이티브 - Unity](/ios/unity/native-ad)
- [리워드 동영상 - Unity](/ios/unity/rewarded-video)
- [동영상 - Unity](/ios/unity/video)
