# WebBridge — 하이브리드 앱 연동 (Android)

> ℹ️ WebBridge 연동 전, [Android SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

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

> 💡 다이어그램의 `NapMxBridge.*`는 Step 2의 `nap-mx-bridge.js` 래퍼입니다. iOS에는 `window.NapMxBridge` 객체가 없고 래퍼가 `window.webkit.messageHandlers.<메서드명>` 호출로 변환하므로, 웹 페이지는 네이티브 객체를 직접 호출하지 않고 항상 래퍼를 통해 호출합니다.

| 계층 | Android |
|------|---------|
| JS → Native | `@JavascriptInterface` (SDK 내장 `AMMWebBridge`) |
| Native → JS | `webView.evaluateJavascript()` |
| Bridge 객체 | `window.NapMxBridge` |
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

## Step 1. Android 네이티브 구현

SDK가 `AMMWebBridge`를 제공하므로 매체에서 `@JavascriptInterface` 핸들러를 직접 작성할 필요가 없습니다. WebView와 함께 브릿지를 생성하고 `attach()`를 호출하면 됩니다.

```java
import android.app.Activity;
import android.os.Bundle;
import android.webkit.WebView;

import com.nasmedia.admixerssp.ads.AMMWebBridge;

public class WebBridgeActivity extends Activity {

    private WebView webView;
    private AMMWebBridge bridge;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_web_bridge);

        webView = findViewById(R.id.web_view);
        webView.getSettings().setJavaScriptEnabled(true);

        // 브릿지 생성 후 attach — 페이지 로드 전후 어느 시점이든 무관합니다.
        bridge = new AMMWebBridge(webView, this);
        if (!bridge.attach()) {
            // 같은 WebView에 다른 브릿지가 이미 연결된 경우 false
            return;
        }

        webView.loadUrl("https://example.com/your-page.html");
    }

    @Override
    protected void onDestroy() {
        // 화면 종료 시 해제 — 보관 중인 광고를 모두 정리하고 인터페이스 등록을 해제합니다.
        if (bridge != null) {
            bridge.detach();
        }
        super.onDestroy();
    }
}
```

`activity_web_bridge.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <WebView
        android:id="@+id/web_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</FrameLayout>
```

| API | 설명 |
|-----|------|
| `AMMWebBridge(WebView, Activity)` | 브릿지 생성. 전달한 Activity 위에 전체 화면 광고가 표시되며, 참조는 약하게 보관됩니다. |
| `boolean attach()` | `window.NapMxBridge` 등록. 동일 WebView에 다른 브릿지가 살아 있으면 `false`를 반환하고 기존 연결을 유지합니다. |
| `void detach()` | 보관 중인 광고 전체 해제 + 인터페이스 등록 해제. |
| `void destroyAll()` | 보관 중인 광고만 해제(인터페이스 등록은 유지). JS `destroyAll()`과 동일합니다. |

> ⚠️ 생성자와 `attach()` / `detach()` / `destroyAll()`는 **메인 스레드**에서 호출해야 합니다. 다른 스레드에서 호출하면 `IllegalStateException`이 발생합니다.

> ℹ️ WebView에서 JavaScript가 활성화되어 있어야 합니다(`setJavaScriptEnabled(true)`).

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

    // 브릿지 탑재 여부 — 브릿지가 없는 앱에서는 아래 호출이 에러나 콜백 없이 무시되므로 호출 전에 확인합니다.
    // iOS는 messageHandlers 존재만으로는 판단할 수 없어(앱의 다른 핸들러) 메서드 단위로 확인합니다.
    // 브릿지는 모든 메서드 핸들러를 한 번에 등록하므로 하나만 확인하면 충분합니다.
    const isAvailable = () => isIOS()
        ? !!window.webkit.messageHandlers.requestInterstitial
        : !!window.NapMxBridge;

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
        // 브릿지 탑재 여부
        isAvailable,

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
| `adUnitId` | 문자열 또는 숫자 | 전체 | 필수. 정수 형태가 아니면 `errorCode: -1`, `errorMsg: "invalid adUnitId"` 실패 콜백이 전달됩니다. |
| `customParams` | 객체 | 전체 | 선택. 값은 문자열로 정규화되어 `AdInfo.Builder.setCustomParams()`로 전달됩니다. |

```javascript
NapMxBridge.requestInterstitial({
    adUnitId: AD_CONFIG.INTERSTITIAL,
    customParams: { age: "20", gender: "M" }
});
```

#### 브릿지 미탑재 앱 대응

브릿지가 없는 앱(Android SDK v2.2.1 미만, 또는 SDK는 최신이지만 앱이 `AMMWebBridge`를 연결하지 않은 경우)에서는 래퍼의 `request*()` / `show*()` 호출이 **에러나 콜백 없이 무시**됩니다. 타임아웃으로도 감지할 수 없으므로 호출 전에 `NapMxBridge.isAvailable()`로 탑재 여부를 확인하고 대체 동작(광고 진입점 숨김, 웹 광고 노출 등)을 준비하세요.

```javascript
if (!NapMxBridge.isAvailable()) {
    // 브릿지가 없는 앱 버전 — 광고 버튼을 숨기거나 대체 광고를 노출
    document.getElementById('btn-watch').style.display = 'none';
} else {
    NapMxBridge.requestRewardVideo({ adUnitId: "YOUR_REWARD_ADUNIT_ID" });
}
```

> ⚠️ iOS에서 `window.webkit.messageHandlers` 존재 여부만으로 판단하면 앱의 다른 메시지 핸들러 때문에 브릿지가 있다고 오판합니다. `isAvailable()`은 메서드 단위(`requestInterstitial`)로 확인합니다.

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

#### errorCode

`errorCode`는 iOS·Android 공통 값입니다. SDK 원본 오류는 `nativeErrorCode` / `nativeErrorMsg` 필드에 함께 전달되므로 로그·문의용으로만 사용하고, 분기는 `errorCode`로 하세요.

> ⚠️ `nativeErrorCode`는 플랫폼마다 출처가 다릅니다. Android는 항상 SDK 코드(`AdMixer.AX_ERR_*`)이며 광고 네트워크 원본 코드·메시지는 `nativeErrorMsg` 문자열 안에 있습니다. iOS는 광고 네트워크 원본 코드(없으면 SDK 코드)입니다. 이 값으로 분기하지 마세요.

| errorCode | errorMsg | 발생 상황 | 권장 처리 |
|-----------|----------|-----------|-----------|
| `-1` | `invalid adUnitId` | `adUnitId`가 없거나 정수 형태가 아님 | 요청 파라미터 확인 |
| `-1` | `ad is not ready. request first` | 로드되지 않은 상태에서 `show*()` 호출 | `*Loaded` 수신 후 `show*()` |
| `-10` | `unsupported format` | 미지원 포맷(배너·네이티브·인라인 동영상) 요청 | 전체 화면 3종만 사용 |
| `-20` | `no fill` | 모든 광고 네트워크에서 광고 없음 | 잠시 후 재요청 |
| `-21` | `ad unit unavailable` | 광고 유닛을 확정할 수 없음 (서버 설정에 유닛 없음 또는 설정 수신 실패·타임아웃) | 유닛 설정 확인 후 1회 재시도 |
| `-22` | `load timed out` | 로드 전체 시간 초과 | 재요청 |
| `-23` | `show failed` | 로드된 광고의 표시 실패 | 재요청 |
| `-30` | `sdk error` | 그 외 SDK 오류 | 로그 후 재요청 |

```json
{
    "adUnitId": "ADUNIT_ID",
    "adapterName": "",
    "errorCode": -20,
    "errorMsg": "no fill",
    "timestamp": 1718089200000,
    "nativeErrorCode": -2147483640,
    "nativeErrorMsg": "No ad filled after repeated reloads."
}
```

> ⚠️ 통일 errorCode는 **Android SDK v2.3.0 이상**에서 전달됩니다. v2.2.2 이하의 브릿지는 `-1`·`-10` 외의 값을 광고 네트워크 원본 코드 그대로 `errorCode`에 전달하므로, 앱의 SDK 버전을 v2.3.0 이상으로 맞추세요.


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
- 로드가 진행 중인 포맷에 다시 `request*()`를 호출하면 중복 요청은 무시됩니다. 이전 요청의 콜백을 기다리세요. 브릿지가 in-flight 상태를 직접 관리하므로 웹에서 별도 상태 조회를 할 필요가 없습니다.
- 로드가 끝난 뒤 같은 포맷을 다시 요청하면 이전 광고는 해제되고 새 광고로 교체됩니다.
- 리워드 지급 판정은 `onRewardEarned`로만 하고, 콜백의 `transactionId`로 서버 지급 원장과 대사하세요. `onRewardVideoCompleted`는 광고 네트워크에 따라 발생하지 않을 수 있습니다. `transactionId`는 노출당 1개 발급되며 서버 earned 로그·매체 콜백 URL의 `transaction_id`와 같은 값입니다. `transactionId`가 없는 `onRewardEarned`는 정상 노출 경로에서는 발생하지 않으므로 지급을 보류하고 확인하세요.
- `*Failed`는 로드 실패, 표시 실패, 미준비 상태의 `show*()`(-1), `adUnitId` 오류(-1) 모두에서 전달됩니다. 로드 실패는 사용자에게 광고가 보이기 전이므로, 실패를 사용자에게 안내할지는 `*Showed` 수신 여부와 `errorCode`로 구분해 매체 정책으로 결정하세요.

### Lifecycle 관리

| 이벤트 | 처리 |
|--------|------|
| 화면 종료 | `onDestroy()`에서 `bridge.detach()` |
| 광고만 정리 | `bridge.destroyAll()` 또는 웹에서 `NapMxBridge.destroyAll()` |

> ⚠️ 화면 종료 시 `detach()`를 호출하지 않으면 광고 객체가 해제되지 않습니다.

### Context

전체 화면 광고 노출에는 Activity가 필요합니다. 브릿지 생성자에 전달한 Activity가 사용되며, 참조는 약하게 보관되므로 Activity가 종료된 뒤에는 노출 요청이 실패 콜백으로 응답됩니다. `getApplicationContext()`를 전달할 수 없습니다.

### AdUnit ID 관리

```javascript
const AD_CONFIG = {
    INTERSTITIAL:         "your-interstitial-adunit-id",
    REWARD_VIDEO:         "your-reward-adunit-id",
    VIDEO_INTERSTITIAL:   "your-video-interstitial-adunit-id"
};
```

> ⚠️ 하나의 AdUnit ID는 하나의 광고 객체에서만 사용하세요. Media Key는 네이티브 코드(Application)에서 설정합니다.

### ProGuard 설정

추가할 규칙이 없습니다. `AMMWebBridge`와 `@JavascriptInterface` 메서드를 보호하는 규칙이 AAR의 `consumer-rules.pro`에 포함되어 자동 병합됩니다([ProGuard 설정](proguard.md) 참고).
