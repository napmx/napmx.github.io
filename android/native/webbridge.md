# WebBridge ???òÏù¥Î∏åÎ¶¨?????∞Îèô

> ?πÔ∏è WebBridge ?∞Îèô ?? [SDK ?úÏûë?òÍ∏∞](getting-started.md)??Step 1~4 ?§Ï†ï???ÑÎ£å?òÏóà?îÏ? ?ïÏù∏?òÏÑ∏??

WebBridge???òÏù¥Î∏åÎ¶¨????WebView Í∏∞Î∞ò) ?òÍ≤Ω?êÏÑú nap ssp ?§Ïù¥?∞Î∏å SDKÎ•?JavaScript BridgeÎ•??µÌï¥ ?∏Ï∂ú?òÏó¨ Í¥ëÍ≥†Î•??úÏãú?òÎäî ?∞Îèô Î∞©Ïãù?ÖÎãà??

> ?í° **?∏Ï∂ú Î∞©Ïãù**  
> Î∞∞ÎÑà¬∑?§Ïù¥?∞Î∏å¬∑?∏Îùº???ôÏòÅ?ÅÏ? **WebView ?ÑÏóê ?§Ïù¥?∞Î∏å Î∑∞Î? ?§Î≤Ñ?àÏù¥**?òÎäî Î∞©Ïãù?ÖÎãà?? HTML ?¥Î???ÏßÅÏ†ë Í¥ëÍ≥†Î•??åÎçîÎßÅÌïò??Í≤ÉÏù¥ ?ÑÎãà?? ?§Ïù¥?∞Î∏å Î∑∞Í? WebView ?ÑÏóê Í≤πÏ≥ê???úÏãú?©Îãà??

---

## ?ÑÌÇ§?çÏ≤ò

```
[Web JS] ?Ä?Ä?Ä?Ä NapMxBridge.requestBanner() ?Ä?Ä?Ä?Ä??[Native Bridge Handler]
                                                        ??                                                        ??                                                  [nap ssp SDK]
                                                   loadAd() / load()
                                                        ??[Web JS] ?ê‚??Ä NapMxBridgeCallback.onBannerLoaded() ?Ä?Ä?Ä?Ä  ??                                                        ??[Web JS] ?Ä?Ä?Ä?Ä NapMxBridge.showInterstitial() ?Ä?Ä?Ä?Ä?Ä??[SDK show()]
```

| Í≥ÑÏ∏µ | Android | iOS |
|------|---------|-----|
| JS ??Native | `@JavascriptInterface` | `WKScriptMessageHandler` |
| Native ??JS | `webView.evaluateJavascript()` | `webView.evaluateJavaScript()` |
| Bridge ?¥Î¶Ñ | `NapMxBridge` | `napMxBridge` (message handler) |
| ÏΩúÎ∞± Í∞ùÏ≤¥ | `window.NapMxBridgeCallback` | `window.NapMxBridgeCallback` |

---

## ÏßÄ??Í¥ëÍ≥† ?¨Îß∑

| Í¥ëÍ≥† ?¨Îß∑ | ?∏Ï∂ú Î∞©Ïãù | JS ?îÏ≤≠ | JS ?úÏãú |
|-----------|-----------|---------|---------|
| Î∞∞ÎÑà | ?§Ïù¥?∞Î∏å Î∑??§Î≤Ñ?àÏù¥ | `requestBanner()` | ?êÎèô (addView ???∏Ï∂ú) |
| ?ÑÎ©¥ Î∞∞ÎÑà | ?ÑÏ≤¥ ?îÎ©¥ ?ùÏóÖ | `requestInterstitial()` | `showInterstitial()` |
| ?§Ïù¥?∞Î∏å | ?§Ïù¥?∞Î∏å Î∑??§Î≤Ñ?àÏù¥ | `requestNative()` | ?êÎèô |
| Î¶¨Ïõå???ôÏòÅ??| ?ÑÏ≤¥ ?îÎ©¥ ?ôÏòÅ??| `requestRewardVideo()` | `showRewardVideo()` |
| ?∏Îùº???ôÏòÅ??| ?§Ïù¥?∞Î∏å Î∑??§Î≤Ñ?àÏù¥ | `requestVideo()` | ?êÎèô |
| ?ÑÎ©¥ ?ôÏòÅ??| ?ÑÏ≤¥ ?îÎ©¥ ?ôÏòÅ??| `requestVideoInterstitial()` | `showVideoInterstitial()` |

