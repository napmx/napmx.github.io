# nap mx 공식 연동 가이드

nap mx에서 지원하는 연동 방식 및 광고 포맷에 대한 공식 가이드 문서입니다.

문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 연동 방식

| 연동 방식 | 지원 네트워크 |
|-----------|--------------|
| **SDK** | NAP, Google AdManager, Naver Ad Manager, KakaoAdfit, Pangle, AppLovin, Unity Ads, Teads |
| **Script** | NAP, Google ADfit |
| **API** | NAP, Criteo |

> NAP 네트워크는 nap mx에 연동된 디맨드(Adpacker, Criteo, Appier 등) 물량을 제공합니다.

---

## 플랫폼 · 광고 포맷 · 지원 네트워크

### Android SDK

| 광고 포맷 | 어댑터가 제공되는 네트워크 |
|-----------|--------------|
| 배너 (일반) | NAP, Google, Naver, KakaoAdfit, Pangle, AppLovin, Unity Ads |
| 배너 (전면) | NAP, Google, Naver, KakaoAdfit, Pangle, AppLovin, Unity Ads |
| 네이티브 | NAP, Google, Naver, KakaoAdfit, Pangle, AppLovin |
| 동영상 (아웃스트림) | NAP, Teads |
| 동영상 (인스트림) | NAP |
| 리워드 동영상 | NAP, Google, Naver, Pangle, AppLovin, Unity Ads |

> ⚠️ 위 표는 **SDK가 어댑터를 제공하는 범위**입니다. 실제로 어떤 네트워크가 광고를 채우는지는 애드유닛별 서버 설정과 각 네트워크의 재고·정책에 따라 달라집니다.
> 포맷 지원 여부와 세부 동작은 **네트워크 SDK 버전에 따라 변경될 수 있으므로**, 도입 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 지면 구성을 문의하시길 권장합니다.
> 실제 연동 시 추가하는 어댑터 의존성은 [Android SDK 시작하기](/android/native/getting-started)에서 확인할 수 있습니다.

### iOS SDK

지원 포맷·네트워크 구성은 Android와 다를 수 있습니다. [iOS SDK 시작하기](/ios/native/getting-started)를 참고하시거나 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의해 주세요.

### Flutter SDK (Beta)

Android와 iOS 네이티브 SDK를 하나의 타입 기반 Dart API로 연결합니다. 배너·전면·네이티브·인스트림/아웃스트림 동영상·리워드를 지원하며, 현재는 `pub.dev`가 아닌 검증된 Git 태그로 설치합니다.

> Flutter 지원은 초기 공개 Beta입니다. 플러그인·Sample 앱 빌드는 검증했지만, 실제 광고 응답은 발급받은 테스트 지면과 Android/iOS 실기기에서 별도로 확인해야 합니다. [Flutter SDK 지원 범위](/flutter/)에서 미지원·미검증 항목을 확인하세요.

### M.Web

| 광고 포맷 | 지원 네트워크 |
|-----------|--------------|
| 배너 (일반/전면) | NAP, Google, ADfit |
| 네이티브 (일반/전면) | NAP |
| 리워드 | NAP |

### PC.Web

| 광고 포맷 | 사이즈 |
|-----------|--------|
| 배너 (일반) | 120x600 등 |

---

## 빠른 시작

- [Android SDK 시작하기](/android/native/getting-started)
- [iOS SDK 시작하기](/ios/native/getting-started)
- [Flutter SDK (Beta) 시작하기](/flutter/getting-started)
- [Android Unity 시작하기](/android/unity/getting-started)
- [iOS Unity 시작하기](/ios/unity/getting-started)
