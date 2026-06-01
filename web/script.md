# WEB Script 연동

nap ssp Script를 mobile web 사이트와 PC web 사이트에 연동하기 위한 가이드 문서입니다.  
연동 및 이용 방법 문의: nap_adx@nasmedia.co.kr

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에 가입 후 미디어 등록 및 애드유닛 생성을 완료하면 연동에 필요한 **Media Key**와 **Adunit ID**를 확인할 수 있습니다.

> nap ssp에서 제공하는 Google 광고를 사용하기 위해서는 Google 광고 진행에 대한 안내사항에 따른 절차가 모두 마무리되어야 합니다.

---

## 1. Script 확인 및 삽입 방법

nap ssp 파트너 사이트([https://publisher.admixer.co.kr/](https://publisher.admixer.co.kr/))에 접속하여 Adunit별 Script를 확인합니다.

- **확인방법 1**: 미디어 > 미디어 관리 > 미디어 상세보기 > 스크립트 확인
- **확인방법 2**: 미디어 > 미디어 관리 > Adunit 상세보기 > 스크립트 확인

확인한 Script를 복사하여 광고가 노출되기 원하는 위치에 삽입합니다.

---

## 2. Parameter 정의

| Parameter | 필수 | 설명 | 값 |
|-----------|------|------|-----|
| `media_key` | ✔️ | 미디어 키 | 파트너 사이트에서 발급 |
| `adunits` | ✔️ | Adunit 목록 (Array Of Objects) | — |
| `adunit_id` | ✔️ | 애드유닛 아이디 | 파트너 사이트에서 발급 |
| `target_id` | ✔️ (인스트림 비디오 제외) | 광고 노출 영역 DIV ID | `admixer_{mediakey}_{adunitID}` |
| `video_id` | ✔️ (인스트림 비디오 시) | 인스트림 영역의 ID | 퍼블리셔가 수동 입력 |
| `close_btn` | — | 닫기 버튼 사용 여부 (Fullscreen·비디오 제외 배너·네이티브에 제공) | `true` / `false` |
| `callback` | — | 광고 노출 성공·실패·비디오 complete 시 호출 | `success`, `fail`, `complete` 함수 |
| `style` | ✔️ (하이브리드 비디오 시) | 스크립트 확인을 통해 복사하여 사용 | — |
| `coppa` | — | 어린이 온라인 사생활 보호법 준수 여부 (미설정 시 `0`) | `0` / `1` |
| `log` | — | 광고 호출·수신·수신실패 등 횟수를 최상단에 표시 | `true` / `false` |

---

## 3. Sample

### 배너, 네이티브 — 기본 (필수 Parameter만)

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID}",
    target_id: "{DIV ID}"
  }]
});
```

### 배너, 네이티브 — custom 설정 1 (close_btn, callback)

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID 1}",
    target_id: "{DIV ID 1}",
    close_btn: false,
    callback: {
      success: () => console.log("퍼블리셔 Callback Success"),
      fail: (error_code, error_msg) => console.log("퍼블리셔 Callback Fail : ", error_code, error_msg)
    }
  }]
});
```

### 배너, 네이티브 — custom 설정 2 (복수 adunit, log, coppa)

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [
    {
      adunit_id: "{ADUNIT ID 1}",
      target_id: "{DIV ID 1}"
    },
    {
      adunit_id: "{ADUNIT ID 2}",
      target_id: "{DIV ID 2}"
    }
  ],
  coppa: 1,
  log: true
});
```

### 배너 — 리워드 유형

버튼 클릭 시 SDK를 호출하여 리워드 광고를 노출하는 방식입니다.  
배너 소재와 비디오 소재가 함께 송출됩니다.

- 배너 소재: 5초 카운트(초수 고정) → 광고 닫기 → 리워드 이벤트 발생
- 비디오 소재: 비디오 광고 노출 → 광고 닫기 → 리워드 이벤트 발생

```html
<!-- 버튼 클릭 시 nap ssp WEB SDK 호출 -->
<button id="testBtn">테스트 버튼 클릭</button>
<div id="{DIV ID}"></div>

