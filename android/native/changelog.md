# 릴리즈 노트

---

## v2.0.0 (미출시)

> v1 → v2 메이저 전환. 공개 광고 클래스가 GMA / iOS-AdMixer 정합의 `AMM*` 네이밍으로 통일되고,
> 풀스크린 광고가 GMA 스타일 정적 `load()` + `FullScreenContentCallback` 구조로 바뀝니다.
> 자세한 교체 방법은 [v2 마이그레이션 가이드](migration-to-v2.md)를 참고하세요.

### 새로운 기능

- **NaverAdManager 어댑터 추가** — Naver Ad Manager(NAM) 미디에이션 지원 (`admixer-naveradmanager`)
- **Teads 어댑터 추가** — Teads 미디에이션 지원 (`admixer-teads`)
- **Jetpack Compose 지원** — `@Composable AdMixerBanner(...)` 등 제공. 코어에 Compose 의존성을 강제하지 않는 선택 모듈 (`admixer-compose`). [Compose 가이드](compose.md)
- **GAM 스타일 풀스크린 광고 API (전면/리워드/전면 동영상)** — 정적 `AMMInterstitial.load(...)` / `AMMRewardVideo.load(...)` / `AMMVideoInterstitial.load(...)`가 로드 완료 시 콜백으로 광고 객체를 반환(인스턴스 생성 불필요). 노출 이벤트는 `FullScreenContentCallback`(`onAdShowedFullScreenContent`/`onAdClicked`/`onAdDismissedFullScreenContent`/`onAdFailedToShowFullScreenContent`)로 수신. ([전면](interstitial.md)·[동영상](video.md)·[리워드](rewarded-video.md))
- **통합 개인정보 동의/테스트 API** — `AdMixer.setGdprConsent`/`setCcpaDoNotSell`/`setUsPrivacy`/`setTestMode`/`setTestDeviceIds`. 워터폴에서 각 네트워크로 자동 전파. [개인정보/테스트 설정](privacy.md)
- **어댑터 자동 등록** — Gradle 의존성에 포함된 어댑터를 자동 탐지·등록. `registerAdapter()` 수동 호출 불필요
- **`cancelLoad()`** — 표시 중인 광고를 끊지 않고 진행 중 로드만 취소 (전면·리워드·전면 동영상)
- **클라이언트 키 주입 `setAdapterConfig(adapterName, Map)`** — 서버 미제공 시 네트워크 키(예: AppLovin `sdkKey`)를 매체가 주입

### 주요 변경 (Breaking Changes)

> 교체 방법 상세는 [v2 마이그레이션 가이드](migration-to-v2.md) 참고.

- **공개 광고 클래스 6종 → `AMM*` 네이밍 통일** (메서드 시그니처 동일, 클래스명만 변경):
  `AdView`→`AMMBannerView`, `NativeAdView`→`AMMNativeAdView`, `VideoAdView`→`AMMVideoView`, `InterstitialAd`→`AMMInterstitial`, `RewardInterstitialVideoAd`→`AMMRewardVideo`, `InterstitialVideoAd`→`AMMVideoInterstitial`. 레이아웃 XML의 클래스 경로도 변경해야 합니다.
- **`AdListener` 이벤트 콜백 분리** — 단일 `onEventAd(adView, AdEvent)` → 이름 있는 메서드(`onAdDisplayed`/`onAdClicked`/`onAdClosed`/`onAdCompleted`/`onAdSkipped`/`onAdRewarded` 등). `AdListener`는 `abstract class`로 전환되어 필요한 메서드만 override(필수 구현 없음). `onReceivedAd`/`onFailedToReceiveAd` 시그니처는 동일. ([Step 5-B](migration-to-v2.md))
- **전면 광고 Basic 전용** — 전면은 Basic 타입만 제공. `AdInfo.Builder`의 `interstitialAdType`/`setInterstitialAdType`/`popupAdOption`/`setPopupAdOption` 제거, **CountDown 타입 완전 제거**(`AdMixer.GAUGE`/`TEXT` 등 관련 상수 삭제). BACK 키 제어는 `setDisableBackKey` 사용. ([전면 가이드](interstitial.md))
- **전면 BACK 키 기본 차단** — `AdInfo.Builder.setDisableBackKey` 기본값 `true`(전면·비디오·리워드 전 풀스크린 공통). 종전처럼 BACK으로 닫으려면 `false` 명시. ([Step 8](migration-to-v2.md))
- **네이티브 View ID prefix** — `tv_title` 등 6개 → `nap_mx_tv_title` 등으로 변경(타 라이브러리 리소스 충돌 방지). 레이아웃 및 `NativeAdViewBinder` 수정 필요. ([Step 7](migration-to-v2.md#step-7-view-id))
- **deprecated 별칭 API 제거** — `onDestroy()`/`closeInterstitial()`→`stopInterstitial()`, `AdInfo.Builder.isUseBackgroundAlpha(Boolean)`→`setUseBackgroundAlpha(boolean)` 등. 동일 동작의 정식 메서드로 교체. ([Step 5](migration-to-v2.md#step-5))
- **배너·네이티브 자동 갱신 옵션 정리** — 클라이언트 `isRetry`/`maxRetryCountInSlot` 제거. 자동 갱신/재로드는 서버 광고단위 `interval`(초) > 0일 때만 동작(무한 루프는 내부 가드 차단).

### 네트워크 버전 업데이트

> 별도 명시한 경우를 제외하고 호스트 앱 API 변경은 없습니다.

- **Pangle `7.7.0.2` → `8.0.0.5`** — GDPR 동의가 퍼블리셔 CMP의 TCF v2 동의문자열로 자동 처리됨(`setGdprConsent` 값은 Pangle로 미전파, CCPA는 유지). 최소 GMA(play-services-ads) `25.1.0`+ 권장.
- **Google AdManager(play-services-ads) `24.8.0` → `25.2.0`** — (`25.3.0`+는 호환 이슈로 미채택)
- **AppLovin `13.5.0` → `13.6.3`**
- **Naver NAM(nam-bom) `8.14.0` → `8.16.0`**
- **Mobwith(mobwithSDK) `1.0.2` → `1.0.83`** — ⚠️ `settings.gradle`(또는 프로젝트 `build.gradle`) repositories에 `maven { url 'https://jitpack.io' }` 추가 필요(BlurView 전이 의존).

### 버그 수정 및 안정성

- **네이티브 광고가 일부 매체/소재에서 표시되지 않던 문제 수정** — Adfit(난독화 빌드 에셋 추출), Mobwith(메인 이미지 렌더), Pangle(광고 로고), Naver(고지 라벨), 일부 에셋이 비어 있거나 누락된 소재 처리 등.
- **BACK 키 차단이 Android 13+ 예측형 뒤로가기에서 무력화되던 문제 수정** — `targetSdk 35` 등 predictive back 환경(전면·비디오·리워드).
- **media-conf 재동기화 시 풀스크린 광고 중복 로드/컨트롤러 중복 생성 문제 수정**.
- **Naver `PUBLISHER_CD`를 SDK가 제공·고정** — 호스트 앱 매니페스트 설정 불필요.
- 풀스크린(전면/리워드/동영상) 표시 안정성 및 리소스 해제 개선, 노출/클릭 로그 정확도 개선.

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
