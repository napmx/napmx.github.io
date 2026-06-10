# v2.0.0 마이그레이션 가이드

이 문서는 nap ssp Android SDK **v1.x → v2.0.0** 업그레이드 시 필요한 변경 사항을 설명합니다.

---

## 빠른 요약

| 구분 | 내용 |
|------|------|
| 새 기능 | NaverAdManager·Teads 어댑터 추가, 광고 신고 및 닫기 버튼 제어 추가 |
| **어댑터 등록 간소화** | **`registerAdapter()` 호출 불필요 — Gradle 의존성 추가만으로 자동 등록** |
| **네이티브 View ID 변경** | **`tv_title` 등 → `nap_mx_tv_title` 등 — 레이아웃 및 ViewBinder 코드 수정 필요** |
| **`setViewIds()` 제거** | **v2.0.0에서 완전 제거 — `NativeAdViewBinder`가 모든 어댑터 View ID 처리** |
| **`setAdapterConfig()` 추가** | **어댑터별 String 초기화 파라미터 설정 (AppLovin `sdkKey` 등)** |
| **전면 BACK 키 기본 차단** | **`AdInfo.Builder.setDisableBackKey` 기본값 `true`로 변경 — 뒤로가기 닫기 의존 시 `false` 명시 필요** |
| **전면 타입 Basic 전용 (Breaking)** | **전면 광고는 Basic만 제공 — `AdInfo.Builder.interstitialAdType`/`popupAdOption`/`setInterstitialAdType`/`setPopupAdOption` 제거. Popup은 내부(서버 설정) 전용, **CountDown 타입은 완전 제거**(관련 상수·뷰 삭제)** |
| 신규 API | `cancelLoad()` (로드만 취소), `AdMixer.setGdprConsent/setUsPrivacy/setCcpaDoNotSell/setTestMode/setTestDeviceIds` (개인정보·테스트 전파), `AdInfo.Builder.showReportIcon/showCloseButton` (광고 제어) |
| Naver PUBLISHER_CD | SDK 제공으로 변경 — 호스트 매니페스트 설정 불필요 |
| **제거된 API (Breaking)** | **`onDestroy()`, `closeInterstitial()`, `AdInfo.Builder.isUseBackgroundAlpha(Boolean)`, `AdMixer.GAUGE`/`AdMixer.TEXT` 등 — v1.x deprecated 별칭을 v2.0.0에서 완전 제거. 정식 메서드로 교체 필요** |
| **AdListener 이벤트 콜백 분리 (Breaking)** | **`onEventAd(AdEvent)` 제거 → `onAdDisplayed`/`onAdClicked`/`onAdClosed`/`onAdCompleted` 및 노출 실패 `onAdShowFailed` 등 이름 있는 메서드. `AdListener`는 abstract class(필요한 것만 override). Step 5-B 참고** |
| ProGuard | 규칙 강화 — 아래 최신 규칙으로 교체 필요 |
| Gradle 버전 | `2.0.0.SNAPSHOT` → `2.0.0` |

---

## Step 1. Gradle 버전 업데이트

`build.gradle` 의존성 버전을 `2.0.0`으로 변경하세요.

```gradle
// Before
implementation 'io.github.nasmedia-tech:admixer-ssp:1.0.21'
implementation 'io.github.nasmedia-tech:admixer-admanager:1.0.21'
// ...

// After
implementation 'io.github.nasmedia-tech:admixer-ssp:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-admanager:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-adfit:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-pangle:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-applovin:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-unity:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-mobwith:2.0.0'
// 신규 — v2.0.0에서 추가됨
implementation 'io.github.nasmedia-tech:admixer-naveradmanager:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-teads:2.0.0'
```

**Mobwith 내장 SDK 버전 변경**: `1.0.2` → `1.0.68`

---

## Step 2. ProGuard 규칙 업데이트

v2.0.0에서 ProGuard 규칙이 강화되었습니다. `proguard-rules.pro`의 기존 AdMixer 관련 규칙을 아래로 교체하세요.

