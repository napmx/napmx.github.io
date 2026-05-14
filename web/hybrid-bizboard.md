# Hybrid App (WebView) 비즈보드

네이티브 앱의 WebView에서 네이버 비즈보드 광고를 연동하는 방법입니다.  
비즈보드는 네이버 성과형DA 광고 상품으로, **NaverAdManager 어댑터**가 필요합니다.

> 비즈보드 연동은 네이버 성과형DA 계정 및 별도 계약이 필요합니다.  
> 문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## Android WebView 설정

비즈보드는 일반 WebView 설정에 추가로 네이버 광고 SDK 초기화가 필요합니다.

```java
WebSettings settings = webView.getSettings();
settings.setJavaScriptEnabled(true);
settings.setDomStorageEnabled(true);
settings.setUserAgentString(
    settings.getUserAgentString() + " NAVERADS"
);
```

---

## iOS WKWebView 설정

```swift
let config = WKWebViewConfiguration()
config.preferences.javaScriptEnabled = true

// 비즈보드용 커스텀 User-Agent
let webView = WKWebView(frame: .zero, configuration: config)
webView.evaluateJavaScript("navigator.userAgent") { result, _ in
    if let ua = result as? String {
        webView.customUserAgent = ua + " NAVERADS"
    }
}
```

---

## 비즈보드 광고 코드

```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.admixer.co.kr/napmx.js" async></script>
</head>
<body>

  <!-- 비즈보드 컨테이너 (권장 사이즈: 320x50 또는 전면) -->
  <div id="napmx-bizboard"></div>

  <script>
    napmx.cmd.push(function() {
      napmx.defineSlot('발급받은_BIZBOARD_ADUNIT_ID', [320, 50], 'napmx-bizboard')
           .setTargeting('adtype', 'bizboard')
           .addService(napmx.pubads());
      napmx.pubads().enableSingleRequest();
      napmx.enableServices();
    });
  </script>

</body>
</html>
```

---

## 주의사항

- 비즈보드는 **네이버 앱 또는 네이버 제휴 지면**에서만 정상 노출됩니다.
- User-Agent에 `NAVERADS` 식별자가 포함되어야 합니다.
- 일반 모바일 웹 환경에서는 비즈보드 대신 일반 배너 광고가 노출될 수 있습니다.
