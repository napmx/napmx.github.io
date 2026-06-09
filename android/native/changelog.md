# 릴리즈 노트

---

## v2.0.0 (2026-05-12)

### 새로운 기능

- **광고 신고하기 기능 추가**: `AdInfo.Builder.showReportIcon(true)` 설정으로 광고 소재 위에 신고 아이콘(ⓘ) 표시. PixelCopy 기반 소재 자동 캡처 지원 (Android 8.0+)
- **NaverAdManager 어댑터 추가**: Naver Ad Manager (NAM) 미디에이션 지원 (`admixer-naveradmanager:2.0.0`)
- **Teads 어댑터 추가**: Teads 미디에이션 지원 (`admixer-teads:2.0.0`)
- **통합 개인정보 동의/테스트 API 추가**: `AdMixer.setGdprConsent/setCcpaDoNotSell/setUsPrivacy/setTestMode/setTestDeviceIds` — 워터폴에서 각 네트워크 privacy/test API로 자동 전파 (AppLovin/Unity/Pangle/AdManager)
- **`cancelLoad()` 추가**: 표시 중인 광고를 끊지 않고 진행 중 로드만 취소 (전면·리워드·전면 동영상)
- **풀스크린 정적 로드 API 추가**: 전면/리워드/전면 동영상에 정적 `load()` + `FullScreenContentCallback`(리워드는 `OnUserEarnedRewardListener`) 패턴 제공 — GAM(Google Mobile Ads) 스타일 노출/클릭/닫힘/보상 콜백. 기존 인스턴스 메서드도 유지
- **클라이언트 키 주입(`setAdapterConfig`)**: 서버 미제공 시 네트워크 키를 매체가 주입 (Server-Precedence 병합)
- **Jetpack Compose 지원(`admixer-compose`)**: `@Composable AdMixerBanner(...)` 제공 — `AndroidView` 호스팅 + 생명주기/dispose 자동 처리. 코어에 Compose 의존성을 강제하지 않는 선택 모듈 (`admixer-compose:2.0.0`). [Compose 가이드](compose.md) 참고

### 개선 사항

