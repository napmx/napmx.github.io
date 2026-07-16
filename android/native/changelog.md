# 릴리즈 노트

---

## v2.1.0 (2026-07-16)

> 변경된 모듈만 개별 버전으로 배포됩니다(모듈별 버전 상이). BOM(`admixer-bom:2026.07.02`)으로 버전을 묶어 연동하는 것을 권장합니다.
>
> | 아티팩트 | 버전 |
> |---|---|
> | `admixer-ssp` (코어) | **2.1.0** |
> | `admixer-adfit` | **2.0.2** |
> | `admixer-admanager` · `admixer-gma-nextgen` · `admixer-applovin` · `admixer-unity` · `admixer-naveradmanager` · `admixer-pangle` · `admixer-teads` · `admixer-compose` | **2.0.1** |
> | `admixer-unity-nativeadlayout` | 2.0.0 (변경 없음) |
> | `admixer-bom` | **2026.07.02** |

### ⚠️ Breaking — 조치 필요

- **`NativeAdViewBinder.Builder.setPrivacyViewId(int)` 제거** — 호출 중이면 컴파일 오류입니다. `setAdChoicesPosition(AdChoicesPosition)`으로 교체하세요(4모서리, 기본 우측 상단). 레이아웃의 `nap_mx_privacy_container` 슬롯은 삭제 가능(SDK가 자동 오버레이). ([네이티브 가이드](native-ad.md#광고-정보-고지adchoices-위치-지정))
- **어댑터 식별자 값 변경 — 재컴파일 필수** — `AdMixer.ADAPTER_*` 상수의 **문자열 값**이 바뀌었습니다(`ADAPTER_ADMANAGER` `"AdManager"`→`"GoogleAdManager"`, `ADAPTER_ADFIT` `"KaKao Adfit"`→`"AdFit"`, `ADAPTER_ADMIXER_HOUSE` `"houseAd"`→`"HouseAd"`). 상수 이름은 그대로라 컴파일 오류가 없어, SDK만 올리고 재컴파일하지 않으면 문자열 비교·`setAdapterConfig` 매칭이 **조용히 어긋납니다.** 반드시 재컴파일하세요.

### ⚠️ 화면 변경 주의 (코드 변경 불요)

- **네이티브 메인 미디어 슬롯이 선언한 크기를 그대로 준수** — 자체 광고도 슬롯을 채웁니다(이전엔 소재 비율로 축소). 슬롯 비율 ≠ 소재 비율이면 여백 위치만 달라짐(총량 동일). 여백을 없애려면 슬롯 비율을 소재(대개 1.91:1)에 맞추세요.
- **Pangle 광고 로고 좌측 상단 → 우측 상단** — 전 네트워크 기본값 통일. 좌측 상단 유지는 `setAdChoicesPosition(AdChoicesPosition.LEFT_TOP)`.

---

## v2.0.1 (2026-07-02)

> 변경된 모듈만 개별 배포: `admixer-ssp`·`admixer-adfit` **2.0.1**, `admixer-gma-nextgen` **2.0.0**(첫 출시), `admixer-bom` **2026.07.01**(첫 출시). 나머지 어댑터는 2.0.0을 유지하며 코어 2.0.1과 호환됩니다. 상세: [Release Notes 2.0.1](../../RELEASE_NOTES_2.0.1.md)

### ⚠️ Breaking

- **`AdInfo.Builder.setAdViewBinder(NativeAdViewBinder)` 제거** — 네이티브 바인더는 뷰 경로 `AMMNativeAdView.setViewBinder(...)`로 설정하세요(업계 표준: 바인딩은 뷰의 렌더링 관심사).

---

## v2.0.0 (2026-06-17)

> v1 → v2 메이저 전환. 공개 광고 클래스가 iOS-AdMixer 정합의 `AMM*` 네이밍으로 통일되고,
> 풀스크린 광고가 정적 `loadAd()` + `FullScreenContentCallback` 구조로 바뀝니다.
> 자세한 교체 방법은 [v2 마이그레이션 가이드](migration.md)를 참고하세요.

### 새로운 기능

- **NaverAdManager 어댑터 추가** — Naver Ad Manager(NAM) 미디에이션 지원 (`admixer-naveradmanager`)
- **NAM Native Simple(템플릿형) 광고 지원** — NAM 통합형 네이티브 지면에서 Native Simple 응답을 별도 연동 코드 없이 자동 렌더링. 템플릿 렌더링 방식이므로 `NativeAdViewBinder` 자산 매핑은 적용되지 않습니다. ([네이티브 가이드](native-ad.md))
- **Teads 어댑터 추가** — Teads 미디에이션 지원 (`admixer-teads`)
- **Jetpack Compose 지원** — `@Composable AdMixerBanner(...)` 등 제공. 코어에 Compose 의존성을 강제하지 않는 선택 모듈 (`admixer-compose`). [Compose 가이드](compose.md)
- **풀스크린 광고 API (전면/리워드/전면 동영상)** — 정적 `AMMInterstitial.loadAd(...)` / `AMMRewardVideo.loadAd(...)` / `AMMVideoInterstitial.loadAd(...)`가 로드 완료 시 콜백으로 광고 객체를 반환(인스턴스 생성 불필요). 노출 이벤트는 `FullScreenContentCallback`(`onAdShowedFullScreenContent`/`onAdClicked`/`onAdDismissedFullScreenContent`/`onAdFailedToShowFullScreenContent`)로 수신. ([전면](interstitial.md)·[동영상](video.md)·[리워드](rewarded-video.md))
- **통합 개인정보 동의/테스트 API** — `AdMixer.setGdprConsent`/`setCcpaDoNotSell`/`setUsPrivacy`/`setTestMode`/`setTestDeviceIds`. 워터폴에서 각 네트워크로 자동 전파. [개인정보/테스트 설정](privacy.md)
- **어댑터 자동 등록** — Gradle 의존성에 포함된 어댑터를 자동 탐지·등록. `registerAdapter()` 수동 호출 불필요
- **`cancelLoad()`** — 표시 중인 광고를 끊지 않고 진행 중 로드만 취소 (전면·리워드·전면 동영상)
- **클라이언트 키 주입 `setAdapterConfig(adapterName, Map)`** — 서버 미제공 시 네트워크 키(예: AppLovin `sdkKey`)를 매체가 주입
- **인라인 광고 addView 시점 자동 노출** — 배너·네이티브·인라인 동영상은 뷰가 화면에 부착(`addView` 또는 XML 배치)되는 시점에 자동 노출됩니다. `showAd()` 직접 호출은 더 이상 필요 없습니다.
- **사용자 API 표준화** — 로드는 `loadAd()`(인라인 인스턴스·풀스크린 정적 공통), 해제는 `stop()`으로 통일했습니다.

### 주요 변경 (Breaking Changes)

> 교체 방법 상세는 [v2 마이그레이션 가이드](migration.md) 참고.

- **공개 광고 클래스 6종 → `AMM*` 네이밍 통일** (메서드 시그니처 동일, 클래스명만 변경):
  `AdView`→`AMMBannerView`, `NativeAdView`→`AMMNativeAdView`, `VideoAdView`→`AMMVideoView`, `InterstitialAd`→`AMMInterstitial`, `RewardInterstitialVideoAd`→`AMMRewardVideo`, `InterstitialVideoAd`→`AMMVideoInterstitial`. 레이아웃 XML의 클래스 경로도 변경해야 합니다.
- **`AdListener` 이벤트 콜백 분리** — 단일 `onEventAd(adView, AdEvent)` → 이름 있는 메서드(`onAdDisplayed`/`onAdClicked`/`onAdClosed`/`onAdCompleted`/`onAdSkipped`/`onAdRewarded` 등). `AdListener`는 `abstract class`로 전환되어 필요한 메서드만 override(필수 구현 없음). `onReceivedAd`/`onFailedToReceiveAd` 시그니처는 동일. ([Step 5-B](migration.md))
- **전면 광고 Basic 전용** — 전면은 Basic 타입만 제공합니다. 전면 타입/팝업/카운트다운 관련 `AdInfo.Builder` 옵션(`interstitialAdType`/`setInterstitialAdType`/`popupAdOption`/`setPopupAdOption`)과 관련 상수(`AdMixer.GAUGE`/`TEXT` 등)가 제거되었습니다. ([전면 가이드](interstitial.md))
- **네이티브 View ID prefix** — `tv_title` 등 6개 → `nap_mx_tv_title` 등으로 변경(타 라이브러리 리소스 충돌 방지). 레이아웃 및 `NativeAdViewBinder` 수정 필요. ([v2 마이그레이션 가이드](migration.md))
- **deprecated 별칭 API 제거** — `onDestroy()`/`closeInterstitial()`→`stopInterstitial()` 등 동일 동작의 정식 메서드로 교체. 배경 알파 옵션(`isUseBackgroundAlpha`/`setUseBackgroundAlpha`)은 무효화(전면 배경 디밍은 고정값 자동 적용). ([v2 마이그레이션 가이드](migration.md))
- **배너·네이티브 자동 갱신 옵션 정리** — 클라이언트 `isRetry`/`maxRetryCountInSlot` 제거. 자동 갱신/재로드는 서버 광고단위 `interval`(초) > 0일 때만 동작(무한 루프는 내부 가드 차단).
- **즉시 노출(start*) API 제거** — `startInterstitial()`/`startInterstitialVideoAd()`/`startRewardVideoAd()` 및 `loadAd(Activity, AdInfo)` 제거. 모든 광고는 로드(수신)와 노출(인라인=`addView`, 풀스크린=`show()`)이 분리됩니다.

### 네트워크 버전 업데이트

> 별도 명시한 경우를 제외하고 호스트 앱 API 변경은 없습니다.

- **Pangle `7.7.0.2` → `8.0.0.5`** — GDPR 동의가 퍼블리셔 CMP의 TCF v2 동의문자열로 자동 처리됨(`setGdprConsent` 값은 Pangle로 미전파, CCPA는 유지). 최소 GMA(play-services-ads) `25.1.0`+ 권장.
- **Google AdManager(play-services-ads) `24.8.0` → `25.2.0`** — (`25.3.0`+는 호환 이슈로 미채택)
- **AdManager 표준 배너 → anchored adaptive 전환** — iOS와 동일하게 표준 배너를 디바이스 너비 기반 anchored adaptive 배너로 요청(높이는 SDK가 산출). MREC(300x250)·320x480 고정 슬롯은 종전대로 유지. 호스트 앱 API 변경 없음(렌더 사이즈만 변동).
- **AppLovin `13.5.0` → `13.6.3`**
- **Naver NAM(nam-bom) `8.14.0` → `8.16.0`**

### 버그 수정 및 안정성

- **서버 interval 자동 갱신(롤링)이 동작하지 않던 문제 수정** — 노출 중(SHOWING) 재로드가 내부 가드에 차단되던 문제, 갱신 중 배너 영역이 화면 전체로 팽창하던 문제, 네이티브 갱신이 무음 중단되던 문제를 수정. 배너·네이티브가 서버 `interval`마다 정상 갱신됩니다.
- **전면 동영상이 인라인 경로로 잘못 로드되어 화면에 표시되지 않을 수 있던 문제 수정**.
- **네이티브 광고가 일부 네트워크/소재에서 표시되지 않던 문제 수정** — 일부 에셋이 비어 있거나 누락된 소재의 렌더링 안정성 개선.
- **media-conf 재동기화 시 풀스크린 광고 중복 로드/컨트롤러 중복 생성 문제 수정**.
- **Naver `PUBLISHER_CD`를 SDK가 제공·고정** — 호스트 앱 매니페스트 설정 불필요.
- 풀스크린(전면/리워드/동영상) 표시 안정성 및 리소스 해제 개선, 노출/클릭 로그 정확도 개선.

---

## v1.0.25 (2026-06-04)

- 전면배너 — 미디에이션 전체 성공/실패가 아닌 어댑터 개별 실패 콜백이 전달되던 이슈 수정
- 전면배너·전면 동영상·리워드 — `hasInterstitial` 초기화되도록 수정

---

## v1.0.24 (2026-05-12)

- 일반 배너 — 특정 소재 노출 이슈 수정
- `network_req_type` 3·4·5·6 추가

---

## v1.0.23 (2026-04-28)

- 전면배너 임프레션(imp) 중복 수정
- maven 배포 시 소스코드 제외하도록 수정

---

## v1.0.22 (2026-04-27)

- 전면배너 load & show 되지 않던 이슈 수정

---

## v1.0.21 (2026-03-12)

- 리워드 전면 BACK 키 비활성화 수정
- 배너 렌더링 클리핑 해제

---

## v1.0.20 (2026-02-19)

- 전략 URL 요청 주기 변경
- 네트워크 로그 파라미터 `uid` → `id` 변경

---

## v1.0.19 (2026-01-16)

- 타 네트워크 SDK 리워드 획득 커스텀 파라미터 추가
- 리워드 획득 내부 로깅 URL 추가
- Google AdManager 버전 업데이트

---

## v1.0.18 (2026-01-07)

- 타 네트워크 SDK 라이브러리 버전 최신 업데이트
- 리워드 획득 이벤트 추가

---

## v1.0.16 (2025-10-30)

- 특정 배너 소재 사이즈 관련 수정

---

## v1.0.15 (2025-10-30)

- Unity Ads 어댑터 배포

---

## v1.0.14 (2025-10-02)

- 미디에이션 처리 로직 수정 (SDK·네트워크 설정·네트워크 ID 매핑), targetSdk 35

---

## v1.0.13 (2025-08-29)

- 전면 옵션 기능 추가 및 고도화

---

## v1.0.11 (2025-08-13)

- 전면 옵션 기능 추가

---

## v1.0.10 (2025-08-13)

- 네트워크 파라미터 추가 및 AdManager·Adfit 클래스명 변경

---

## v1.0.9 (2025-05-22)

- 전면 동영상 메모리 누수 수정
- 요청 불필요 파라미터 제거 및 정리

---

## v1.0.8 (2025-04-14)

- 전면 `onEvent` 누락 수정
- Kakao Adfit 어댑터 추가
- 제휴사 네트워크 재요청 추가

---

## v1.0.5

- 네이티브 노출 추가

---

## v1.0.4

- 노출 네트워크 로그 이슈 수정

---

## v1.0.3

- 백그라운드 광고 요청 기능 추가 (어드민 설정 필수)

---

## v1.0.2

- 노출 네트워크 로그 이슈 수정 (실패한 네트워크를 모두 네트워크 로그 URL에 포함하도록 구현)

---

## v1.0.1

- GA SHA 암호화 방식을 AES로 변경, 패키지 구분자 `,` 처리

---

## v1.0.0 (2024-10-07)

- 최초 릴리즈
