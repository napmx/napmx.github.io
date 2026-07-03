# API 레퍼런스

---

## AdMixer (초기화 및 설정)

```java
// 초기화 — build.gradle에 추가된 어댑터 모듈은 initialize() 호출 시 자동 등록됩니다
AdMixer.getInstance().initialize(Context context, String mediaKey, ArrayList<String> adUnitIds)

// COPPA(아동 대상 앱) 여부 설정 — int 상수 사용
// AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE  = 1
// AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_FALSE = 0
AdMixer.setTagForChildDirectedTreatment(int id)

// 개인정보 동의 (워터폴에서 각 네트워크로 자동 전파) — 선택
AdMixer.setGdprConsent(boolean hasUserConsent)   // GDPR 사용자 동의
AdMixer.setCcpaDoNotSell(boolean doNotSell)      // CCPA Do-Not-Sell 플래그
AdMixer.setUsPrivacy(String usPrivacyString)     // CCPA US Privacy 문자열(예 "1YNN")

// 테스트 설정 — 선택
AdMixer.setTestMode(boolean enabled)
AdMixer.setTestDeviceIds(List<String> ids)
```

| 개인정보/테스트 메서드 | 설명 |
|------|------|
| `setGdprConsent(boolean)` | GDPR 사용자 동의 여부. 미설정 시 적용 안 함 |
| `setCcpaDoNotSell(boolean)` | CCPA "Do Not Sell" 플래그 |
| `setUsPrivacy(String)` | CCPA US Privacy 문자열(IAB CMP 연동 시) |
| `setTestMode(boolean)` / `isTestMode()` | 전역 테스트 모드 |
| `setTestDeviceIds(List<String>)` / `getTestDeviceIds()` | 테스트 디바이스 광고 ID 목록 |
| `setTagForChildDirectedTreatment(int)` | COPPA(-1 미설정/0 false/1 true) |

> 네트워크별 전파 매핑은 [개인정보 동의 및 테스트 설정](privacy.md) 참고.

> ℹ️ **어댑터 자동 등록 (v2.0.0)**: `initialize()` 내부에서 클래스패스(Gradle 의존성)에 포함된 어댑터를 자동으로 탐지하여 등록합니다. `registerAdapter()` 수동 호출은 더 이상 필요하지 않습니다.

| 어댑터 상수 | 대상 네트워크 |
|-----------|-------------|
| `AdMixer.ADAPTER_ADMANAGER` | Google AdManager |
| `AdMixer.ADAPTER_ADFIT` | Kakao Adfit |
| `AdMixer.ADAPTER_PANGLE` | Pangle (TikTok) |
| `AdMixer.ADAPTER_APPLOVIN` | AppLovin |
| `AdMixer.ADAPTER_UNITY` | Unity Ads |
| `AdMixer.ADAPTER_NAVER_ADMANAGER` | Naver Ad Manager |
| `AdMixer.ADAPTER_TEADS` | Teads |

---

## AdInfo.Builder

| 메서드 | 타입 | 기본값 | 설명 |
|--------|------|--------|------|
| `new Builder(String adUnitId)` | - | - | 필수. adUnit ID 지정 |
| `interstitialTimeout(int)` | `int` | `0` | 전면 광고 타임아웃 (초, 0: 서버 지정) |
| `setMute(boolean)` | `boolean` | `false` | 동영상 음소거 |
| `setCloseButtonBound(int)` | `int` | `100` | 전면 광고 닫기 'X' 버튼 터치 영역 비율(%, 20~100 범위로 클램프) |
| `setAdapterConfig(String adapterName, Map<String,String> config)` | - | `{}` | 어댑터별 초기화 파라미터 설정 (예: AppLovin `sdkKey`). `AdMixer.ADAPTER_*` 상수를 adapterName으로 사용 |
| `setCustomParams(Map<String,String>)` | - | `{}` | S2S Callback 커스텀 파라미터 |
| `build()` | `AdInfo` | - | AdInfo 인스턴스 생성 |

---