```proguard
# ✅ 필수 — AdMixer Core
-keep class com.nasmedia.admixerssp.** { *; }

# 사용하는 어댑터 모듈만 추가
-keep class com.nasmedia.admanager.** { *; }
-keep class com.nasmedia.adfit.** { *; }
-keep class com.nasmedia.pangle.** { *; }
-keep class com.nasmedia.applovin.** { *; }
-keep class com.nasmedia.unity.** { *; }
-keep class com.nasmedia.mobwith.** { *; }
-keep class com.nasmedia.naveradmanager.** { *; }   # 신규
-keep class com.nasmedia.teads.** { *; }            # 신규
```

> ℹ️ 각 어댑터 AAR에 `consumer-rules.pro`가 포함되어 있어 대부분의 규칙이 자동 적용됩니다. 위 규칙은 추가 안전망입니다.

---

## Step 3. `registerAdapter()` 호출 제거

v2.0.0부터 `initialize()` 내부에서 클래스패스(Gradle 의존성)에 포함된 어댑터를 **자동으로 탐지·등록**합니다. `Application.onCreate()`의 `registerAdapter()` 호출을 모두 제거하세요.

```java
// Before (v1.x) — 수동 등록 필요
AdMixer.registerAdapter(AdMixer.ADAPTER_ADMANAGER);
AdMixer.registerAdapter(AdMixer.ADAPTER_ADFIT);
AdMixer.registerAdapter(AdMixer.ADAPTER_PANGLE);
// ...

// After (v2.0.0) — 불필요, 제거하세요
// (Gradle 의존성에 포함된 어댑터는 initialize() 호출 시 자동 등록)
```

> ℹ️ `registerAdapter()` 메서드는 하위 호환성을 위해 남아 있으며 호출 시 동작은 하지만, `initialize()` 내부에서 `discoverAdapters()`가 이미 모든 어댑터를 자동 등록하므로 중복 호출입니다. 제거를 권장합니다.

---

## Step 4. 새 어댑터 추가 (선택)

### NaverAdManager 추가 시

`build.gradle`에 의존성을 추가하면 어댑터는 `initialize()` 호출 시 자동으로 등록됩니다.

> ℹ️ Naver Ad Manager의 `com.naver.gfpsdk.PUBLISHER_CD`는 nap ssp가 SDK(`admixer-naveradmanager` aar)에서 제공·관리합니다. **호스트 앱 매니페스트에 별도로 설정하지 마세요.** (이전 안내에서 호스트가 Publisher ID를 추가하도록 했으나, SDK 제공 방식으로 변경되었습니다.)

### Teads 추가 시

**`settings.gradle`** Maven 저장소 추가:

```gradle
repositories {
    maven { url "https://sdk.teads.tv/android/repo" }
    maven { url "https://teads.jfrog.io/artifactory/SDKAndroid-maven-prod" }
}
```

어댑터는 `initialize()` 호출 시 자동으로 등록됩니다. 별도의 `registerAdapter()` 호출이 필요하지 않습니다.

---

## Step 5. 제거된 클래스 및 API 교체 (필수 — Breaking Change)

### ⚠️ 구버전 클래스 제거 및 AMM* 클래스 전환

v1.x에서 제공되던 기존 광고 클래스들은 v2.0.0에서 완전히 제거되었습니다. 아래와 같이 새 클래스로 교체하여 빌드 오류를 해결하십시오.

| 구 클래스 (v1.x) | 신 클래스 (v2.0.0) | 설명 |
|---|---|---|
| `AdView` | `AMMBannerView` | 배너 광고 뷰 |
| `NativeAdView` | `AMMNativeAdView` | 네이티브 광고 뷰 |
| `InterstitialAd` | `AMMInterstitial` | 전면 광고 매니저 |
| `RewardInterstitialVideoAd` | `AMMRewardVideo` | 보상형 전면 비디오 광고 매니저 |
| `InterstitialVideoAd` | `AMMVideoInterstitial` | 전면 비디오 광고 매니저 |
| `VideoAdView` | `AMMVideoView` | 비디오 광고 뷰 |

