# WebBridge — 하이브리드 앱 연동 (iOS)

> ℹ️ WebBridge 연동 전, 각 플랫폼의 SDK 시작하기([Android](/android/native/getting-started) / [iOS](/ios/native/getting-started))의 Step 1~4 설정이 완료되었는지 확인하세요.

WebBridge는 하이브리드 앱(WebView 기반) 환경에서 nap ssp 네이티브 SDK를 JavaScript Bridge를 통해 호출하여 광고를 표시하는 연동 방식입니다.

> 💡 **노출 방식**  
> 배너·네이티브·인라인 동영상은 **WebView 위에 네이티브 뷰를 오버레이**하는 방식입니다. HTML 내부에 직접 광고를 렌더링하는 것이 아니라, 네이티브 뷰가 WebView 위에 겹쳐져 표시됩니다.

---

## 아키텍처

```
[Web JS] ──── NapMxBridge.requestBanner() ────→ [Native Bridge Handler]
                                                        │
                                                        ▼
                                                  [nap ssp SDK]
                                                   loadAd() / load()
                                                        │
[Web JS] ←── NapMxBridgeCallback.onBannerLoaded() ────  │
                                                        │
[Web JS] ──── NapMxBridge.showInterstitial() ─────→ [SDK show()]
```

| 계층 | Android | iOS |
|------|---------|-----|
| JS → Native | `@JavascriptInterface` | `WKScriptMessageHandler` |
| Native → JS | `webView.evaluateJavascript()` | `webView.evaluateJavaScript()` |
| Bridge 이름 | `NapMxBridge` | `napMxBridge` (message handler) |
| 콜백 객체 | `window.NapMxBridgeCallback` | `window.NapMxBridgeCallback` |

---

## 지원 광고 포맷

| 광고 포맷 | 노출 방식 | JS 요청 | JS 표시 |
|-----------|-----------|---------|---------|
| 배너 | 네이티브 뷰 오버레이 | `requestBanner()` | 자동 (addView 시 노출) |
| 전면 배너 | 전체 화면 팝업 | `requestInterstitial()` | `showInterstitial()` |
| 네이티브 | 네이티브 뷰 오버레이 | `requestNative()` | 자동 |
| 리워드 동영상 | 전체 화면 동영상 | `requestRewardVideo()` | `showRewardVideo()` |
| 인라인 동영상 | 네이티브 뷰 오버레이 | `requestVideo()` | 자동 |
| 전면 동영상 | 전체 화면 동영상 | `requestVideoInterstitial()` | `showVideoInterstitial()` |

---

## Step 1. 플랫폼 통합 JS 래퍼

Android와 iOS의 호출 방식 차이를 추상화하는 JS 래퍼입니다. 웹 페이지에서는 이 래퍼를 통해 플랫폼을 신경 쓰지 않고 호출할 수 있습니다.

#### `nap-mx-bridge.js`

```javascript
/**
 * nap ssp WebBridge — 플랫폼 통합 래퍼
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
        requestBanner:             (params) => call("requestBanner", params),
        requestInterstitial:       (params) => call("requestInterstitial", params),
        requestNative:             (params) => call("requestNative", params),
        requestRewardVideo:        (params) => call("requestRewardVideo", params),
        requestVideo:              (params) => call("requestVideo", params),
        requestVideoInterstitial:  (params) => call("requestVideoInterstitial", params),

        // 광고 제어
        showInterstitial:          () => callNoArgs("showInterstitial"),
        showRewardVideo:           () => callNoArgs("showRewardVideo"),
        showVideoInterstitial:     () => callNoArgs("showVideoInterstitial"),
        hideBanner:                () => callNoArgs("hideBanner"),
        showBanner:                () => callNoArgs("showBanner"),
        destroyBanner:             () => callNoArgs("destroyBanner"),
        destroyAll:                () => callNoArgs("destroyAll")
    };
})();
```

---

## Step 2. 콜백 핸들러 등록

네이티브에서 광고 이벤트 발생 시 JavaScript 함수를 호출하여 웹 페이지에 알립니다.

