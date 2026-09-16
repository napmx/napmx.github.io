# iOS SDK 시작하기 - Unity

Unity 프로젝트에서 nap mx iOS SDK를 연동하기 위한 가이드 문서이며, nap mx Mediation을 지원합니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. SDK 다운로드 및 설치

GitHub에서 최신 `NAPSSPSDK-x.y.z.unitypackage`를 다운로드합니다.

다운로드한 `.unitypackage`를 Unity 프로젝트 내에 추가합니다.

```
Assets > Import Package > Custom Package > NAPSSPSDK-x.y.z.unitypackage
```

패키지에 포함된 파일은 다음과 같습니다.

| 파일 | 역할 |
|---|---|
| `Assets/Scripts/AdMixer.cs` | 단일 진입점 (싱글톤 `MonoBehaviour`). Inspector에서 Media Key / Adunit ID 설정, `Awake()`에서 SDK 자동 초기화 |
| `Assets/Scripts/NAPSSPPluginIOS.cs` | iOS 네이티브 함수 바인딩 및 이벤트(`static event`) 노출 |
| `Assets/Plugins/iOS/NAPSSPBridge.swift` | 네이티브 브릿지 |
| `Assets/Plugins/iOS/iOSATTRequest.mm` | ATT 권한 요청 (앱 활성화 후 자동 호출) |
| `Assets/Plugins/iOS/NativeTemplates/NapSSPNativeTemplateView.xib` | 네이티브 광고 템플릿 |
| `Assets/Editor/iOSPostProcess.cs` | 빌드 후처리 — `Info.plist` 항목 · `AppTrackingTransparency.framework` · `Podfile` 생성 |

---

## 2. Xcode 설정하기

nap mx SDK는 **Xcode 16 이상**, **iOS deployment target 13.0 이상** 환경에서 동작합니다.

nap mx SDK는 CocoaPods를 지원합니다. 플러그인이 대응하는 `AdMixerMediation` 버전은 빌드 시 자동 생성되는 `Podfile`에 명시되어 있으며, 그 버전을 그대로 사용하세요.

### 2-1. CocoaPods를 통한 설치

CocoaPods가 없는 경우 먼저 설치합니다.

```bash
sudo gem install cocoapods
```

Unity에서 iOS 빌드를 생성하면 플러그인의 빌드 후처리(`iOSPostProcess.cs`)가 Xcode 프로젝트 디렉토리에 **`Podfile`을 자동 생성**합니다. (이미 `Podfile`이 있으면 건드리지 않습니다) `pod init`을 별도로 실행할 필요가 없습니다.

생성되는 `Podfile`은 다음과 같습니다. nap mx Mediation과 미디에이션에 추가할 네트워크 SDK가 `UnityFramework` 타겟에 포함되어 있으며, 사용하지 않는 어댑터는 지우세요.

