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
| `maxRetryCountInSlot(int)` | `int` | `-1` | 슬롯 내 최대 재시도 횟수 (-1 또는 0: 무제한, 양수: 해당 횟수까지) |
| `interstitialTimeout(int)` | `int` | `0` | 전면 광고 타임아웃 (초, 0: 서버 지정) |
| `interstitialAdType(InterstitialAdType)` | enum | `Basic` | 전면 광고 형태 |
| `popupAdOption(PopupInterstitialAdOption)` | - | `null` | 팝업형 옵션 |
| `setUseBackgroundAlpha(boolean)` | `boolean` | `true` | 전면 광고 배경 반투명 |
| `setMute(boolean)` | `boolean` | `false` | 동영상 음소거 |
| `showCloseButton(boolean)` | `boolean` | `true` | 전면 광고 닫기 버튼 표시 |
| `closeButtonDelay(int)` | `int` | `0` | 닫기 버튼 지연 표시 (초) |
| `showReportIcon(boolean)` | `boolean` | `false` | 신고 아이콘 표시 |
| `setAdViewBinder(NativeAdViewBinder)` | - | `null` | 네이티브 광고 바인더 설정 (`NativeAdView.setViewBinder()` 권장, 이 메서드는 AdInfo를 통한 대안) |
| `setAdapterConfig(String adapterName, Map<String,String> config)` | - | `{}` | 어댑터별 초기화 파라미터 설정 (예: AppLovin `sdkKey`). `AdMixer.ADAPTER_*` 상수를 adapterName으로 사용 |
| `setCustomParams(Map<String,String>)` | - | `{}` | S2S Callback 커스텀 파라미터 |
| `build()` | `AdInfo` | - | AdInfo 인스턴스 생성 |

---

## AdView (배너)

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

## InterstitialAd (전면 배너)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setAdListener(AdListener)` | 이벤트 리스너 등록 |
| `setListener(AdListener)` | `setAdListener()`의 별칭 |
| `loadAd()` | 광고 로드 시작 (수신 후 `showInterstitial()` 호출 필요) |
| `startInterstitial()` | 광고 로드 시작 + 수신 즉시 자동 노출 |
| `showInterstitial()` | 전면 광고 표시 (Activity context 필요) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stopInterstitial()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

---

## NativeAdView (네이티브)

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

## RewardInterstitialVideoAd (리워드 동영상)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setListener(AdListener)` | 이벤트 리스너 등록 |
| `loadRewardVideoAd()` | 광고 로드 시작 (수신 후 `showRewardVideoAd()` 호출 필요) |
| `startRewardVideoAd()` | 광고 로드 시작 + 수신 즉시 자동 노출 |
| `showRewardVideoAd()` | 광고 표시 |
| `closeInterstitialVideoAd()` | 광고 화면 닫기 (CLOSE/SKIPPED 이벤트 시 호출) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stopRewardVideoAd()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

---

## VideoAdView (인라인 동영상)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setAdViewListener(AdListener)` | 이벤트 리스너 등록 |
| `loadAd()` | 광고 로드 시작 |
| `onResume()` | Activity onResume에서 호출 (필수) |
| `onPause()` | Activity onPause에서 호출 (필수) |
| `destroy()` | 리소스 해제 — onDestroy에서 호출 (필수) |

---

## InterstitialVideoAd (전면 동영상)

| 메서드 | 설명 |
|--------|------|
| `setAdInfo(AdInfo)` | 광고 정보 설정 (필수) |
| `setListener(AdListener)` | 이벤트 리스너 등록 |
| `loadInterstitialVideoAd()` | 광고 로드 시작 (수신 후 `showInterstitialVideoAd()` 호출 필요) |
| `startInterstitialVideoAd()` | 광고 로드 시작 + 수신 즉시 자동 노출 |
| `showInterstitialVideoAd()` | 광고 표시 |
| `showInterstitialVideoAd(Activity activity)` | 광고 표시 (Activity 명시 지정) |
| `closeInterstitialVideoAd()` | 광고 닫기 (CLOSE/SKIPPED 이벤트 시 필수) |
| `cancelLoad()` | 진행 중인 **로드만 취소** (표시 중 광고는 보존). 로딩 중이 아니면 no-op |
| `stopInterstitialVideoAd()` | 광고 정지 및 리소스 해제 — onDestroy에서 호출 (필수) |
| `hasInterstitial` | 광고 수신 여부 (boolean 필드) |

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

## AdListener (이벤트 콜백)

```java
public interface AdListener {
    // 광고 수신 성공 — 메인 스레드에서 호출됨
    void onReceivedAd(@NonNull String adapterName, @NonNull Object adView);

    // 광고 수신 실패 — 메인 스레드에서 호출됨
    void onFailedToReceiveAd(@Nullable Object adView, @NonNull String adapterName,
                              int errorCode, @Nullable String errorMsg);

    // 광고 이벤트 발생 (표시, 클릭, 닫기 등) — 메인 스레드에서 호출됨
    void onEventAd(@NonNull Object adView, @NonNull AdEvent event);

    // 팝업 전면광고: 왼쪽 버튼 클릭 (default 빈 구현 제공)
    default void onLeftClicked(@NonNull String adapterName) {}

    // 팝업 전면광고: 오른쪽 버튼 클릭 (default 빈 구현 제공)
    default void onRightClicked(@NonNull String adapterName) {}
}
```

> ⚠️ `AdListener`는 내부적으로 `WeakReference`로 보유됩니다. 익명 클래스로 구현하면 GC에 의해 수집될 수 있으므로 반드시 **멤버 변수**로 선언하세요.

---

## AdEvent (이벤트 종류)

| 이벤트 | 설명 | 발생 포맷 |
|--------|------|----------|
| `DISPLAYED` | 광고가 화면에 표시됨 | 배너, 전면, 네이티브 |
| `CLICK` | 광고 소재 또는 링크 클릭 | 전체 |
| `CLOSE` | 광고 창 닫힘 | 전면, 동영상 |
| `LEFT_CLICK` | 팝업형: 왼쪽 버튼 클릭 | 전면(Popup) |
| `RIGHT_CLICK` | 팝업형: 오른쪽 버튼 클릭 | 전면(Popup) |
| `COMPLETION` | 동영상 재생 완료 | 동영상, 리워드 |
| `SKIPPED` | Skip 버튼 클릭 | 동영상, 리워드 |
| `EARNEDREWARD` | 리워드 획득 (동영상 시청 완료) | 리워드 |

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