<script>
document.getElementById("testBtn").addEventListener("click", function () {
  admixer_m({
    media_key: "{MEDIA KEY}",
    adunits: [{
      adunit_id: "{ADUNIT ID}",
      target_id: "{DIV ID}"
    }]
  });
});

// 리워드 적용 시: nap ssp WEB SDK REWARD 종료 후 message 이벤트 수신
window.addEventListener('message', function(event) {
  if (event.data?.type === 'napsspBR') {
    switch (event.data.msg) {
      case "videoCompleted":
        console.log("비디오 시청완료");
        break;
      case "rewardGranted":
        console.log("리워드 지급");
        break;
      case "adClosed":
        console.log("광고 종료");
        break;
    }
  }
});
</script>
```

### 비디오 — 아웃스트림 비디오 (기본)

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID}",
    target_id: "{DIV ID}"
  }]
});
```

### 비디오 — 아웃스트림 비디오 (close_btn, callback)

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID 1}",
    target_id: "{DIV ID 1}",
    close_btn: false,
    callback: {
      success: () => console.log("퍼블리셔 Callback Success"),
      fail: (error_code, error_msg) => console.log("퍼블리셔 Callback Fail : ", error_code, error_msg),
      complete: () => console.log("비디오 Complete Callback Success")
    }
  }]
});
```

### 비디오 — 아웃스트림 비디오 (다중 Adunit, coppa, log, 하이브리드 비디오용 style)

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [
    {
      adunit_id: "{ADUNIT ID 1}",
      target_id: "{DIV ID 1}"
    },
    {
      adunit_id: "{ADUNIT ID 2}",
      target_id: "{DIV ID 2}",
      style: {"top":"0","left":"10","right":"0","bottom":"10","floating_size":"200","floating_position":"4"}
    }
  ],
  coppa: 1,
  log: true
});
```