```ruby
platform :ios, '13.0'

target 'Unity-iPhone' do
  use_frameworks!
end

target 'UnityFramework' do
  use_frameworks!

  pod 'AdMixerMediation', '<플러그인 대응 버전>'   # 자동 생성된 Podfile 의 값을 그대로 사용

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
> ⚠️ 플러그인의 Swift 브릿지는 `AdMixerMediation`의 API에 직접 의존하므로, 자동 생성된 `Podfile`의 버전을 임의로 올리면 빌드가 실패할 수 있습니다. SDK 업데이트는 플러그인 업데이트와 함께 진행하세요.

### Google 네트워크 - SDK 입찰 광고 소스 설정

Google 네트워크를 사용하시는 경우, SDK 입찰 광고 소스 사용을 위해 아래 광고 소스 라이브러리를 모두 추가해주세요.

- Pangle / AppLovin / DT Exchange / InMobi / Liftoff Monetize / Meta / Moloco / Unity Ads / Mintegral

---

## 3. SDK 연동하기

### 3-1. IOSPostProcess - IDFA

ATT(App Tracking Transparency) 프레임워크를 사용하여 추적 권한을 요청합니다.

광고 요청 시 IDFA를 사용하려면 사용자의 추적 권한 승인 여부를 먼저 확인해야 합니다. 따라서 추적 권한 요청은 반드시 광고 요청 전에 이루어져야 하며, 플러그인의 `iOSATTRequest.mm`이 앱 활성화(`UIApplicationDidBecomeActive`) 1초 후 자동으로 요청합니다. 별도 코드가 필요 없습니다.

`Info.plist`의 추적 동의 문구(`NSUserTrackingUsageDescription`)와 `AppTrackingTransparency.framework` 추가는 `Assets > Editor > iOSPostProcess.cs`가 빌드 시 자동으로 처리합니다. 추적 권한 요청 팝업에 표시되는 문구를 수정하고 싶은 경우, 해당 파일에서 문구를 변경해 주세요.

```csharp
// iOSPostProcess.cs
plist.root.SetString("NSUserTrackingUsageDescription", "사용자에게 최적화된 맞춤형 광고를 제공하기 위해 기기 정보를 사용합니다.");
```

### 3-2. IOSPostProcess - GADApplicationIdentifier

Google AdManager 네트워크를 사용하는 경우, `AdMixer` Inspector의 `iOS GAD Application Id`에 **발급받은 App ID**(`ca-app-pub-xxx~yyy`)를 입력합니다.

빌드 시 `iOSPostProcess.cs`가 이 값을 `Info.plist`의 `GADApplicationIdentifier`로 기록합니다. 비워 두면 기록하지 않습니다.

### 3-3. Delegate 설정

각 광고 타입별 델리게이트가 정상적으로 동작하기 위해서는 아래의 두 가지가 필수로 설정되어 있어야 합니다.

**① GameObject의 이름 설정 및 전달**

네이티브 브릿지는 `UnitySendMessage`로 이벤트를 전달하므로, 수신할 GameObject의 이름을 SDK에 전달해야 합니다.

- `AdMixer` 컴포넌트를 사용하는 경우 — `Awake()`에서 자기 GameObject 이름을 **자동으로** 전달합니다. 별도 코드가 필요 없습니다.
- `NAPSSPPluginIOS`를 직접 호출하는 경우 — 초기화 전에 `SetUnityCallbackHandler`로 이름을 전달합니다.

```csharp
void Awake()
{
    gameObject.name = "NAPSSPPluginIOS";
    NAPSSPPluginIOS.SetUnityCallbackHandler("NAPSSPPluginIOS");
}
```

> ⚠️ 이름을 전달한 뒤 **Inspector에서 해당 GameObject의 이름을 변경하면 이벤트가 수신되지 않습니다.**

**② GameObject에 ssp plugin 파일 추가**

nap mx의 delegate를 이용하려면 `Assets/Scripts` 폴더 밑에 있는 `NAPSSPPluginIOS.cs` 파일을 ①에서 이름을 전달한 GameObject에 추가해주세요.

- `AdMixer` 컴포넌트를 사용하는 경우 — `Awake()`에서 **자동으로** 추가됩니다.
- 직접 호출하는 경우 — Inspector에서 컴포넌트를 추가하거나 `AddComponent<NAPSSPPluginIOS>()`를 호출합니다.

**③ 이벤트 구독**

이벤트는 `NAPSSPPluginIOS`의 **C# `static event`** 로 제공됩니다. 원하는 이벤트에 핸들러를 구독하세요. 이벤트 목록은 광고 타입별 페이지를 참고하세요.

```csharp
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
```

---

## 4. SDK 초기화

반드시 한 번 초기화 호출이 필요합니다. 광고 호출 전 앱에서 1회 초기화해주세요.

- **MEDIA_KEY**: nap mx 파트너 사이트에서 발급받은 미디어 키 (**int**)
- **ADUNIT**: nap mx 파트너 사이트에서 발급받은 애드유닛 ID 리스트 (**int[]**)

### 방법 A — `AdMixer` 컴포넌트 (권장)

씬에 GameObject를 만들고 `AdMixer.cs` 컴포넌트를 추가한 뒤, Inspector에 `iOS MediaKey`와 사용할 광고 타입의 `iOS ... AdUnitId`를 입력합니다. `Awake()`에서 자동으로 초기화되며, 씬에 **하나만** 존재해야 합니다 (`DontDestroyOnLoad` 처리).

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

`Awake()` 시점에 다음이 자동으로 수행됩니다.

- 이벤트 수신 오브젝트 등록 (`AdMixer` GameObject 자신)
- Inspector의 Adunit ID 전체로 SDK 초기화
- 배너 · 동영상 · 네이티브 뷰 인스턴스 생성 (`*ViewInit`) 및 델리게이트 등록

### 방법 B — `NAPSSPPluginIOS` 직접 호출

`AdMixer` 없이 직접 제어하려면 `NAPSSPPluginIOS`의 정적 메서드를 호출합니다.

```csharp
using UnityEngine;

public class AdManager : MonoBehaviour
{
    void Awake()
    {
        // 이벤트 수신 설정 (3-3 참고)
        gameObject.name = "NAPSSPPluginIOS";
        gameObject.AddComponent<NAPSSPPluginIOS>();
        NAPSSPPluginIOS.SetUnityCallbackHandler("NAPSSPPluginIOS");

        int[] adUnitIds = { ADUNIT_ID_BANNER, ADUNIT_ID_INTERSTITIAL, ADUNIT_ID_NATIVE };
        NAPSSPPluginIOS.Initialize(MEDIA_KEY, adUnitIds);
    }
}
```

### 미디에이션 네트워크 초기화 (선택)

일부 네트워크는 앱 시작 시 초기화 함수를 호출해야 합니다. SDK 초기화 이후에 호출하세요.

```csharp
// Google AdManager 초기화 (해당 네트워크 사용 시)
NAPSSPPluginIOS.initGAM();
// Pangle 초기화 (해당 네트워크 사용 시, 앱 ID 값 적용 필수)
NAPSSPPluginIOS.initPangle("앱 ID");
// AppLovin 초기화 (해당 네트워크 사용 시, SDK Key 값 적용 필수)
NAPSSPPluginIOS.initAppLovin("sdkKey");
// UnityAds 초기화 (해당 네트워크 사용 시, Game ID 값 적용 필수)
NAPSSPPluginIOS.initUnityAds("게임 ID");
```

> Pangle, UnityAds App ID 발급은 nap mx 운영팀([nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr))으로 문의해주세요.
>
> AppLovin SDK Key: `nObIkviLd_FQIkP6yMGsTI7vKdDheVRJfwRkxzH7ie0T2o2slTnPIBcbTRelfXPuwGQcPf2bVGKTtaxtTrR0c9`

---

## 광고 타입별 흐름

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

## 다음 단계

- [배너 - Unity](/ios/unity/banner)
- [네이티브 - Unity](/ios/unity/native-ad)
- [리워드 동영상 - Unity](/ios/unity/rewarded-video)
- [동영상 - Unity](/ios/unity/video)
