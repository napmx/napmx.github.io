# Hybrid App (WebView 지면)

> **필수 설정** — Hybrid App(Web View 지면)의 경우, 광고 타겟팅을 위해 **ADID 전송**과 **Google WEB View API 적용**이 필수로 진행되어야 합니다.

---

## 1. WEB Script 연동

[WEB Script 연동 가이드](/web/script)를 참고하여 웹 페이지에 Script를 구현합니다.

---

## 2. ADID 전송

ADID 전송 시 사용되는 프로토콜은 아래와 같습니다.

- **AOS**: [WEB Script - ADID 전송 가이드](/web/script#adid-전송) 참고
- **iOS**: [WEB Script - iOS 앱 내 웹뷰 ADID 저장 방법](/web/script) 참고

---

## 3. Google WEB View API

Google 수익화를 포함하는 경우 적용해주세요.

> 미적용 시 eCPM 및 Fill-Rate에 영향이 있을 수 있습니다.

공식 가이드:
- Android: https://developers.google.com/ad-manager/mobile-ads-sdk/android/browser/webview/api-for-ads?hl=ko
- iOS: https://developers.google.com/ad-manager/mobile-ads-sdk/ios/browser/webview/api-for-ads?hl=ko

### 적용 프로세스

**1단계. 앱에 Google Ad Manager SDK 적용**

웹뷰 태그에서 앱 신호를 활용하기 위해 SDK 적용이 필요합니다.

**2단계. 앱 설정 파일에 "webview 전용 사용" 값 추가 (초기화 검사 우회)**

- **Android**: `AndroidManifest.xml`에 `<meta-data>` 태그를 추가하여 `APPLICATION_ID` 검사를 우회합니다.

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy"/>
```

- **iOS**: `Info.plist`에 `GADIntegrationManager`를 추가하여 `GADApplicationIdentifier` 검사를 우회합니다.

```xml
<key>GADIntegrationManager</key>
<string>GAM</string>
```

**3단계. 웹 뷰 등록**

해당 WebView를 SDK에 등록하여 광고 이벤트를 SDK가 감지할 수 있도록 설정합니다.

- **Android**: `registerWebView()` 호출

```java
MobileAds.registerWebView(webView);
```

- **iOS**: `register(_:)` 호출

```swift
GADMobileAds.sharedInstance().register(webView)
```

**4단계. WebView에서 광고 포함 웹 페이지 로드**

Google 광고 스크립트가 포함된 웹 페이지를 WebView로 로드합니다.

**5단계. 광고 노출**

웹 페이지 내 Google 광고 태그가 실행되어 광고 이벤트가 발생하면, SDK에 등록된 WebView를 통해 앱 신호가 결합된 상태로 광고가 송출됩니다.
