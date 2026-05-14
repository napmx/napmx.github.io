# Hybrid App (WebView 지면)

네이티브 앱의 WebView에서 nap mx Script 광고를 연동하는 방법입니다.

> **필수 설정** — WebView에서 광고가 정상 동작하려면 아래 설정이 반드시 필요합니다.

---

## Android WebView 설정

```java
// Java
WebView webView = findViewById(R.id.webView);
WebSettings settings = webView.getSettings();

// JavaScript 활성화 (필수)
settings.setJavaScriptEnabled(true);

// DOM Storage 활성화
settings.setDomStorageEnabled(true);

// 혼합 콘텐츠 허용 (Android 5.0+)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
}

// 광고 페이지 로드
webView.loadUrl("https://your-page.com/ad-page");
```

```kotlin
// Kotlin
val webView: WebView = findViewById(R.id.webView)
webView.settings.apply {
    javaScriptEnabled = true
    domStorageEnabled = true
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW
    }
}
webView.loadUrl("https://your-page.com/ad-page")
```

---

## iOS WKWebView 설정

```swift
import WebKit

class AdWebViewController: UIViewController {

    var webView: WKWebView!

    override func viewDidLoad() {
        super.viewDidLoad()

        let config = WKWebViewConfiguration()
        // JavaScript 허용 (기본값)
        config.preferences.javaScriptEnabled = true
        // 미디어 자동재생 허용 (동영상 광고)
        config.mediaTypesRequiringUserActionForPlayback = []

        webView = WKWebView(frame: view.bounds, configuration: config)
        view.addSubview(webView)

        let url = URL(string: "https://your-page.com/ad-page")!
        webView.load(URLRequest(url: url))
    }
}
```

---

## WEB 페이지 광고 코드

WebView에 로드되는 HTML 페이지에 nap mx Script를 삽입합니다.

```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.admixer.co.kr/napmx.js" async></script>
</head>
<body>

  <div id="napmx-banner"></div>

  <script>
    napmx.cmd.push(function() {
      napmx.defineSlot('발급받은_ADUNIT_ID', [320, 50], 'napmx-banner')
           .addService(napmx.pubads());
      napmx.pubads().enableSingleRequest();
      napmx.enableServices();
    });
  </script>

</body>
</html>
```

---

## AndroidManifest 권한

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

---

## 주의사항

- WebView의 `javaScriptEnabled` 설정이 **반드시 `true`** 여야 합니다.
- 광고 클릭 시 외부 브라우저로 이동하도록 `WebViewClient`를 설정하세요.

```java
webView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (url.startsWith("http")) {
            Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
            startActivity(intent);
            return true;
        }
        return false;
    }
});
```
