# v2.0.0 마이그레이션 가이드

이 문서는 nap ssp Android SDK **v1.x → v2.0.0** 업그레이드 시 필요한 변경 사항을 설명합니다.

---

## 빠른 요약

| 구분 | 내용 |
|------|------|
| **광고 클래스명 변경 (Breaking)** | **`AdView`→`AMMBannerView`, `InterstitialAd`→`AMMInterstitial`, `NativeAdView`→`AMMNativeAdView`, `RewardInterstitialVideoAd`→`AMMRewardVideo`, `VideoAdView`→`AMMVideoView`, `InterstitialVideoAd`→`AMMVideoInterstitial` — 구 클래스명 완전 제거(미교체 시 컴파일 오류)** |
| **풀스크린 로드 방식 (권장)** | **전면/리워드/전면 동영상에 정적 `load()` + `FullScreenContentCallback` 패턴 추가 — 기존 인스턴스 메서드도 그대로 유지** |
| 새 기능 | NaverAdManager·Teads 어댑터, 광고 신고 기능 추가 |
| **어댑터 등록 간소화** | **`registerAdapter()` 호출 불필요 — Gradle 의존성 추가만으로 자동 등록** |
| **네이티브 View ID 변경** | **`tv_title` 등 → `nap_mx_tv_title` 등 — 레이아웃 및 ViewBinder 코드 수정 필요** |
| **`setViewIds()` 제거** | **v2.0.0에서 완전 제거 — `NativeAdViewBinder`가 모든 어댑터 View ID 처리** |
| **`setAdapterConfig()` 추가** | **어댑터별 String 초기화 파라미터 설정 (AppLovin `sdkKey` 등)** |
| **전면 BACK 키 기본 차단** | **`PopupInterstitialAdOption.setDisableBackKey` 기본값 `true`로 변경 — 뒤로가기 닫기 의존 시 `false` 명시 필요** |
| 신규 API | `cancelLoad()`(로드만 취소), `AdMixer.setGdprConsent/setCcpaDoNotSell/setTestMode/setTestDeviceIds`(개인정보·테스트 전파) |
| Naver PUBLISHER_CD | SDK 제공으로 변경 — 호스트 매니페스트 설정 불필요 |
| **제거된 API (Breaking)** | **`onDestroy()`, `closeInterstitial()`, `AdInfo.Builder.isUseBackgroundAlpha(Boolean)`, `AdMixer.GAUGE`/`AdMixer.TEXT` 등 — v1.x deprecated 별칭을 v2.0.0에서 완전 제거. 정식 메서드로 교체 필요** |
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

## Step 5. 클래스명 변경 및 제거된 API 교체 (필수 — Breaking Change)

### 5-1. 광고 클래스명 변경 (필수)

v2.0.0에서 광고 포맷 클래스가 `AMM` prefix로 일괄 변경되었습니다. **구 클래스명은 완전히 제거**되어 그대로 두면 컴파일 오류가 발생합니다. 클래스명만 교체하면 기존 인스턴스 메서드(`setAdInfo`/`setListener`/`load*`/`show*`/`stop*`)는 그대로 사용할 수 있습니다.

| 포맷 | 기존 (v1.x) | v2.0.0 |
|------|------------|--------|
| 배너 | `AdView` | `AMMBannerView` |
| 전면 | `InterstitialAd` | `AMMInterstitial` |
| 네이티브 | `NativeAdView` | `AMMNativeAdView` |
| 리워드 동영상 | `RewardInterstitialVideoAd` | `AMMRewardVideo` |
| 인라인 동영상 | `VideoAdView` | `AMMVideoView` |
| 전면 동영상 | `InterstitialVideoAd` | `AMMVideoInterstitial` |

```java
// Before — 구 클래스명 제거됨, 컴파일 오류
InterstitialAd interstitialAd = new InterstitialAd(this);

// After — 클래스명만 교체
AMMInterstitial interstitialAd = new AMMInterstitial(this);
```

XML 레이아웃의 배너 태그도 함께 교체하세요.

```xml
<!-- Before -->
<com.nasmedia.admixerssp.ads.AdView ... />
<!-- After -->
<com.nasmedia.admixerssp.ads.AMMBannerView ... />
```

> 💡 **권장 — 풀스크린 정적 로드 패턴**: 전면·리워드·전면 동영상은 정적 `load()` + `FullScreenContentCallback` 패턴으로 전환하면 GAM(Google Mobile Ads) 스타일의 명확한 노출/클릭/닫힘/보상 콜백을 사용할 수 있습니다. 자세한 코드는 [전면 광고](interstitial.md)·[리워드 동영상](rewarded-video.md)·[동영상 광고](video.md) 가이드를 참고하세요. (기존 인스턴스 메서드 `loadInterstitial()`/`showInterstitial()` 등도 계속 동작하므로, 클래스명만 교체하는 최소 마이그레이션도 가능합니다.)

### 5-2. 제거된 별칭 API 교체 (필수)

v1.x에서 `@Deprecated`로 표시되었던 **별칭(alias) 메서드·상수는 v2.0.0에서 완전히 제거**되었습니다. 이 메서드들은 동일 동작을 하는 정식 메서드의 별칭이었으며, v1→v2는 메이저 버전 전환이므로 클린 브레이크로 정리했습니다. 아래 정식 메서드로 교체하세요(미교체 시 컴파일 오류).

