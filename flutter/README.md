# Flutter SDK (Beta)

> 🧪 **초기 공개 Beta** · 플러그인 `v0.1.1` · Android/iOS 빌드 검증 완료

`nap_mx_flutter`는 Flutter 앱에서 nap mx Android·iOS 네이티브 SDK를 같은 Dart API로 사용하는 공개 플러그인입니다.

- 공개 저장소: [Nasmedia-Tech/nap_mx_flutter](https://github.com/Nasmedia-Tech/nap_mx_flutter)
- 검증된 릴리스: [`v0.1.1`](https://github.com/Nasmedia-Tech/nap_mx_flutter/releases/tag/v0.1.1)
- 실행 예제: [Sample 앱](https://github.com/Nasmedia-Tech/nap_mx_flutter/tree/v0.1.1/example)
- 상세 API 문서: [플러그인 저장소 가이드](https://github.com/Nasmedia-Tech/nap_mx_flutter/blob/v0.1.1/doc/integration-guide.md)

## 지원 포맷

`지원`은 공개 네이티브 SDK API와 Flutter 연결 코드가 구현되어 있다는 뜻입니다. `실기기 미검증`은 컴파일과 자동 테스트는 통과했지만, 발급된 지면으로 실제 광고 응답을 받는 검증이 아직 공개 결과에 포함되지 않았다는 뜻입니다.

| 광고 포맷 | Android | iOS | Flutter 구현 | 실제 광고 검증 |
|---|---|---|---|---|
| 배너 | 지원 | 지원 | `NapMxAdView` | 실기기 미검증 |
| 전면 | 지원 | 지원 | `NapMxFullscreenAdController` | 실기기 미검증 |
| 네이티브 | 지원 | 지원 | SDK 템플릿을 포함한 `NapMxAdView` | 실기기 미검증 |
| 인스트림(인라인) 동영상 | 지원 | 지원 | `NapMxAdView` | 실기기 미검증 |
| 아웃스트림(전면) 동영상 | 지원 | 지원 | `NapMxFullscreenAdController` | 실기기 미검증 |
| 리워드 동영상 | 지원 | 지원 | SDK 보상 콜백 + `transactionId` | 실기기 미검증 |

네이티브 광고의 제목·이미지·CTA를 Flutter 위젯으로 분해해 다시 조립하지 않습니다. SDK가 노출·클릭을 등록한 네이티브 View를 그대로 사용하므로 `광고/Ad` 표시, AdChoices, CTA와 미디어 영역이 유지됩니다.

## 미디에이션 네트워크

플러그인은 Core만 기본 설치합니다. 앱은 실제 사용하는 어댑터만 선택해야 하며, 특정 네트워크를 강제로 선택하는 Flutter API는 제공하지 않습니다.

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

포맷별 실제 응답 네트워크는 애드유닛의 서버 설정, 각 네트워크 심사·정책·재고에 따라 달라집니다. Android 지원 여부를 iOS 지원으로 간주하지 말고 각 플랫폼 표를 확인하세요.

## 검증된 버전과 최소 환경

| 항목 | 최소/고정 버전 |
|---|---|
| Flutter / Dart | Flutter 3.24+, Dart 3.5+ (CI: Flutter 3.44.9) |
| Android | API 21+, compile SDK 36, Java 17, Kotlin 2.3.20 |
| nap mx Android | `admixer-ssp:2.3.0`, BOM `2026.09.03` |
| iOS | iOS 13+, Xcode 16+, Swift 5.9 |
| nap mx iOS | `AdMixerMediation` 2.5.0, `AdMixer` 1.3.0 |

선택 어댑터에 따라 최소 OS가 올라갑니다. Android Google/Naver는 API 23, AppLovin/GMA NextGen은 API 24가 필요하며, iOS AdFit/Teads는 iOS 14가 필요합니다.

## 플러그인과 매체 앱의 책임

| 항목 | 플러그인 | 매체 앱/서버 |
|---|---|---|
| nap mx Core 초기화 | 수행 | 발급값과 호출 시점 결정 |
| 광고 load/show/해제 | 공통 API 제공 | 화면 수명주기와 UX 결정 |
| 개인정보 동의 UI/CMP | 자동 수행하지 않음 | 실제 사용자 선택 수집 |
| iOS ATT 요청 | 자동 수행하지 않음 | 요청 여부·문구·시점 결정 |
| 선택 어댑터/App ID | 강제 설치하지 않음 | 사용하는 것만 설정 |
| iOS 네트워크 초기화 | 자동 수행하지 않음 | 각 네트워크 가이드대로 수행 |
| 리워드 클라이언트 콜백 | `transactionId` 전달 | 한 번만 지급하도록 저장 |
| S2S 리워드 콜백 | 앱 SDK 범위 밖 | 콜백 URL 등록·서명/값 검증·멱등 처리 |

## 현재 제한사항

- `pub.dev`에는 아직 배포하지 않았습니다. 브랜치가 아닌 `v0.1.1` 태그로 설치하세요.
- 공개 범용 테스트 Media Key/AdUnit ID가 없습니다. 운영 담당자를 통해 별도 테스트 지면을 발급받아야 합니다.
- CI는 정적 분석, 단위 테스트, Android debug/release, iOS Simulator 빌드를 검증합니다. 실제 노출·클릭·보상 성공을 뜻하지 않습니다.
- Android `testMode`에 대응하는 iOS 전역 API가 없어 iOS에서 `true`를 전달하면 명시적으로 실패합니다.
- `mediation` 설정 맵은 Android 전용입니다. iOS 네트워크 초기화는 호스트 앱에서 수행합니다.

다음 단계: [설치와 초기화](/flutter/getting-started)
