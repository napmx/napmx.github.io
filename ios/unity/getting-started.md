# iOS SDK 시작하기 - Unity

Unity 프로젝트에서 nap ssp iOS SDK를 연동하기 위한 가이드 문서이며, nap ssp Mediation을 지원합니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. SDK 다운로드 및 설치

GitHub에서 최신 `NAPSSPSDK.unitypackage`를 다운로드합니다.

다운로드한 `.unitypackage`를 Unity 프로젝트 내에 추가합니다.

```
Assets > Import Package > Custom Package > NAPSSPSDK.unitypackage
```

---

## 2. Xcode 설정

nap ssp SDK는 **Xcode 16 이상**, **iOS deployment target 13.0 이상** 환경에서 동작합니다.

nap ssp SDK는 CocoaPods를 지원합니다.

### 2-1. CocoaPods를 통한 설치

CocoaPods가 없는 경우, 설치 후 초기화합니다.

```bash
sudo gem install cocoapods
pod init
```

초기화 시 생성된 `Podfile`에 nap ssp Mediation과 미디에이션에 추가할 네트워크 SDK를 아래와 같이 추가합니다.

```ruby
pod 'NapSSP'

# 미디에이션 네트워크 (선택)
pod 'NapSSP/GoogleAdManager'
pod 'NapSSP/Pangle'
pod 'NapSSP/AppLovin'
pod 'NapSSP/ADfit'
```

pod를 업데이트합니다.

```bash
pod install --repo-update
```

이후 `.xcworkspace`를 열어 남은 설정을 진행해주세요.

### Google 네트워크 - SDK 입찰 광고 소스 설정

Google 네트워크를 사용하시는 경우, SDK 입찰 광고 소스 사용을 위해 아래 광고 소스 라이브러리를 모두 추가해주세요.

- Pangle / AppLovin / DT Exchange / InMobi / Liftoff Monetize / Meta / Moloco / Unity Ads / Mintegral

---

## 3. SDK 연동

### 3-1. IDFA 설정

ATT(App Tracking Transparency) 프레임워크를 사용하여 추적 권한을 요청합니다.

`Info.plist`에 추적 동의 문구를 추가합니다.

```xml
<key>NSUserTrackingUsageDescription</key>
<string>맞춤형 광고 제공을 위해 광고 추적 권한이 필요합니다.</string>
```

> ATT 팝업 실행 로직은 `Assets > Editor > iOSPostProcess.cs` 파일에 자동으로 구현되어 있습니다. 팝업에 표시되는 문구를 수정하고 싶은 경우 해당 파일에서 변경해주세요.

Google AdManager 사용 시:

```xml
<key>GADApplicationIdentifier</key>
<string>발급받은_GAD_APP_ID</string>
```

AppLovin 사용 시:

```xml
<key>AppLovinSdkKey</key>
<string>발급받은 키</string>
```

### 3-2. Delegate 설정

각 광고 타입별 델리게이트가 정상적으로 동작하기 위해 아래 두 가지를 반드시 설정해야 합니다.

**① GameObject 이름 설정 및 전달**

SDK 초기화 시 Delegate를 수신할 GameObject의 이름을 함께 전달합니다.

**② NAPSSPPluginIOS.cs 파일 추가**

nap ssp의 delegate를 이용하려면 `Plugins/IOS` 폴더 하위의 `NAPSSPPluginIOS.cs` 파일을 해당 GameObject에 추가해주세요.

---

## 4. SDK 초기화

반드시 한 번 초기화 호출이 필요합니다. 광고 호출 전 앱에서 1회 호출해주세요.

- **MEDIA_KEY**: nap ssp 파트너 사이트에서 발급받은 미디어 키
- **ADUNIT**: nap ssp 파트너 사이트에서 발급받은 애드유닛 ID 리스트

```csharp
using UnityEngine;

public class AdManager : MonoBehaviour
{
    void Awake()
    {
        string[] adUnitIds = { "발급받은_ADUNIT_ID" };
        NapAdMixer.InitSdk("발급받은_MEDIA_KEY", adUnitIds, gameObject.name);
    }
}
```

> Pangle, UnityAds App ID 발급은 nap mx 운영팀(nap_mx@nasmedia.co.kr)으로 문의해주세요.
>
> AppLovin SDK Key: `nObIkviLd_FQIkP6yMGsTI7vKdDheVRJfwRkxzH7ie0T2o2slTnPIBcbTRelfXPuwGQcPf2bVGKTtaxtTrR0c9`

---

## 광고 타입별 흐름

| 광고 타입 | 인스턴스 생성 | 광고 요청 | 제거 |
|-----------|-------------|----------|------|
| 배너 | `BannerViewInit()` | `LoadBanner()` | `DestroyBanner()` |
| 전면 배너 | `InterstitialInit()` | `LoadInterstitial()` | — |
| 네이티브 | `NativeViewInit()` | `LoadNativeAd()` | `DestroyNativeAd()` |
| 리워드 동영상 | `RewardAdInit()` | `LoadRewardVideo()` | — |
| 동영상 | `VideoViewInit()` | `LoadVideoAd()` | — |
| 전면 동영상 | `VideoInterstitialInit()` | `LoadVideoInterstitial()` | — |

---

## 다음 단계

- [배너 - Unity](/ios/unity/banner)
- [네이티브 - Unity](/ios/unity/native-ad)
- [리워드 동영상 - Unity](/ios/unity/rewarded-video)
- [동영상 - Unity](/ios/unity/video)
