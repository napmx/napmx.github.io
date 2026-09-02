# WebBridge — 하이브리드 앱 연동 (iOS)

> ℹ️ WebBridge 연동 전, [iOS SDK 시작하기](getting-started.md)의 Step 1~3 설정이 완료되었는지 확인하세요.

WebBridge는 하이브리드 앱(WebView 기반) 환경에서 nap mx 네이티브 SDK를 JavaScript Bridge를 통해 호출하여 광고를 표시하는 연동 방식입니다.

> 💡 **제공 포맷은 전체 화면 광고 3종입니다**  
> 전면, 리워드 동영상, 전면 동영상만 제공합니다. 배너·네이티브·인라인 동영상은 WebBridge로는 제공하지 않습니다. 해당 포맷은 네이티브 SDK 연동을 사용하세요.

---

## 아키텍처

```
[Web JS] ──── NapMxBridge.requestInterstitial() ────→ [AMMWebBridge]
                                                            │
                                                            ▼
                                                      [nap mx SDK]
                                                        loadAd()
                                                            │
[Web JS] ←── NapMxBridgeCallback.onInterstitialLoaded() ──  │
                                                            │
[Web JS] ──── NapMxBridge.showInterstitial() ────────→ [SDK show()]
```

| 계층 | iOS |
|------|-----|
| JS → Native | `WKScriptMessageHandler` (SDK 내장 `AMMWebBridge`) |
| Native → JS | `webView.evaluateJavaScript()` |
| 콜백 객체 | `window.NapMxBridgeCallback` |

---

## 지원 광고 포맷

| 광고 포맷 | 노출 방식 | JS 요청 | JS 표시 |
|-----------|-----------|---------|---------|
| 전면 배너 | 전체 화면 팝업 | `requestInterstitial()` | `showInterstitial()` |
| 리워드 동영상 | 전체 화면 동영상 | `requestRewardVideo()` | `showRewardVideo()` |
| 전면 동영상 | 전체 화면 동영상 | `requestVideoInterstitial()` | `showVideoInterstitial()` |

배너·네이티브·인라인 동영상(`requestBanner` / `requestNative` / `requestVideo`)을 요청하면 광고가 로드되지 않고 **`errorCode: -10`, `errorMsg: "unsupported format"`** 실패 콜백이 즉시 전달됩니다. 요청이 무시되어 응답이 오지 않는 상황은 발생하지 않으므로, 웹에서 대기 상태를 정리할 수 있습니다.

---

## Step 1. iOS 네이티브 구현

SDK가 `AMMWebBridge`를 제공하므로 매체에서 message handler를 직접 등록할 필요가 없습니다. WebView와 함께 브릿지를 생성하고 `attach()`를 호출하면 됩니다.

```swift
import UIKit
import WebKit
import AdMixerMediation

final class WebBridgeViewController: UIViewController {

    private var webView: WKWebView!
    private var bridge: AMMWebBridge!

    override func viewDidLoad() {
        super.viewDidLoad()

        let configuration = WKWebViewConfiguration()
        configuration.allowsInlineMediaPlayback = true

        webView = WKWebView(frame: view.bounds, configuration: configuration)
        webView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(webView)

        // 브릿지 생성 후 attach — 페이지 로드 전후 어느 시점이든 무관합니다.
        bridge = AMMWebBridge(webView: webView, rootViewController: self)
        guard bridge.attach() else {
            // 같은 WKUserContentController에 다른 브릿지가 이미 연결된 경우 false
            return
        }

        if let url = URL(string: "https://example.com/your-page.html") {
            webView.load(URLRequest(url: url))
        }
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)

        // 화면 종료 시 해제 — 보관 중인 광고를 모두 정리하고 핸들러 등록을 해제합니다.
        if isMovingFromParent || isBeingDismissed {
            bridge.detach()
        }
    }
}
```

| API | 설명 |
|-----|------|
| `init(webView:rootViewController:)` | 브릿지 생성. 전달한 뷰 컨트롤러 위에 전체 화면 광고가 표시됩니다. |
| `attach() -> Bool` | 메시지 핸들러 등록. 동일한 `WKUserContentController`에 다른 브릿지가 살아 있으면 `false`를 반환하고 기존 연결을 유지합니다. |
| `detach()` | 보관 중인 광고 전체 해제 + 핸들러 등록 해제. |
| `destroyAll()` | 보관 중인 광고만 해제(핸들러 등록은 유지). JS `destroyAll()`과 동일합니다. |

> ⚠️ 생성과 `attach()` / `detach()` / `destroyAll()` 호출은 **메인 스레드**에서 수행해야 합니다.

---

## Step 2. 플랫폼 통합 JS 래퍼

Android와 iOS의 호출 방식 차이를 추상화하는 JS 래퍼입니다. 웹 페이지에서는 이 래퍼를 통해 플랫폼을 신경 쓰지 않고 호출할 수 있습니다.