#### AMMInterstitial (전면)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `closeInterstitial()` | `stopInterstitial()` | 동일 동작 별칭이었음 |
| `onDestroy()` | `stopInterstitial()` | `destroy()`의 별칭 + Activity 메서드와 혼동 유발 → 제거 |

> **로드 메서드 (참고)**: v2.0.0 권장 방식은 정적 `AMMInterstitial.load()` + `FullScreenContentCallback`입니다([전면 광고](interstitial.md) 참고). 기존 인스턴스 메서드 `loadInterstitial()`(로드)·`startInterstitial()`(로드+노출)·`showInterstitial()`(표시)도 유지됩니다. `AMMInterstitial`에는 `loadAd()`가 없습니다(`loadAd()`는 배너 `AMMBannerView`의 메서드).

```java
// Before — 제거됨, 컴파일 오류
interstitialAd.onDestroy();
interstitialAd.closeInterstitial();

// After
interstitialAd.stopInterstitial();   // 정식 해제 (loadInterstitial()은 그대로 사용)
```

#### AMMRewardVideo (리워드 동영상)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stopRewardVideoAd()` | `destroy()` 별칭이었음 |

```java
// Before — 제거됨
rewardAd.onDestroy();

// After
rewardAd.stopRewardVideoAd();
```

#### AMMVideoInterstitial (전면 동영상)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stopInterstitialVideoAd()` | `destroy()` 별칭이었음 |

```java
// Before — 제거됨
interstitialVideoAd.onDestroy();

// After
interstitialVideoAd.stopInterstitialVideoAd();
```

#### AMMBannerView / AMMNativeAdView (배너·네이티브)

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `destroy()` | `destroy()` 별칭이었음. 생명주기 자동 연결(`bindLifecycle`) 사용 시 호출 불필요 |

```java
// Before — 제거됨
adView.onDestroy();

// After
adView.destroy();
```

#### AdInfo.Builder

| 제거됨 (v1.x) | v2.0.0 정식 메서드 | 비고 |
|------------|------------|------|
| `.isUseBackgroundAlpha(Boolean)` | `.setUseBackgroundAlpha(boolean)` | `Boolean` → `boolean` (null 위험 제거) |

```java
// Before — 제거됨
new AdInfo.Builder(ADUNIT_ID)
    .isUseBackgroundAlpha(true)
    .build();

// After
new AdInfo.Builder(ADUNIT_ID)
    .setUseBackgroundAlpha(true)
    .build();
```

#### AdMixer 상수

| 제거됨 (v1.x) | v2.0.0 정식 상수 | 비고 |
|------------|------------|------|
| `AdMixer.GAUGE` | `AdMixer.AX_COUNT_TYPE_GAUGE` | 동일 값 별칭이었음 |
| `AdMixer.TEXT` | `AdMixer.AX_COUNT_TYPE_TEXT` | 동일 값 별칭이었음 |

```java
// Before — 제거됨
int type = AdMixer.GAUGE;

// After
int type = AdMixer.AX_COUNT_TYPE_GAUGE;
```

---

## Step 6. 새 기능 적용 (선택)

### 광고 신고하기

v2.0.0에서 추가된 광고 소재 신고 기능입니다. Android 8.0(API 26) 이상에서 PixelCopy 기반 소재 자동 캡처를 지원합니다.

```java
AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
    .showReportIcon(true)  // ← 신고 아이콘(ⓘ) 활성화
    .build();
```

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
PopupInterstitialAdOption opt = new PopupInterstitialAdOption();
opt.setDisableBackKey(false); // 명시적 false → 뒤로가기 닫기 허용
```

`PopupInterstitialAdOption.setDisableBackKey`의 **기본값이 `true`(차단)** 로 변경되었습니다.

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
AdMixer.setCcpaDoNotSell(false);
AdMixer.setTestMode(true);
AdMixer.setTestDeviceIds(Arrays.asList("..."));
```

자세한 매핑은 [개인정보 동의 및 테스트 설정](privacy.md)을 참고하세요.

---

## 변경 없는 항목 (하위 호환 유지)

아래 Public API는 **v2.0.0에서 클래스명·시그니처가 변경되지 않습니다**. 기존 코드를 그대로 사용할 수 있습니다.

| 클래스 | 변경 여부 |
|--------|----------|
| `AdListener` | 변경 없음 (콜백 메서드 시그니처 동일) |
| `AdEvent` | 변경 없음 |
| `AdInfo` / `AdInfo.Builder` (제거 API 제외) | 변경 없음 |
| `NativeAdViewBinder` | 변경 없음 |
| `PopupInterstitialAdOption` | 변경 없음 |
| `AdMixer` (제거 상수 제외) | 변경 없음 |

> ℹ️ 광고 포맷 클래스(`AdView`·`InterstitialAd`·`NativeAdView`·`RewardInterstitialVideoAd`·`VideoAdView`·`InterstitialVideoAd`)는 v2.0.0에서 `AMM*`로 **이름이 변경**되었습니다. [Step 5](#step-5-클래스명-변경-및-제거된-api-교체-필수--breaking-change)를 참고하세요.

---

## 문의

업그레이드 관련 문의사항은 **nap_mx@nasmedia.co.kr**로 연락하세요.