### 비디오 - 인스트림 비디오

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID 1}",
    video_id: "{PUBLISHER VIDEO ID}"
  }]
});
```

---

## 4. Android 앱 내 웹뷰를 통해 Script 구현할 경우 가이드

### 기본 WebView 설정

```java
// JavaScript 활성화 (필수)
webView.getSettings().setJavaScriptEnabled(true);
// 로컬스토리지 활성화 (필수)
webView.getSettings().setDomStorageEnabled(true);
```

### 광고 클릭 시 _blank 처리

광고 클릭 후 새 창을 띄워 랜딩페이지로 이동하는 경우, `_blank` 처리를 통해 광고 영역 내에서 랜딩페이지로 이동하는 것을 방지합니다.

### 뒤로가기 버튼 클릭 시 이전 페이지 이동 처리

```java
webView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // 마켓(market://) 및 앱 링크 처리
        if (url.startsWith("market://") || !url.startsWith("http")) {
            try {
                Intent intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME);
                startActivity(intent);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return true;
        }
        return false;
    }
});
```

### ADID 전송

ADID를 localStorage에 저장 후 전송합니다.

```java
new Thread(() -> {
    String adId = "";
    try {
        AdvertisingIdClient.Info adInfo = AdvertisingIdClient.getAdvertisingIdInfo(this);
        if (adInfo != null && !adInfo.isLimitAdTrackingEnabled()) {
            adId = adInfo.getId();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    final String finalAdId = adId;
    runOnUiThread(() -> {
        webView.setWebViewClient(new WebViewClient() {
            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                view.evaluateJavascript(setAppInfo(), null);
            }
            @Override
            public void onPageCommitVisible(WebView view, String url) {
                if (!finalAdId.isEmpty()) {
                    String js = "localStorage.setItem('admixer_adid', '" + finalAdId + "');";
                    view.evaluateJavascript(js, null);
                }
            }
            @Override
            public void onPageFinished(WebView view, String url) {
                view.evaluateJavascript(setAppInfo(), null);
            }
        });
    });
}).start();

// 앱 정보(OS, 모델, OS 버전, 패키지명)를 window.__NAPSSP_INFO__로 전달
private String setAppInfo() {
    try {
        JSONObject obj = new JSONObject();
        obj.put("os", "android");
        obj.put("model", Build.MODEL);
        obj.put("osv", Build.VERSION.RELEASE);
        obj.put("bundle", getPackageName());
        String safeJson = obj.toString();
        return "(function() {" +
            " window.__NAPSSP_INFO__ = window.__NAPSSP_INFO__ || {};" +
            " var data = " + safeJson + ";" +
            " for (var k in data) { window.__NAPSSP_INFO__[k] = data[k]; }" +
            "})();";
    } catch (Exception e) {
        e.printStackTrace();
    }
    return "";
}
```

---

## 5. iOS 앱 내 웹뷰를 통해 Script 구현할 경우 가이드

### ADID 저장

adid 값을 WKWebView의 localStorage에 저장합니다.

```swift
// adid를 localStorage에 저장
let js = "localStorage.setItem('admixer_adid', '\(idfa)');"
webView.evaluateJavaScript(js, completionHandler: nil)
```

### 앱 정보 전달 (WKWebView message handler)

WKWebView에서 웹-앱 통신을 구현합니다.  
message handler: `iOSBridge`, message body: `getAppInfo`

```swift
import WebKit

class ViewController: UIViewController, WKScriptMessageHandler {

    var webView: WKWebView!
    var lastHandledPopupURL: URL?

    override func viewDidLoad() {
        super.viewDidLoad()
        let config = WKWebViewConfiguration()
        config.userContentController.add(self, name: "iOSBridge")
        webView = WKWebView(frame: view.bounds, configuration: config)
        webView.navigationDelegate = self
        webView.uiDelegate = self
        view.addSubview(webView)
    }

    // iOSBridge 메세지 수신 시 앱 정보 전달
    func userContentController(_ userContentController: WKUserContentController,
                                didReceive message: WKScriptMessage) {
        if message.name == "iOSBridge", let body = message.body as? String, body == "getAppInfo" {
            let model = getDeviceModel()
            let osv = UIDevice.current.systemVersion
            let bundle = Bundle.main.bundleIdentifier ?? ""
            let idfa = ASIdentifierManager.shared().advertisingIdentifier.uuidString

            let json = """
            {"os":"ios","model":"\(model)","osv":"\(osv)","bundle":"\(bundle)","idfa":"\(idfa)"}
            """
            let js = "window.onNativeMessage && window.onNativeMessage(\(json));"
            webView.evaluateJavaScript(js, completionHandler: nil)
        }
    }

    private func getDeviceModel() -> String {
        var systemInfo = utsname()
        uname(&systemInfo)
        let machine = systemInfo.machine
        return Mirror(reflecting: machine).children.reduce("") { result, element in
            guard let value = element.value as? Int8, value != 0 else { return result }
            return result + String(UnicodeScalar(UInt8(value)))
        }
    }
}
```

### WKWebView 클릭 동작 최적화

```swift
extension ViewController: WKNavigationDelegate {

    func webView(_ webView: WKWebView,
                 decidePolicyFor navigationAction: WKNavigationAction,
                 decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
        if shouldHandleExternalNavigation(for: navigationAction) {
            return decisionHandler(.cancel)
        }
        decisionHandler(.allow)
    }

    func shouldHandleExternalNavigation(for navigationAction: WKNavigationAction) -> Bool {
        guard let targetURL = navigationAction.request.url else { return false }
        let type = navigationAction.navigationType

        // createWebViewWith에서 이미 처리한 URL이면 내부 로딩 막음
        if let handledURL = lastHandledPopupURL,
           handledURL.absoluteString == targetURL.absoluteString {
            lastHandledPopupURL = nil
            return true
        }
        // 유저가 클릭한 링크로 이동
        if type == .linkActivated {
            UIApplication.shared.open(targetURL)
            return true
        }
        return false
    }
}

extension ViewController: WKUIDelegate {

    func webView(_ webView: WKWebView,
                 createWebViewWith configuration: WKWebViewConfiguration,
                 for navigationAction: WKNavigationAction,
                 windowFeatures: WKWindowFeatures) -> WKWebView? {
        let isPopup = (navigationAction.targetFrame == nil)
        if navigationAction.navigationType == .other && isPopup {
            if let url = navigationAction.request.url {
                lastHandledPopupURL = url
                UIApplication.shared.open(url)
            }
        }
        return nil
    }
}
```

---

## 6. Callback Sample

`callback` 파라미터의 응답이 `fail`인 경우(광고 노출 실패), 특정 URL 혹은 스크립트를 호출하도록 설정합니다.

### 1) iFrame을 특정 URL로 호출하는 경우

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID 1}",
    target_id: "{DIV ID 1}",
    close_btn: false,
    callback: {
      success: () => console.log("퍼블리셔 Callback Success"),
      fail: (error_code, error_msg) => {
        const container = document.getElementById("{DIV ID 1}");
        if (!document.getElementById("{passback_DIV ID 1}")) {
          const iframe = document.createElement("iframe");
          iframe.id = "{passback_DIV ID 1}";
          iframe.src = "{passback URL}";
          iframe.width = "{width}";
          iframe.height = "{height}";
          iframe.frameBorder = "0";
          iframe.allowFullscreen = true;
          container.appendChild(iframe);
        }
      }
    }
  }],
  coppa: 0,
  log: false
});
```

### 2) iFrame 내부에서 특정 스크립트를 load하는 경우

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID 1}",
    target_id: "{DIV ID 1}",
    close_btn: false,
    callback: {
      success: () => console.log("퍼블리셔 Callback Success"),
      fail: (error_code, error_msg) => {
        const container = document.getElementById("{DIV ID 1}");
        if (!document.getElementById("{passback_DIV ID 1}")) {
          const iframe = document.createElement("iframe");
          iframe.id = "{passback_DIV ID 1}";
          iframe.width = "{width}";
          iframe.height = "{height}";
          iframe.frameBorder = "0";
          iframe.allowFullscreen = true;
          container.appendChild(iframe);
          const iframeDocument = iframe.contentDocument || iframe.contentWindow.document;
          const script = document.createElement("script");
          script.src = "{script URL}";
          if (typeof iframeDocument.body === "undefined") {
            iframeDocument.appendChild(document.createElement("body"));
          }
          iframeDocument.body.appendChild(script);
        }
      }
    }
  }],
  coppa: 0,
  log: false
});
```

### 3) iFrame 내부에서 HTML 삽입

```javascript
admixer_m({
  media_key: "{MEDIA KEY}",
  adunits: [{
    adunit_id: "{ADUNIT ID 1}",
    target_id: "{DIV ID 1}",
    close_btn: false,
    callback: {
      success: () => console.log("퍼블리셔 Callback Success"),
      fail: (error_code, error_msg) => {
        const container = document.getElementById("{DIV ID 1}");
        if (!document.getElementById("{passback_DIV ID 1}")) {
          const iframe = document.createElement("iframe");
          iframe.id = "{passback_DIV ID 1}";
          iframe.width = "{width}";
          iframe.height = "{height}";
          iframe.frameBorder = "0";
          iframe.allowFullscreen = true;
          container.appendChild(iframe);
          const iframeDocument = iframe.contentDocument || iframe.contentWindow.document;
          iframeDocument.open();
          iframeDocument.write(`
            <script>
              window.googletag = window.googletag || {cmd: []};
              googletag.cmd.push(function() {
                googletag.defineSlot("Slot ID", [{width}, {height}], "gpt-passback").addService(googletag.pubads());
                googletag.enableServices();
                googletag.display("gpt-passback");
              });
            </script>
          `);
          iframeDocument.close();
        }
      }
    }
  }],
  coppa: 0,
  log: false
});
```
