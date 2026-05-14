# 소개 및 지원 포맷

## nap mx란?

nap mx(Supply-Side Platform)는 kt nasmedia가 운영하는 광고 수익화 플랫폼입니다.  
퍼블리셔는 SDK, Script, API 세 가지 방식으로 연동할 수 있으며, 다수의 광고 네트워크를 단일 SDK로 통합 운영합니다.

파트너 사이트: [publisher.admixer.co.kr](https://publisher.admixer.co.kr/)  
문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 연동 방식

| 방식 | 설명 | 지원 네트워크 |
|------|------|--------------|
| **SDK** | Android/iOS 앱에 라이브러리 직접 연동 | NAP, Google AdManager, KakaoAdfit, Pangle, AppLovin, Unity Ads |
| **Script** | 웹뷰/모바일웹에 스크립트 삽입 | NAP, Google ADfit, MobWith |
| **API** | 서버-서버 직접 API 호출 | NAP, Criteo, MobWith |

> **참고**: Google 수익화(AdSense 등)를 함께 사용하는 경우 Script 연동을 권장합니다.

---

## 플랫폼 · 광고 포맷 지원 현황

### Android / iOS SDK

| 포맷 | 사이즈 | 주요 네트워크 |
|------|--------|--------------|
| 배너 | 320×50, 320×100, 300×250, 320×480 | NAP, Google, KakaoAdfit, Pangle, AppLovin |
| 전면 배너 | 320×480 | NAP, Google, Pangle, AppLovin |
| 네이티브 | 자유 | NAP, Google, KakaoAdfit, Pangle |
| 동영상 (아웃스트림) | 16:9 | NAP, Pangle, AppLovin |
| 리워드 동영상 | 전면 | NAP, Google, Pangle, AppLovin, Unity Ads |

### M.Web

| 포맷 | 사이즈 | 주요 네트워크 |
|------|--------|--------------|
| 배너 | 320×50 ~ 320×480 | NAP, ADfit, MobWith |
| 전면 배너 | 320×480 | NAP, MobWith |
| 네이티브 | 다양 | NAP, MobWith |
| 리워드 | 300×250, 320×480 | NAP |

### PC.Web

| 포맷 | 사이즈 |
|------|--------|
| 배너 | 120×600 등 |

---

## 지원 광고 네트워크

| 어댑터명 | 네트워크 | 플랫폼 |
|---------|---------|--------|
| `AdMixer` | nap mx 자체 | Android / iOS |
| `AdManager` | Google Ad Manager | Android / iOS |
| `KaKao Adfit` | 카카오 ADfit | Android / iOS |
| `MobWith` | 모비위드 | Android / iOS / Web |
| `AppLovin` | AppLovin MAX | Android / iOS |
| `Pangle` | ByteDance Pangle | Android / iOS |
| `UnityAds` | Unity Ads | Android / iOS (Unity) |
| `NaverAdManager` | 네이버 성과형DA | Android / iOS |
| `Teads` | Teads | Android / iOS |
