# 릴리즈 노트

---

## vNEXT (미출시)

### 새로운 기능

- **GAM 스타일 풀스크린 광고 API 추가 (전면/리워드/전면 동영상)**: GMA(Google Mobile Ads) / iOS-AdMixer와 동일한 구조로 통일.
  - 정적 로드 — `AMMInterstitial.load(...)` / `AMMRewardVideo.load(...)` / `AMMVideoInterstitial.load(context, adInfo, callback)` 가 로드 완료 시 콜백으로 **로드된 광고 객체**를 반환 (인스턴스 생성 불필요)
  - 노출 콜백 — `FullScreenContentCallback`(`onAdShowedFullScreenContent` / `onAdClicked` / `onAdDismissedFullScreenContent` / `onAdFailedToShowFullScreenContent(AdError)`, AdMixer 확장 `onAdCompleted` / `onAdLeftClicked` / `onAdRightClicked`). Kotlin은 `ad.fullScreenContentCallback = ...` 프로퍼티 접근 지원
  - 리워드 보상 적립 — `AMMRewardVideo.show(activity, OnUserEarnedRewardListener)`의 `onUserEarnedReward()` (영상 재생 완료 `onAdCompleted()`와 별개)
  - 노출 실패 신호(`onAdFailedToShowFullScreenContent`) 신설 — 미준비 상태 `show()` 등 표시 실패를 로드 실패와 분리. ([전면](interstitial.md)·[동영상](video.md)·[리워드](rewarded-video.md)·[API 레퍼런스](api-reference.md) 가이드)

### 변경 사항 (Breaking — 구 클래스 제거)

- **구 광고 클래스 6종 제거**: 모든 공개 광고 클래스가 GMA(Google Mobile Ads) / iOS-AdMixer 정합의 `AMM*` 네이밍으로 통일되었습니다. 호스트 앱은 구 클래스 참조를 아래 `AMM*`로 교체해야 합니다(**메서드 시그니처는 동일** — 클래스명만 변경).
  - `AdView` → `AMMBannerView`
  - `NativeAdView` → `AMMNativeAdView`
  - `VideoAdView` → `AMMVideoView`
  - `InterstitialAd` → `AMMInterstitial`
  - `RewardInterstitialVideoAd` → `AMMRewardVideo`
  - `InterstitialVideoAd` → `AMMVideoInterstitial`
  - 레이아웃 XML의 `<com.nasmedia.admixerssp.ads.AdView .../>`도 `<com.nasmedia.admixerssp.ads.AMMBannerView .../>`로 변경해야 합니다.
  - 공개 API 제거이므로 **메이저 버전 상향**이 필요합니다.
- **`AdListener` 이벤트 콜백 분리 (Breaking)**: 단일 `onEventAd(adView, AdEvent)`를 이름 있는 메서드로 분리 — `onAdDisplayed`/`onAdClicked`/`onAdClosed`/`onAdCompleted`/`onAdSkipped`/`onAdLeftClicked`/`onAdRightClicked`/`onAdRewarded`. `AdListener`는 `abstract class`로 전환되어 **필요한 메서드만 override**(필수 구현 없음). `onReceivedAd`/`onFailedToReceiveAd`는 시그니처 동일. `AdEvent` enum은 SDK 내부 전용으로 전환. ([마이그레이션 Step 5-B](migration-to-v2.md)·[API 레퍼런스](api-reference.md))
- **전면 광고 타입 Basic 전용 (Breaking)**: 전면(Interstitial)은 Basic 타입만 제공 — `AdInfo.Builder`의 `interstitialAdType`/`setInterstitialAdType`/`popupAdOption`/`setPopupAdOption` 제거. Popup 렌더링은 내부(서버 설정 기반)로만 유지. BACK 키 제어는 `AdInfo.Builder.setDisableBackKey` 사용. ([전면 가이드](interstitial.md))
- **전면 CountDown 타입 완전 제거 (Breaking)**: 카운트다운 전면 타입과 관련 코드를 SDK에서 모두 삭제 — `InterstitialAdType.CountDown`, `CountdownProgressView`, `PopupInterstitialAdOption.setCountDown`/`COUNTDOWN_MODE_*`, `AdMixer.AX_COUNT_TYPE_GAUGE`/`TEXT`, 어댑터의 `countdown_type` 전송/파싱 등. (Basic/Popup만 유지)
- **배너·네이티브 자동 갱신/재시도 옵션 정리 (Breaking)**: `AdInfo.Builder`의 `isRetry`·`maxRetryCountInSlot` 제거 — 자동 갱신/재로드는 서버 광고단위 `interval`(초)>0일 때만 동작, 무한 루프는 내부 가드가 차단. 네이티브도 배너와 동일 로직 적용. ([정책](../ADMIXER_SDK_POLICY.md))

