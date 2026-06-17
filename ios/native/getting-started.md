# SDK 시작하기

iOS 앱에 연동하기 위한 가이드 문서이며, nap mx Mediation을 지원합니다.

> 최신 버전의 Admixer SDK와 최신 버전의 Xcode 사용을 권장합니다.  
> Admixer iOS SDK는 **iOS 13.0 이상**, **Xcode 15.3 이상** 에서 사용하실 수 있으며, CocoaPods와 SPM을 이용한 설치를 지원합니다.  
> 연동 및 이용 방법 문의: nap_mx@nasmedia.co.kr

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에 가입 후 미디어 등록 및 애드유닛 생성을 완료하면 연동에 필요한 **Media Key**와 **Adunit ID**를 확인할 수 있습니다.
**앱 당 1개의 Media key만 사용 가능합니다 **

> 하기 네트워크는 연동을 위해 별도 key 값이 필요합니다. 발급은 nap_mx@nasmedia.co.kr로 문의해주세요.
> - Google App ID / Pangle App ID / Unity Ads App ID

---

## Step 1. SDK 설치

### 1-1. CocoaPods를 통한 설치

CocoaPods가 없는 경우 설치 후 초기화합니다.

```bash
sudo gem install cocoapods
pod init
```

초기화 시 생성된 `Podfile`에 nap ssp Mediation과 미디에이션에 추가할 네트워크 SDK를 아래와 같이 추가합니다.

```ruby
target 'MyApp' do
  use_frameworks!

  pod 'AdMixerMediation'
  pod 'AdMixerMediationGAM'      # Google AdManager
  pod 'AdMixerMediationAdFit'    # Kakao AdFit
  pod 'AdMixerMediationPangle'   # Pangle
  pod 'AdMixerMediationAppLovin' # AppLovin
  pod 'AdMixerMediationUnityAds' # UnityAds
  pod 'AdMixerMediationNAM'      # Naver AdManager
end
```

pod를 업데이트합니다.

```bash
pod install --repo-update
```

### 1-2. SPM을 통한 설치

nap ssp Mediation과 미디에이션에 추가할 네트워크 SDK를 각각 추가합니다.

**Project > Package Dependencies 탭** 이동 후 아래 패키지를 추가합니다.

| 패키지 | 설명 | Repository URL |
|--------|------|----------------|
| nap mx Mediation (Mediation) | 필수 | `https://github.com/Nasmedia-Tech/iOS-SSP-Mediation-SPM.git` |
| nap mx Mediation (Core) | 필수 | `https://github.com/Nasmedia-Tech/iOS-SSP-SPM.git` |
| Google AdManager | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-GAM-SPM.git` |
| Kakao AdFit | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-AdFit-SPM.git` |
| Pangle | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-Pangle-SPM.git` |
| Unity Ads | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-UnityAds-SPM.git` |
| AppLovin | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-AppLovin-SPM.git` |
| Naver AdManager | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-NAM-SPM` |

> Unity Ads를 nap mx Mediation과 함께 사용하는 경우, Google SDK 입찰 광고 소스에서 Unity Ads는 중복 추가가 불가합니다.

### 1-3. Google 네트워크 - SDK 입찰 광고 소스 설정

Google 네트워크를 사용하는 경우, SDK 입찰 광고 소스 사용을 위해 아래 라이브러리를 추가해주세요.

**CocoaPods** — Podfile에 추가

| Pod | 네트워크 |
|-----|---------|
| `GoogleMobileAdsMediationPangle` | Pangle |
| `GoogleMobileAdsMediationFacebook` | Meta |
| `GoogleMobileAdsMediationAppLovin` | AppLovin |
| `GoogleMobileAdsMediationUnity` | Unity Ads |
| `GoogleMobileAdsMediationVungle` | Liftoff Monetize |
| `GoogleMobileAdsMediationMintegral` | Mintegral |
| `GoogleMobileAdsMediationFyber` | DT Exchange |
| `GoogleMobileAdsMediationInMobi` | InMobi |
| `GoogleMobileAdsMediationMoloco` | Moloco |