```javascript
window.NapMxBridgeCallback = {
    // 배너
    onBannerLoaded:     function(data) { /* 배너 로드 성공 */ },
    onBannerFailed:     function(data) { /* 배너 로드 실패 */ },
    onBannerClicked:    function(data) { /* 배너 클릭 */ },
    onBannerDisplayed:  function(data) { /* 배너 표시됨 */ },

    // 전면 배너
    onInterstitialLoaded:    function(data) { /* 로드 성공 → showInterstitial() 호출 가능 */ },
    onInterstitialFailed:    function(data) { /* 로드 실패 */ },
    onInterstitialShowed:    function(data) { /* 표시됨 */ },
    onInterstitialClicked:   function(data) { /* 클릭 */ },
    onInterstitialDismissed: function(data) { /* 닫힘 */ },

    // 리워드 동영상
    onRewardVideoLoaded:    function(data) { /* 로드 성공 → showRewardVideo() 호출 가능 */ },
    onRewardVideoFailed:    function(data) { /* 로드 실패 */ },
    onRewardVideoShowed:    function(data) { /* 표시됨 */ },
    onRewardVideoCompleted: function(data) { /* 재생 완료 */ },
    onRewardEarned:         function(data) { /* ✅ 리워드 지급 시점 */ },
    onRewardVideoDismissed: function(data) { /* 닫힘 */ },

    // 인라인 동영상
    onVideoLoaded:    function(data) { /* 로드 성공 */ },
    onVideoFailed:    function(data) { /* 로드 실패 */ },
    onVideoCompleted: function(data) { /* 재생 완료 */ },
    onVideoClicked:   function(data) { /* 더보기 클릭 */ },
    onVideoSkipped:   function(data) { /* 스킵 */ },

    // 전면 동영상
    onVideoInterstitialLoaded:    function(data) { /* 로드 성공 */ },
    onVideoInterstitialFailed:    function(data) { /* 로드 실패 */ },
    onVideoInterstitialShowed:    function(data) { /* 표시됨 */ },
    onVideoInterstitialCompleted: function(data) { /* 재생 완료 */ },
    onVideoInterstitialClicked:   function(data) { /* 더보기 클릭 */ },
    onVideoInterstitialDismissed: function(data) { /* 닫힘 */ }
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

---

## Step 3. iOS 네이티브 구현

> iOS 코드는 [iOS Native 가이드](https://napmx.github.io/#/ios/native/getting-started) 기준으로 작성되었습니다.

### 3-1. WKWebView 설정

```swift
import UIKit
import WebKit
import AdMixerMediation

class WebBridgeViewController: UIViewController {
    var webView: WKWebView!
    var bridgeHandler: NapMxAdBridgeHandler!

    override func viewDidLoad() {
        super.viewDidLoad()

        let config = WKWebViewConfiguration()
        let ucc = WKUserContentController()

        bridgeHandler = NapMxAdBridgeHandler(viewController: self)

        let messageNames = [
            "requestBanner", "requestInterstitial", "requestNative",
            "requestRewardVideo", "requestVideo", "requestVideoInterstitial",
            "showInterstitial", "showRewardVideo", "showVideoInterstitial",
            "hideBanner", "showBanner", "destroyBanner", "destroyAll"
        ]
        for name in messageNames {
            ucc.add(bridgeHandler, name: name)
        }

        config.userContentController = ucc
        config.allowsInlineMediaPlayback = true

        webView = WKWebView(frame: view.bounds, configuration: config)
        webView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(webView)
        bridgeHandler.webView = webView

        if let url = URL(string: "https://your-web-app.com") {
            webView.load(URLRequest(url: url))
        }
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        if isMovingFromParent || isBeingDismissed {
            bridgeHandler.destroyAll()
        }
    }
}
```

### 3-2. Bridge 핸들러

```swift
import WebKit
import AdMixerMediation

class NapMxAdBridgeHandler: NSObject, WKScriptMessageHandler {
    weak var viewController: UIViewController?
    weak var webView: WKWebView?