### 네트워크 버전 업데이트

- **Pangle `7.7.0.2` → `8.0.0.5`**: 호스트 앱 API 변경 없음. 다만 `PAGConfig.setGDPRConsent`가 Pangle `7.9.0.9`에서 제거되어, 8.x는 GDPR 동의를 퍼블리셔 CMP의 **TCF v2 동의문자열**로 자동 처리합니다(`AdMixer.setGdprConsent` 값은 더 이상 Pangle로 전파되지 않음). CCPA(`setPAConsent`)는 유지. Pangle 8.0.0.5는 최소 GMA(play-services-ads) `25.1.0` 이상 권장.
- **Google AdManager(play-services-ads) `24.8.0` → `25.2.0`**: 호스트 앱 API 변경 없음. (`25.3.0`+는 Kotlin 메타데이터 호환 이슈로 광범위한 호스트 지원을 위해 `25.2.0`에 고정)
- **AppLovin `13.5.0` → `13.6.3`**: 호스트 앱 API 변경 없음.
- **Naver NAM(nam-bom) `8.14.0` → `8.16.0`**: 호스트 앱 API 변경 없음.
- **Mobwith(mobwithSDK) `1.0.68` → `1.0.83`**: 호스트 앱 API 변경 없음. ⚠️ **저장소 추가 필요** — 1.0.83이 `com.github.Dimezis:BlurView`를 전이 의존하므로 `settings.gradle`(또는 프로젝트 `build.gradle`) repositories에 `maven { url 'https://jitpack.io' }` 추가가 필요합니다.

### 버그 수정

- **Adfit 네이티브 광고가 표시되지 않던 문제 수정**: Adfit `3.21.17`이 R8 난독화로 출시되며 내부 에셋 추출(리플렉션)이 실패해 정상 광고를 "빈 광고"로 오인·거부하던 현상 → 추출 실패 시 SDK 로드 결과를 신뢰하도록 수정. (호스트 앱 변경 불필요)
- **Mobwith 네이티브 메인 이미지가 렌더되지 않던 문제 수정**: 메인 이미지 뷰 슬롯 매핑 오류로 SDK가 대표 이미지를 그리지 못하던 현상 수정. (호스트 앱 변경 불필요)
- **Pangle 네이티브 광고 로고(ad logo) 누락 수정**: Pangle 정책상 필수인 광고 로고 뷰를 등록하도록 수정. (호스트 앱 변경 불필요)
- **Naver 네이티브 광고 고지(notice) 라벨 누락 수정**: 기본 레이아웃에서 광고 고지 텍스트가 바인딩되지 않던 현상 수정. (호스트 앱 변경 불필요)

---

## v2.0.0 (2026-05-12)

### 새로운 기능

- **광고 신고하기 기능 추가**: `AdInfo.Builder.showReportIcon(true)` 설정으로 광고 소재 위에 신고 아이콘(ⓘ) 표시. PixelCopy 기반 소재 자동 캡처 지원 (Android 8.0+)
- **NaverAdManager 어댑터 추가**: Naver Ad Manager (NAM) 미디에이션 지원 (`admixer-naveradmanager:2.0.0`)
- **Teads 어댑터 추가**: Teads 미디에이션 지원 (`admixer-teads:2.0.0`)
- **통합 개인정보 동의/테스트 API 추가**: `AdMixer.setGdprConsent/setCcpaDoNotSell/setUsPrivacy/setTestMode/setTestDeviceIds` — 워터폴에서 각 네트워크 privacy/test API로 자동 전파 (AppLovin/Unity/Pangle/AdManager)
- **`cancelLoad()` 추가**: 표시 중인 광고를 끊지 않고 진행 중 로드만 취소 (전면·리워드·전면 동영상)
- **클라이언트 키 주입(`setAdapterConfig`)**: 서버 미제공 시 네트워크 키를 매체가 주입 (Server-Precedence 병합)
- **Jetpack Compose 지원(`admixer-compose`)**: `@Composable AdMixerBanner(...)` 제공 — `AndroidView` 호스팅 + 생명주기/dispose 자동 처리. 코어에 Compose 의존성을 강제하지 않는 선택 모듈 (`admixer-compose:2.0.0`). [Compose 가이드](compose.md) 참고

