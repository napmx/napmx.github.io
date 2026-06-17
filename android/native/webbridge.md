# WebBridge — 하이브리드 앱 연동 (Android)

> ℹ️ WebBridge 연동 전, [Android SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

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

    // 네이티브
    onNativeLoaded:     function(data) { /* 네이티브 광고 로드 성공 */ },
    onNativeFailed:     function(data) { /* 네이티브 광고 로드 실패 */ },
    onNativeClicked:    function(data) { /* 네이티브 광고 클릭 */ },

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

## Step 3. Android 네이티브 구현

### 3-1. 레이아웃 XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container_webbridge"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```

### 3-2. WebView Activity

```java
public class WebBridgeActivity extends AppCompatActivity {
    private WebView webView;
    private NapMxAdBridgeHandler bridgeHandler;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_webbridge);

        webView = findViewById(R.id.webView);

        // WebView 기본 설정
        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);
        settings.setDomStorageEnabled(true);
        settings.setMediaPlaybackRequiresUserGesture(false);

        // Bridge 핸들러 등록
        bridgeHandler = new NapMxAdBridgeHandler(this, webView);
        webView.addJavascriptInterface(bridgeHandler, "NapMxBridge");

        webView.loadUrl("https://your-web-app.com");
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (bridgeHandler != null) bridgeHandler.onResume();
    }

    @Override
    protected void onPause() {
        if (bridgeHandler != null) bridgeHandler.onPause();
        super.onPause();
    }

    @Override
    protected void onDestroy() {
        if (bridgeHandler != null) {
            bridgeHandler.destroyAll();
            bridgeHandler = null;
        }
        super.onDestroy();
    }
}
```

### 3-3. Bridge 핸들러

> ⚠️ `@JavascriptInterface` 메서드는 WebView의 JS 스레드에서 호출됩니다. 모든 UI 조작은 반드시 `runOnUiThread()`로 래핑하세요.  
> ⚠️ `AdListener`는 내부적으로 `WeakReference`로 보유됩니다. 반드시 **멤버 변수**로 선언하세요.

```java
public class NapMxAdBridgeHandler {
    private final Activity activity;
    private final WebView webView;
    private AMMBannerView banner;
    private AMMNativeAdView nativeAdView;
    private AMMInterstitial loadedInterstitial;
    private AMMRewardVideo loadedRewardVideo;
    private AMMVideoView videoView;
    private AMMVideoInterstitial loadedVideoInterstitial;

    public NapMxAdBridgeHandler(@NonNull Activity activity, @NonNull WebView webView) {
        this.activity = activity;
        this.webView = webView;
    }

    // ── 배너 광고 ─────────────────────────────────

    private final AdListener bannerListener = new AdListener() {
        @Override
        public void onReceivedAd(String adapterName, Object adView) {
            sendCallback("onBannerLoaded", "", adapterName, 0, "");
        }
        @Override
        public void onFailedToReceiveAd(Object adView, String adapterName,
                                        int errorCode, String errorMsg) {
            sendCallback("onBannerFailed", "", adapterName, errorCode, errorMsg);
        }
        @Override
        public void onAdDisplayed() {
            sendCallback("onBannerDisplayed", "", "", 0, "");
        }
        @Override
        public void onAdClicked() {
            sendCallback("onBannerClicked", "", "", 0, "");
        }
    };

    @JavascriptInterface
    public void requestBanner(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");
                String position = params.optString("position", "bottom");

                AdInfo adInfo = new AdInfo.Builder(adUnitId).build();

                // Adfit 사용 시 Activity Context 필수
                banner = new AMMBannerView(activity);
                RelativeLayout.LayoutParams lp = new RelativeLayout.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT
                );

                banner.setAdInfo(adInfo);
                banner.setAdViewListener(bannerListener);

                ViewGroup parent = (ViewGroup) webView.getParent();
                if (parent instanceof RelativeLayout) {
                    if ("top".equals(position)) {
                        lp.addRule(RelativeLayout.ALIGN_PARENT_TOP);
                    } else {
                        lp.addRule(RelativeLayout.ALIGN_PARENT_BOTTOM);
                    }
                    parent.addView(banner, lp);
                }
                banner.loadAd();
            } catch (JSONException e) {
                sendCallback("onBannerFailed", "", "", -1, e.getMessage());
            }
        });
    }

    @JavascriptInterface
    public void hideBanner() {
        activity.runOnUiThread(() -> { if (banner != null) banner.setVisibility(View.GONE); });
    }

    @JavascriptInterface
    public void showBanner() {
        activity.runOnUiThread(() -> { if (banner != null) banner.setVisibility(View.VISIBLE); });
    }

    @JavascriptInterface
    public void destroyBanner() {
        activity.runOnUiThread(() -> {
            if (banner != null) {
                ViewGroup parent = (ViewGroup) banner.getParent();
                if (parent != null) parent.removeView(banner);
                banner.stop();
                banner = null;
            }
        });
    }

    // ── 네이티브 광고 ──────────────────────────────

    private final AdListener nativeListener = new AdListener() {
        @Override
        public void onReceivedAd(String adapterName, Object adView) {
            activity.runOnUiThread(() -> {
                if (nativeAdView != null && nativeAdView.hasAd) {
                    ViewGroup parent = (ViewGroup) webView.getParent();
                    if (parent != null) {
                        parent.removeView(nativeAdView);
                        RelativeLayout.LayoutParams lp = new RelativeLayout.LayoutParams(
                            ViewGroup.LayoutParams.MATCH_PARENT,
                            ViewGroup.LayoutParams.WRAP_CONTENT
                        );
                        lp.addRule(RelativeLayout.ALIGN_PARENT_BOTTOM);
                        parent.addView(nativeAdView, lp);
                    }
                    sendCallback("onNativeLoaded", nativeAdView.getAdInfo().getAdUnitId(), adapterName, 0, "");
                }
            });
        }
        @Override
        public void onFailedToReceiveAd(Object adView, String adapterName, int errorCode, String errorMsg) {
            String adUnitId = nativeAdView != null ? nativeAdView.getAdInfo().getAdUnitId() : "";
            sendCallback("onNativeFailed", adUnitId, adapterName, errorCode, errorMsg);
        }
        @Override
        public void onAdClicked() {
            String adUnitId = nativeAdView != null ? nativeAdView.getAdInfo().getAdUnitId() : "";
            sendCallback("onNativeClicked", adUnitId, "", 0, "");
        }
    };

    @JavascriptInterface
    public void requestNative(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");
                
                NativeAdViewBinder viewBinder = new NativeAdViewBinder.Builder(
                    activity.getResources().getIdentifier("item_native_ad", "layout", activity.getPackageName())
                )
                .setIconImageId(activity.getResources().getIdentifier("nap_mx_iv_icon", "id", activity.getPackageName()))
                .setTitleId(activity.getResources().getIdentifier("nap_mx_tv_title", "id", activity.getPackageName()))
                .setAdvertiserId(activity.getResources().getIdentifier("nap_mx_tv_adv", "id", activity.getPackageName()))
                .setDescriptionId(activity.getResources().getIdentifier("nap_mx_tv_desc", "id", activity.getPackageName()))
                .setMainViewId(activity.getResources().getIdentifier("nap_mx_iv_main", "id", activity.getPackageName()))
                .setCtaId(activity.getResources().getIdentifier("nap_mx_btn_cta", "id", activity.getPackageName()))
                .setPrivacyViewId(activity.getResources().getIdentifier("nap_mx_privacy_container", "id", activity.getPackageName())) // ✅ 선택 (미지정 시 우측 상단 자동 오버레이)
                .build();

                AdInfo adInfo = new AdInfo.Builder(adUnitId).build();

                nativeAdView = new AMMNativeAdView(activity);
                nativeAdView.setAdInfo(adInfo);
                nativeAdView.setViewBinder(viewBinder);
                nativeAdView.setAdViewListener(nativeListener);
                nativeAdView.loadAd();
            } catch (Exception e) {
                sendCallback("onNativeFailed", "", "", -1, e.getMessage());
            }
        });
    }

    // ── 전면 배너 광고 ───────────────────────────

    @JavascriptInterface
    public void requestInterstitial(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");

                AdInfo adInfo = new AdInfo.Builder(adUnitId)
                    .build();

                AMMInterstitial.loadAd(activity, adInfo, new AMMInterstitialLoadCallback() {
                    @Override
                    public void onSuccessLoadInterstitial(@NonNull String adapterName,
                                                          @NonNull AMMInterstitial ad) {
                        loadedInterstitial = ad;
                        ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                            @Override public void onAdShowedFullScreenContent() {
                                sendCallback("onInterstitialShowed", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdClicked() {
                                sendCallback("onInterstitialClicked", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdDismissedFullScreenContent() {
                                loadedInterstitial = null;
                                sendCallback("onInterstitialDismissed", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                                loadedInterstitial = null;
                                sendCallback("onInterstitialFailed", adUnitId, adapterName,
                                    adError.getCode(), adError.getMessage());
                            }
                        });
                        sendCallback("onInterstitialLoaded", adUnitId, adapterName, 0, "");
                    }

                    @Override
                    public void onFailLoadInterstitial(int errorCode, @Nullable String errorMsg) {
                        sendCallback("onInterstitialFailed", adUnitId, "", errorCode,
                            errorMsg != null ? errorMsg : "");
                    }
                });
            } catch (JSONException e) {
                sendCallback("onInterstitialFailed", "", "", -1, e.getMessage());
            }
        });
    }

    @JavascriptInterface
    public void showInterstitial() {
        activity.runOnUiThread(() -> {
            if (loadedInterstitial != null) loadedInterstitial.show(activity);
        });
    }

    // ── 리워드 동영상 광고 ───────────────────────

    @JavascriptInterface
    public void requestRewardVideo(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");
                boolean mute = params.optBoolean("mute", true);

                AdInfo.Builder builder = new AdInfo.Builder(adUnitId).setMute(mute);

                if (params.has("customParams")) {
                    JSONObject customObj = params.getJSONObject("customParams");
                    Map<String, String> customParams = new HashMap<>();
                    Iterator<String> keys = customObj.keys();
                    while (keys.hasNext()) {
                        String key = keys.next();
                        customParams.put(key, customObj.getString(key));
                    }
                    builder.setCustomParams(customParams);
                }

                AMMRewardVideo.loadAd(activity, builder.build(), new AMMRewardVideoLoadCallback() {
                    @Override
                    public void onSuccessLoadReward(@NonNull String adapterName,
                                                    @NonNull AMMRewardVideo ad) {
                        loadedRewardVideo = ad;
                        ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                            @Override public void onAdShowedFullScreenContent() {
                                sendCallback("onRewardVideoShowed", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdCompleted() {
                                sendCallback("onRewardVideoCompleted", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdDismissedFullScreenContent() {
                                loadedRewardVideo = null;
                                sendCallback("onRewardVideoDismissed", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                                loadedRewardVideo = null;
                                sendCallback("onRewardVideoFailed", adUnitId, adapterName,
                                    adError.getCode(), adError.getMessage());
                            }
                        });
                        sendCallback("onRewardVideoLoaded", adUnitId, adapterName, 0, "");
                    }

                    @Override
                    public void onFailLoadReward(int errorCode, @Nullable String errorMsg) {
                        sendCallback("onRewardVideoFailed", adUnitId, "", errorCode,
                            errorMsg != null ? errorMsg : "");
                    }
                });
            } catch (JSONException e) {
                sendCallback("onRewardVideoFailed", "", "", -1, e.getMessage());
            }
        });
    }

    @JavascriptInterface
    public void showRewardVideo() {
        activity.runOnUiThread(() -> {
            if (loadedRewardVideo != null) {
                loadedRewardVideo.show(activity, () ->
                    sendCallback("onRewardEarned", "", "", 0, ""));
            }
        });
    }

    // ── 인라인 동영상 광고 ───────────────────────

    private final AdListener videoListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            activity.runOnUiThread(() -> {
                if (videoView != null) {
                    ViewGroup parent = (ViewGroup) webView.getParent();
                    if (parent != null) {
                        parent.removeView(videoView);
                        RelativeLayout.LayoutParams lp = new RelativeLayout.LayoutParams(
                            ViewGroup.LayoutParams.MATCH_PARENT,
                            ViewGroup.LayoutParams.WRAP_CONTENT
                        );
                        lp.addRule(RelativeLayout.CENTER_IN_PARENT);
                        parent.addView(videoView, lp);
                    }
                    sendCallback("onVideoLoaded", videoView.getAdInfo().getAdUnitId(), adapterName, 0, "");
                }
            });
        }
        @Override
        public void onFailedToReceiveAd(Object adView, String adapterName, int errorCode, String errorMsg) {
            String adUnitId = videoView != null ? videoView.getAdInfo().getAdUnitId() : "";
            sendCallback("onVideoFailed", adUnitId, adapterName, errorCode, errorMsg);
        }
        @Override
        public void onAdCompleted() {
            String adUnitId = videoView != null ? videoView.getAdInfo().getAdUnitId() : "";
            sendCallback("onVideoCompleted", adUnitId, "", 0, "");
        }
        @Override
        public void onAdSkipped() {
            String adUnitId = videoView != null ? videoView.getAdInfo().getAdUnitId() : "";
            sendCallback("onVideoSkipped", adUnitId, "", 0, "");
        }
        @Override
        public void onAdClicked() {
            String adUnitId = videoView != null ? videoView.getAdInfo().getAdUnitId() : "";
            sendCallback("onVideoClicked", adUnitId, "", 0, "");
        }
    };

    @JavascriptInterface
    public void requestVideo(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");

                AdInfo adInfo = new AdInfo.Builder(adUnitId).build();

                videoView = new AMMVideoView(activity);
                videoView.setAdInfo(adInfo);
                videoView.setAdViewListener(videoListener);
                videoView.loadAd();
            } catch (JSONException e) {
                sendCallback("onVideoFailed", "", "", -1, e.getMessage());
            }
        });
    }

    // ── 전면 동영상 광고 ─────────────────────────

    @JavascriptInterface
    public void requestVideoInterstitial(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");

                AdInfo adInfo = new AdInfo.Builder(adUnitId)
                    .build();

                AMMVideoInterstitial.loadAd(activity, adInfo, new AMMVideoInterstitialLoadCallback() {
                    @Override
                    public void onSuccessLoadVideoInterstitial(@NonNull String adapterName,
                                                                @NonNull AMMVideoInterstitial ad) {
                        loadedVideoInterstitial = ad;
                        ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                            @Override public void onAdShowedFullScreenContent() {
                                sendCallback("onVideoInterstitialShowed", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdClicked() {
                                sendCallback("onVideoInterstitialClicked", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdCompleted() {
                                sendCallback("onVideoInterstitialCompleted", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdDismissedFullScreenContent() {
                                loadedVideoInterstitial = null;
                                sendCallback("onVideoInterstitialDismissed", adUnitId, adapterName, 0, "");
                            }
                            @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                                loadedVideoInterstitial = null;
                                sendCallback("onVideoInterstitialFailed", adUnitId, adapterName,
                                    adError.getCode(), adError.getMessage());
                            }
                        });
                        sendCallback("onVideoInterstitialLoaded", adUnitId, adapterName, 0, "");
                    }

                    @Override
                    public void onFailLoadVideoInterstitial(int errorCode, @Nullable String errorMsg) {
                        sendCallback("onVideoInterstitialFailed", adUnitId, "", errorCode,
                            errorMsg != null ? errorMsg : "");
                    }
                });
            } catch (JSONException e) {
                sendCallback("onVideoInterstitialFailed", "", "", -1, e.getMessage());
            }
        });
    }

    @JavascriptInterface
    public void showVideoInterstitial() {
        activity.runOnUiThread(() -> {
            if (loadedVideoInterstitial != null) loadedVideoInterstitial.show(activity);
        });
    }

    // ── Lifecycle & 정리 ─────────────────────────

    @JavascriptInterface
    public void destroyNative() {
        activity.runOnUiThread(() -> {
            if (nativeAdView != null) {
                ViewGroup parent = (ViewGroup) nativeAdView.getParent();
                if (parent != null) parent.removeView(nativeAdView);
                nativeAdView.stop();
                nativeAdView = null;
            }
        });
    }

    @JavascriptInterface
    public void destroyAll() {
        activity.runOnUiThread(() -> {
            destroyBanner();
            destroyNative();
            if (loadedInterstitial != null)      { loadedInterstitial.stop(); loadedInterstitial = null; }
            if (loadedRewardVideo != null)        { loadedRewardVideo.stop(); loadedRewardVideo = null; }
            if (videoView != null)                { videoView.stop(); videoView = null; }
            if (loadedVideoInterstitial != null)  { loadedVideoInterstitial.stop(); loadedVideoInterstitial = null; }
        });
    }

    public void onResume() {
        if (banner != null) banner.onResume();
        if (nativeAdView != null) nativeAdView.onResume();
        if (videoView != null) videoView.onResume();
    }

    public void onPause() {
        if (banner != null) banner.onPause();
        if (nativeAdView != null) nativeAdView.onPause();
        if (videoView != null) videoView.onPause();
    }

    private void sendCallback(String callbackName, String adUnitId,
                               String adapterName, int errorCode, String errorMsg) {
        String json = String.format(
            "{\"adUnitId\":\"%s\",\"adapterName\":\"%s\",\"errorCode\":%d,\"errorMsg\":\"%s\",\"timestamp\":%d}",
            adUnitId, adapterName, errorCode,
            errorMsg != null ? errorMsg.replace("\"", "\\\"") : "",
            System.currentTimeMillis()
        );
        activity.runOnUiThread(() ->
            webView.evaluateJavascript(
                String.format("javascript:if(window.NapMxBridgeCallback&&window.NapMxBridgeCallback.%s)" +
                    "{window.NapMxBridgeCallback.%s(%s);}", callbackName, callbackName, json),
                null)
        );
    }
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

### 네이티브 광고

```html
<script>
window.NapMxBridgeCallback = {
    onNativeLoaded:  function(d) { console.log('네이티브 로드 성공:', d.adapterName); },
    onNativeFailed:  function(d) { console.error('네이티브 실패:', d.errorCode); },
    onNativeClicked: function(d) { console.log('네이티브 클릭'); }
};

NapMxBridge.requestNative({
    adUnitId: "YOUR_NATIVE_ADUNIT_ID"
});
</script>
```

### 인라인 동영상 광고

```html
<script>
window.NapMxBridgeCallback = {
    onVideoLoaded:    function(d) { console.log('동영상 로드 성공'); },
    onVideoFailed:    function(d) { console.error('동영상 로드 실패'); },
    onVideoCompleted: function(d) { console.log('동영상 재생 완료'); },
    onVideoSkipped:   function(d) { console.log('동영상 스킵됨'); },
    onVideoClicked:   function(d) { console.log('동영상 클릭(더보기)'); }
};

NapMxBridge.requestVideo({
    adUnitId: "YOUR_VIDEO_ADUNIT_ID"
});
</script>
```

### 전면 동영상 광고

```html
<script>
window.NapMxBridgeCallback = {
    onVideoInterstitialLoaded: function(d) {
        NapMxBridge.showVideoInterstitial(); // 즉시 노출 또는 원하는 시점에 호출
    },
    onVideoInterstitialCompleted: function(d) {
        console.log('전면 동영상 재생 완료');
    },
    onVideoInterstitialDismissed: function(d) {
        console.log('전면 동영상 닫힘');
    }
};

NapMxBridge.requestVideoInterstitial({
    adUnitId: "YOUR_VIDEO_INTERSTITIAL_ADUNIT_ID"
});
</script>
```

---

## 주의사항

### Lifecycle 관리

| 이벤트 | Android |
|--------|---------|
| 화면 전환 | `onPause()` / `onResume()` 에서 Bridge 핸들러 호출 |
| 화면 종료 | `onDestroy()`에서 `destroyAll()` |

> ⚠️ 네이티브 광고 객체는 반드시 화면 종료 시 해제해야 합니다.

### Context (Android)

| 항목 | 권장 | 주의 |
|------|------|------|
| 배너 생성 | Activity Context (Adfit 필수) | `getApplicationContext()` 사용 금지 |
| 전면/리워드 `show()` | Activity 인스턴스 | Fragment Context 전달 시 이슈 가능 |

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

### ProGuard 설정 (Android)

```proguard
# nap ssp Core (필수)
-keep class com.nasmedia.admixerssp.** { *; }

# WebBridge Handler
-keep class com.your.package.NapMxAdBridgeHandler { *; }
-keepclassmembers class com.your.package.NapMxAdBridgeHandler {
    @android.webkit.JavascriptInterface <methods>;
}
```