---

## Step 1. ?åÎû´???µÌï© JS ?òÌçº

Android?Ä iOS???∏Ï∂ú Î∞©Ïãù Ï∞®Ïù¥Î•?Ï∂îÏÉÅ?îÌïò??JS ?òÌçº?ÖÎãà?? ???òÏù¥ÏßÄ?êÏÑú?????òÌçºÎ•??µÌï¥ ?åÎû´?ºÏùÑ ?†Í≤Ω ?∞Ï? ?äÍ≥† ?∏Ï∂ú?????àÏäµ?àÎã§.

#### `nap-mx-bridge.js`

```javascript
/**
 * nap ssp WebBridge ???åÎû´???µÌï© ?òÌçº
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
        // Í¥ëÍ≥† ?îÏ≤≠
        requestBanner:             (params) => call("requestBanner", params),
        requestInterstitial:       (params) => call("requestInterstitial", params),
        requestNative:             (params) => call("requestNative", params),
        requestRewardVideo:        (params) => call("requestRewardVideo", params),
        requestVideo:              (params) => call("requestVideo", params),
        requestVideoInterstitial:  (params) => call("requestVideoInterstitial", params),

        // Í¥ëÍ≥† ?úÏñ¥
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

## Step 2. ÏΩúÎ∞± ?∏Îì§???±Î°ù

?§Ïù¥?∞Î∏å?êÏÑú Í¥ëÍ≥† ?¥Î≤§??Î∞úÏÉù ??JavaScript ?®ÏàòÎ•??∏Ï∂ú?òÏó¨ ???òÏù¥ÏßÄ???åÎ¶Ω?àÎã§.

```javascript
window.NapMxBridgeCallback = {
    // Î∞∞ÎÑà
    onBannerLoaded:     function(data) { /* Î∞∞ÎÑà Î°úÎìú ?±Í≥µ */ },
    onBannerFailed:     function(data) { /* Î∞∞ÎÑà Î°úÎìú ?§Ìå® */ },
    onBannerClicked:    function(data) { /* Î∞∞ÎÑà ?¥Î¶≠ */ },
    onBannerDisplayed:  function(data) { /* Î∞∞ÎÑà ?úÏãú??*/ },

    // ?ÑÎ©¥ Î∞∞ÎÑà
    onInterstitialLoaded:    function(data) { /* Î°úÎìú ?±Í≥µ ??showInterstitial() ?∏Ï∂ú Í∞Ä??*/ },
    onInterstitialFailed:    function(data) { /* Î°úÎìú ?§Ìå® */ },
    onInterstitialShowed:    function(data) { /* ?úÏãú??*/ },
    onInterstitialClicked:   function(data) { /* ?¥Î¶≠ */ },
    onInterstitialDismissed: function(data) { /* ?´Ìûò */ },

    // Î¶¨Ïõå???ôÏòÅ??    onRewardVideoLoaded:    function(data) { /* Î°úÎìú ?±Í≥µ ??showRewardVideo() ?∏Ï∂ú Í∞Ä??*/ },
    onRewardVideoFailed:    function(data) { /* Î°úÎìú ?§Ìå® */ },
    onRewardVideoShowed:    function(data) { /* ?úÏãú??*/ },
    onRewardVideoCompleted: function(data) { /* ?¨ÏÉù ?ÑÎ£å */ },
    onRewardEarned:         function(data) { /* ??Î¶¨Ïõå??ÏßÄÍ∏??úÏ†ê */ },
    onRewardVideoDismissed: function(data) { /* ?´Ìûò */ },

    // ?∏Îùº???ôÏòÅ??    onVideoLoaded:    function(data) { /* Î°úÎìú ?±Í≥µ */ },
    onVideoFailed:    function(data) { /* Î°úÎìú ?§Ìå® */ },
    onVideoCompleted: function(data) { /* ?¨ÏÉù ?ÑÎ£å */ },
    onVideoClicked:   function(data) { /* ?îÎ≥¥Í∏??¥Î¶≠ */ },
    onVideoSkipped:   function(data) { /* ?§ÌÇµ */ },

    // ?ÑÎ©¥ ?ôÏòÅ??    onVideoInterstitialLoaded:    function(data) { /* Î°úÎìú ?±Í≥µ */ },
    onVideoInterstitialFailed:    function(data) { /* Î°úÎìú ?§Ìå® */ },
    onVideoInterstitialShowed:    function(data) { /* ?úÏãú??*/ },
    onVideoInterstitialCompleted: function(data) { /* ?¨ÏÉù ?ÑÎ£å */ },
    onVideoInterstitialClicked:   function(data) { /* ?îÎ≥¥Í∏??¥Î¶≠ */ },
    onVideoInterstitialDismissed: function(data) { /* ?´Ìûò */ }
};
```

#### ÏΩúÎ∞± ?∞Ïù¥???ïÏãù

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

## Step 3. Android ?§Ïù¥?∞Î∏å Íµ¨ÌòÑ

### 3-1. ?àÏù¥?ÑÏõÉ XML

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

        // WebView Í∏∞Î≥∏ ?§Ï†ï
        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);
        settings.setDomStorageEnabled(true);
        settings.setMediaPlaybackRequiresUserGesture(false);

        // Bridge ?∏Îì§???±Î°ù
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

### 3-3. Bridge ?∏Îì§??
> ?†Ô∏è `@JavascriptInterface` Î©îÏÑú?úÎäî WebView??JS ?§Î†à?úÏóê???∏Ï∂ú?©Îãà?? Î™®Îì† UI Ï°∞Ïûë?Ä Î∞òÎìú??`runOnUiThread()`Î°??òÌïë?òÏÑ∏??  
> ?†Ô∏è `AdListener`???¥Î??ÅÏúºÎ°?`WeakReference`Î°?Î≥¥Ïú†?©Îãà?? Î∞òÎìú??**Î©§Î≤Ñ Î≥Ä??*Î°??†Ïñ∏?òÏÑ∏??

```java
public class NapMxAdBridgeHandler {
    private final Activity activity;
    private final WebView webView;
    private AMMBannerView banner;
    private AMMInterstitial loadedInterstitial;
    private AMMRewardVideo loadedRewardVideo;
    private AMMVideoView videoView;
    private AMMVideoInterstitial loadedVideoInterstitial;