또한, v1.x에서 `@Deprecated`로 표시되었던 **별칭(alias) 메서드·상수는 v2.0.0에서 완전히 제거**되었습니다. 아래의 정식 메서드로 교체하세요(미교체 시 컴파일 오류).

### AMMInterstitial (전면 광고)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `closeInterstitial()` | `stopInterstitial()` | 동일 동작 별칭이었음 |
| `onDestroy()` | `stopInterstitial()` | `destroy()`의 별칭 + Activity 메서드와 혼동 유발 → 제거 |

> **로드 메서드 (참고)**: 전면 광고의 정식 로드 메서드는 **`loadInterstitial()`**(유지됨)입니다. `AMMInterstitial`에는 `loadAd()`가 없습니다(`loadAd()`는 배너 `AMMBannerView`의 메서드). 자동 노출은 `startInterstitial()`(로드+노출), 수신 후 표시는 `showInterstitial()`.

```java
// Before (v1.x) — 제거됨, 컴파일 오류
InterstitialAd interstitialAd = new InterstitialAd(context);
interstitialAd.onDestroy();
interstitialAd.closeInterstitial();

// After (v2.0.0) — 신규 클래스 및 메서드 사용
AMMInterstitial interstitialAd = new AMMInterstitial(context);
interstitialAd.stopInterstitial();   // 정식 해제 (loadInterstitial()은 그대로 사용)
```

### AMMRewardVideo (보상형 전면 비디오 광고)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stopRewardVideoAd()` (또는 `destroy()`) | `destroy()` 별칭이었음. `stopRewardVideoAd()`는 하위 호환 메서드로 유지됨 |

```java
// Before (v1.x) — 제거됨
RewardInterstitialVideoAd rewardAd = new RewardInterstitialVideoAd(context);
rewardAd.onDestroy();

// After (v2.0.0)
AMMRewardVideo rewardAd = new AMMRewardVideo(context);
rewardAd.stopRewardVideoAd(); // 또는 rewardAd.destroy();
```

### AMMVideoInterstitial (전면 비디오 광고)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stopInterstitialVideoAd()` (또는 `destroy()`) | `destroy()` 별칭이었음. `stopInterstitialVideoAd()`는 하위 호환 메서드로 유지됨 |

```java
// Before (v1.x) — 제거됨
InterstitialVideoAd interstitialVideoAd = new InterstitialVideoAd(context);
interstitialVideoAd.onDestroy();

// After (v2.0.0)
AMMVideoInterstitial interstitialVideoAd = new AMMVideoInterstitial(context);
interstitialVideoAd.stopInterstitialVideoAd(); // 또는 interstitialVideoAd.destroy();
```

### AMMBannerView / AMMNativeAdView (배너·네이티브)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `destroy()` | `destroy()` 별칭이었음. 생명주기 자동 연결(`bindLifecycle`) 사용 시 호출 불필요 |

```java
// Before (v1.x) — 제거됨
AdView adView = findViewById(R.id.ad_view);
adView.onDestroy();

// After (v2.0.0)
AMMBannerView adView = findViewById(R.id.ad_view);
adView.destroy();
```

### AdInfo.Builder

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `.isUseBackgroundAlpha(Boolean)` | `.setUseBackgroundAlpha(boolean)` | `Boolean` → `boolean` (null 위험 제거) |

```java
// Before (v1.x) — 제거됨
new AdInfo.Builder(ADUNIT_ID)
    .isUseBackgroundAlpha(true)
    .build();

// After (v2.0.0)
new AdInfo.Builder(ADUNIT_ID)
    .setUseBackgroundAlpha(true)
    .build();
```

### AdMixer 상수 (전면 CountDown — 제거됨)

전면 카운트다운(CountDown) 타입이 완전히 제거되어 관련 상수도 모두 삭제되었습니다(`AdMixer.GAUGE`/`TEXT`, `AdMixer.AX_COUNT_TYPE_GAUGE`/`TEXT`). 해당 상수를 사용하던 코드는 제거하세요.

