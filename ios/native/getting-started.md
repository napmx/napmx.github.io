# SDK 시작하기

iOS 앱에 연동하기 위한 가이드 문서이며, nap mx Mediation을 지원합니다.

> 최신 버전의 Admixer SDK와 최신 버전의 Xcode 사용을 권장합니다.  
> Admixer iOS SDK는 **iOS 13.0 이상**, **Xcode 16 이상**(Teads 어댑터 사용 시 Xcode 26 이상) 에서 사용하실 수 있으며, CocoaPods와 SPM을 이용한 설치를 지원합니다.  
> 연동 및 이용 방법 문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에 가입 후 미디어 등록 및 애드유닛 생성을 완료하면 연동에 필요한 **Media Key**와 **Adunit ID**를 확인할 수 있습니다.
**앱 당 1개의 Media key만 사용 가능합니다 **

> 하기 네트워크는 연동을 위해 별도 key 값이 필요합니다. 발급은 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의해주세요.
> - Google App ID / Pangle App ID / Unity Ads App ID

---

## Step 1. SDK 설치

### 1-1. CocoaPods를 통한 설치

CocoaPods가 없는 경우 설치 후 초기화합니다.

```bash
sudo gem install cocoapods
pod init
```

초기화 시 생성된 `Podfile`에 nap mx Mediation과 미디에이션에 추가할 네트워크 SDK를 아래와 같이 추가합니다.

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
  pod 'AdMixerMediationTeads'    # Teads
end
```

pod를 업데이트합니다.

```bash
pod install --repo-update
```

### 1-2. SPM을 통한 설치

nap mx Mediation과 미디에이션에 추가할 네트워크 SDK를 각각 추가합니다.

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
| Teads | 선택 | `https://github.com/Nasmedia-Tech/iOS-SSP-Teads-SPM.git` |

> Unity Ads를 nap mx Mediation과 함께 사용하는 경우, Google SDK 입찰 광고 소스에서 Unity Ads는 중복 추가가 불가합니다.

### 1-3. Google 네트워크 - SDK 입찰 광고 소스 설정

Google AdManager를 미디에이션으로 사용하는 경우, 아래 광고 소스 라이브러리를 모두 추가해야 최적 수익화가 가능합니다.

* [Google 공식 가이드 — 네트워크 선택](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/choose-networks?hl=ko&_gl=1*1mk7mlq*_up*MQ..*_ga*NDk3NjA1MDI0LjE3ODcxODk2MzQ.*_ga_SM8HXJ53K2*czE3ODcxODk2MzMkbzEkZzAkdDE3ODcxODk2MzMkajYwJGwwJGgw)
**추가해야 할 광고 소스 (모두 추가 권장)**

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

**SPM** — 각 광고 소스 의 어댑터를 개별 package dependency로 추가

<p>1단계: Xcode에서 <strong>File &gt; Add Package Dependencies</strong>로 이동합니다.</p>

<p>2단계: 연동하고자 하는 광고 네트워크의 GitHub Repository URL을 입력합니다.</p>

<p>3단계: 중요 업데이트 및 버그 수정을 지속적으로 수신할 수 있도록 Dependency Rule을 <strong>Up to Next Major Version</strong>(권장)으로 설정합니다.</p>

<p>4단계: 앱 타겟(App target)에 추가합니다.</p>

| GitHub Repository URL | 네트워크 | 비고 |
|-----------------------|----------|------|
| `https://github.com/googleads/googleads-mobile-ios-mediation-pangle` | Pangle | nap mx 미디에이션으로 Pangle 사용하는 경우, v7.9.600 버전 고정|
| `https://github.com/googleads/googleads-mobile-ios-mediation-meta` | Meta | |
| `https://github.com/googleads/googleads-mobile-ios-mediation-applovin` | AppLovin | nap mx 미디에이션으로 AppLovin 사용하는 경우, v13.5.100 버전 고정 |
| `https://github.com/googleads/googleads-mobile-ios-mediation-liftoffmonetize` | Liftoff Monetize | |
| `https://github.com/googleads/googleads-mobile-ios-mediation-mintegral` | Mintegral | |
| `https://github.com/googleads/googleads-mobile-ios-mediation-dtexchange` | DT Exchange | |
| `https://github.com/googleads/googleads-mobile-ios-mediation-inmobi` | InMobi | |
| `https://github.com/googleads/googleads-mobile-ios-mediation-moloco` | Moloco | |

