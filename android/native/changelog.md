# 릴리즈 노트

---

## v2.0.0 (2026-05-12)

### 새로운 기능

- **광고 신고하기 기능 추가**: `AdInfo.Builder.showReportIcon(true)` 설정으로 신고 아이콘(ⓘ) 표시. PixelCopy 기반 소재 자동 캡처 지원 (Android 8.0+)
- **NaverAdManager 어댑터 추가**: Naver Ad Manager (NAM) 미디에이션 지원 (`admixer-naveradmanager:2.0.0`)
- **Teads 어댑터 추가**: Teads 미디에이션 지원 (`admixer-teads:2.0.0`)

### 개선 사항

- **어댑터 자동 등록**: `initialize()` 호출 시 Gradle 의존성에 포함된 어댑터를 자동 탐지·등록 — `registerAdapter()` 수동 호출 불필요
- **Mobwith 버전 업데이트**: mobwithSDK `1.0.2` → `1.0.68`
- **ProGuard 최적화**: `NativeAdViewBinder$Builder` R8 난독화 버그 수정, 서버 응답 파싱 클래스 난독화 방지 규칙 강화
- **아키텍처 개선**: Delegate 패턴 기반 SRP 적용, 생성자 주입 완성
- **전면/리워드 비디오 경로 정규화**: Activity 기반 경로로 통일 — back/close 정책 안정성 개선
- **네이티브 View ID prefix 추가**: `tv_title` 등 → `nap_mx_tv_title` 등으로 변경 — 리소스 충돌 방지 ([마이그레이션 Step 7](migration.md#step-7) 참고)
- **`setViewIds()` 제거 / `setAdapterConfig()` 추가**: 모든 어댑터가 `NativeAdViewBinder` 단일 경로로 통합. 어댑터별 String 파라미터(AppLovin `sdkKey` 등)는 `setAdapterConfig(adapterName, Map<String,String>)` 사용
- **AppLovin 12.x/13.x 초기화 API 호환성**: SDK 버전별 `builder()` 시그니처 차이를 폴백 처리로 해결

### 버그 수정

- 전면/리워드 광고 콜백에서 `adInfo` null 체크 누락 수정
- Adfit 어댑터 `closeAdapter()` 시 메모리 누수 방지
- 노출/클릭 로그 `nSkip` 파라미터 null 버그 수정
- NaverAd/Unity 어댑터 콜백 누락 및 리소스 해제 수정

### 하위 호환성

모든 Public API 변경 없음 — 기존 연동 코드 수정 불필요.

---

## v1.0.21 (2026-02-20)

- 미디에이션 기능 업데이트
- 네트워크 버전 업데이트: Adfit `3.21.17`, Pangle `7.7.0.2`, Unity `4.15.0`
- 리워드 콜백 추가

---

## v1.0.20 (2026-02-20)

- 미디에이션 기능 업데이트
- 네트워크 버전 업데이트 (Adfit, Pangle, Unity Ads)
- 리워드 콜백 추가

---

## v1.0.19 (2026-01-21)

- 네트워크 SDK에서 리워드 획득 커스텀 파라미터 추가

---

## v1.0.18 (2026-01-07)

- 네트워크 버전 업데이트
- 리워드 이벤트 `EARNEDREWARD` 추가

---

## v1.0.16 (2025-10-30)

- 소재 사이즈 수정 기능 추가

---

## v1.0.15 (2025-10-16)

- Unity Ads 어댑터 추가

---

## v1.0.14 (2025-10-01)

- 미디에이션 처리 로직 수정

---

## v1.0.13 (2025-08-28)

- 전면 배너 옵션 추가

---

## v1.0.12 (2025-08-18)

- AppLovin 어댑터 추가
- Mobwith 버전 업데이트

---

## v1.0.10 (2025-07-24)

- Mobwith, Pangle 어댑터 추가

---

## v1.0.8 (2025-04-14)

- Kakao Adfit 어댑터 추가

---

## v1.0.0 (2024-10-07)

- 최초 릴리즈