#### `nap-mx-bridge.js`

```javascript
/**
 * nap mx WebBridge — 플랫폼 통합 래퍼
 *
 * Android: window.NapMxBridge.methodName(JSON.stringify(params))
 * iOS:     window.webkit.messageHandlers.methodName.postMessage(params)
 */
const NapMxBridge = (() => {
    const isIOS = () => !!(window.webkit && window.webkit.messageHandlers);

    const call = (method, params) => {
        if (isIOS()) {
            if (window.webkit.messageHandlers[method]) {
                window.webkit.messageHandlers[method].postMessage(params || {});
            }
        } else if (window.NapMxBridge) {
            window.NapMxBridge[method](JSON.stringify(params || {}));
        }
    };

    const callNoArgs = (method) => {
        if (isIOS()) {
            if (window.webkit.messageHandlers[method]) {
                window.webkit.messageHandlers[method].postMessage({});
            }
        } else if (window.NapMxBridge) {
            window.NapMxBridge[method]();
        }
    };

    return {
        // 광고 요청
        requestInterstitial:       (params) => call("requestInterstitial", params),
        requestRewardVideo:        (params) => call("requestRewardVideo", params),
        requestVideoInterstitial:  (params) => call("requestVideoInterstitial", params),

        // 광고 표시
        showInterstitial:          () => callNoArgs("showInterstitial"),
        showRewardVideo:           () => callNoArgs("showRewardVideo"),
        showVideoInterstitial:     () => callNoArgs("showVideoInterstitial"),

        // 전체 해제
        destroyAll:                () => callNoArgs("destroyAll")
    };
})();
```

#### 요청 파라미터

| 파라미터 | 타입 | 적용 대상 | 설명 |
|----------|------|-----------|------|
| `adUnitId` | 문자열 또는 숫자 | 전체 | 필수. 숫자로 변환할 수 없으면 `errorCode: -1`, `errorMsg: "invalid adUnitId"` 실패 콜백이 전달됩니다. |
| `customParams` | 객체 | 리워드 동영상 | 선택. 값은 문자열로 정규화되어 전달됩니다. |

---

## Step 3. 콜백 핸들러 등록

네이티브에서 광고 이벤트 발생 시 JavaScript 함수를 호출하여 웹 페이지에 알립니다.

```javascript
window.NapMxBridgeCallback = {
    // 전면 배너
    onInterstitialLoaded:    function(data) { /* 로드 성공 → showInterstitial() 호출 가능 */ },
    onInterstitialFailed:    function(data) { /* 로드 실패 또는 표시 실패 */ },
    onInterstitialShowed:    function(data) { /* 표시됨 */ },
    onInterstitialClicked:   function(data) { /* 클릭 */ },
    onInterstitialDismissed: function(data) { /* 닫힘 */ },

    // 리워드 동영상
    onRewardVideoLoaded:    function(data) { /* 로드 성공 → showRewardVideo() 호출 가능 */ },
    onRewardVideoFailed:    function(data) { /* 로드 실패 또는 표시 실패 */ },
    onRewardVideoShowed:    function(data) { /* 표시됨 */ },
    onRewardVideoCompleted: function(data) { /* 재생 완료 (네트워크에 따라 발생하지 않을 수 있음) */ },
    onRewardEarned:         function(data) { /* ✅ 리워드 지급 시점 — 지급 판정은 이 콜백으로만 합니다 */ },
    onRewardVideoClicked:   function(data) { /* 클릭 */ },
    onRewardVideoDismissed: function(data) { /* 닫힘 */ },

    // 전면 동영상
    onVideoInterstitialLoaded:    function(data) { /* 로드 성공 → showVideoInterstitial() 호출 가능 */ },
    onVideoInterstitialFailed:    function(data) { /* 로드 실패 또는 표시 실패 */ },
    onVideoInterstitialShowed:    function(data) { /* 표시됨 */ },
    onVideoInterstitialCompleted: function(data) { /* 재생 완료 */ },
    onVideoInterstitialClicked:   function(data) { /* 더보기 클릭 */ },
    onVideoInterstitialDismissed: function(data) { /* 닫힘 */ },

    // 미지원 포맷 요청 시 (errorCode: -10)
    onBannerFailed: function(data) { /* requestBanner 요청 시 */ },
    onNativeFailed: function(data) { /* requestNative 요청 시 */ },
    onVideoFailed:  function(data) { /* requestVideo 요청 시 */ }
};
```

#### 콜백 데이터 형식

```json
{
    "adUnitId": "ADUNIT_ID",
    "adapterName": "AdMixer",
    "errorCode": 0,
    "errorMsg": "",
    "timestamp": 1718089200000
}
```

`onRewardEarned`에는 지급 건별 고유 ID가 추가됩니다. 서버 지급 원장과 대사할 때 사용하세요.

