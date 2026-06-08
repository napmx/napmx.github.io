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
| `AdMixer.ADAPTER_MOBWITH` | Mobwith |
| `AdMixer.ADAPTER_NAVER_ADMANAGER` | Naver Ad Manager |
| `AdMixer.ADAPTER_TEADS` | Teads |

| 카운트다운 타입 상수 | 값 | 설명 |
|------------------|----|------|
| `AdMixer.AX_COUNT_TYPE_GAUGE` | `0` | 게이지(원형 프로그레스) 타입 |
| `AdMixer.AX_COUNT_TYPE_TEXT` | `1` | 숫자 텍스트 타입 |

---

## AdInfo.Builder

| 메서드 | 타입 | 기본값 | 설명 |
|--------|------|--------|------|
| `new Builder(String adUnitId)` | - | - | 필수. adUnit ID 지정 |
| `isRetry(boolean)` | `boolean` | `true` | 광고 수신 실패 시 자동 재시도 |
| `isLoadOnly(boolean)` | `boolean` | `false` | `true` 설정 시 광고 수신 후 자동 노출 안 함 (지연 노출용) |
| `maxRetryCountInSlot(int)` | `int` | `-1` | 슬롯 내 최대 자동 재시도 횟수. `-1`/`0` = 무제한, 양수 N = 최대 N회. **주로 배너 자동 재시도에 적용**(재시도 간격 최소 5초). **권장: 유한값(예 3~5).** 전면/리워드는 별도 루프 가드로 무한 재로드가 차단됨 |
| `interstitialTimeout(int)` | `int` | `0` | 전면 광고 타임아웃 (초, 0: 서버 지정) |
| `interstitialAdType(InterstitialAdType)` | enum | `Basic` | 전면 광고 형태 |
| `popupAdOption(PopupInterstitialAdOption)` | - | `null` | 팝업형 옵션 |
| `setUseBackgroundAlpha(boolean)` | `boolean` | `true` | 전면 광고 배경 반투명 |
| `setMute(boolean)` | `boolean` | `false` | 동영상 음소거 |
| `showCloseButton(boolean)` | `boolean` | `true` | 전면 광고 닫기 버튼 표시 |
| `setCloseButtonBound(int)` | `int` | `100` | Basic/CountDown 전면 닫기(X) 버튼 터치 영역 비율 (20~100%, 범위 밖은 클램프) |
| `showReportIcon(boolean)` | `boolean` | `false` | 신고 아이콘 표시 |
| `setAdViewBinder(NativeAdViewBinder)` | - | `null` | 네이티브 광고 바인더 설정 (`AMMNativeAdView.setViewBinder()` 권장, 이 메서드는 AdInfo를 통한 대안) |
| `setAdapterConfig(String adapterName, Map<String,String> config)` | - | `{}` | 어댑터별 초기화 파라미터 설정 (예: AppLovin `sdkKey`). `AdMixer.ADAPTER_*` 상수를 adapterName으로 사용 |
| `setCustomParams(Map<String,String>)` | - | `{}` | S2S Callback 커스텀 파라미터 |
| `build()` | `AdInfo` | - | AdInfo 인스턴스 생성 |

---

## AMMBannerView (배너)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadAd()` | 광고 로드 시작 (`AdInfo.isLoadOnly(true)` 설정 시 자동 노출 안 함) |
| `showAd()` | 지연 노출 모드(`isLoadOnly=true`)에서 광고 표시 |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `destroy()` | 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasAd` | 광고 수신 여부 (boolean 필드) |

---

## AMMInterstitial (전면 배너)

| 메서드 | 설명 |
|--------|------|
| `static load(Context, AdInfo, AMMInterstitialLoadCallback)` | **정적 로드.** 로드 완료 시 콜백으로 로드된 광고 객체 전달 |
| `setFullScreenContentCallback(FullScreenContentCallback)` | 노출/클릭/닫힘/표시실패 콜백 등록 |
| `show(Activity)` | 전면 광고 표시 (Activity Context 필요) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stopInterstitial()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

---

## AMMNativeAdView (네이티브)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setViewBinder(NativeAdViewBinder)` | 레이아웃 바인더 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadNativeAd()` | 광고 로드 시작 (항상 지연 노출 모드 — `showAd()` 명시 호출 필수) |
| `showAd()` | 광고 소재 렌더링 및 노출 — `onReceivedAd()` 콜백에서 호출 (필수) |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `destroy()` | 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasAd` | 광고 수신 여부 (boolean 필드) |