### 1-4. 타 네트워크 SDK 버전 정보

| Adapter SDK | 이름 | 버전 | 비고 |
|-------------|------|------|------|
| `AdMixerMediationGAM` | Google-Mobile-Ads-SDK | `12.7.0` 이상 &#126; `13.8` 미만 | |
| `AdMixerMediationAdFit` | AdFitSDK | `3.14.7` 이상 &#126; `3.18.6` 미만 | 최소 지원 OS 14 |
| `AdMixerMediationPangle` | Ads-Global | `7.4.0.8` 이상 &#126; `8.1.1` 미만 | |
| `AdMixerMediationUnityAds` | UnityAds | `4.15.1` 이상 &#126; `4.16.6` 미만 | |
| `AdMixerMediationAppLovin` | AppLovinSDK | `13.3.1` 이상 &#126; `13.5.2` 미만 | |
| `AdMixerMediationNAM` | NAMSDK | `8.0` 이상 &#126; `8.23` 미만 | |
| `AdMixerMediationTeads` | TeadsSDK | `6.2` 이상 &#126; `7.0` 미만 | 최소 지원 OS 14 |

> 기존 적용 중인 네트워크사 버전이 있는 경우, 매체 버전과 nap mx 버전 중 더 낮은 버전으로 탑재됩니다.
> 별도로 사용 중이신 네트워크 버전이 없으신 경우, 범위 내에서 가장 최신 버전으로 탑재됩니다.

<div class="callout info">
  <strong>중복 사용 가능 네트워크</strong>
  <ul>
    <li><strong>Google</strong> — 기존 운영 중인 지면과 다른 지면의 경우 사용 가능합니다.</li>
    <li><strong>Pangle</strong> — 기존 운영 중인 지면과 다른 지면의 경우 사용 가능합니다.</li>
    <li><strong>Adfit</strong> — 네트워크사의 앱 심사 진행 후 사용 가능합니다.</li>
  </ul>
</div>

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

> Google App ID 발급은 nap mx 운영팀([nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr))으로 문의해주세요.

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

## Step 4. 에러 코드

광고 수신에 실패한 경우, delegate 메서드를 통해 nap mx Error 객체를 전달받을 수 있습니다.
각각의 에러에 대한 설명은 아래 표를 참고해 주세요.

| no | 에러 코드 | 설명 |
|----|----------|------|
| 0 | `missingBaseURL` | api 요청에 필요한 base URL이 누락된 경우 |
| 1 | `invalidURLString` | 유효하지 않은 URL로 요청하는 경우 |
| 2 | `invalidServerResponse` | 서버로부터 유효하지 않은 응답을 받은 경우. 네트워크 상태를 확인하거나 관리자에게 문의하세요. |
| 3 | `decodeError` | 데이터 처리에 오류가 있는 경우 |
| 4 | `apiResponseFail` | 서버 통신에 실패한 경우. 서버 상태를 확인하거나 잠시 후 다시 시도해 주세요. |
| 5 | `vastParsingError` | 비디오 광고 데이터 처리에 오류가 있는 경우 |
| 6 | `emptyAd` | 노출 가능한 광고가 없는 경우. 잠시 후 다시 광고 요청을 시도해 주세요. |

---

## 다음 단계

- [배너 광고 연동하기](/ios/native/banner)
- [네이티브 광고 연동하기](/ios/native/native-ad)
- [리워드 동영상 연동하기](/ios/native/rewarded-video)
- [동영상 광고 연동하기](/ios/native/video)
- [비즈보드 연동하기](/ios/native/bizboard)