## AMMBannerView (배너)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadAd()` | 광고 로드 시작. 뷰가 화면에 부착(addView)되는 시점에 자동 표시 |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `stop()` | 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasAd` | 광고 수신 여부 (boolean 필드) |

---

## AMMInterstitial (전면 배너)

> ℹ️ 정적 `loadAd()` + `FullScreenContentCallback` 구조. 구 `InterstitialAd` 클래스는 제거되었습니다(→ `AMMInterstitial`).

| 멤버 | 설명 |
|--------|------|
| `static loadAd(Context, AdInfo, AMMInterstitialLoadCallback)` | 정적 로드. 완료 시 콜백으로 로드된 광고 객체 반환 |
| `setFullScreenContentCallback(FullScreenContentCallback)` | 노출/클릭/닫힘/표시실패 콜백 등록 (Kotlin: `fullScreenContentCallback` 프로퍼티) |
| `show(Activity)` | 전면 광고 표시 (Activity Context 필요) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stop()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

> ℹ️ 인스턴스 메서드(`setAdInfo`/`setAdListener`/`loadAd()`/`showAd()`)도 제공됩니다. **즉시 노출 `startInterstitial()`은 v2.0.0에서 제거**되었습니다 — 로드 후 `show()`로 노출하세요. 신규 연동은 정적 `loadAd()`를 권장합니다. (구 `InterstitialAd` 클래스는 제거됨)

---

## AMMVideoInterstitial (전면 동영상)

> ℹ️ 전면 광고와 동일한 정적 `loadAd()` + `FullScreenContentCallback` 구조. 구 `InterstitialVideoAd` 클래스는 제거되었습니다(→ `AMMVideoInterstitial`).

| 멤버 | 설명 |
|--------|------|
| `static loadAd(Context, AdInfo, AMMVideoInterstitialLoadCallback)` | 정적 로드. 완료 시 콜백으로 로드된 광고 객체 반환 |
| `setFullScreenContentCallback(FullScreenContentCallback)` | 노출/클릭/재생완료/닫힘/표시실패 콜백 등록 |
| `show(Activity)` | 전면 동영상 표시 (Activity Context 필요) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존) |
| `stop()` | 광고 정지 및 리소스 해제 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

---

## AMMNativeAdView (네이티브)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setViewBinder(NativeAdViewBinder)` | 레이아웃 바인더 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadAd()` | 광고 로드 시작. 뷰가 화면에 부착(addView)되는 시점에 자동 렌더링 |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `stop()` | 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasAd` | 광고 수신 여부 (boolean 필드) |

---

## AMMRewardVideo (리워드 동영상)

> ℹ️ 정적 `loadAd()` + `show(activity, OnUserEarnedRewardListener)` 구조. 구 `RewardInterstitialVideoAd` 클래스는 제거되었습니다(→ `AMMRewardVideo`).

| 멤버 | 설명 |
|--------|------|
| `static loadAd(Context, AdInfo, AMMRewardVideoLoadCallback)` | 정적 로드. 완료 시 콜백으로 로드된 광고 객체 반환 |
| `setFullScreenContentCallback(FullScreenContentCallback)` | 노출/클릭/재생완료/닫힘/표시실패 콜백 등록 |
| `show(Activity, OnUserEarnedRewardListener)` | 광고 표시 + 보상 적립 리스너 등록 |
| `show(Activity)` | 광고 표시 (보상 리스너 없이) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존) |
| `stop()` | 광고 정지 및 리소스 해제 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

> 보상 적립은 `OnUserEarnedRewardListener.onUserEarnedReward()`로 수신합니다(영상 재생 완료 `onAdCompleted()`와는 별개).

---

## AMMVideoView (인라인 동영상)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadAd()` | 광고 로드 시작. 뷰가 화면에 부착(addView)되는 시점에 자동 노출 |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `stop()` | 리소스 해제 — onDestroy에서 호출 (필수) |

---

## 풀스크린 로드 콜백 / 노출 콜백

