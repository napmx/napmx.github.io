# 릴리즈 노트

---

## v2.1.0 (2026-07-16) — 콜드스타트 로딩 최적화 · 미디에이션 응답 일관성 · 네이티브 렌더링 정합

> 광고 요청부터 노출까지의 로딩 지연을 단축하고, 어댑터 동작 편차와 무관하게 매체 앱에 일관된 응답을 보장하도록 코어를 강화했습니다. 네이티브 광고에는 **레이아웃 코드 수정이 필요한 변경 1건**(`setPrivacyViewId` 제거)과 **화면이 바뀔 수 있는 변경 2건**(메인 미디어 슬롯 크기, Pangle 광고 로고 위치)이 포함됩니다.

> ℹ️ 이번 릴리스는 변경된 모듈만 개별 버전으로 배포됩니다(모듈별 버전 상이). BOM(`admixer-bom:2026.07.02`)으로 버전을 묶어 연동하는 것을 권장합니다.
>
> | 아티팩트 | 버전 |
> |---|---|
> | `admixer-ssp` (코어) | **2.1.0** |
> | `admixer-adfit` | **2.0.2** |
> | `admixer-admanager` · `admixer-gma-nextgen` · `admixer-applovin` · `admixer-unity` · `admixer-naveradmanager` · `admixer-pangle` · `admixer-teads` · `admixer-compose` | **2.0.1** |
> | `admixer-unity-nativeadlayout` | 2.0.0 (변경 없음) |
> | `admixer-bom` | **2026.07.02** |

### ⚠️ 어댑터 식별자 **값** 변경 (Breaking — 재컴파일 필수)

`AdMixer.ADAPTER_*` 상수의 **문자열 값**이 바뀌었습니다. 상수 이름은 그대로라 **컴파일 오류가 나지 않습니다.**

| 상수 | 이전 값 | 새 값 |
|---|---|---|
| `AdMixer.ADAPTER_ADMANAGER` | `"AdManager"` | `"GoogleAdManager"` |
| `AdMixer.ADAPTER_ADFIT` | `"KaKao Adfit"` | `"AdFit"` |
| `AdMixer.ADAPTER_ADMIXER_HOUSE` | `"houseAd"` | `"HouseAd"` |

> 🚨 **반드시 재컴파일하세요.** Java는 `public static final String` 상수를 **컴파일 시점에 앱 바이너리로 복사(인라인)** 합니다. 이전 버전으로 빌드된 앱에는 옛 문자열이 그대로 박혀 있어, SDK만 올리고 재컴파일하지 않으면 **오류 없이 조용히 동작이 어긋납니다.**
>
> 영향받는 코드:
> - `adapterName.equals(AdMixer.ADAPTER_ADFIT)` 같은 **문자열 비교 분기** → 항상 `false`
> - `setAdapterConfig(AdMixer.ADAPTER_ADFIT, ...)` 같은 **어댑터 키 주입** → 매칭 실패로 설정 미반영
>
> **권장** — 문자열 비교 대신 아래의 `AdNetworkType` enum 오버로드로 전환하면 이런 문제가 구조적으로 사라집니다.

### 콜백 — 네트워크 식별을 `AdNetworkType` enum으로 (신규 · 기존 방식 Deprecated)

- **`AdNetworkType` enum 오버로드 추가** — `onReceivedAd` / `onFailedToReceiveAd` / `onAdShowFailed` 3종이 네트워크를 문자열(`String adapterName`) 대신 **타입 안전한 enum**(`AdNetworkType networkType`)으로 전달합니다. `switch(networkType) { case PANGLE: ... }`처럼 분기할 수 있어 오타로 인한 분기 누락이 사라집니다.
- **기존 `String adapterName` 오버로드는 `@Deprecated`** — 지금은 그대로 동작하며 **다음 메이저(3.0)에서 제거 예정**입니다. 신규 구현은 enum 버전을 사용하세요. 문자열이 필요하면 `networkType.getAdapterName()`으로 얻습니다.
  > ⚠️ 단, **전 네트워크 No-Ad 등 내부 실패**(어댑터가 아닌 SDK/미디에이션 레벨 실패)는 enum에 대응 값이 없어 **`String` 오버로드로만 통지**됩니다. 최종 No-Ad를 놓치지 않으려면 **두 오버로드를 함께 유지**하세요. ([API 레퍼런스](api-reference.md))

