# nap mx iOS SDK 연동 가이드

> **최소 지원 OS**: iOS 13.0 이상, Xcode 15.3 이상  
> **설치 방식**: CocoaPods, Swift Package Manager(SPM) 지원  
> **문의**: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

### 지원하는 광고 포맷

| 포맷 | 클래스 | 설명 |
|------|--------|------|
| 배너 광고 | `AMMBannerView` | 화면 상단/하단에 고정 표시되는 배너 |
| 전면 광고 | `AMMInterstitial` | 화면 전체를 덮는 전면 광고 |
| 네이티브 광고 | `AMMNativeAdViewContainer` | 앱 UI에 자연스럽게 통합되는 커스텀 형태 |
| 리워드 동영상 | `AMMRewardVideo` | 시청 완료 시 리워드를 지급하는 전면 동영상 |
| 인라인 동영상 | `AMMVideoAdView` | 화면 내 인라인으로 재생되는 동영상 |
| 전면 동영상 | `AMMVideoInterstitial` | 화면 전체를 덮는 전면 동영상 |

### 지원하는 미디에이션 네트워크

| 네트워크 |
|---------|
| Google AdManager |
| Kakao Adfit |
| Pangle (TikTok) |
| AppLovin |
| Unity Ads |
| Naver Ad Manager |
| Teads |

---

## 빠른 시작

1. [SDK 시작하기](/ios/native/getting-started) — CocoaPods/SPM 설치, 초기화
2. [배너 구현](/ios/native/banner) — 첫 번째 광고 노출
3. [샘플 프로젝트](#샘플-프로젝트) — 완성된 예제 코드 확인

---

## 샘플 프로젝트

연동 시작 시 샘플 프로젝트를 먼저 적용하여 광고 응답을 확인하는 방식을 권장합니다.

* [iOS SDK Sample](https://github.com/Nasmedia-Tech/iOS-AdMixerSSP-TestApp)

---

## 사전 요구사항

연동 작업 전에 아래 사항을 준비하세요.

> **📌 참고** **nap mx 파트너 사이트**에 가입 후 미디어 등록 및 애드유닛 생성을 완료해야 **media key**와 **adunit id**를 확인할 수 있습니다. media key와 adunit id가 파트너 사이트에 등록된 내용과 상이할 경우 광고가 원활히 노출되지 않을 수 있습니다.

| 항목 | 발급 방법 |
|------|----------|
| media key | nap mx 파트너 사이트에서 미디어 등록 후 발급 |
| adunit id | nap mx 파트너 사이트에서 애드유닛 생성 후 발급 |
| Google App ID | [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr) 문의 |
| Pangle App ID | [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr) 문의 |
| Unity Ads App ID | [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr) 문의 |

---

## Unity Plugin

Unity 엔진을 사용하는 경우 Unity Plugin 가이드를 참고하세요.

- [iOS Unity 시작하기](/ios/unity/getting-started)