---

## Step 5-B. AdListener 이벤트 콜백 분리 (필수 — Breaking Change)

**[REQ-20260609]** `AdListener`의 단일 `onEventAd(adView, AdEvent)`가 **이름 있는 이벤트 메서드로 분리**되었습니다. 전면용 `FullScreenContentCallback`과 동일한 named-method 스타일로 통일하기 위함입니다.

- `AdListener`가 `interface` → `abstract class`로 바뀌어 **모든 메서드가 기본 no-op**입니다(필요한 것만 override, 필수 구현 없음).
- `onReceivedAd` / `onFailedToReceiveAd`는 시그니처 동일(그대로 동작).
- 로드 실패(`onFailedToReceiveAd`)와 구분되는 **표시 단계 실패** 시 호출되는 `onAdShowFailed`가 추가되었습니다.
- `AdEvent` enum은 SDK 내부 전용으로 전환되었습니다(외부 콜백에서 미사용).

### `onEventAd(AdEvent.X)` → 이름 있는 메서드 매핑

| 구 `onEventAd(AdEvent.X)` | 신 콜백 메서드 | 설명 |
|---|---|---|
| `DISPLAYED` | `onAdDisplayed()` | 광고 노출됨 |
| `CLICK` | `onAdClicked()` | 광고 클릭 |
| `CLOSE` | `onAdClosed()` | 광고 닫힘 |
| `COMPLETION` | `onAdCompleted()` | 비디오 재생 완료 |
| `SKIPPED` | `onAdSkipped()` | 비디오 스킵 |
| `LEFT_CLICK` | `onAdLeftClicked()` | 팝업형 전면 좌측 버튼 클릭 |
| `RIGHT_CLICK` | `onAdRightClicked()` | 팝업형 전면 우측 버튼 클릭 |
| `EARNEDREWARD` | `onAdRewarded()` | 보상 적립 |
| - | `onAdShowFailed(adView, adapterName, errorCode, errorMsg)` | 광고 노출(show) 실패 (신규 추가) |

```java
// Before (v1.x) — onEventAd 제거됨, 컴파일 오류
adView.setAdViewListener(new AdListener() {
    @Override public void onReceivedAd(String adapterName, Object ad) { /* ... */ }
    @Override public void onFailedToReceiveAd(Object ad, String name, int code, String msg) { /* ... */ }
    @Override public void onEventAd(Object ad, AdEvent event) {
        if (event == AdEvent.CLICK) { /* 클릭 */ }
        else if (event == AdEvent.COMPLETION) { /* 완료 */ }
    }
});

// After (v2.0.0) — 필요한 이벤트만 override
adView.setAdViewListener(new AdListener() {
    @Override public void onReceivedAd(String adapterName, Object ad) { /* ... */ }
    @Override public void onFailedToReceiveAd(Object ad, String name, int code, String msg) { /* ... */ }
    @Override public void onAdShowFailed(Object ad, String name, int code, String msg) { /* 표시 실패 */ }
    @Override public void onAdClicked() { /* 클릭 */ }
    @Override public void onAdCompleted() { /* 완료 */ }
});
```

> ℹ️ 전면형태(전면/리워드/전면 동영상)는 `FullScreenContentCallback`(GAM 규약)도 그대로 사용할 수 있습니다(변경 없음).

---

## Step 6. 새 기능 적용 (선택)

### 광고 신고하기 및 닫기 버튼 제어

v2.0.0에서 광고 소재 신고 기능 및 전면 광고 'X' 닫기 버튼 표시 제어 기능이 추가되었습니다. Android 8.0(API 26) 이상에서 PixelCopy 기반 소재 자동 캡처를 지원합니다.

```java
AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
    .showReportIcon(true)   // ← 신고 아이콘(ⓘ) 활성화 (기본값: false)
    .showCloseButton(true)  // ← 전면 광고 'X' 닫기 버튼 표시 여부 (기본값: true)
    .build();
```