### 네이티브 — 광고 정보 고지(AdChoices) 위치 지정 (신규)

- **`NativeAdViewBinder.Builder.setAdChoicesPosition(AdChoicesPosition)` 추가** — 광고 정보 고지 아이콘을 4개 모서리(`LEFT_TOP` / `RIGHT_TOP` / `LEFT_BOTTOM` / `RIGHT_BOTTOM`) 중 원하는 곳에 노출합니다. 미지정 시 **우측 상단**(기존 동작과 동일). **모든 네이티브 네트워크**(AdMixer 자체·AdManager·GMA NextGen·NaverAd·Pangle·Adfit)에 동일하게 적용됩니다. ([네이티브 가이드](native-ad.md#광고-정보-고지adchoices-위치-지정))
- **레이아웃에 고지 슬롯이 더 이상 필요 없습니다** — SDK가 자동으로 오버레이합니다.
- **GMA NextGen 고지 아이콘 위치 명시** — 기존에는 네트워크 SDK 기본값에 의존했으나, 이제 우측 상단(또는 지정 모서리)으로 명시합니다.

### 네이티브 — 주요 변경 (Breaking Changes)

- **`NativeAdViewBinder.Builder.setPrivacyViewId(int)` 제거** — 호출 중이라면 컴파일 오류가 발생하므로 **`setAdChoicesPosition()`으로 교체**하세요. 레이아웃의 `nap_mx_privacy_container` 슬롯도 삭제할 수 있습니다(SDK가 자동 오버레이).
  > 임의 위치(예: 상단 중앙)를 지정하는 기능은 제공하지 않습니다. 네트워크 SDK마다 아이콘 소유권이 달라(일부는 SDK가 자기 뷰 계층에 직접 그림) 모서리 밖 위치는 네트워크 간 동작을 보장할 수 없기 때문입니다. 실제로 Google AdManager는 매체가 지정한 컨테이너를 무시하고 자체 오버레이를 그립니다. 워터폴은 어느 네트워크가 채울지 매체가 통제할 수 없으므로, 지면마다 위치가 달라지는 것을 막기 위해 4모서리로 일원화했습니다.

### 네이티브 — 렌더링 정합 (⚠️ 화면 변경 가능)

- **메인 미디어 슬롯이 선언한 크기를 그대로 지킵니다** — 이전에는 AdMixer 자체 광고에 한해 SDK가 소재 비율에 맞춰 슬롯보다 작게 렌더링했고, 그래서 같은 레이아웃이라도 워터폴 승자에 따라 높이가 달라졌습니다(예: 144×96 카드에 1200×628 소재 → 144×75로 축소, 카드 하단에 배경 노출). 이제 AdManager·Pangle·NaverAd·Adfit과 동일하게 슬롯을 채웁니다.
  - **영향**: 슬롯 비율 ≠ 소재 비율인 경우 여백(레터박스)의 **위치**가 하단 몰림 → 상·하 분산으로 바뀝니다(총량 동일). 슬롯 비율 = 소재 비율이면 변화 없음. `wrap_content` 슬롯도 변화 없음.
  - **권장**: 여백을 없애려면 슬롯 비율을 소재 비율(대부분 1.91:1)에 맞추세요. ([네이티브 가이드](native-ad.md))
- **Pangle 광고 로고 위치 — 좌측 상단 → 우측 상단** — 전 네트워크 기본값을 우측 상단으로 통일했습니다. 좌측 상단을 유지하려면 `setAdChoicesPosition(AdChoicesPosition.LEFT_TOP)`을 지정하세요.

### 성능 (콜드스타트 로딩 최적화)

- **콜드스타트 첫 광고 로딩 단축** — ① 마지막 성공 서버 config를 캐시로 즉시 웜업하여 첫 로드가 서버 응답을 기다리지 않고 미디에이션을 시작, ② config에 포함된 네트워크 SDK를 백그라운드에서 선(先)초기화(Pre-Init), ③ WebView 엔진 1회 선(先)워밍으로 첫 배너/전면의 렌더러 초기화 비용 제거, ④ 블로킹 HTTP를 전용 네트워크 스레드 풀로 분리해 저속망·멀티슬롯 큐 대기 해소, ⑤ config 첫 도착 디바운스 0ms.
- **광고 식별자(IFA) 영속화** — GMS 응답 전/실패 시에도 직전 IFA로 조기 요청이 가능하도록 보강.

### 미디에이션 응답 일관성

- **리워드 1회 지급 보장** — 네트워크마다 완료/획득 이벤트 발화 방식이 달라도, 매체 앱에는 리워드 콜백이 **정확히 1회만** 전달됩니다.
- **표시 실패의 일관 통지** — 전면/리워드가 로드 후 표시되지 못한 경우, 5초 내 결과가 없으면 표시 실패 콜백을 백스톱으로 전달합니다(무음 방지). 로드만 성공하고 표시하지 않으면 임프레션을 집계하지 않고, 실제 표시 시에만 통지·집계합니다.
- **단일 결정적 결과 보장** — 백스톱이 표시 실패를 통지한 뒤 뒤늦게 도착하는 어댑터의 상반된 콜백(중복 표시 실패, 표시 실패 후 닫힘/스킵/표시됨)은 억제되어, 매체 앱은 한 번의 광고당 **정확히 하나의 결정적 결과**(표시됨 또는 표시 실패)만 받습니다. 리워드 적립 콜백은 유실 방지를 위해 예외적으로 항상 전달됩니다. **호스트 앱 API·콜백 시그니처 변경 없음.**
- **콜백 스레드 일관성** — 백그라운드에서 콜백하는 네트워크와 무관하게 광고 이벤트는 항상 메인스레드로 전달됩니다.

### 임프레션 정확화 (⚠️ 자체 광고 지표 영향)

- **실제 노출 시점 발사** — 자체(AdMixer) 배너·전면의 임프레션 비콘을 '로드 완료'가 아닌 **'실제 화면 노출'** 시점(배너: 화면 뷰포트 50% 이상 노출 / 전면: Activity 실제 표시)에 발사하도록 글로벌 표준에 정렬했습니다.
- **영향**: 로드 후 미표시분·오프스크린 프리페치의 과다 집계가 제거되어 **자체 광고 임프레션 수치가 소폭 하락(정확화)**할 수 있습니다. **정산 로직/앱 이벤트(`onAdDisplayed`)는 변경 없음.** 네트워크 SDK가 집계하는 임프레션(GAM/AppLovin 등)은 해당 없음.

### 안정성

- 전면 광고 닫힘 후 Activity 누수, SDK 종료 시 메인스레드 블로킹(~수초) 제거, 취소된 요청의 커넥션·풀 점유 해소.
- 어댑터 개별 버그 수정: AdManager 배너 로드 NPE, GMA NextGen 초기화 고착, Pangle 네이티브 늦은 뷰 부착, Mobwith 전면 표시 안정성, Teads 늦은 응답 정리, Adfit·AppLovin 배너 일시정지/재개.

### 연동 요구사항 안내 (신규 — 확인 권장)

- **Kotlin 툴체인**: Google AdManager·Kakao Adfit·Naver Ad Manager 어댑터를 포함하는 앱은 이들 네트워크 SDK가 요구하는 최신 Kotlin 런타임에 맞춰 **호스트 앱을 Kotlin 2.0 이상(권장 2.1+)으로 빌드**하세요. 코어(`admixer-ssp`)만 사용하는 앱은 종전대로 Kotlin 1.8+/Java-only로 무방합니다. ([시작하기](getting-started.md))
- **네트워크 보안 설정**: 코어 SDK가 광고 생태계 호환을 위해 `networkSecurityConfig`를 앱 레벨로 선언합니다. 매체 앱이 **자체 `android:networkSecurityConfig`를 선언하는 경우** 매니페스트 병합 충돌을 피하려면 `<application>`에 `tools:replace="android:networkSecurityConfig"`를 지정하세요. ([시작하기](getting-started.md))

### 네트워크 SDK 버전 · 16KB 페이지 대응

- 번들 네트워크 SDK 버전은 현행 유지(변경 없음): AdManager `play-services-ads 25.2.0` · Adfit `3.21.17` · Pangle `8.0.0.5` · AppLovin `13.6.3` · Unity `4.18.1` · Naver `nam-bom 8.16.0` · Teads `6.1.0`.
- **Android 15 16KB 페이지 크기**: 네이티브 라이브러리를 포함하는 Pangle·AppLovin의 `.so`가 모두 16KB 정렬로 출하됨을 실측 확인했습니다(그 외 네트워크는 네이티브 라이브러리 미포함). **전 네트워크 16KB 안전.**
- (참고) Unity Ads는 벤더가 직접연동 방식의 지원을 축소하고 있어, 중기적으로 대체 연동 방식 전환을 검토 중입니다.

---

## v2.0.1 (2026-07-02)

> 네이티브 뷰바인더 연동을 뷰 경로로 정리하고, 변경 모듈만 독립 배포합니다.
> `admixer-ssp`/`admixer-adfit` 2.0.1, `admixer-gma-nextgen` 2.0.0(첫 출시), `admixer-bom` 2026.07.01(첫 출시).
> 나머지 어댑터는 2.0.0을 유지하며 코어 2.0.1과 호환됩니다. 상세: [Release Notes 2.0.1](../../RELEASE_NOTES_2.0.1.md)

### 새로운 기능

- **GMA NextGen 어댑터 첫 출시** — Google Mobile Ads NextGen SDK 연동 (`admixer-gma-nextgen`). classic `admixer-admanager`와 별도 모듈이며 통합 시 택1.
- **Adfit 네이티브 비즈보드(BizBoard) 지원** — Adfit 네이티브 경로에서 비즈보드 요청 처리 (`admixer-adfit`).
- **BOM(Bill of Materials) 첫 배포** — `admixer-bom`(POM-only). `platform('io.github.nasmedia-tech:admixer-bom:2026.07.02')`로 import 시 멤버 아티팩트 버전을 생략할 수 있습니다.

### 버그 수정

- **네이티브 뷰바인더 브릿지** — 뷰의 `AMMNativeAdView.setViewBinder(...)`로만 설정한 바인더가 어댑터까지 전달되지 않아 모든 네이티브 어댑터가 `"No value for nativeViewBinder"`로 실패하던 문제 수정. 로드 직전 뷰의 바인더를 `AdInfo`로 자동 브릿지(최초 로드·롤링 재로드 모두).

### 주요 변경 (Breaking Changes)

- **`AdInfo.Builder.setAdViewBinder(NativeAdViewBinder)` 제거** — 네이티브 바인더는 뷰 경로 `AMMNativeAdView.setViewBinder(...)`로 설정합니다(업계 표준: 바인딩은 뷰의 렌더링 관심사).

### 기타

- **어댑터-코어 버전 호환성 검사 제거** — 어댑터 `initAdapter`의 코어-어댑터 버전 강제 검사를 제거해, 변경된 모듈만 독립적으로 새 버전을 배포할 수 있습니다(어댑터-코어 lockstep 불요).

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
