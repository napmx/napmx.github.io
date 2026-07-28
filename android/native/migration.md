# v2 마이그레이션 가이드

이 문서는 nap mx Android SDK **v1.x → v2 최신 버전** 업그레이드 시 필요한 변경 사항을 설명합니다.

> ℹ️ **이미 v2.0.x를 쓰고 계신다면** Step 1~7은 건너뛰고 [v2.0.x → 최신 버전](#v20x--최신-버전-업그레이드)만 확인하세요.
>
> 코드 예제는 **최신 버전 기준**입니다. 버전은 [BOM](#step-1-gradle-버전-업데이트)으로 관리하는 것을 권장합니다.

---

## 빠른 요약

| 구분 | 내용 |
|------|------|
| 새 기능 | NaverAdManager·Teads 어댑터 추가 |
| **어댑터 등록 간소화** | **`registerAdapter()` 호출 불필요 — Gradle 의존성 추가만으로 자동 등록** |
| **네이티브 View ID 변경** | **`tv_title` 등 → `nap_mx_tv_title` 등 — 레이아웃 및 ViewBinder 코드 수정 필요** |
| **`setViewIds()` 제거** | **v2.0.0에서 완전 제거 — 네이티브 렌더링에 `NativeAdViewBinder`의 View ID를 사용** |
| **`setAdapterConfig()` 추가** | **어댑터별 String 초기화 파라미터 설정 (AppLovin `sdkKey` 등)** |
| **전면 타입 Basic 전용 (Breaking)** | **전면 광고는 Basic만 제공 — `AdInfo.Builder.interstitialAdType`/`popupAdOption`/`setInterstitialAdType`/`setPopupAdOption` 제거(앱에서 전면 타입을 지정할 수 없음). CountDown 타입은 완전 제거(관련 상수·뷰 삭제)** |
| 신규 API | `cancelLoad()` (로드만 취소), `AdMixer.setTestMode`/`setTestDeviceIds` (테스트 설정) |
| Naver PUBLISHER_CD | SDK 제공으로 변경 — 호스트 매니페스트 설정 불필요 |
| **제거된 API (Breaking)** | **`onDestroy()`, `closeInterstitial()`, 배경 알파 빌더 옵션 `isUseBackgroundAlpha(...)`, `AdMixer.GAUGE`/`AdMixer.TEXT` 등 — v1.x deprecated 별칭/무효 옵션을 v2.0.0에서 제거. 정식 메서드로 교체 필요 (`setUseBackgroundAlpha(boolean)`는 남아 있으나 **동작하지 않습니다** — 제거 권장)** |
| **AdListener 이벤트 콜백 분리 (Breaking)** | **`onEventAd(AdEvent)` 제거 → `onAdDisplayed`/`onAdClicked`/`onAdClosed`/`onAdCompleted` 및 노출 실패 `onAdShowFailed` 등 이름 있는 메서드. `AdListener`는 abstract class(필요한 것만 override). Step 5-B 참고** |
| ProGuard | `consumer-rules.pro` 자동 병합 — v1.x에서 수동 추가한 AdMixer keep 규칙은 삭제 가능 |
| Gradle 버전 | v1.x 버전 → 최신 버전 (BOM 권장) |

---

## 마이그레이션 순서 (한눈에)

1. **Gradle 버전** 최신으로 변경(BOM 권장) + 새 어댑터(NaverAdManager/Teads) 저장소 추가 → [Step 1](#step-1-gradle-버전-업데이트)
2. **ProGuard 규칙** 정리(수동 규칙 삭제 가능) → [Step 2](#step-2-proguard-규칙-업데이트)
3. **`registerAdapter()` 호출 제거**(자동 등록) → [Step 3](#step-3-registeradapter-호출-제거)
4. **구 클래스 → `AMM*` 전환** + 제거된 별칭 메서드 교체 → [Step 5](#step-5-제거된-클래스-및-api-교체-필수--breaking-change)
5. **`AdListener` 이벤트 콜백 분리**(`onEventAd` → named 메서드) → [Step 5-B](#step-5-b-adlistener-이벤트-콜백-분리-필수--breaking-change)
6. **네이티브 View ID** `nap_mx_` prefix + `setViewIds()` 제거 → [Step 7](#step-7-네이티브-광고-view-id-업데이트-필수)
7. **빌드 → 컴파일 오류 해소**(아래 표) → **광고 노출 검증**([맨 아래 체크리스트](#업그레이드-후-검증-체크리스트))

---

## 컴파일 오류 빠른 해결

업그레이드 직후 가장 흔한 컴파일 오류와 해결 매핑입니다. 대부분 클래스명/메서드명 변경이라 기계적으로 교체하면 됩니다.

| 증상 (컴파일/런타임) | 원인 | 해결 |
|---|---|---|
| `cannot find symbol: class AdView`/`InterstitialAd` 등 | 구 광고 클래스 제거 | `AMM*` 클래스로 교체 ([Step 5](#step-5-제거된-클래스-및-api-교체-필수--breaking-change)) |
| `method onEventAd ... does not override` | `AdListener.onEventAd` 제거 | 이름 있는 메서드로 분리 ([Step 5-B](#step-5-b-adlistener-이벤트-콜백-분리-필수--breaking-change)) |
| `cannot find symbol: method onDestroy()` | 별칭 제거 | `stop()`으로 교체 (인라인·풀스크린 공통) |
| `cannot find symbol: method startInterstitial()` 등 | 즉시 노출 API 제거 | 수신 콜백 이후 `show(activity)` 호출 |
| `cannot find symbol: method setViewIds(...)` | 제거됨 | `NativeAdViewBinder` 설정으로 통합 ([Step 7](#step-7-네이티브-광고-view-id-업데이트-필수)) |
| `cannot find symbol: method isUseBackgroundAlpha(...)` | v1.x 빌더 메서드 제거 | 호출 제거 (전면 배경 디밍은 자동 적용). 이름이 비슷한 `setUseBackgroundAlpha(boolean)`은 남아 있으나 **동작하지 않습니다** |
| `cannot find symbol: method setInterstitialAdType/setPopupAdOption(...)` | 전면 Basic 전용 | 해당 호출 제거 |
| `cannot find symbol: AdMixer.GAUGE`/`TEXT` | CountDown 상수 제거 | 관련 코드 제거 |
| 빌드는 되나 네이티브가 비거나 미표시 | View ID prefix 변경 | 레이아웃·바인더에 `nap_mx_` 적용 ([Step 7](#step-7-네이티브-광고-view-id-업데이트-필수)) |
| `cannot find symbol: method setPrivacyViewId(int)` | **v2.0.x 이후 제거** | `setAdChoicesPosition()`으로 교체 ([v2.0.x → 최신](#v20x--최신-버전-업그레이드)) |
| `cannot find symbol: method setAdViewBinder(...)` | **v2.0.1에서 제거** | `AMMNativeAdView.setViewBinder()`로 교체 ([v2.0.x → 최신](#v20x--최신-버전-업그레이드)) |

> 🚨 **컴파일 오류 없이 조용히 깨지는 변경도 있습니다** — 어댑터 식별자 상수의 **값**이 바뀌어 재컴파일이 필요합니다. [v2.0.x → 최신 버전](#v20x--최신-버전-업그레이드)을 반드시 확인하세요.

---

## Step 1. Gradle 버전 업데이트

**BOM 사용을 권장합니다.** 모듈마다 버전이 갈릴 수 있어(코어와 어댑터가 독립 배포됨) BOM이 조합 호환을 보장하고, 업그레이드 시 **BOM 버전 한 줄만** 바꾸면 됩니다.

```gradle
// Before (v1.x)
implementation 'io.github.nasmedia-tech:admixer-ssp:1.0.21'
implementation 'io.github.nasmedia-tech:admixer-admanager:1.0.21'
// ...

// After (권장 — BOM으로 버전 관리, 멤버는 버전 생략)
implementation platform('io.github.nasmedia-tech:admixer-bom:2026.07.05')
implementation 'io.github.nasmedia-tech:admixer-ssp'
implementation 'io.github.nasmedia-tech:admixer-admanager'
implementation 'io.github.nasmedia-tech:admixer-adfit'
implementation 'io.github.nasmedia-tech:admixer-pangle'
implementation 'io.github.nasmedia-tech:admixer-applovin'
implementation 'io.github.nasmedia-tech:admixer-unity'
// 신규 — v2에서 추가됨
implementation 'io.github.nasmedia-tech:admixer-naveradmanager'
implementation 'io.github.nasmedia-tech:admixer-teads'
```

> ℹ️ **최신 BOM 버전**은 [SDK 시작하기](getting-started.md)에서 확인하세요. BOM 없이 개별 버전을 직접 지정하는 방법도 시작하기 가이드에 있습니다.

---

## Step 2. ProGuard 규칙 업데이트

v2.0.0에서 Core SDK와 전 어댑터 AAR이 `consumer-rules.pro`를 내장하도록 바뀌었습니다. Gradle이 이 규칙을 호스트 앱의 릴리즈 빌드에 **자동으로 병합**하므로, **앱의 `proguard-rules.pro`에 AdMixer 관련 keep 규칙을 둘 필요가 없습니다.**

v1.x에서 수동으로 추가해 두었던 AdMixer keep 규칙은 **삭제해도 됩니다.**

```proguard
# Before (v1.x) — 수동 규칙. v2에서는 불필요하므로 삭제 가능
-keep class com.nasmedia.admixerssp.** { *; }
-keep class com.nasmedia.admanager.** { *; }
# ... 어댑터별 규칙

# After (v2) — 별도 규칙 없음 (consumer-rules.pro 자동 병합)
```

> ℹ️ 앱이 SDK 클래스를 **리플렉션으로 직접 참조**하는 특수한 경우에만 해당 클래스를 개별 keep하세요. 자세한 내용은 [ProGuard 설정](proguard.md)을 참고하세요.

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

> ℹ️ `registerAdapter(String)` 메서드는 하위 호환성을 위해 남아 있지만, `initialize()` 내부의 자동 탐지가 클래스패스에 포함된 어댑터를 이미 등록하므로 중복 호출입니다. 제거를 권장합니다. 또한 현재 버전에서 이 메서드는 **SDK 내장 어댑터 이름만 처리**하며, 그 외 이름은 경고 로그를 남기고 무시됩니다(외부 커스텀 어댑터 등록은 지원하지 않습니다).

---

## Step 4. 새 어댑터 추가 (선택)

### NaverAdManager 추가 시

`build.gradle`에 의존성을 추가하면 어댑터는 `initialize()` 호출 시 자동으로 등록됩니다.

> ℹ️ Naver Ad Manager의 `com.naver.gfpsdk.PUBLISHER_CD`는 nap mx가 SDK(`admixer-naveradmanager` aar)에서 제공·관리합니다. **호스트 앱 매니페스트에 별도로 설정하지 마세요.** (이전 안내에서 호스트가 Publisher ID를 추가하도록 했으나, SDK 제공 방식으로 변경되었습니다.)

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

> ℹ️ **포맷별 상세 전환 매핑**(이벤트 콜백 포함)은 각 가이드 하단 "구 API에서 전환" 표에서도 확인할 수 있습니다 — [배너](banner.md) · [전면](interstitial.md) · [네이티브](native-ad.md) · [동영상](video.md) · [리워드 동영상](rewarded-video.md).

또한, v1.x에서 `@Deprecated`로 표시되었던 **별칭(alias) 메서드·상수는 v2.0.0에서 완전히 제거**되었습니다. 아래의 정식 메서드로 교체하세요(미교체 시 컴파일 오류).

### AMMInterstitial (전면 광고)

| 제거됨 (v1.x) | 현행 정식 메서드 | 비고 |
|------------|------------|------|
| `closeInterstitial()` | `stop()` | 동일 동작 별칭이었음 |
| `onDestroy()` | `stop()` | `destroy()`의 별칭 + Activity 메서드와 혼동 유발 → 제거 |

> ℹ️ **`stopInterstitial()`은 하위 호환용 `@Deprecated` 별칭**으로만 남아 있습니다. 신규 코드는 `stop()`을 사용하세요. `destroy()`도 `@Deprecated`입니다.

> **로드 메서드 (참고)**: 전면 광고의 정식 로드 메서드는 **`loadAd()`**(인스턴스) 또는 정적 `AMMInterstitial.loadAd(context, adInfo, callback)`입니다. 구 명칭 `loadInterstitial()`/`showInterstitial()`은 `@Deprecated` 별칭으로 유지됩니다. 즉시 노출 `startInterstitial()`은 **제거**되었습니다 — 수신 후 `show(activity)`로 노출하세요.

```java
// Before (v1.x) — 제거됨, 컴파일 오류
InterstitialAd interstitialAd = new InterstitialAd(context);
interstitialAd.onDestroy();
interstitialAd.closeInterstitial();

// After (v2) — 신규 클래스 및 정식 메서드 사용
AMMInterstitial interstitialAd = new AMMInterstitial(context);
interstitialAd.loadAd();   // 로드 (구 loadInterstitial()은 @Deprecated 별칭)
interstitialAd.stop();     // 정식 해제
```

### AMMRewardVideo (보상형 전면 비디오 광고)

| 제거됨 (v1.x) | 현행 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stop()` | `destroy()` 별칭이었음. `stopRewardVideoAd()`·`destroy()`는 `@Deprecated` 별칭으로만 유지됨 |

```java
// Before (v1.x) — 제거됨
RewardInterstitialVideoAd rewardAd = new RewardInterstitialVideoAd(context);
rewardAd.onDestroy();

// After (v2)
AMMRewardVideo rewardAd = new AMMRewardVideo(context);
rewardAd.stop();
```

### AMMVideoInterstitial (전면 비디오 광고)

| 제거됨 (v1.x) | 현행 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stop()` | `destroy()` 별칭이었음. `stopInterstitialVideoAd()`·`destroy()`는 `@Deprecated` 별칭으로만 유지됨 |

```java
// Before (v1.x) — 제거됨
InterstitialVideoAd interstitialVideoAd = new InterstitialVideoAd(context);
interstitialVideoAd.onDestroy();

// After (v2)
AMMVideoInterstitial interstitialVideoAd = new AMMVideoInterstitial(context);
interstitialVideoAd.stop();
```

### AMMBannerView / AMMNativeAdView (배너·네이티브)

| 제거됨 (v1.x) | 현행 정식 메서드 | 비고 |
|------------|------------|------|
| `onDestroy()` | `stop()` | `destroy()` 별칭이었음. `destroy()` 자체도 `@Deprecated` — `stop()`을 사용하세요 |

> ℹ️ 인라인 뷰(배너·네이티브·동영상)는 호스트의 `LifecycleOwner`(Activity/Fragment)를 자동으로 탐색해 관찰하며, `ON_DESTROY` 시점에 SDK가 스스로 해제합니다. `LifecycleOwner`를 찾을 수 없는 Context에서 사용한다면 화면 종료 시 직접 `stop()`을 호출하세요.

```java
// Before (v1.x) — 제거됨
AdView adView = findViewById(R.id.ad_view);
adView.onDestroy();

// After (v2)
AMMBannerView adView = findViewById(R.id.ad_view);
adView.stop();
```

### AdInfo.Builder — 배경 알파 옵션 (동작하지 않음)

전면 광고 배경 디밍은 v2.0.0에서 **SDK가 고정값으로 자동 적용**합니다. 배경 알파 옵션은 두 메서드의 상태가 서로 다르니 구분하세요.

| 메서드 | 현재 상태 | 증상 |
|---|---|---|
| `isUseBackgroundAlpha(Boolean)` (v1.x 빌더 메서드) | **제거됨** | 호출 시 컴파일 오류 |
| `setUseBackgroundAlpha(boolean)` | **남아 있으나 동작하지 않음** | 컴파일은 그대로 통과하고 **값만 무시**됨 |

`setUseBackgroundAlpha(boolean)`은 값을 보관만 하고 **읽는 곳이 없습니다.** 호출해도 무해하지만 설정이 반영되지 않으므로 **제거를 권장**합니다(차기 메이저 버전에서 삭제 예정).

```java
// Before (v1.x)
new AdInfo.Builder(ADUNIT_ID)
    .isUseBackgroundAlpha(true)   // 제거됨 — 컴파일 오류
    .build();

// After (v2.0.0) — 옵션 불필요 (배경 디밍 자동 적용)
new AdInfo.Builder(ADUNIT_ID)
    .build();
```

### AdMixer 상수 (전면 CountDown — 제거됨)

전면 카운트다운(CountDown) 타입이 완전히 제거되어 관련 상수도 모두 삭제되었습니다(`AdMixer.GAUGE`/`TEXT`, `AdMixer.AX_COUNT_TYPE_GAUGE`/`TEXT`). 해당 상수를 사용하던 코드는 제거하세요.

---

## Step 5-B. AdListener 이벤트 콜백 분리 (필수 — Breaking Change)

`AdListener`의 단일 `onEventAd(adView, AdEvent)`가 **이름 있는 이벤트 메서드로 분리**되었습니다. 전면용 `FullScreenContentCallback`과 동일한 named-method 스타일로 통일하기 위함입니다.

- `AdListener`가 `interface` → `abstract class`로 바뀌어 **모든 메서드가 기본 no-op**입니다(필요한 것만 override, 필수 구현 없음).
- `onReceivedAd`는 v1.x 시그니처(`String adapterName, Object ad`)가 그대로 동작하지만 현재는 `@Deprecated`이며, `AdNetworkType` enum 오버로드가 표준입니다. `onFailedToReceiveAd`는 이후 릴리스에서 `(errorCode, errorMsg)` 2-인자 표준 콜백으로 단순화되었습니다(아래 3단계 참고).
- 로드 실패(`onFailedToReceiveAd`)와 구분되는 **표시 단계 실패** 시 호출되는 `onAdShowFailed`가 추가되었습니다.
- `AdEvent` enum은 공개 콜백에서 더 이상 사용되지 않습니다(SDK 내부 디스패치 전용). 앱 코드에서 직접 참조할 필요가 없습니다.

### `onEventAd(AdEvent.X)` → 이름 있는 메서드 매핑

| 구 `onEventAd(AdEvent.X)` | 신 콜백 메서드 | 설명 |
|---|---|---|
| `DISPLAYED` | `onAdDisplayed()` | 광고 노출됨 |
| `CLICK` | `onAdClicked()` | 광고 클릭 |
| `CLOSE` | `onAdClosed()` | 광고 닫힘 |
| `COMPLETION` | `onAdCompleted()` | 비디오 재생 완료 |
| `SKIPPED` | `onAdSkipped()` | 비디오 스킵 |
| `EARNEDREWARD` | `onAdRewarded()` | 보상 적립 |
| - | `onAdShowFailed(adView, networkType, errorCode, errorMsg)` | 광고 노출(show) 실패 (신규 추가) |

> ℹ️ 좌측 `AdEvent` 상수명은 **v1.x 기준**입니다. v2.0.0에서 `AdEvent`는 **SDK 내부 전용**으로 전환되었고, 내부적으로 `COMPLETION`은 `COMPLETE`로 리네임되었습니다. 앱 코드는 더 이상 `AdEvent`를 직접 참조하지 않으므로(이름 있는 메서드만 override) 이 리네임은 연동에 영향을 주지 않습니다.

### 로드 실패 콜백은 3단계를 거쳤습니다

| 세대 | 로드 실패 시그니처 | 상태 |
|---|---|---|
| ① v1.x (구) | `onFailedToReceiveAd(Object ad, String adapterName, int code, String msg)` | `@Deprecated` — 3.0 제거 예정 |
| ② 과도기 | `onFailedToReceiveAd(Object ad, AdNetworkType networkType, int code, String msg)` | `@Deprecated` — 3.0 제거 예정 |
| ③ **현행 표준** | **`onFailedToReceiveAd(int errorCode, String errorMsg)`** | ✅ 정식 |

①·②의 기본 구현은 위임 체인(enum → String → 표준)을 통해 ③으로 이어지므로, **표준 콜백 하나만 구현하면 내부 No-Ad(전 네트워크 실패)·SDK 미초기화를 포함한 모든 수신 실패를 받습니다.** 실패 경로의 네트워크 식별자는 항상 내부 합성값(`"SDK"`/`"Mediation"`)이라 매체에 유의미하지 않아 전달하지 않습니다. ①·② 구현이 이미 있다면 지우지 않아도 그대로 동작합니다(override가 위임 체인의 해당 지점을 대체).

```java
// ① Before (v1.x) — onEventAd 제거됨, 컴파일 오류. v1은 네트워크를 String으로 전달했습니다.
adView.setAdViewListener(new AdListener() {
    @Override public void onReceivedAd(String adapterName, Object ad) { /* ... */ }
    @Override public void onFailedToReceiveAd(Object ad, String adapterName, int code, String msg) { /* ... */ }
    @Override public void onEventAd(Object ad, AdEvent event) {
        if (event == AdEvent.CLICK) { /* 클릭 */ }
        else if (event == AdEvent.COMPLETION) { /* 완료 */ }
    }
});

// ③ After (현행 표준) — 필요한 이벤트만 override
adView.setAdViewListener(new AdListener() {
    @Override public void onReceivedAd(AdNetworkType networkType, Object ad) {
        // networkType로 switch: switch(networkType){ case PANGLE: ... }
    }
    @Override public void onFailedToReceiveAd(int errorCode, String errorMsg) {
        // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
    }
    @Override public void onAdShowFailed(Object ad, AdNetworkType networkType, int code, String msg) { /* 표시 실패 */ }
    @Override public void onAdClicked() { /* 클릭 */ }
    @Override public void onAdCompleted() { /* 완료 */ }
});
```

> 로드 성공 `onReceivedAd`와 노출 실패 `onAdShowFailed`는 **`AdNetworkType networkType` enum 오버로드가 표준**입니다. 기존 `String adapterName` 오버로드는 `@Deprecated`(3.0에서 제거 예정)이며, 문자열이 필요하면 `networkType.getAdapterName()`으로 얻으세요.

> 🗺️ **Deprecation 로드맵** — 차기 메이저 버전(3.0)에서 로드 실패의 String·enum 4-인자 오버로드와 노출 실패의 String 오버로드가 **제거**됩니다. 남는 것은 로드 실패 `onFailedToReceiveAd(int, String)`, 노출 실패 `onAdShowFailed(Object, AdNetworkType, int, String)`입니다.

> ℹ️ 전면형태(전면/리워드/전면 동영상)는 `FullScreenContentCallback`(GAM 규약)도 그대로 사용할 수 있습니다(변경 없음).

---

## Step 6. 새 기능 적용 (선택)

v2.0.0의 선택 신규 기능(진행 중 로드만 취소 `cancelLoad()`, 개인정보 동의/테스트 설정 전파)은 아래 **Step 8~9**를 참고하세요.

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
NativeAdViewBinder viewBinder = new NativeAdViewBinder.Builder(R.layout.item_native_ad)
    .setTitleId(R.id.nap_mx_tv_title)
    .setIconImageId(R.id.nap_mx_iv_icon)
    .setAdvertiserId(R.id.nap_mx_tv_adv)
    .setDescriptionId(R.id.nap_mx_tv_desc)
    .setMainViewId(R.id.nap_mx_iv_main)
    .setCtaId(R.id.nap_mx_btn_cta)
    .setAdChoicesPosition(AdChoicesPosition.RIGHT_TOP) // ✅ 선택 — AdChoices 모서리, 기본 RIGHT_TOP
    .build();

// ✅ 필수 — 만든 바인더를 뷰에 연결해야 적용됩니다 (누락 시 네이티브가 표시되지 않음)
nativeAdView.setViewBinder(viewBinder);
```

> ⚠️ **바인더는 반드시 `AMMNativeAdView.setViewBinder()`로 연결하세요.** `AdInfo.Builder.setAdViewBinder()`는 v2.0.1에서 제거되었습니다(바인딩은 뷰의 렌더링 관심사라는 업계 표준에 맞춤). 연결하지 않으면 어댑터가 바인더를 받지 못해 네이티브 광고가 표시되지 않습니다.

### setViewIds 제거 (v2.0.0 Breaking Change)

v2.0.0에서 `setViewIds()`가 **완전히 제거**되었습니다. 네이티브 렌더링을 자체 레이아웃으로 수행하는 어댑터(AdMixer 하우스애드·AdManager·Adfit·Pangle·NaverAdManager 등)가 `NativeAdViewBinder`를 직접 읽으므로, `setViewIds()` 호출을 제거하고 위의 `NativeAdViewBinder` 설정만으로 동작합니다. 네트워크에 따라 자체 렌더링 규격을 사용하는 경우 일부 슬롯이 적용되지 않을 수 있으니, 표시 결과는 각 네트워크별로 확인하십시오.

```java
// Before (v1.x) — 제거하세요
Map<String, Integer> ids = new HashMap<>();
ids.put("iv_image", R.id.my_image_view);
new AdInfo.Builder(ADUNIT_ID)
    .setViewIds(AdMixer.ADAPTER_ADFIT, ids)
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

## Step 8. 진행 중 로드만 취소 — `cancelLoad()` (선택)

표시 중인 광고를 끊지 않고 미완료 로드만 취소하는 `cancelLoad()`가 추가되었습니다(전면·리워드·전면 동영상 공통).

```java
// 화면 전환·백그라운드: 진행 중 로드만 취소 (표시 중이면 no-op)
interstitialAd.cancelLoad();
// 화면 종료: 전체 정리
interstitialAd.stop();
```

---

## Step 9. 아동 대상 설정 / 테스트 설정 (선택)

```java
// 아동 대상 앱이라면 필수 — Google Play Families 정책
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE);

// 테스트 설정
AdMixer.setTestMode(true);
AdMixer.setTestDeviceIds(Arrays.asList("..."));
```

자세한 내용은 [개인정보 / 테스트 설정](privacy.md)을 참고하세요.

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
| 전면 타입 설정 (`interstitialAdType`/`popupAdOption`) | **제거(Breaking)** — 관련 빌더 메서드가 모두 삭제되었고 전면은 Basic 전용입니다 |

---

## v2.0.x → 최신 버전 업그레이드

이미 **v2.0.0 / v2.0.1을 사용 중**이라면 위 Step 1~9는 해당되지 않습니다. 아래만 확인하세요.

### 컴파일 오류가 나는 변경

| 증상 | 원인 | 조치 |
|---|---|---|
| `cannot find symbol: method setPrivacyViewId(int)` | `NativeAdViewBinder.Builder.setPrivacyViewId()` 제거 | `setAdChoicesPosition(AdChoicesPosition.RIGHT_TOP)`으로 교체. 레이아웃의 `nap_mx_privacy_container` 슬롯도 삭제 가능(SDK가 자동 오버레이) |
| `cannot find symbol: method setAdViewBinder(...)` | `AdInfo.Builder.setAdViewBinder()` 제거 **(v2.0.1)** | `AMMNativeAdView.setViewBinder(viewBinder)`로 교체 |

### 🚨 컴파일 오류 **없이** 조용히 깨지는 변경

**어댑터 식별자 상수의 값이 바뀌었습니다.** 이름은 그대로라 빌드는 통과합니다.

| 상수 | 이전 값 | 새 값 |
|---|---|---|
| `AdMixer.ADAPTER_ADMANAGER` | `"AdManager"` | `"GoogleAdManager"` |
| `AdMixer.ADAPTER_ADFIT` | `"KaKao Adfit"` | `"AdFit"` |
| `AdMixer.ADAPTER_ADMIXER_HOUSE` | `"houseAd"` | `"HouseAd"` |

Java는 `public static final String` 상수를 **컴파일 시점에 앱 바이너리로 복사**합니다. SDK만 올리고 재컴파일하지 않으면 옛 문자열이 앱에 남아, `adapterName.equals(AdMixer.ADAPTER_ADFIT)` 같은 분기와 `setAdapterConfig(AdMixer.ADAPTER_ADFIT, ...)` 키가 **오류 없이 어긋납니다.**

> ✅ **조치** — 반드시 **클린 빌드로 재컴파일**하세요. 근본적으로는 문자열 비교 대신 `AdNetworkType` enum 오버로드로 전환하면 이 문제가 재발하지 않습니다.

### 화면이 바뀔 수 있는 변경

- **네이티브 메인 미디어 슬롯이 선언한 크기를 그대로 지킵니다.** 이전에는 AdMixer 자체 광고만 소재 비율로 축소돼 슬롯보다 작게 그려졌습니다. 슬롯 비율 ≠ 소재 비율이면 여백 위치가 하단 몰림 → 상·하 분산으로 바뀝니다. ([네이티브 가이드](native-ad.md))
- **Pangle 광고 로고가 좌측 상단 → 우측 상단**으로 이동합니다. 좌측 상단 유지는 `setAdChoicesPosition(AdChoicesPosition.LEFT_TOP)`.

### 권장 (선택)

- 로드 실패 처리를 **표준 콜백 `onFailedToReceiveAd(int errorCode, String errorMsg)` 하나로 정리** — String·enum 4-인자 오버로드는 둘 다 `@Deprecated`(3.0 제거 예정)이며, 기본 구현이 표준 콜백으로 위임하므로 기존 구현도 동작은 유지됩니다. 로드 성공(`onReceivedAd`)·노출 실패(`onAdShowFailed`)는 `AdNetworkType` enum 오버로드로 전환하세요([위 Step 5-B](#step-5-b-adlistener-이벤트-콜백-분리-필수--breaking-change)).

전체 변경 내역은 [릴리즈 노트](changelog.md)를 확인하세요.

---

## 업그레이드 후 검증 체크리스트

빌드가 통과한 뒤 아래 항목을 실기기에서 확인하면 대부분의 연동 이슈를 사전에 잡을 수 있습니다.

- [ ] 빌드 성공(컴파일 오류 0) — [컴파일 오류 빠른 해결](#컴파일-오류-빠른-해결) 표로 잔여 오류 해소
- [ ] `AdMixer.getInstance().initialize(...)`가 `Application.onCreate()`에서 1회 호출
- [ ] (AdManager 사용 시) 매니페스트 `com.google.android.gms.ads.APPLICATION_ID` 설정
- [ ] `registerAdapter()` 및 네트워크 SDK 수동 init(`PAGSdk.init`/`MobileAds.initialize`) 호출 제거
- [ ] 각 포맷 광고 **수신·노출** 확인(VERBOSE 로그 `adb logcat | grep AdMixerSDK`)
- [ ] 네이티브: `nap_mx_` View ID + `setViewBinder()` 적용, 누락 asset 케이스에서도 레이아웃 정상
- [ ] 전면/리워드/전면 동영상: 수신 후 `show(activity)` 노출, 닫힘/`onDestroy`에서 `stop()` 호출
- [ ] 배너/네이티브: `onResume`/`onPause`/`stop` 생명주기 연결
- [ ] 리워드: `onUserEarnedReward()`(또는 `onAdRewarded()`)에서만 보상 지급
- [ ] 아동 대상 앱이면 `setTagForChildDirectedTreatment` 설정 / 테스트 설정 적용 ([privacy.md](privacy.md))

> 광고가 안 나오거나 빌드가 막히면 [FAQ](faq.md)·[Q&A](qna.md)·[에러 코드](error-codes.md)를 먼저 확인하세요.

---

## 문의

업그레이드 관련 문의사항은 **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)**로 연락하세요.