> 버전 고정이 필요한 경우: Pangle `v7.9.600`, AppLovin `v13.5.100`

**SPM** — File > Add Package Dependencies에서 GitHub Repository URL 입력 후 Dependency Rule을 `Up to Next Major Version`(권장)으로 설정합니다.

---

## Step 2. SDK 설정

### 추적 권한 요청 (ATT)

`Info.plist`의 `Privacy - Tracking Usage Description`에 사용자에게 보여줄 문구를 입력합니다.

```xml
<key>NSUserTrackingUsageDescription</key>
<string>맞춤형 광고 제공을 위해 광고 추적 권한이 필요합니다.</string>
```

ATT 팝업 실행 코드는 AppDelegate에 아래와 같이 추가합니다.

```swift
import AppTrackingTransparency

func applicationDidBecomeActive(_ application: UIApplication) {
    requestTrackingAuthorization()
}

private func requestTrackingAuthorization() {
    Task {
        _ = await ATTrackingManager.requestTrackingAuthorization()
    }
}
```

### Info.plist 추가 설정

파트너 네트워크의 가이드를 참고하여 SKAdNetwork ID와 연동 전 체크사항들을 확인해주세요.

Google AdManager 사용 시:

```xml
<!-- Ad Manager 앱 ID (형식: ca-app-pub-################~##########) -->
<key>GADApplicationIdentifier</key>
<string>발급받은_GAD_APP_ID</string>
```

> Google App ID 발급은 nap ssp 운영팀(nap_mx@nasmedia.co.kr)으로 문의해주세요.

---

## Step 3. SDK 초기화

반드시 한 번 초기화 호출이 필요합니다. 광고 호출 전 앱에서 1회 호출해주세요.

```swift
import UIKit
import AdMixerMediation
import GoogleMobileAds
import PAGAdSDK
import AppLovinSDK
import UnityAds
import GFPSDK

class AppDelegate: NSObject, UIApplicationDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil) -> Bool {

        // AdMixer 초기화 (필수)
        AMMediation.shared.initialize(
            mediaKey: MEDIA_KEY,
            adunitID: [ADUNIT_ID_BANNER, ADUNIT_ID_INTERSTITIAL_BANNER, ADUNIT_ID_NATIVE]
        )

        // Google AdManager 초기화 (해당 네트워크 사용 시)
        MobileAds.shared.start()

        // Pangle 초기화 (해당 네트워크 사용 시)
        let pangleConfig = PAGConfig.share()
        pangleConfig.appID = "앱ID"
        PAGSdk.start(with: pangleConfig) { isSuccess, error in }

        // AppLovin 초기화 (해당 네트워크 사용 시, key 값 적용 필수)
        let appLovinKey = "nObIkviLd_FQIkP6yMGsTI7vKdDheVRJfwRkxzH7ie0T2o2slTnPIBcbTRelfXPuwGQcPf2bVGKTtaxtTrR0c9"
        let config = ALSdkInitializationConfiguration(sdkKey: appLovinKey)
        ALSdk.shared().initialize(with: config) { _ in }

        // UnityAds 초기화 (해당 네트워크 사용 시)
        UnityAds.initialize("앱ID")

        // Naver AdManager 초기화 (해당 네트워크 사용 시, key 값 적용 필수)
        let namKey = "N248971943"
        GFPAdManager.setup(withPublisherCd: namKey, target: nil) { (error: GFPError?) in
            if let error {
                print("NAM init Error: \(error.description)")
            } else {
                print("NAM init success, isSdkInitialized: \(GFPAdManager.isSdkInitialized())")
            }
        }

        return true
    }
}
```

---

## 다음 단계

- [배너 광고 연동하기](/ios/native/banner)
- [네이티브 광고 연동하기](/ios/native/native-ad)
- [리워드 동영상 연동하기](/ios/native/rewarded-video)
- [동영상 광고 연동하기](/ios/native/video)
- [비즈보드 연동하기](/ios/native/bizboard)