    public NapMxAdBridgeHandler(@NonNull Activity activity, @NonNull WebView webView) {
        this.activity = activity;
        this.webView = webView;
    }

    // ?Ä?Ä Î∞∞ÎÑà Í¥ëÍ≥† ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä

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

                // Adfit ?¨Ïö© ??Activity Context ?ÑÏàò
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
                banner.destroy();
                banner = null;
            }
        });
    }

    // ?Ä?Ä ?ÑÎ©¥ Î∞∞ÎÑà Í¥ëÍ≥† ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä

    @JavascriptInterface
    public void requestInterstitial(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");
                boolean disableBackKey = params.optBoolean("disableBackKey", true);

                AdInfo adInfo = new AdInfo.Builder(adUnitId)
                    .setDisableBackKey(disableBackKey)
                    .build();

                AMMInterstitial.load(activity, adInfo, new AMMInterstitialLoadCallback() {
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

    // ?Ä?Ä Î¶¨Ïõå???ôÏòÅ??Í¥ëÍ≥† ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä

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

                AMMRewardVideo.load(activity, builder.build(), new AMMRewardVideoLoadCallback() {
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

    // ?Ä?Ä ?ÑÎ©¥ ?ôÏòÅ??Í¥ëÍ≥† ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä

    @JavascriptInterface
    public void requestVideoInterstitial(String jsonParams) {
        activity.runOnUiThread(() -> {
            try {
                JSONObject params = new JSONObject(jsonParams);
                String adUnitId = params.getString("adUnitId");

                AdInfo adInfo = new AdInfo.Builder(adUnitId)
                    .setDisableBackKey(params.optBoolean("disableBackKey", true))
                    .build();

                AMMVideoInterstitial.load(activity, adInfo, new AMMVideoInterstitialLoadCallback() {
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

    // ?Ä?Ä Lifecycle & ?ïÎ¶¨ ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä

    @JavascriptInterface
    public void destroyAll() {
        activity.runOnUiThread(() -> {
            destroyBanner();
            if (loadedInterstitial != null)      { loadedInterstitial.destroy(); loadedInterstitial = null; }
            if (loadedRewardVideo != null)        { loadedRewardVideo.stopRewardVideoAd(); loadedRewardVideo = null; }
            if (videoView != null)                { videoView.destroy(); videoView = null; }
            if (loadedVideoInterstitial != null)  { loadedVideoInterstitial.stopInterstitialVideoAd(); loadedVideoInterstitial = null; }
        });
    }

    public void onResume() {
        if (banner != null) banner.onResume();
        if (videoView != null) videoView.onResume();
    }

    public void onPause() {
        if (banner != null) banner.onPause();
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

## Step 4. iOS ?§Ïù¥?∞Î∏å Íµ¨ÌòÑ

> iOS ÏΩîÎìú??[iOS Native Í∞Ä?¥Îìú](https://napmx.github.io/#/ios/native/getting-started) Í∏∞Ï??ºÎ°ú ?ëÏÑ±?òÏóà?µÎãà??

### 4-1. WKWebView ?§Ï†ï

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

### 4-2. Bridge ?∏Îì§??
```swift
import WebKit
import AdMixerMediation

class NapMxAdBridgeHandler: NSObject, WKScriptMessageHandler {
    weak var viewController: UIViewController?
    weak var webView: WKWebView?

    private var bannerView: AMMBannerView?
    private var interstitial: AMMInterstital?
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

    // ?Ä?Ä Î∞∞ÎÑà ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä
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

    // ?Ä?Ä ?ÑÎ©¥ Î∞∞ÎÑà ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä
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

    // ?Ä?Ä Î¶¨Ïõå???Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä
    private func requestRewardVideo(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        let customParam = params["customParams"] as? [String: String]

        AMMRewardVideo.load(adUnitID: adUnitId, customParam: customParam) { [weak self] reward, error in
            guard let self else { return }
            if let error { self.sendCallback("onRewardVideoFailed", adUnitId: adUnitId, errorMsg: error.localizedDescription); return }
            if let reward { self.rewardVideo = reward; self.rewardVideo?.delegate = self; self.sendCallback("onRewardVideoLoaded", adUnitId: adUnitId) }
        }
    }

    // ?Ä?Ä ?∏Îùº???ôÏòÅ???Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä
    private func requestVideo(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        videoView = AMMVideoView(rootViewController: vc)
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

    // ?Ä?Ä ?ÑÎ©¥ ?ôÏòÅ???Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä
    private func requestVideoInterstitial(params: [String: Any], in vc: UIViewController) {
        let adUnitId = params["adUnitId"] as? String ?? ""
        AMMVideoInterstitial.load(adUnitID: adUnitId) { [weak self] vi, error in
            guard let self else { return }
            if let error { self.sendCallback("onVideoInterstitialFailed", adUnitId: adUnitId, errorMsg: error.localizedDescription); return }
            if let vi { self.videoInterstitial = vi; self.videoInterstitial?.delegate = self; self.sendCallback("onVideoInterstitialLoaded", adUnitId: adUnitId) }
        }
    }

    // ?Ä?Ä ?ïÎ¶¨ ?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä?Ä
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

## ???òÏù¥ÏßÄ ?¨Ïö© ?àÏ†ú

### Î∞∞ÎÑà Í¥ëÍ≥†

```html
<script src="nap-mx-bridge.js"></script>
<script>
window.NapMxBridgeCallback = {
    onBannerLoaded:  function(d) { console.log('Î∞∞ÎÑà Î°úÎìú ?±Í≥µ:', d.adapterName); },
    onBannerFailed:  function(d) { console.error('Î∞∞ÎÑà ?§Ìå®:', d.errorCode); },
    onBannerClicked: function(d) { console.log('Î∞∞ÎÑà ?¥Î¶≠'); }
};

NapMxBridge.requestBanner({
    adUnitId: "YOUR_BANNER_ADUNIT_ID",
    position: "bottom"
});
</script>
```

### ?ÑÎ©¥ Í¥ëÍ≥†

```html
<script>
window.NapMxBridgeCallback = {
    onInterstitialLoaded: function(d) {
        NapMxBridge.showInterstitial();   // Ï¶âÏãú ?∏Ï∂ú ?êÎäî ?êÌïò???úÏ†ê???∏Ï∂ú
    },
    onInterstitialDismissed: function(d) {
        console.log('?ÑÎ©¥ Í¥ëÍ≥† ?´Ìûò');
    }
};

NapMxBridge.requestInterstitial({
    adUnitId: "YOUR_INTERSTITIAL_ADUNIT_ID",
    viewType: "basic"
});
</script>
```

### Î¶¨Ïõå???ôÏòÅ??Í¥ëÍ≥†

```html
<script>
window.NapMxBridgeCallback = {
    onRewardVideoLoaded: function(d) {
        document.getElementById('btn-watch').disabled = false;
    },
    onRewardEarned: function(d) {
        alert('Î≥¥ÏÉÅ ?çÎìù! ÏΩîÏù∏ +100');    // ??Î¶¨Ïõå??ÏßÄÍ∏??úÏ†ê
    }
};

// 1. ÎØ∏Î¶¨ Î°úÎìú
NapMxBridge.requestRewardVideo({ adUnitId: "YOUR_REWARD_ADUNIT_ID", mute: false });

// 2. ?¨Ïö©???¥Î¶≠ ???∏Ï∂ú
document.getElementById('btn-watch').onclick = () => NapMxBridge.showRewardVideo();
</script>
```

---

## Ï£ºÏùò?¨Ìï≠

### Lifecycle Í¥ÄÎ¶?
| ?¥Î≤§??| Android | iOS |
|--------|---------|-----|
| ?îÎ©¥ ?ÑÌôò | `onPause()` / `onResume()` ?êÏÑú Bridge ?∏Îì§???∏Ï∂ú | `viewDidDisappear`?êÏÑú `stop()` ?∏Ï∂ú |
| ?îÎ©¥ Ï¢ÖÎ£å | `onDestroy()`?êÏÑú `destroyAll()` | `viewDidDisappear` + `isMovingFromParent`?êÏÑú `destroyAll()` |

> ?†Ô∏è ?§Ïù¥?∞Î∏å Í¥ëÍ≥† Í∞ùÏ≤¥??Î∞òÎìú???îÎ©¥ Ï¢ÖÎ£å ???¥Ï†ú?¥Ïïº ?©Îãà??

### Context (Android)

| ??™© | Í∂åÏû• | Ï£ºÏùò |
|------|------|------|
| Î∞∞ÎÑà ?ùÏÑ± | Activity Context (Adfit ?ÑÏàò) | `getApplicationContext()` ?¨Ïö© Í∏àÏ? |
| ?ÑÎ©¥/Î¶¨Ïõå??`show()` | Activity ?∏Ïä§?¥Ïä§ | Fragment Context ?ÑÎã¨ ???¥Ïäà Í∞Ä??|

### AdUnit ID Í¥ÄÎ¶?
```javascript
const AD_CONFIG = {
    BANNER:               "your-banner-adunit-id",
    INTERSTITIAL:         "your-interstitial-adunit-id",
    REWARD_VIDEO:         "your-reward-adunit-id",
    VIDEO:                "your-video-adunit-id",
    VIDEO_INTERSTITIAL:   "your-video-interstitial-adunit-id"
};
```

> ?†Ô∏è ?òÎÇò??AdUnit ID???òÎÇò??Í¥ëÍ≥† Í∞ùÏ≤¥?êÏÑúÎß??¨Ïö©?òÏÑ∏?? Media Key???§Ïù¥?∞Î∏å ÏΩîÎìú(Application/AppDelegate)?êÏÑú ?§Ï†ï?©Îãà??

### ProGuard ?§Ï†ï (Android)

```proguard
# nap ssp Core (?ÑÏàò)
-keep class com.nasmedia.admixerssp.** { *; }

# WebBridge Handler
-keep class com.your.package.NapMxAdBridgeHandler { *; }
-keepclassmembers class com.your.package.NapMxAdBridgeHandler {
    @android.webkit.JavascriptInterface <methods>;
}
```