- **어댑터 자동 등록**: `initialize()` 호출 시 Gradle 의존성에 포함된 어댑터를 자동 탐지·등록 — `registerAdapter()` 수동 호출 불필요
- **Mobwith 버전 업데이트**: mobwithSDK `1.0.2` → `1.0.68`
- **Pangle 버전 업데이트**: pag-sdk `7.7.0.2` → `8.0.0.5`. GDPR 동의는 8.x에서 `setGDPRConsent` 제거에 따라 퍼블리셔 CMP의 TCF v2 동의문자열로 자동 처리됩니다(`AdMixer.setGdprConsent` 값은 Pangle로 전파되지 않음). CCPA는 기존대로 전파.
- **Google AdManager 버전 업데이트**: play-services-ads `24.8.0` → `25.2.0`. (`25.3.0`+는 Kotlin 메타데이터 호환 이슈로 광범위한 호스트 지원을 위해 `25.2.0`에 고정)
- **AppLovin 버전 업데이트**: applovin-sdk `13.5.0` → `13.6.3`.
- **Naver NAM 버전 업데이트**: nam-bom `8.14.0` → `8.16.0`.
- **Mobwith 버전 업데이트**: mobwithSDK `1.0.68` → `1.0.83`. ⚠️ 1.0.83이 `com.github.Dimezis:BlurView`를 전이 의존하므로 `settings.gradle` repositories에 `maven { url 'https://jitpack.io' }` 추가가 필요합니다([시작하기](getting-started.md) 참고).
- **ProGuard 최적화**: ConfigMapper, AdStrategy 등 서버 응답 파싱 클래스 난독화 방지 규칙 강화. `NativeAdViewBinder$Builder` R8 난독화 버그 수정 (`consumer-rules.pro` `$**` wildcard 추가)
- **아키텍처 개선**: Delegate 패턴 기반 단일 책임 원칙(SRP) 적용으로 유지보수성 향상
- **생성자 주입 완성**: 모든 내부 Delegate 클래스가 Service Locator 대신 생성자 주입 사용
- **전면/리워드 비디오 경로 정규화**: `AXAdInterstitialVideoAd → InterstitialVideoAdActivity` 기반 Activity 경로로 통일 — back/close 정책 안정성 개선
- **네이티브 View ID prefix 추가**: `tv_title` 등 6개 View ID → `nap_mx_tv_title` 등으로 변경 — 타 라이브러리 리소스 충돌 방지 (마이그레이션 필요, [Step 7](migration.md#step-7-view-id) 참고)
- **`setViewIds()` 제거 / `setAdapterConfig()` 추가**: 모든 어댑터(Adfit·Pangle·Mobwith 포함)가 `NativeAdViewBinder` 단일 경로로 통합. 어댑터별 String 초기화 파라미터(AppLovin `sdkKey` 등)는 `setAdapterConfig(adapterName, Map<String,String>)` 사용
- **AppLovin 12.x/13.x 초기화 API 호환성**: `AppLovinSdkInitializationConfiguration.builder()` 시그니처가 버전별로 다른 문제를 폴백 처리로 해결
- **전면 광고 BACK 키 기본 차단(동작 변경)**: `PopupInterstitialAdOption.setDisableBackKey` 기본값 `true` — 정적 전면도 비디오·리워드와 동일하게 뒤로가기 기본 차단. `false` 명시 시 종전 동작 ([마이그레이션 Step 8](migration.md))
- **BACK 키 공통 제어 API 추가**: `AdInfo.Builder.setDisableBackKey(boolean)`(기본 `true`) — Basic 전면·비디오·리워드 전 풀스크린 타입 공통 적용. (Popup/CountDown은 `PopupInterstitialAdOption`이 우선) ([전면](interstitial.md)·[동영상](video.md)·[리워드](rewarded-video.md) 가이드)
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

#### 광고 클래스명 변경

광고 포맷 클래스가 `AMM` prefix로 일괄 변경되었습니다. **구 클래스명은 완전히 제거**되어 미교체 시 컴파일 오류가 발생합니다. 클래스명만 교체하면 기존 인스턴스 메서드는 그대로 동작합니다.

| 기존 (v1.x) | v2.0.0 |
|---|---|
| `AdView` | `AMMBannerView` |
| `InterstitialAd` | `AMMInterstitial` |
| `NativeAdView` | `AMMNativeAdView` |
| `RewardInterstitialVideoAd` | `AMMRewardVideo` |
| `VideoAdView` | `AMMVideoView` |
| `InterstitialVideoAd` | `AMMVideoInterstitial` |

> 전면·리워드·전면 동영상은 정적 `load()` + `FullScreenContentCallback` 패턴(권장)이 추가되었습니다. 교체 방법은 [마이그레이션 Step 5](migration.md#step-5)를 참고하세요.

#### 제거된 별칭 API

v1→v2 메이저 전환에 맞춰 v1.x `@Deprecated` 별칭 API를 완전히 제거했습니다. 모두 동일 동작의 정식 메서드/상수로 대체되며, 교체 방법은 [마이그레이션 Step 5](migration.md#step-5)를 참고하세요.

| 제거된 API | 정식 대체 |
|---|---|
| `AMMInterstitial.onDestroy()`, `closeInterstitial()` | `stopInterstitial()` |
| `AMMRewardVideo.onDestroy()` | `stopRewardVideoAd()` |
| `AMMVideoInterstitial.onDestroy()` | `stopInterstitialVideoAd()` |
| `AMMBannerView`/`AMMNativeAdView`.`onDestroy()` | `destroy()` |
| `AdInfo.Builder.isUseBackgroundAlpha(Boolean)` | `setUseBackgroundAlpha(boolean)` |
| `AdMixer.GAUGE` / `AdMixer.TEXT` | `AdMixer.AX_COUNT_TYPE_GAUGE` / `AX_COUNT_TYPE_TEXT` |

> 광고 포맷 클래스(위 표)를 제외한 기존 Public API(`AdListener`, `AdEvent`, `AdInfo`, `AdMixer`, `NativeAdViewBinder`, `PopupInterstitialAdOption`)의 **정식 메서드는 변경되지 않았습니다**. 단, 광고 클래스명 변경, 네이티브 View ID 변경([Step 7](migration.md))과 전면 BACK 키 기본값 변경([Step 8](migration.md))은 별도 확인이 필요합니다.

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
