# nap mx Android SDK 연동 가이드

> **코어 SDK 버전**: 2.1.2 (BOM `2026.07.05`)  
> **최소 지원 OS**: 코어 SDK 기준 Android 5.0 (API 21) 이상  
> **문의**: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

> ⚠️ **추가하는 미디에이션 어댑터에 따라 앱의 `minSdkVersion` 요구치가 올라갑니다.** 네트워크 SDK마다 요구 API 레벨이 달라, 코어 SDK의 API 21이 그대로 적용되지 않을 수 있습니다. 어댑터별 최소 요구 버전은 [SDK 시작하기](/android/native/getting-started)에서 확인하세요.

---


### 지원하는 광고 포맷

| 포맷 | 클래스 | 설명 |
|------|--------|------|
| 배너 광고 | `AMMBannerView` | 화면 상단/하단에 고정 표시되는 배너 |
| 전면 광고 | `AMMInterstitial` | 화면 전체를 덮는 전면 광고 |
| 네이티브 광고 | `AMMNativeAdView` | 앱 UI에 자연스럽게 통합되는 커스텀 형태 |
| 리워드 동영상 | `AMMRewardVideo` | 시청 완료 시 리워드를 지급하는 전면 동영상 |
| 인라인 동영상 | `AMMVideoView` | 화면 내 인라인으로 재생되는 동영상 |
| 전면 동영상 | `AMMVideoInterstitial` | 화면 전체를 덮는 전면 동영상 |

### 지원하는 미디에이션 네트워크

| 네트워크 | 비고 |
|---------|------|
| Google AdManager | |
| Google Mobile Ads NextGen | 🧪 beta — AdManager와 택1 |
| Naver Ad Manager | |
| Kakao Adfit | |
| Pangle (TikTok) | |
| AppLovin | |
| Unity Ads | |
| Teads | |

> ℹ️ 포맷별로 어떤 네트워크가 채워지는지는 애드유닛 설정과 각 네트워크의 재고·정책에 따라 달라집니다. 네트워크마다 지원 포맷과 동작이 다를 수 있으므로, 특정 네트워크의 동작을 전제로 화면을 설계하지 마세요.

---

## 빠른 시작

1. [SDK 시작하기](/android/native/getting-started) — Gradle 설정, 초기화, ProGuard
2. [배너 광고](/android/native/banner) — 첫 번째 광고 노출
3. [샘플 프로젝트](#샘플-프로젝트) — 완성된 예제 코드 확인

> 💡 **Jetpack Compose 앱**이라면 [Compose 연동 가이드](/android/native/compose)를 참고하세요. 생명주기 연결과 해제를 자동 처리하는 헬퍼 모듈(`admixer-compose`)을 제공합니다.

---

## 샘플 프로젝트

연동 시작 시 샘플 프로젝트를 먼저 적용하여 광고 응답을 확인하는 방식을 권장합니다.

* [Android Java SDK Sample](https://github.com/Nasmedia-Tech/AOS-AdMixerSSP-TestApp/tree/main/AdMixerSDKSample)
* [Android Kotlin SDK Sample](https://github.com/Nasmedia-Tech/AOS-AdMixerSSP-TestApp/tree/main/AdMixerSDKKotlinSample)

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

---

## Unity Plugin

Unity 엔진을 사용하는 경우 Unity Plugin 가이드를 참고하세요.

- [Android Unity 시작하기](/android/unity/getting-started)