```json
{
    "adUnitId": "ADUNIT_ID",
    "adapterName": "AdMixer",
    "errorCode": 0,
    "errorMsg": "",
    "timestamp": 1718089200000,
    "transactionId": "TRANSACTION_ID"
}
```

#### 주요 errorCode

| errorCode | 발생 상황 |
|-----------|-----------|
| `-1` | `adUnitId`가 없거나 숫자로 변환할 수 없음 / 로드되지 않은 상태에서 `show*()` 호출 |
| `-10` | 미지원 포맷(배너·네이티브·인라인 동영상) 요청 |

그 외 값은 광고 네트워크가 반환한 오류 코드입니다.

---

## 웹 페이지 사용 예제

### 전면 광고

```html
<script src="nap-mx-bridge.js"></script>
<script>
window.NapMxBridgeCallback = {
    onInterstitialLoaded: function(data) {
        NapMxBridge.showInterstitial();   // 즉시 노출 또는 원하는 시점에 호출
    },
    onInterstitialFailed: function(data) {
        console.log("전면 실패: " + data.errorCode + " " + data.errorMsg);
    },
    onInterstitialDismissed: function(data) {
        console.log("전면 닫힘");
    }
};

NapMxBridge.requestInterstitial({
    adUnitId: "YOUR_INTERSTITIAL_ADUNIT_ID"
});
</script>
```

### 리워드 동영상 광고

```html
<script src="nap-mx-bridge.js"></script>
<script>
window.NapMxBridgeCallback = {
    onRewardVideoLoaded: function(data) {
        document.getElementById('btn-watch').disabled = false;
    },
    onRewardEarned: function(data) {
        // 리워드 지급 처리 — transactionId로 서버 지급 원장과 대사
        console.log("리워드 지급: " + data.transactionId);
    },
    onRewardVideoDismissed: function(data) {
        document.getElementById('btn-watch').disabled = true;
    },
    onRewardVideoFailed: function(data) {
        console.log("리워드 실패: " + data.errorCode + " " + data.errorMsg);
    }
};

NapMxBridge.requestRewardVideo({
    adUnitId: "YOUR_REWARD_ADUNIT_ID"
});

document.getElementById('btn-watch').onclick = () => NapMxBridge.showRewardVideo();
</script>
```

### 전면 동영상 광고

```html
<script src="nap-mx-bridge.js"></script>
<script>
window.NapMxBridgeCallback = {
    onVideoInterstitialLoaded: function(data) {
        NapMxBridge.showVideoInterstitial();   // 즉시 노출 또는 원하는 시점에 호출
    },
    onVideoInterstitialCompleted: function(data) {
        console.log("재생 완료");
    },
    onVideoInterstitialFailed: function(data) {
        console.log("전면 동영상 실패: " + data.errorCode + " " + data.errorMsg);
    }
};

NapMxBridge.requestVideoInterstitial({
    adUnitId: "YOUR_VIDEO_INTERSTITIAL_ADUNIT_ID"
});
</script>
```

---

## 주의사항

### 로드와 표시

- `show*()`는 해당 포맷의 `*Loaded` 콜백을 받은 뒤에 호출하세요. 준비되지 않은 상태에서 호출하면 `errorCode: -1`, `errorMsg: "ad is not ready. request first"` 실패 콜백이 전달됩니다.
- 로드가 진행 중인 포맷에 다시 `request*()`를 호출하면 중복 요청은 무시됩니다. 이전 요청의 콜백을 기다리세요.
- 로드가 끝난 뒤 같은 포맷을 다시 요청하면 이전 광고는 해제되고 새 광고로 교체됩니다.
- 리워드 지급 판정은 `onRewardEarned`로만 하세요. `onRewardVideoCompleted`는 광고 네트워크에 따라 발생하지 않을 수 있습니다.

### Lifecycle 관리

| 이벤트 | 처리 |
|--------|------|
| 화면 종료 | `viewDidDisappear` + `isMovingFromParent`(또는 `isBeingDismissed`)에서 `bridge.detach()` |
| 광고만 정리 | `bridge.destroyAll()` 또는 웹에서 `NapMxBridge.destroyAll()` |

> ⚠️ 화면 종료 시 `detach()`를 호출하지 않으면 광고 객체가 해제되지 않습니다.

### AdUnit ID 관리

```javascript
const AD_CONFIG = {
    INTERSTITIAL:         "your-interstitial-adunit-id",
    REWARD_VIDEO:         "your-reward-adunit-id",
    VIDEO_INTERSTITIAL:   "your-video-interstitial-adunit-id"
};
```

> ⚠️ 하나의 AdUnit ID는 하나의 광고 객체에서만 사용하세요. Media Key는 네이티브 코드(AppDelegate)에서 설정합니다.