전면 / 리워드 / 전면 동영상의 정적 `loadAd()` 결과 콜백과 노출 단계 콜백입니다.

**Load 콜백 (abstract class — 로드 결과 1회 통지)**

| 콜백 클래스 | 성공 콜백 | 실패 콜백 |
|---|---|---|
| `AMMInterstitialLoadCallback` | `onSuccessLoadInterstitial(String adapterName, AMMInterstitial ad)` | `onFailLoadInterstitial(int errorCode, String errorMsg)` |
| `AMMRewardVideoLoadCallback` | `onSuccessLoadReward(String adapterName, AMMRewardVideo ad)` | `onFailLoadReward(int errorCode, String errorMsg)` |
| `AMMVideoInterstitialLoadCallback` | `onSuccessLoadVideoInterstitial(String adapterName, AMMVideoInterstitial ad)` | `onFailLoadVideoInterstitial(int errorCode, String errorMsg)` |

**FullScreenContentCallback (abstract class — 노출 단계, 필요한 메서드만 오버라이드)**

| 콜백 | 설명 | 비고 |
|------|------|------|
| `onAdShowedFullScreenContent()` | 광고가 풀스크린으로 표시됨 (임프레션) | |
| `onAdClicked()` | 광고 클릭 | |
| `onAdDismissedFullScreenContent()` | 광고 닫힘 | |
| `onAdFailedToShowFullScreenContent(AdError)` | 광고 표시 실패 | |
| `onAdCompleted()` | 비디오 재생 완료 | **AdMixer 확장** (전면비디오/리워드) |

**OnUserEarnedRewardListener (interface — 리워드 보상 적립)**

| 콜백 | 설명 |
|------|------|
| `onUserEarnedReward()` | 사용자가 시청을 완료해 보상을 획득함. `AMMRewardVideo.show(activity, listener)`로 등록 |

**AdError (class — 표시 실패 정보)**

| 메서드 | 설명 |
|--------|------|
| `getCode()` | 에러 코드 (`AX_ERR_*`) |
| `getMessage()` | 에러 메시지 |
| `getDomain()` | 에러 도메인 (기본 `com.nasmedia.admixerssp`) |

> ℹ️ **제거된 구 클래스**: `AdView`(→`AMMBannerView`), `NativeAdView`(→`AMMNativeAdView`), `VideoAdView`(→`AMMVideoView`), `InterstitialAd`(→`AMMInterstitial`), `RewardInterstitialVideoAd`(→`AMMRewardVideo`), `InterstitialVideoAd`(→`AMMVideoInterstitial`)는 모두 **제거**되었습니다. `AMM*` 클래스로 전환하세요(메서드 시그니처 동일).

---

## 풀스크린 광고 생명주기 권장 호출 순서 (#56)

전면 / 리워드 / 전면 동영상 공통:

| 상황 | 권장 호출 | 효과 |
|------|----------|------|
| 호스트 화면 전환·백그라운드 진입 (표시 중 광고는 유지) | `cancelLoad()` | 진행 중인 **로드만** 취소. SHOWING 중이면 no-op이라 광고가 끊기지 않음 |
| 광고 닫힘/실패 콜백 수신 후 | `stopXxx()` → `onDestroy()` | 미디에이션 컨트롤러 destroy + 서버 config 리스너 해제(재동기화 재로드 대상에서 제외) |
| 화면 완전 종료 (`Activity#onDestroy`) | `stopXxx()` (필수) | 전체 리소스 해제 |

> `cancelLoad()`는 "로드만 취소", `stop()`은 "전체 정리(리스너 해제 포함)"로 구분됩니다.
> `media-conf` 재동기화 시 SHOWING/이미-로드 유닛은 SDK가 자동으로 재로드하지 않습니다(REQ-LIFECYCLE-RESYNC-56).

---

## AdListener (이벤트 콜백)

