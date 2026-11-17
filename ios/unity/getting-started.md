# iOS SDK 시작하기 - Unity

Unity 프로젝트에서 nap mx iOS SDK를 연동하기 위한 가이드 문서이며 nap mx Mediation을 지원합니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. SDK 다운로드 및 설치

[GitHub — Unity-SSP-Download](https://github.com/Nasmedia-Tech/Unity-SSP-Download)에서 최신 `NAPSSPSDK-x.y.z.unitypackage`를 다운로드합니다.

다운로드한 `.unitypackage`를 Unity 프로젝트에 추가합니다.

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
| `Assets/Editor/iOSPostProcess.cs` | 빌드 후처리 — `Info.plist` 항목 · `AppTrackingTransparency.framework` · SDK Swift Package 추가 |
| `Assets/Editor/AdMixerGAMBuild.cs` | 빌드 시 GAM App ID 검증 및 `Info.plist` 기록 |

---

## 2. Xcode 설정하기

nap mx SDK는 **iOS deployment target 13.0 이상** 환경에서 동작합니다.

> ⚠️ **Xcode 버전**: Xcode 27(iOS 27 SDK)은 배포 타깃 15.0 미만을 빌드하지 못합니다. 최소 iOS 버전을 13.0~14.x로 두려면 **Xcode 26.x**로 빌드하고, Xcode 27을 쓰려면 Unity Player Settings의 **Target minimum iOS Version**을 15.0 이상으로 올리세요.

### 2-1. Swift Package Manager를 통한 설치 (자동)

Unity에서 iOS 빌드를 생성하면 플러그인의 빌드 후처리(`iOSPostProcess.cs`)가 Xcode 프로젝트에 nap mx SDK와 미디에이션 어댑터를 **Swift Package로 자동 추가**합니다. CocoaPods 설치나 `pod install`은 필요 없습니다. 생성된 `Unity-iPhone.xcodeproj`를 Xcode로 열면 패키지를 자동으로 받습니다(최초 1회 네트워크 필요).

기본으로 추가되는 패키지는 다음과 같습니다. 각 패키지는 같은 메이저 버전 안의 최신 배포 버전을 받습니다.

| 패키지 | 네트워크 |
|---|---|
| `AdMixerMediation` | nap mx Mediation (필수, nap mx SDK 포함) |
| `AdMixerMediationGAM` | Google AdManager |
| `AdMixerMediationPangle` | Pangle |
| `AdMixerMediationAppLovin` | AppLovin |
| `AdMixerMediationUnityAds` | UnityAds |
| `AdMixerMediationAdFit` | Kakao AdFit — **기본 제외** (iOS 15.0 이상 필요) |

사용할 네트워크는 `Assets/Editor/iOSPostProcess.cs`의 `SwiftPackages` 목록에서 켜고 끕니다.

```csharp
// iOSPostProcess.cs — (패키지 URL, 최소 버전, 제품 이름, 필요 최소 iOS, 사용 여부)
new SwiftPackage("https://github.com/Nasmedia-Tech/iOS-SSP-AdFit-SPM.git", "2.0.1", "AdMixerMediationAdFit", "15.0", false),
```

- **Kakao AdFit**을 쓰려면 Target minimum iOS Version을 15.0 이상으로 올린 뒤 해당 항목을 `true`로 바꾸세요. 요구 버전보다 낮으면 빌드 시작 전에 실패합니다.
- 네트워크를 끈 뒤에는 Xcode 프로젝트를 **Replace**로 다시 빌드하세요. 이전 빌드에서 추가된 패키지가 남아 있으면 Append 빌드는 실패합니다.

> ⚠️ **CocoaPods 방식(이전 버전)에서 업그레이드하는 경우** — 이전 플러그인이 만든 `Podfile`에 `AdMixer*` pod이 남아 있으면 SDK가 중복 링크되지 않도록 빌드가 실패합니다. `Podfile`에서 `AdMixer*` pod을 지우고(다른 pod이 없으면 `Podfile`과 `Pods` 폴더 삭제) Xcode 프로젝트를 **Replace**로 다시 빌드하세요.
>
> 플러그인의 Swift 브릿지는 `AdMixerMediation`의 API에 직접 의존합니다. SDK 업데이트는 플러그인 업데이트와 함께 진행하세요.

### 2-2. Google 네트워크 - SDK 입찰 광고 소스 설정

Google 네트워크를 사용하는 경우 SDK 입찰 광고 소스를 쓰려면 아래 광고 소스 라이브러리를 모두 추가해주세요.

- Pangle / AppLovin / DT Exchange / InMobi / Liftoff Monetize / Meta / Moloco / Unity Ads / Mintegral

---

## 3. SDK 연동하기

### 3-1. IOSPostProcess - IDFA

ATT(App Tracking Transparency) 프레임워크로 추적 권한을 요청합니다.

광고 요청에 IDFA를 쓰려면 사용자가 추적을 허용했는지 먼저 확인해야 하므로, 추적 권한은 반드시 광고 요청 전에 요청해야 합니다. 플러그인의 `iOSATTRequest.mm`이 앱 활성화(`UIApplicationDidBecomeActive`) 1초 후 자동으로 요청하므로 별도 코드는 필요 없습니다.

`Info.plist`의 추적 동의 문구(`NSUserTrackingUsageDescription`)와 `AppTrackingTransparency.framework` 추가는 `Assets > Editor > iOSPostProcess.cs`가 빌드 시 자동으로 처리합니다. 추적 권한 요청 팝업의 문구를 바꾸려면 이 파일에서 수정해주세요.

```csharp
// iOSPostProcess.cs
plist.root.SetString("NSUserTrackingUsageDescription", "사용자에게 최적화된 맞춤형 광고를 제공하기 위해 기기 정보를 사용합니다.");
```

### 3-2. IOSPostProcess - GADApplicationIdentifier

Google AdManager 네트워크를 사용하는 경우, `AdMixer` Inspector의 `iOS GAD Application Id`에 **발급받은 App ID**(`ca-app-pub-xxx~yyy`)를 입력합니다.

빌드 시 `AdMixerGAMBuild.cs`가 이 값을 `Info.plist`의 `GADApplicationIdentifier`로 기록합니다.

- GAM 패키지(`AdMixerMediationGAM`)가 포함된 빌드에서 값이 비어 있으면 **빌드가 실패**합니다 (App ID 없이 실행하면 앱 시작 시 크래시하므로 빌드 단계에서 막습니다).
- Google 테스트 App ID(`ca-app-pub-3940256099942544~...`)는 개발 빌드(Development Build)에서만 허용되며, 릴리스 빌드에서는 실패합니다.
- 빌드 씬에 배치된 `AdMixer`에서 값을 읽으므로 `AdMixer`는 코드로 생성하지 말고 **씬에 배치**하세요.

### 3-3. Delegate 설정

광고 타입별 델리게이트가 정상적으로 동작하려면 아래 세 가지를 설정해야 합니다.

**① GameObject의 이름 설정 및 전달**

네이티브 브릿지는 `UnitySendMessage`로 이벤트를 전달하므로, 수신할 GameObject의 이름을 SDK에 전달해야 합니다.

- `AdMixer` 컴포넌트를 사용하는 경우 — `Awake()`에서 자기 GameObject 이름을 **자동으로** 전달하므로 별도 코드는 필요 없습니다.
- `NAPSSPPluginIOS`를 직접 호출하는 경우 — 초기화 전에 `SetUnityCallbackHandler`로 이름을 전달합니다.

```csharp
void Awake()
{
    gameObject.name = "NAPSSPPluginIOS";
    NAPSSPPluginIOS.SetUnityCallbackHandler("NAPSSPPluginIOS");
}
```

> ⚠️ 이름을 전달한 뒤 **Inspector에서 해당 GameObject의 이름을 변경하면 이벤트가 수신되지 않습니다.**

**② GameObject에 SSP 플러그인 파일 추가**

nap mx 델리게이트를 사용하려면 `Assets/Scripts` 폴더의 `NAPSSPPluginIOS.cs` 파일을 ①에서 이름을 전달한 GameObject에 추가해주세요.

- `AdMixer` 컴포넌트를 사용하는 경우 — `Awake()`에서 **자동으로** 추가됩니다.
- 직접 호출하는 경우 — Inspector에서 컴포넌트를 추가하거나 `AddComponent<NAPSSPPluginIOS>()`를 호출합니다.

**③ 이벤트 구독**

이벤트는 `NAPSSPPluginIOS`의 **C# `static event`**로 제공됩니다. 필요한 이벤트에 핸들러를 구독하세요. 이벤트 목록은 광고 타입별 페이지를 참고하세요.

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

광고를 호출하기 전에 앱에서 초기화를 반드시 1회 호출해주세요.

- **MEDIA_KEY**: nap mx 파트너 사이트에서 발급받은 미디어 키 (**int**)
- **ADUNIT**: nap mx 파트너 사이트에서 발급받은 애드유닛 ID 리스트 (**int[]**)

### 4-1. 방법 A — `AdMixer` 컴포넌트 (권장)

씬에 GameObject를 만들고 `AdMixer.cs` 컴포넌트를 추가한 뒤, Inspector에 `iOS MediaKey`와 사용할 광고 타입의 `iOS ... AdUnitId`를 입력합니다. `Awake()`에서 자동으로 초기화됩니다. `DontDestroyOnLoad`로 처리되므로 씬에는 **하나만** 있어야 합니다.

| Inspector 필드 | 타입 | 설명 |
|---|---|---|
| `iOS MediaKey` | int | 발급받은 Media Key |
| `iOS Banner AdUnitId` | int | 배너 Adunit ID |
| `iOS Interstitial AdUnitId` | int | 전면 Adunit ID |
| `iOS Reward AdUnitId` | int | 리워드 동영상 Adunit ID |
| `iOS Video AdUnitId` | int | 동영상 Adunit ID |
| `iOS Video Interstitial AdUnitId` | int | 전면 동영상 Adunit ID |
| `iOS Native AdUnitId` | int | 네이티브 Adunit ID |
| `iOS Banner Top Position` | bool | 배너 위치 (`true`: 상단, `false`: 하단) |
| `iOS Native Top Position` | bool | 네이티브 위치 (`true`: 상단, `false`: 하단) |
| `iOS GAD Application Id` | string | Google AdManager App ID (GAM 사용 시) |

`Awake()`에서 다음 작업이 자동으로 수행됩니다.

- 이벤트 수신 오브젝트 등록 (`AdMixer` GameObject 자신)
- Inspector의 Adunit ID 전체로 SDK 초기화
- 배너 · 동영상 · 네이티브 뷰 인스턴스 생성 (`*ViewInit`) 및 델리게이트 등록

### 4-2. 방법 B — `NAPSSPPluginIOS` 직접 호출

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

### 4-3. 미디에이션 네트워크 초기화 (선택)

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

공통 API는 `AdMixer.Instance`로 호출합니다. Adunit ID는 Inspector 값을 사용하므로 인자로 넘기지 않습니다.

| 광고 타입 | 로드 | 노출 | 제거 |
|-----------|------|------|------|
| 배너 | `LoadBanner()` | `ShowBanner()` | `DestroyBanner()` |
| 전면 | `LoadInterstitial()` | `ShowInterstitial()` | `DestroyInterstitial()` |
| 네이티브 | `LoadNativeAd()` | `ShowNativeAd()` | `DestroyNativeAd()` |
| 리워드 동영상 | `LoadRewardVideo()` | `ShowRewardVideo()` | `DestroyRewardVideo()` |
| 동영상 | `LoadVideoAd()` | `ShowVideoAd()` | `DestroyVideoAd()` |
| 전면 동영상 | `LoadVideoInterstitial()` | `ShowVideoInterstitial()` | `DestroyVideoInterstitial()` |

모든 광고는 `Load` → (로드 성공 이벤트 수신 후) `Show` 순서로 호출합니다. 로드 전이거나 이미 노출 중일 때 `Show`를 호출하면 무시됩니다.

---

## 다음 단계

- [배너 - Unity](/ios/unity/banner)
- [네이티브 - Unity](/ios/unity/native-ad)
- [리워드 동영상 - Unity](/ios/unity/rewarded-video)
- [동영상 - Unity](/ios/unity/video)