v2.0.0의 기타 선택 신규 기능(진행 중 로드만 취소 `cancelLoad()`, 개인정보 동의/테스트 설정 전파)은 아래 **Step 9~10**을 참고하세요.

---

## Step 7. 네이티브 광고 View ID 업데이트 (필수)

v2.0.0에서 네이티브 광고 레이아웃의 View ID에 `nap_mx_` prefix가 추가되었습니다. 타 라이브러리와의 리소스 ID 충돌을 방지하기 위한 변경입니다.

### 변경 ID 목록

| 기존 (v1.x) | v2.0.0 |
|------------|--------|
| `tv_title` | `nap_mx_tv_title` |
| `iv_icon` | `nap_mx_iv_icon` |
| `tv_adv` | `nap_mx_tv_adv` |
| `tv_desc` | `nap_mx_tv_desc` |
| `iv_main` | `nap_mx_iv_main` |
| `btn_cta` | `nap_mx_btn_cta` |

### 레이아웃 XML 수정

```xml
<!-- Before -->
<TextView android:id="@+id/tv_title" ... />
<ImageView android:id="@+id/iv_icon" ... />

<!-- After -->
<TextView android:id="@+id/nap_mx_tv_title" ... />
<ImageView android:id="@+id/nap_mx_iv_icon" ... />
```

### NativeAdViewBinder 코드 수정 (Java)

```java
// Before
new NativeAdViewBinder.Builder(R.layout.item_native_ad)
    .setTitleId(R.id.tv_title)
    .setIconImageId(R.id.iv_icon)
    .setAdvertiserId(R.id.tv_adv)
    .setDescriptionId(R.id.tv_desc)
    .setMainViewId(R.id.iv_main)
    .setCtaId(R.id.btn_cta)
    .build();

// After
new NativeAdViewBinder.Builder(R.layout.item_native_ad)
    .setTitleId(R.id.nap_mx_tv_title)
    .setIconImageId(R.id.nap_mx_iv_icon)
    .setAdvertiserId(R.id.nap_mx_tv_adv)
    .setDescriptionId(R.id.nap_mx_tv_desc)
    .setMainViewId(R.id.nap_mx_iv_main)
    .setCtaId(R.id.nap_mx_btn_cta)
    .build();
```

### setViewIds 제거 (v2.0.0 Breaking Change)

v2.0.0에서 `setViewIds()`가 **완전히 제거**되었습니다. AdMixer·AdManager·Adfit·Pangle·Mobwith·NaverAd 모든 어댑터가 `NativeAdViewBinder`를 직접 읽으므로, `setViewIds()` 호출을 제거하고 위의 `NativeAdViewBinder` 설정만으로 동작합니다.

```java
// Before (v1.x) — 제거하세요
Map<String, Integer> ids = new HashMap<>();
ids.put("iv_image", R.id.my_image_view);
new AdInfo.Builder(ADUNIT_ID)
    .setViewIds(AdMixer.ADAPTER_MOBWITH, ids)
    .build();

// After (v2.0.0) — NativeAdViewBinder 설정만으로 충분
new NativeAdViewBinder.Builder(R.layout.item_native_ad)
    .setMainViewId(R.id.nap_mx_iv_main)
    // ... 나머지 View ID 설정
    .build();
```

### setAdapterConfig 추가 (어댑터별 초기화 파라미터)

어댑터에 String 타입 초기화 파라미터를 전달해야 하는 경우 `setAdapterConfig()`를 사용합니다. 대표적인 사용 예는 AppLovin SDK Key 재정의입니다.

```java
// AppLovin SDK Key를 기본값 대신 직접 지정하는 경우
Map<String, String> applovinConfig = new HashMap<>();
applovinConfig.put("sdkKey", "YOUR_APPLOVIN_SDK_KEY");

AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
    .setAdapterConfig(AdMixer.ADAPTER_APPLOVIN, applovinConfig)
    .build();
```

