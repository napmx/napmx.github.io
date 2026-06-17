# Hybrid App (WebView) 비즈보드

KaKao Adfit 비즈보드 연동을 위한 가이드 문서입니다.  
문의: nap_mx@nasmedia.co.kr

> **비즈보드 지면 정책**
> - 비즈보드 지면은 비즈보드만 단독 사용 (타사 네트워크 등 미디에이션 불가)
> - 비즈보드 심사 과정 필수 (앱 내에 비즈보드 테스트 광고가 적용된 지면 스크린샷 전달이 필요합니다.)

---

## 0. 비즈보드 상품 설명

- 앱 사용자에게 최적화된 디자인으로 맞춤형 광고를 제공합니다.
- 철저한 심사를 거쳐 높은 퀄리티의 소재를 보장합니다.

---

## 1. SDK 연동 및 WEB View 설정 가이드

KaKao Adfit SDK 가이드를 참고하여 진행합니다.

- Android: https://github.com/adfit/adfit-android-sdk/blob/master/docs/WEBVIEWAD.md
- iOS: https://adfit.github.io/adfit-ios-sdk/documentation/adfitsdk/hybridad/

### 적용 프로세스

1. **앱에 AdFit SDK 적용** — 앱 단에서 AdFit SDK를 먼저 연동합니다. 이 단계는 "앱이 광고를 사용할 수 있는 상태"를 만드는 기본 준비 과정입니다.
2. **광고용 WebView 준비 및 SDK 등록** — 광고를 노출할 WebView를 생성하고 광고 동작에 필요한 설정(JavaScript·쿠키 등 필수 설정)을 적용합니다. 이후 해당 WebView를 AdFit SDK에 등록합니다.
3. **WebView에서 광고 포함 웹 페이지 로드** — AdFit 광고 스크립트가 포함된 웹 페이지를 WebView로 로드합니다.
4. **Web SDK를 통해 광고 요청 및 노출** — 웹 페이지 내 AdFit Web SDK가 실행되며, 광고를 요청하고 WebView 안에서 광고가 노출됩니다.

---

## 2. 비즈보드 코드 발급 및 리포트 매핑

### 코드 발급

운영팀에 문의하여 코드 발급 요청 부탁드립니다.

### 리포트 매핑

1. nap mx 파트너 사이트에서 애드유닛 이름 **'비즈보드'** 기입 후 리포트용 애드유닛 발급
   - 포맷/사이즈: **Banner - 320x50** 선택
2. 발급 후 운영팀에 문의하여 해당 애드유닛에 비즈보드 리포트 매핑 요청

---

## 3. 비즈보드 지면 심사 (중요)

비즈보드 지면은 지면 심사 과정이 필수입니다.

SDK 연동 테스트 과정에서 **비즈보드 테스트 광고가 노출된 실제 지면 캡처 이미지**를 운영팀에 전달해주세요.
