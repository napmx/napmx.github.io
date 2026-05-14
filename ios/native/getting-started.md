# iOS SDK 시작하기 (Native)

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. CocoaPods 설정

`Podfile`에 SDK를 추가합니다.

```ruby
platform :ios, '13.0'

target 'YourApp' do
  use_frameworks!

  # nap SSP 메인 SDK (필수)
  pod 'NapSSP'

  # --- 선택적 어댑터 ---
  pod 'NapSSP/GoogleAdManager'   # Google Ad Manager
  pod 'NapSSP/Pangle'            # ByteDance Pangle
  pod 'NapSSP/AppLovin'          # AppLovin MAX
  pod 'NapSSP/KakaoAdfit'        # 카카오 ADfit
end
```

```bash
pod install
```

---

## 2. 권한 설정 (Info.plist)

```xml
<!-- 앱 추적 권한 (iOS 14+) -->
<key>NSUserTrackingUsageDescription</key>
<string>맞춤형 광고 제공을 위해 광고 추적 권한이 필요합니다.</string>

<!-- Google AdManager 사용 시 -->
<key>GADApplicationIdentifier</key>
<string>발급받은_GAD_APP_ID</string>
```

---

## 3. 앱 추적 권한 요청 (iOS 14+)

```swift
import AppTrackingTransparency

func requestTrackingAuthorization() {
    if #available(iOS 14, *) {
        ATTrackingManager.requestTrackingAuthorization { status in
            // 권한 요청 후 SDK 초기화
            self.initNapSdk()
        }
    } else {
        initNapSdk()
    }
}
```

---

## 4. SDK 초기화

`AppDelegate`의 `application(_:didFinishLaunchingWithOptions:)` 또는 첫 화면 진입 시 초기화합니다.

```swift
import NapSSP

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        NapAdMixer.shared.initialize(mediaKey: "발급받은_MEDIA_KEY")
        return true
    }
}
```

```objc
// Objective-C
#import <NapSSP/NapSSP.h>

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    [[NapAdMixer shared] initializeWithMediaKey:@"발급받은_MEDIA_KEY"];
    return YES;
}
```

---

## 5. COPPA 설정 (선택)

```swift
// 아동 대상 앱
NapAdMixer.shared.tagForChildDirectedTreatment = true

// 일반 앱
NapAdMixer.shared.tagForChildDirectedTreatment = false
```

---

## 다음 단계

- [배너 광고 연동하기](/ios/native/banner)
- [전면 광고 연동하기](/ios/native/interstitial)
- [네이티브 광고 연동하기](/ios/native/native-ad)