> ℹ️ SDK 제공 샘플 레이아웃(`admixer-nativeadlayout` 모듈)을 사용하는 경우 레이아웃 XML은 자동 적용됩니다. `NativeAdViewBinder` 코드만 업데이트하면 됩니다.

---

## Step 8. 전면 광고 뒤로가기(BACK) 키 — 동작 변경 (확인 필요)

v2.0.0부터 **전면 광고는 시스템 뒤로가기(BACK) 키를 기본 차단**합니다(비디오·리워드와 동일 정책). 광고는 'X' 닫기 버튼으로만 닫힙니다.

> ⚠️ 기존(v1.x)에 **뒤로가기로 전면 광고를 닫던 동작에 의존**하는 매체는, 아래와 같이 명시적으로 해제해야 종전 동작이 유지됩니다.

```java
// v1.x 동작(뒤로가기로 닫기)을 유지하려면:
AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
        .setDisableBackKey(false) // 명시적 false → 뒤로가기 닫기 허용
        .build();
```

`AdInfo.Builder.setDisableBackKey`의 **기본값이 `true`(차단)** 로 변경되었습니다.

---

## Step 9. 진행 중 로드만 취소 — `cancelLoad()` (선택)

표시 중인 광고를 끊지 않고 미완료 로드만 취소하는 `cancelLoad()`가 추가되었습니다(전면·리워드·전면 동영상 공통).

```java
// 화면 전환·백그라운드: 진행 중 로드만 취소 (표시 중이면 no-op)
interstitialAd.cancelLoad();
// 화면 종료: 전체 정리
interstitialAd.stopInterstitial();
```

---

## Step 10. 개인정보 동의 / 테스트 설정 전파 (선택)

GDPR/CCPA/COPPA 동의값과 테스트 설정을 `AdMixer` 전역에 설정하면 워터폴에서 각 네트워크로 자동 전파됩니다.

```java
AdMixer.setGdprConsent(true);
AdMixer.setUsPrivacy("1YNN"); // CCPA US Privacy 문자열 설정
AdMixer.setCcpaDoNotSell(false); // CCPA Do Not Sell 플래그 설정
AdMixer.setTestMode(true);
AdMixer.setTestDeviceIds(Arrays.asList("..."));
```

자세한 매핑은 [개인정보 동의 및 테스트 설정](privacy.md)을 참고하세요.

---

## 주요 Public API 변경 여부 요약

v2.0.0에서의 주요 Public API 변경 여부를 요약한 표입니다. '변경 없음'으로 표시된 API는 기존 v1.x 코드를 그대로 사용할 수 있습니다.

| 클래스 | 변경 여부 |
|--------|----------|
| `AMMBannerView` (구 `AdView`) | 클래스명 변경 외 기본 API 동작 유지 |
| `AMMNativeAdView` (구 `NativeAdView`) | 클래스명 변경 외 기본 API 동작 유지 |
| `AMMVideoView` (구 `VideoAdView`) | 클래스명 변경 외 기본 API 동작 유지 |
| `AMMInterstitial` (구 `InterstitialAd`) | 클래스명 변경 및 일부 제거 API 제외 유지 |
| `AMMRewardVideo` (구 `RewardInterstitialVideoAd`) | 클래스명 변경 및 일부 제거 API 제외 유지 |
| `AMMVideoInterstitial` (구 `InterstitialVideoAd`) | 클래스명 변경 및 일부 제거 API 제외 유지 |
| `AdListener` | **변경됨** — `onEventAd` → 이름 있는 메서드 및 노출 실패 콜백 추가 (Step 5-B) |
| `AdEvent` | **내부 전용 전환** — 외부 콜백에서 미사용 (Step 5-B) |
| `AdInfo.Builder` (제거 API 제외) | 변경 없음 |
| 전면 타입 설정 (`interstitialAdType`/`popupAdOption`) | **제거(Breaking)** — 전면은 Basic 전용. `PopupInterstitialAdOption` 클래스는 내부 전용으로 유지 |

---

## 문의

업그레이드 관련 문의사항은 **nap_mx@nasmedia.co.kr**로 연락하세요.