> **[REQ-20260609 / v2.0.0]** 기존 단일 `onEventAd(adView, AdEvent)`는 **이름 있는 이벤트 메서드로 분리**되었습니다. `AdListener`는 `abstract class`이며 **모든 메서드가 기본 no-op**이라 **필요한 메서드만 override**하면 됩니다(필수 구현 없음). 기존 `onEventAd` 구현은 아래 named 메서드로 이전해야 합니다(Breaking Change).

```java
public abstract class AdListener {
    // ── 로드 콜백 ──
    public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {}
    public void onFailedToReceiveAd(@Nullable Object adView, @NonNull String adapterName,
                                    int errorCode, @Nullable String errorMsg) {}
    // 노출(show) 실패 — 로드 실패와 구분되는 표시 단계 실패
    public void onAdShowFailed(@Nullable Object adView, @NonNull String adapterName,
                               int errorCode, @Nullable String errorMsg) {}

    // ── 이벤트 콜백 (필요한 것만 override) ──
    public void onAdDisplayed() {}      // 광고 노출됨
    public void onAdClicked() {}        // 광고 클릭
    public void onAdClosed() {}         // 광고 닫힘
    public void onAdCompleted() {}      // 동영상 재생 완료
    public void onAdSkipped() {}        // 동영상 Skip
    public void onAdRewarded() {}       // 리워드 적립
}
```

> ⚠️ 배너(`AMMBannerView`)는 `AdListener`를 내부적으로 `WeakReference`로 보유합니다. 익명 클래스로 구현하면 GC에 의해 수집될 수 있으므로 반드시 **멤버 변수**로 선언하세요.
>
> ℹ️ 전면형태(전면/리워드/전면 동영상)는 GAM 규약의 [`FullScreenContentCallback`](interstitial.md)으로도 노출 단계 콜백을 받을 수 있습니다.

---

## 이벤트 콜백 메서드 (구 AdEvent 매핑)

기존 `onEventAd(adView, AdEvent)`의 `AdEvent` 분기는 아래 이름 있는 메서드로 대체되었습니다. (`AdEvent` enum은 SDK 내부 전용으로 전환)

| 구 `AdEvent` | 신 콜백 메서드 | 설명 | 발생 포맷 |
|--------|------|------|----------|
| `DISPLAYED` | `onAdDisplayed()` | 광고가 화면에 표시됨 | 배너, 전면, 네이티브, 동영상 |
| `CLICK` | `onAdClicked()` | 광고 소재 또는 링크 클릭 | 전체 |
| `CLOSE` | `onAdClosed()` | 광고 창 닫힘 | 전면, 동영상 |
| `COMPLETION` | `onAdCompleted()` | 동영상 재생 완료 | 동영상, 리워드 |
| `SKIPPED` | `onAdSkipped()` | Skip 버튼 클릭 | 동영상, 리워드 |
| `EARNEDREWARD` | `onAdRewarded()` | 리워드 획득 (동영상 시청 완료) | 리워드 |

> ℹ️ 좌측 `AdEvent` 상수명은 **구(v1.x) 기준**입니다. v2.0.0에서 `AdEvent`는 **SDK 내부 전용**으로 전환되었고, 내부적으로 `COMPLETION`은 `COMPLETE`로 리네임되었습니다. 앱은 `AdEvent`를 직접 참조하지 않고 위의 이름 있는 메서드만 override합니다.

---

## AdMixerLog (디버그 로그)

```java
// 로그 레벨 설정
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE); // 개발 환경
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.ERROR);   // 운영 환경 (에러만 출력)
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.NONE);    // 로그 완전 비활성화
```

| LogLevel | priority | 설명 |
|---------|:--------:|------|
| `NONE` | 0 | 모든 로그 완전 비활성화 |
| `VERBOSE` | 2 | 모든 로그 출력 (개발 권장) |
| `DEBUG` | 3 | 디버그 이상 출력 |
| `INFO` | 4 | 정보 이상 출력 |
| `WARN` | 5 | 경고 이상 출력 (기본값) |
| `ERROR` | 6 | 에러만 출력 (운영 환경 권장) |
| `ASSERT` | 7 | ASSERT(wtf) 레벨만 출력 |
