# Flutter SDK (Beta)

> **Beta** · 플러그인 `v0.1.8`

`nap_mx_flutter`는 Flutter 앱에서 nap mx Android·iOS 네이티브 SDK를 같은 Dart API로 사용하는 공개 플러그인입니다.

- 공개 저장소: [Nasmedia-Tech/nap_mx_flutter](https://github.com/Nasmedia-Tech/nap_mx_flutter)
- 최신 릴리스: [`v0.1.8`](https://github.com/Nasmedia-Tech/nap_mx_flutter/releases/tag/v0.1.8)
- 변경 내역: [릴리즈 노트](/flutter/changelog)
- 실행 예제: [Sample 앱](https://github.com/Nasmedia-Tech/nap_mx_flutter/tree/v0.1.8/example)
- 상세 API 문서: [플러그인 저장소 가이드](https://github.com/Nasmedia-Tech/nap_mx_flutter/blob/v0.1.8/doc/integration-guide.md)

## 지원 포맷

| 광고 포맷 | Android | iOS | Flutter API |
|---|---|---|---|
| 배너 | 지원 | 지원 | `NapMxAdView` |
| 전면 | 지원 | 지원 | `NapMxFullscreenAdController` |
| 네이티브 | 지원 | 지원 | `NapMxAdView` (SDK 네이티브 템플릿) |
| 인스트림(인라인) 동영상 | 지원 | 지원 | `NapMxAdView` |
| 아웃스트림(전면) 동영상 | 지원 | 지원 | `NapMxFullscreenAdController` |
| 리워드 동영상 | 지원 | 지원 | `NapMxFullscreenAdController` |

네이티브 광고는 SDK의 템플릿 뷰를 그대로 표시합니다. `광고/Ad` 표시, AdChoices, CTA 버튼, 미디어 영역이 SDK 규격대로 유지됩니다.

## 미디에이션 네트워크

플러그인에는 Core만 들어 있습니다. 어댑터는 사용하는 것만 앱에 추가하세요. 앱에서 네트워크를 지정하는 API는 없습니다.

| 네트워크 | Android 어댑터 | iOS 어댑터 | 비고 |
|---|---|---|---|
| nap mx 자체 광고 | Core 내장 | Core 내장 | 기본 |
| Google Ad Manager | 지원 | 지원 | Google App ID 필요 |
| Google Mobile Ads NextGen | Beta | 미지원 | Android API 24+, classic AdManager와 동시 사용 금지 |
| Naver Ad Manager | 지원 | 지원 | 플랫폼별 초기화 확인 |
| Kakao AdFit | 지원 | 지원 | iOS 14+ |
| Pangle | 지원 | 지원 | 별도 저장소/초기화 확인 |
| AppLovin | 지원 | 지원 | Android API 24+ |
| Unity Ads | 지원 | 지원 | 플랫폼별 App ID 필요 |
| Teads | 지원 | 지원 | iOS 14+, Xcode 26+ |

어느 네트워크의 광고가 나갈지는 nap mx 서버 설정과 각 네트워크의 정책·재고에 따라 정해집니다. 지원 여부는 플랫폼마다 다르니 위 표에서 확인하세요.

## 버전과 최소 환경

아래 표는 `v0.1.8` 기준입니다. 네이티브 SDK의 최신 버전은 [Android 릴리스 노트](/android/native/changelog)와 [iOS 릴리스 노트](/ios/native/changelog)에서 확인하세요.

| 항목 | 최소/고정 버전 |
|---|---|
| Flutter / Dart | Flutter 3.24+, Dart 3.5+ (CI: Flutter 3.44.9) |
| Android | API 21+, compile SDK 36, Java 17, AGP 8.10.1+, Kotlin 2.1.0+ (권장 2.2.10) |
| nap mx Android | `admixer-ssp:2.4.0`, BOM `2026.10.04` |
| iOS | iOS 13+, Xcode 16+, Swift 5.9 |
| nap mx iOS | `AdMixerMediation` 2.6.0, `AdMixer` 1.5.1 |

선택 어댑터에 따라 최소 OS가 올라갑니다. Android Google/Naver는 API 23, AppLovin/GMA NextGen은 API 24가 필요하며, iOS AdFit/Teads는 iOS 14가 필요합니다.

플러그인은 앱의 AGP·Kotlin 버전을 따릅니다. 설정 방법은 [Android 설정](/flutter/getting-started?id=_4-android-설정), 빌드를 확인한 버전 조합은 [자동 검증](/flutter/events-and-testing?id=자동-검증-명령)에 있습니다.

## 플러그인과 매체 앱의 책임

| 항목 | 플러그인 | 매체 앱/서버 |
|---|---|---|
| nap mx Core 초기화 | 수행 | 발급값과 호출 시점 결정 |
| 광고 load/show/해제 | API 제공 | 화면 수명주기와 UX 결정 |
| 개인정보 동의 UI/CMP | 제공하지 않음 | 사용자 선택 수집 |
| iOS ATT 요청 | 제공하지 않음 | 요청 여부·문구·시점 결정 |
| 선택 어댑터/App ID | 포함하지 않음 | 사용하는 것만 설정 |
| iOS 네트워크 초기화 | 제공하지 않음 | 각 네트워크 가이드대로 수행 |
| 리워드 클라이언트 콜백 | `transactionId` 전달 | 한 번만 지급하도록 저장 |
| S2S 리워드 콜백 | 관여하지 않음 | 콜백 URL 등록, 값 확인, 중복 지급 방지 |

## 현재 제한사항

- `pub.dev`에는 배포하지 않습니다. `v0.1.8` 태그로 설치하세요. [설치 예제](/flutter/getting-started?id=_2-플러그인-설치)를 참고하세요.
- 공용 테스트 Media Key/AdUnit ID는 없습니다. 운영 담당자에게 테스트 지면을 따로 발급받으세요.
- `testMode`는 Android 전용입니다. iOS에서 `true`를 전달하면 초기화가 실패합니다.
- `testDeviceIds`와 `usPrivacyString`도 Android 전용입니다. `v0.1.8`부터 iOS에서 비어 있지 않은 값을 전달하면 초기화가 실패하므로 플랫폼별로 나눠 설정하세요.
- `mediation` 설정 맵은 Android 전용입니다. iOS 네트워크 초기화는 호스트 앱에서 수행합니다.

다음 단계: [설치와 초기화](/flutter/getting-started)