    private var bannerView: AMMBannerView?
    private var interstitial: AMMInterstitial?
    private var rewardVideo: AMMRewardVideo?
    private var videoView: AMMVideoAdView?
    private var videoInterstitial: AMMVideoInterstitial?

    init(viewController: UIViewController) {
        self.viewController = viewController
        super.init()
    }

    func userContentController(_ userContentController: WKUserContentController,
                                didReceive message: WKScriptMessage) {
        guard let vc = viewController else { return }
        let params = message.body as? [String: Any] ?? [:]

        switch message.name {
        case "requestBanner":            requestBanner(params: params, in: vc)
        case "requestInterstitial":      requestInterstitial(params: params, in: vc)
        case "requestRewardVideo":       requestRewardVideo(params: params, in: vc)
        case "requestVideo":             requestVideo(params: params, in: vc)
        case "requestVideoInterstitial": requestVideoInterstitial(params: params, in: vc)
        case "showInterstitial":         interstitial?.show(rootViewController: vc)
        case "showRewardVideo":          rewardVideo?.show(rootViewController: vc)
        case "showVideoInterstitial":    videoInterstitial?.show(rootViewController: vc)
        case "hideBanner":               bannerView?.isHidden = true
        case "showBanner":               bannerView?.isHidden = false
        case "destroyBanner":            destroyBannerView()
        case "destroyAll":               destroyAll()
        default: break
        }
    }

    // ── 배너 ─────────────────────────────────────
    private func requestBanner(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        let position = params["position"] as? String ?? "bottom"

        bannerView = AMMBannerView(rootViewController: vc)
        bannerView?.adUnitId = adUnitId
        bannerView?.delegate = self
        guard let banner = bannerView, let parent = webView?.superview else { return }

        banner.translatesAutoresizingMaskIntoConstraints = false
        parent.addSubview(banner)

        if position == "top" {
            NSLayoutConstraint.activate([
                banner.leadingAnchor.constraint(equalTo: parent.leadingAnchor),
                banner.trailingAnchor.constraint(equalTo: parent.trailingAnchor),
                banner.topAnchor.constraint(equalTo: parent.safeAreaLayoutGuide.topAnchor)
            ])
        } else {
            NSLayoutConstraint.activate([
                banner.leadingAnchor.constraint(equalTo: parent.leadingAnchor),
                banner.trailingAnchor.constraint(equalTo: parent.trailingAnchor),
                banner.bottomAnchor.constraint(equalTo: parent.safeAreaLayoutGuide.bottomAnchor)
            ])
        }
        banner.load()
    }

    // ── 전면 배너 ────────────────────────────────
    private func requestInterstitial(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        let viewType = params["viewType"] as? String ?? "basic"

        let config = AMMInterstitialConfig()
        switch viewType {
        case "popup":    config.viewType = .popup
        case "countDown": config.viewType = .countDown
        default:         config.viewType = .basic
        }

        AMMInterstitial.load(adUnitID: adUnitId, config: config) { [weak self] interstitial, error in
            guard let self else { return }
            if let error { self.sendCallback("onInterstitialFailed", adUnitId: adUnitId, errorMsg: error.localizedDescription); return }
            if let interstitial { self.interstitial = interstitial; self.interstitial?.delegate = self; self.sendCallback("onInterstitialLoaded", adUnitId: adUnitId) }
        }
    }

    // ── 리워드 ───────────────────────────────────
    private func requestRewardVideo(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        let customParam = params["customParams"] as? [String: String]

        AMMRewardVideo.load(adUnitID: adUnitId, customParam: customParam) { [weak self] reward, error in
            guard let self else { return }
            if let error { self.sendCallback("onRewardVideoFailed", adUnitId: adUnitId, errorMsg: error.localizedDescription); return }
            if let reward { self.rewardVideo = reward; self.rewardVideo?.delegate = self; self.sendCallback("onRewardVideoLoaded", adUnitId: adUnitId) }
        }
    }