---

## AMMRewardVideo (리워드 동영상)

| 메서드 | 설명 |
|--------|------|
| `static load(Context, AdInfo, AMMRewardVideoLoadCallback)` | **정적 로드.** 로드 완료 시 콜백으로 로드된 광고 객체 전달 |
| `setFullScreenContentCallback(FullScreenContentCallback)` | 노출/클릭/완료/닫힘/표시실패 콜백 등록 |
| `show(Activity, OnUserEarnedRewardListener)` | 광고 표시 + 시청 완료 시 `onUserEarnedReward()` 보상 적립 콜백 |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stopRewardVideoAd()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

---

## AMMVideoView (인라인 동영상)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadAd()` | 광고 로드 시작 |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `destroy()` | 리소스 해제 — onDestroy에서 호출 (필수) |

---

## AMMVideoInterstitial (전면 동영상)

| 메서드 | 설명 |
|--------|------|
| `static load(Context, AdInfo, AMMVideoInterstitialLoadCallback)` | **정적 로드.** 로드 완료 시 콜백으로 로드된 광고 객체 전달 |
| `setFullScreenContentCallback(FullScreenContentCallback)` | 노출/클릭/완료/닫힘/표시실패 콜백 등록 |
| `show(Activity)` | 광고 표시 (Activity Context 필요) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stopInterstitialVideoAd()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

---

## 풀스크린 로드/콘텐츠 콜백 (전면·리워드·전면 동영상)

v2.0.0부터 전면/리워드/전면 동영상은 정적 `load()`로 로드하고, 콜백으로 받은 광고 객체에 `FullScreenContentCallback`을 등록합니다.

### 로드 콜백

```java
// AMMInterstitial.load() — 전면
public abstract class AMMInterstitialLoadCallback {
    public abstract void onSuccessLoadInterstitial(@NonNull String adapterName, @NonNull AMMInterstitial ad);
    public abstract void onFailLoadInterstitial(int errorCode, @Nullable String errorMsg);
}

// AMMRewardVideo.load() — 리워드
public abstract class AMMRewardVideoLoadCallback {
    public abstract void onSuccessLoadReward(@NonNull String adapterName, @NonNull AMMRewardVideo ad);
    public abstract void onFailLoadReward(int errorCode, @Nullable String errorMsg);
}

// AMMVideoInterstitial.load() — 전면 동영상
public abstract class AMMVideoInterstitialLoadCallback {
    public abstract void onSuccessLoadVideoInterstitial(@NonNull String adapterName, @NonNull AMMVideoInterstitial ad);
    public abstract void onFailLoadVideoInterstitial(int errorCode, @Nullable String errorMsg);
}
```

### FullScreenContentCallback (노출 단계 이벤트)

필요한 메서드만 오버라이드합니다(모두 기본 빈 구현). GAM(Google Mobile Ads)의 `FullScreenContentCallback`과 동일한 구조/네이밍을 따릅니다.

```java
public abstract class FullScreenContentCallback {
    public void onAdShowedFullScreenContent() {}                       // 풀스크린 표시됨
    public void onAdClicked() {}                                       // 광고 클릭
    public void onAdDismissedFullScreenContent() {}                    // 광고 닫힘
    public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {} // 표시 실패
    public void onAdCompleted() {}                                     // [확장] 동영상 재생 완료(전면/리워드 비디오)
    public void onAdLeftClicked() {}                                   // [확장] 팝업형 전면 좌측 버튼 클릭
    public void onAdRightClicked() {}                                  // [확장] 팝업형 전면 우측 버튼 클릭
}
```

### OnUserEarnedRewardListener (리워드 적립)

```java
public interface OnUserEarnedRewardListener {
    void onUserEarnedReward(); // 시청 완료 → 보상 지급 시점 (AMMRewardVideo.show()에 전달)
}
```

> ℹ️ 팝업형 전면의 좌/우 버튼은 SDK가 자동으로 닫지 않습니다. `onAdLeftClicked()`/`onAdRightClicked()`에서 앱이 직접 `stopInterstitial()` 또는 화면 종료를 처리하세요.

---

## 풀스크린 광고 생명주기 권장 호출 순서 (#56)

전면 / 리워드 / 전면 동영상 공통:

| 상황 | 권장 호출 | 효과 |
|------|----------|------|
| 호스트 화면 전환·백그라운드 진입 (표시 중 광고는 유지) | `cancelLoad()` | 진행 중인 **로드만** 취소. SHOWING 중이면 no-op이라 광고가 끊기지 않음 |
| 광고 닫힘/실패 콜백 수신 후 | `stopXxx()` → `onDestroy()` | 미디에이션 컨트롤러 destroy + 서버 config 리스너 해제(재동기화 재로드 대상에서 제외) |
| 화면 완전 종료 (`Activity#onDestroy`) | `stopXxx()` (필수) | 전체 리소스 해제 |

> `cancelLoad()`는 "로드만 취소", `stopXxx()/destroy()`는 "전체 정리(리스너 해제 포함)"로 구분됩니다.
> `media-conf` 재동기화 시 SHOWING/이미-로드 유닛은 SDK가 자동으로 재로드하지 않습니다(REQ-LIFECYCLE-RESYNC-56).

---

## AdListener (이벤트 콜백 — 배너·네이티브·인라인 동영상)

`AMMBannerView` / `AMMNativeAdView` / `AMMVideoView`(View 기반 포맷)의 이벤트 콜백입니다. 전면·리워드·전면 동영상은 [FullScreenContentCallback](#풀스크린-로드콘텐츠-콜백-전면리워드전면-동영상)을 사용합니다.

```java
public interface AdListener {
    // 광고 수신 성공 — 메인 스레드에서 호출됨
    void onReceivedAd(@NonNull String adapterName, @NonNull Object adView);

    // 광고 수신 실패 — 메인 스레드에서 호출됨
    void onFailedToReceiveAd(@Nullable Object adView, @NonNull String adapterName,
                              int errorCode, @Nullable String errorMsg);

    // 광고 이벤트 발생 (표시, 클릭, 닫기 등) — 메인 스레드에서 호출됨
    void onEventAd(@NonNull Object adView, @NonNull AdEvent event);

    // 광고 노출(show) 단계 실패 — 로드 실패와 구분 (default 빈 구현 제공)
    default void onAdShowFailed(@Nullable Object adView, @NonNull String adapterName,
                                int errorCode, @Nullable String errorMsg) {}
}
```

> ⚠️ `AdListener`는 내부적으로 `WeakReference`로 보유됩니다. 익명 클래스로 구현하면 GC에 의해 수집될 수 있으므로 반드시 **멤버 변수**로 선언하세요.

---

## AdEvent (이벤트 종류 — `AdListener.onEventAd`)

`AdListener`(배너·네이티브·인라인 동영상)에서 전달되는 이벤트입니다.

| 이벤트 | 설명 | 발생 포맷 |
|--------|------|----------|
| `DISPLAYED` | 광고가 화면에 표시됨 | 배너, 네이티브, 인라인 동영상 |
| `CLICK` | 광고 소재 또는 링크 클릭 | 배너, 네이티브, 인라인 동영상 |
| `COMPLETION` | 동영상 재생 완료 | 인라인 동영상 |
| `SKIPPED` | Skip 버튼 클릭 | 인라인 동영상 |

> ℹ️ **전면·리워드·전면 동영상**의 노출/클릭/닫힘/완료/보상 이벤트는 `AdEvent`가 아닌 [`FullScreenContentCallback`](#풀스크린-로드콘텐츠-콜백-전면리워드전면-동영상)(`onAdShowedFullScreenContent`/`onAdClicked`/`onAdDismissedFullScreenContent`/`onAdCompleted` 등)과 `OnUserEarnedRewardListener.onUserEarnedReward()`로 전달됩니다.

---

## AdMixerLog (디버그 로그)

```java
// 로그 레벨 설정
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE); // 개발 환경
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.ERROR);   // 운영 환경 (에러만 출력)
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.NONE);    // 로그 완전 비활성화
```

| LogLevel | 설명 |
|---------|------|
| `VERBOSE` | 모든 로그 출력 (개발 권장) |
| `DEBUG` | 디버그 이상 출력 |
| `INFO` | 정보 이상 출력 |
| `WARN` | 경고 이상 출력 (기본값) |
| `ERROR` | 에러만 출력 (운영 환경 권장) |
| `ASSERT` | ASSERT(wtf) 레벨만 출력 |
| `NONE` | 모든 로그 완전 비활성화 |
