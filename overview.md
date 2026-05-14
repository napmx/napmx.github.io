# 소개 및 지원 포맷

## nap mx란?

nap mx(Supply-Side Platform)는 kt nasmedia가 운영하는 광고 수익화 플랫폼입니다.  
퍼블리셔는 SDK, Script, API 세 가지 방식으로 연동할 수 있으며, 다수의 광고 네트워크를 단일 SDK로 통합 운영합니다.

파트너 사이트: [publisher.admixer.co.kr](https://publisher.admixer.co.kr/)  
문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 연동 방식

| 방식 | 설명 | 지원 플랫폼 | 지원 네트워크 |
|------|------|------------|--------------|
| **SDK** | Android/iOS 앱에 라이브러리 직접 연동 | Android / iOS | NAP, Google AdManager, KakaoAdfit, Pangle, AppLovin, Unity Ads, Mobwith, NaverAdManager, Teads |
| **Script** | 웹뷰/모바일웹에 스크립트 삽입 | M.Web / PC.Web | NAP, Google ADfit, Mobwith |
| **API** | 서버-서버 직접 API 호출 | Server | NAP, Criteo, Mobwith |

> **참고**: nap 네트워크는 Adpacker, Criteo, Appier 등 연동된 디멘드 물량을 함께 제공합니다.

---

## 플랫폼 · 광고 포맷 지원 현황

### Android / iOS SDK

#### 배너

| 사이즈 | NAP | Google | KakaoAdfit | AppLovin | Pangle | Unity Ads |
|--------|:---:|:------:|:----------:|:--------:|:------:|:---------:|
| 320×50 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 320×100 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 300×250 | ✅ | ✅ | ✅ | — | ✅ | ✅ |
| 320×480 | ✅ | ✅ | ✅ | — | — | ✅ |

#### 전면 배너

| 사이즈 | NAP | Google | KakaoAdfit | AppLovin | Pangle | Unity Ads |
|--------|:---:|:------:|:----------:|:--------:|:------:|:---------:|
| 300×250 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 320×480 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

#### 네이티브

| NAP | Google | KakaoAdfit | Pangle |
|:---:|:------:|:----------:|:------:|
| ✅ | ✅ | ✅ | ✅ |

#### 동영상

| 포맷 | NAP | Google | AppLovin | Pangle | Unity Ads |
|------|:---:|:------:|:--------:|:------:|:---------:|
| 아웃스트림 (16:9) | ✅ | — | — | — | — |
| 인스트림 (16:9) | ✅ | — | — | — | — |
| 리워드 (전면) | ✅ | ✅ | ✅ | ✅ | ✅ |

---

### M.Web (Script)

#### 배너

| 사이즈 | NAP | Google | KakaoAdfit | Mobwith |
|--------|:---:|:------:|:----------:|:-------:|
| 100×100 | ✅ | ✅ | — | — |
| 300×250 | ✅ | ✅ | ✅ | ✅ |
| 320×50 | ✅ | ✅ | ✅ | ✅ |
| 320×100 | ✅ | ✅ | ✅ | ✅ |
| 320×480 | ✅ | ✅ | ✅ | ✅ |

#### 전면 배너 / 리워드 / 네이티브

| 포맷 | 사이즈 | NAP | Google | KakaoAdfit | Mobwith |
|------|--------|:---:|:------:|:----------:|:-------:|
| 전면 배너 | 300×250, 320×480 | ✅ | ✅ | ✅ | ✅ |
| 리워드 | 300×250, 320×480 | ✅ | ✅ | — | — |
| 네이티브 | 다양 | ✅ | ✅ | — | — |

---

### PC.Web (Script)

| 포맷 | 사이즈 | NAP |
|------|--------|:---:|
| 배너 | 120×600 등 | ✅ |

---

## 지원 광고 네트워크

| 네트워크 | 어댑터 | 플랫폼 |
|---------|--------|--------|
| nap mx (자체) | `AdMixer` | Android / iOS / Web |
| Google Ad Manager | `AdManager` | Android / iOS |
| 카카오 ADfit | `KakaoAdfit` | Android / iOS / M.Web |
| Mobwith | `MobWith` | Android / iOS / M.Web |
| AppLovin MAX | `AppLovin` | Android / iOS |
| ByteDance Pangle | `Pangle` | Android / iOS |
| Unity Ads | `UnityAds` | Android / iOS (Unity) |
| 네이버 성과형DA | `NaverAdManager` | Android / iOS |
| Teads | `Teads` | Android / iOS |
| Criteo | — | API |