    // ── 인라인 동영상 ────────────────────────────
    private func requestVideo(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        videoView = AMMVideoAdView(rootViewController: vc)
        videoView?.adUnitID = adUnitId
        videoView?.delegate = self
        guard let video = videoView, let parent = webView?.superview else { return }
        video.translatesAutoresizingMaskIntoConstraints = false
        parent.addSubview(video)
        NSLayoutConstraint.activate([
            video.leadingAnchor.constraint(equalTo: parent.leadingAnchor),
            video.trailingAnchor.constraint(equalTo: parent.trailingAnchor),
            video.bottomAnchor.constraint(equalTo: parent.safeAreaLayoutGuide.bottomAnchor),
            video.heightAnchor.constraint(equalToConstant: 200)
        ])
        video.load()
    }

    // ── 전면 동영상 ──────────────────────────────
    private func requestVideoInterstitial(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        AMMVideoInterstitial.load(adUnitID: adUnitId) { [weak self] vi, error in
            guard let self else { return }
            if let error { self.sendCallback("onVideoInterstitialFailed", adUnitId: adUnitId, errorMsg: error.localizedDescription); return }
            if let vi { self.videoInterstitial = vi; self.videoInterstitial?.delegate = self; self.sendCallback("onVideoInterstitialLoaded", adUnitId: adUnitId) }
        }
    }

    // ── 정리 ─────────────────────────────────────
    private func destroyBannerView() { bannerView?.stop(); bannerView?.removeFromSuperview(); bannerView = nil }
    func destroyAll() {
        destroyBannerView()
        interstitial?.stop(); interstitial = nil
        rewardVideo?.stop(); rewardVideo = nil
        videoView?.stop(); videoView?.removeFromSuperview(); videoView = nil
        videoInterstitial?.stop(); videoInterstitial = nil
    }

    private func sendCallback(_ name: String, adUnitId: String = "", adapterName: String = "",
                               errorCode: Int = 0, errorMsg: String = "") {
        let escaped = errorMsg.replacingOccurrences(of: "\"", with: "\\\"")
        let json = "{\"adUnitId\":\"\(adUnitId)\",\"adapterName\":\"\(adapterName)\",\"errorCode\":\(errorCode),\"errorMsg\":\"\(escaped)\",\"timestamp\":\(Int(Date().timeIntervalSince1970 * 1000))}"
        let js = "if(window.NapMxBridgeCallback&&window.NapMxBridgeCallback.\(name)){window.NapMxBridgeCallback.\(name)(\(json));}"
        DispatchQueue.main.async { [weak self] in self?.webView?.evaluateJavaScript(js, completionHandler: nil) }
    }
}

// MARK: - Delegates
extension NapMxAdBridgeHandler: AMMBannerViewDelegate {
    func onSuccessBanner()  { sendCallback("onBannerLoaded") }
    func onFailBanner()     { sendCallback("onBannerFailed") }
    func onTapBanner()      { sendCallback("onBannerClicked") }
}

extension NapMxAdBridgeHandler: AMMInterstitialDelegate {
    func onSuccessShowInterstitial()                { sendCallback("onInterstitialShowed") }
    func onFailShowInterstitial(error: Error?)      { sendCallback("onInterstitialFailed", errorMsg: error?.localizedDescription ?? "") }
    func onTapInterstitial()                        { sendCallback("onInterstitialClicked") }
    func onCloseInterstitial()                      { interstitial = nil; sendCallback("onInterstitialDismissed") }
}

extension NapMxAdBridgeHandler: AMMRewardVideoDelegate {
    func onSuccessShowReward()                      { sendCallback("onRewardVideoShowed") }
    func onFailShowReward(error: Error?)             { sendCallback("onRewardVideoFailed", errorMsg: error?.localizedDescription ?? "") }
    func onCloseRewardVideo()                       { rewardVideo = nil; sendCallback("onRewardVideoDismissed") }
    func onTapRewardVideo()                         { sendCallback("onRewardVideoClicked") }
    func onRewardVideoComplete()                    { sendCallback("onRewardVideoCompleted") }
    func onRewardVideoEarned()                      { sendCallback("onRewardEarned") }
}