### 개선 사항

- **어댑터 자동 등록**: `initialize()` 호출 시 Gradle 의존성에 포함된 어댑터를 자동 탐지·등록 — `registerAdapter()` 수동 호출 불필요
- **Mobwith 버전 업데이트**: mobwithSDK `1.0.2` → `1.0.68`
- **ProGuard 최적화**: ConfigMapper, AdStrategy 등 서버 응답 파싱 클래스 난독화 방지 규칙 강화. `NativeAdViewBinder$Builder` R8 난독화 버그 수정 (`consumer-rules.pro` `$**` wildcard 추가)
- **아키텍처 개선**: Delegate 패턴 기반 단일 책임 원칙(SRP) 적용으로 유지보수성 향상
- **생성자 주입 완성**: 모든 내부 Delegate 클래스가 Service Locator 대신 생성자 주입 사용
- **전면/리워드 비디오 경로 정규화**: `AXAdInterstitialVideoAd → InterstitialVideoAdActivity` 기반 Activity 경로로 통일 — back/close 정책 안정성 개선
- **네이티브 View ID prefix 추가**: `tv_title` 등 6개 View ID → `nap_mx_tv_title` 등으로 변경 — 타 라이브러리 리소스 충돌 방지 (마이그레이션 필요, [Step 7](migration-to-v2.md#step-7-view-id) 참고)
- **`setViewIds()` 제거 / `setAdapterConfig()` 추가**: 모든 어댑터(Adfit·Pangle·Mobwith 포함)가 `NativeAdViewBinder` 단일 경로로 통합. 어댑터별 String 초기화 파라미터(AppLovin `sdkKey` 등)는 `setAdapterConfig(adapterName, Map<String,String>)` 사용
- **AppLovin 12.x/13.x 초기화 API 호환성**: `AppLovinSdkInitializationConfiguration.builder()` 시그니처가 버전별로 다른 문제를 폴백 처리로 해결
- **전면 광고 BACK 키 기본 차단(동작 변경)**: `PopupInterstitialAdOption.setDisableBackKey` 기본값 `true` — 정적 전면도 비디오·리워드와 동일하게 뒤로가기 기본 차단. `false` 명시 시 종전 동작 ([마이그레이션 Step 8](migration-to-v2.md))
- **BACK 키 공통 제어 API 추가**: `AdInfo.Builder.setDisableBackKey(boolean)`(기본 `true`) — Basic 전면·비디오·리워드 전 풀스크린 타입 공통 적용. (Popup은 `PopupInterstitialAdOption`이 우선) ([전면](interstitial.md)·[동영상](video.md)·[리워드](rewarded-video.md) 가이드)
- **media-conf 재동기화 안정화**: 표시 중(SHOWING)/이미 로드된 풀스크린 유닛이 config 재동기화로 재로드되거나 MediationController가 중복 생성되던 문제 수정
- **Naver PUBLISHER_CD 관리 방식**: `com.naver.gfpsdk.PUBLISHER_CD`를 SDK(`admixer-naveradmanager`)가 제공·고정 — 호스트 앱 매니페스트 설정 불필요
- **노출(DISPLAYED) 이벤트 일관화**: 모든 포맷(배너·전면·네이티브·동영상)이 노출 시 `onEventAd(AdEvent.DISPLAYED)`를 일관되게 전달. 종전 네이티브 및 일부 NAP 전면배너 경로에서 `DISPLAYED` 콜백이 누락되던 문제 수정 (임프레션 집계에는 영향 없음)

### 버그 수정

- **BACK 키 차단이 Android 13+ 예측형 뒤로가기에서 무력화되던 문제 수정**: `targetSdk 35` 등 predictive back이 켜진 앱에서 `onKeyDown/onKeyUp`이 호출되지 않아 전면·비디오·리워드가 BACK으로 닫히던 현상 → `OnBackInvokedCallback` 등록으로 정상 차단
- 전면/리워드 광고 `onAdReceived`, `onEarnedReward` 콜백에서 `adInfo` null 체크 누락 수정
- Adfit 어댑터 `closeAdapter()` 시 `nativeAdLayout` null 처리로 메모리 누수 방지
- 노출/클릭 로그의 `nSkip` 파라미터가 항상 null로 전달되던 버그 수정
- NaverAd/Unity 어댑터 콜백 누락 및 리소스 해제 불완전 수정

### 주요 변경 (Breaking Changes)

v1→v2 메이저 전환에 맞춰 v1.x `@Deprecated` 별칭 API를 완전히 제거했습니다. 모두 동일 동작의 정식 메서드/상수로 대체되며, 교체 방법은 [마이그레이션 Step 5](migration-to-v2.md#step-5)를 참고하세요.

| 제거된 API | 정식 대체 |
|---|---|
| `InterstitialAd.onDestroy()`, `closeInterstitial()` | `stopInterstitial()` |
| `RewardInterstitialVideoAd.onDestroy()` | `stopRewardVideoAd()` |
| `InterstitialVideoAd.onDestroy()` | `stopInterstitialVideoAd()` |
| `AdView`/`NativeAdView`.`onDestroy()` | `destroy()` |
| `AdInfo.Builder.isUseBackgroundAlpha(Boolean)` | `setUseBackgroundAlpha(boolean)` |
| `AdMixer.GAUGE` / `AdMixer.TEXT` | 제거됨 (전면 CountDown 타입 삭제) |

> 위 표 외의 기존 Public API(`AdView`, `InterstitialAd`, `NativeAdView`, `VideoAdView`, `RewardInterstitialVideoAd`, `InterstitialVideoAd`, `AdListener`, `AdEvent`, `AdInfo`, `AdMixer`, `PopupInterstitialAdOption`)의 **정식 메서드는 변경되지 않았습니다**. 단, 네이티브 View ID 변경([Step 7](migration-to-v2.md))과 전면 BACK 키 기본값 변경([Step 8](migration-to-v2.md))은 별도 확인이 필요합니다.

---

## v1.0.21 (2026-02-20)

- 미디에이션 기능 업데이트
- 네트워크 버전 업데이트: Adfit `3.21.17`, Pangle `7.7.0.2`, Unity `4.15.0`
- 리워드 콜백 추가

---

## v1.0.20 (2026-02-20)

- 미디에이션 기능 업데이트
- 네트워크 버전 업데이트 (Adfit, Pangle, Unity Ads)
- 리워드 콜백 추가

---

## v1.0.19 (2026-01-21)

- 네트워크 SDK에서 리워드 획득 커스텀 파라미터 추가
- 리워드 획득 내부 로깅 URL 추가

---

## v1.0.18 (2026-01-07)

- 네트워크 버전 업데이트
- 리워드 이벤트 `EARNEDREWARD` 추가

---

## v1.0.16 (2025-10-30)

- 소재 사이즈 수정 기능 추가

---

## v1.0.15 (2025-10-16)

- Unity Ads 어댑터 추가

---

## v1.0.14 (2025-10-01)

- 미디에이션 처리 로직 수정

---

## v1.0.13 (2025-08-28)

- 전면 배너 옵션 추가

---

## v1.0.12 (2025-08-18)

- AppLovin 어댑터 추가
- Mobwith 버전 업데이트 (`1.0.2` → `1.0.3`)

---

## v1.0.11 (2025-08-11)

- 전면 배너 옵션 추가
- 버그 수정

---

## v1.0.10 (2025-07-24)

- Mobwith, Pangle 어댑터 추가
- 버그 수정

---

## v1.0.9 (2025-05-22)

- 버그 수정

---

## v1.0.8 (2025-04-14)

- Kakao Adfit 어댑터 추가
- 버그 수정

---

## v1.0.7 (2025-03-10) ~ v1.0.1 (2024-10-14)

- 버그 수정 및 안정성 개선

---

## v1.0.0 (2024-10-07)

- 최초 릴리즈