extension NapMxAdBridgeHandler: AMMVideoViewDelegate {
    func onSuccessVideo()   { sendCallback("onVideoLoaded") }
    func onFailVideo()      { sendCallback("onVideoFailed") }
    func onSkipVideo()      { sendCallback("onVideoSkipped") }
    func onTapAdViewMore()  { sendCallback("onVideoClicked") }
    func onCompleteVideo()  { sendCallback("onVideoCompleted") }
}

extension NapMxAdBridgeHandler: AMMVideoInterstitialDelegate {
    func onSuccessShowVideoInterstitial()           { sendCallback("onVideoInterstitialShowed") }
    func onFailShowVideoInterstitial(error: Error?) { sendCallback("onVideoInterstitialFailed", errorMsg: error?.localizedDescription ?? "") }
    func onCloseVideoInterstitial()                 { videoInterstitial = nil; sendCallback("onVideoInterstitialDismissed") }
    func onTapVideoInterstitialViewMore()           { sendCallback("onVideoInterstitialClicked") }
    func onCompleteVideoInterstitial()              { sendCallback("onVideoInterstitialCompleted") }
}
```

---

## 웹 페이지 사용 예제

### 배너 광고

```html
<script src="nap-mx-bridge.js"></script>
<script>
window.NapMxBridgeCallback = {
    onBannerLoaded:  function(d) { console.log('배너 로드 성공:', d.adapterName); },
    onBannerFailed:  function(d) { console.error('배너 실패:', d.errorCode); },
    onBannerClicked: function(d) { console.log('배너 클릭'); }
};

NapMxBridge.requestBanner({
    adUnitId: "YOUR_BANNER_ADUNIT_ID",
    position: "bottom"
});
</script>
```

### 전면 광고

```html
<script>
window.NapMxBridgeCallback = {
    onInterstitialLoaded: function(d) {
        NapMxBridge.showInterstitial();   // 즉시 노출 또는 원하는 시점에 호출
    },
    onInterstitialDismissed: function(d) {
        console.log('전면 광고 닫힘');
    }
};

NapMxBridge.requestInterstitial({
    adUnitId: "YOUR_INTERSTITIAL_ADUNIT_ID",
    viewType: "basic"
});
</script>
```

### 리워드 동영상 광고

```html
<script>
window.NapMxBridgeCallback = {
    onRewardVideoLoaded: function(d) {
        document.getElementById('btn-watch').disabled = false;
    },
    onRewardEarned: function(d) {
        alert('보상 획득! 코인 +100');    // ✅ 리워드 지급 시점
    }
};

// 1. 미리 로드
NapMxBridge.requestRewardVideo({ adUnitId: "YOUR_REWARD_ADUNIT_ID", mute: false });

// 2. 사용자 클릭 시 노출
document.getElementById('btn-watch').onclick = () => NapMxBridge.showRewardVideo();
</script>
```

---

## 주의사항

### Lifecycle 관리

| 이벤트 | iOS |
|--------|-----|
| 화면 전환 | `viewDidDisappear`에서 `stop()` 호출 |
| 화면 종료 | `viewDidDisappear` + `isMovingFromParent`에서 `destroyAll()` |

> ⚠️ 네이티브 광고 객체는 반드시 화면 종료 시 해제해야 합니다.

### AdUnit ID 관리

```javascript
const AD_CONFIG = {
    BANNER:               "your-banner-adunit-id",
    INTERSTITIAL:         "your-interstitial-adunit-id",
    REWARD_VIDEO:         "your-reward-adunit-id",
    VIDEO:                "your-video-adunit-id",
    VIDEO_INTERSTITIAL:   "your-video-interstitial-adunit-id"
};
```

> ⚠️ 하나의 AdUnit ID는 하나의 광고 객체에서만 사용하세요. Media Key는 네이티브 코드(Application/AppDelegate)에서 설정합니다.


