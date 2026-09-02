# 릴리즈 노트


## v2.4.5 (2026-09-02)

- 하이브리드 앱(WebView) 연동 WebBridge 추가 — JavaScript 에서 전면·리워드·전면 동영상 광고 호출 (WebBridge 가이드 참고)
- 앱 추적 투명성(ATT) 상태 처리 정합성 개선
- 어댑터 버전 업데이트
   - AdMixerMediationGAM v1.2.6
   - AdMixerMediationAdFit v1.1.4
   - AdMixerMediationAppLovin v1.1.4
   - AdMixerMediationPangle v1.2.4
   - AdMixerMediationNAM v1.3.4

---

## v2.4.4 (2026-08-12)

- 네이티브 광고 클릭 동작 개선
- 전면 광고 노출 실패 통지 보강
- 개인정보 동의값 네트워크 전파
- 미디에이션 네트워크 선택 안정성 개선
- 라이센스 Apache-2.0 전환
- 어댑터 버전 업데이트
   - AdMixerMediationGAM v1.2.5
   - AdMixerMediationNAM v1.3.3
   - AdMixerMediationAdFit v1.1.3
   - AdMixerMediationAppLovin v1.1.3
   - AdMixerMediationPangle v1.2.3
   - AdMixerMediationUnityAds v1.1.3

---

## v2.4.3 (2026-07-29)

- 리워드 지급 트랜잭션 ID 추가 — 신규 콜백 `onRewardVideoEarned(rewardInfo:)` 로 지급 건별 고유 ID 수신, 지급 서버 로그·매체 콜백 URL 에 `transaction_id` 부착
   - 기존 `onRewardVideoEarned()` 는 deprecated (하위 호환 유지, 신규 콜백 구현 시 기존 콜백 미호출)
- 리워드 매체 콜백 재시도 추가 (전송 실패 시 최대 3회)
- 리워드 안정성 개선 (닫힘 직후 보상 이벤트 유실 방어 · 노출 실패 통지 보강)
- 네이티브 자동갱신 안정성 개선
- 어댑터 버전 업데이트
   - AdMixerMediationGAM v1.2.2
   - AdMixerMediationAdFit v1.1.2
   - AdMixerMediationAppLovin v1.1.2
   - AdMixerMediationPangle v1.2.2
   - AdMixerMediationUnityAds v1.1.2
   - AdMixerMediationNAM v1.3.2
   - AdMixerMediationGAM v1.2.3 (2026-08-07) — Google-Mobile-Ads-SDK 허용 범위 확대 (`12.7.0` 이상 ~ `13.8` 미만)
   - AdMixerMediationGAM v1.2.4 (2026-08-07) — 네이티브 미디어 뷰 이미지 비율 개선(매체 레이아웃 제약 충돌 해소), 전면 노출 실패 통지 보강, 개인정보 동의값 네트워크 전파

---

## v2.4.2 (2026-07-20)

- 안정성 개선 (메모리 누수 · 스레드 안전성 · 콜백 정합성)
- Teads — deprecated API 마이그레이션, TeadsSDK `6.2` 이상 요구
- 어댑터 버전 업데이트
   - AdMixerMediationGAM v1.2.1
   - AdMixerMediationAdFit v1.1.1
   - AdMixerMediationAppLovin v1.1.1
   - AdMixerMediationPangle v1.2.1
   - AdMixerMediationUnityAds v1.1.1
   - AdMixerMediationNAM v1.3.1
   - AdMixerMediationTeads v1.1.0 (TeadsSDK 6.2 이상)

---

## v2.4.1 (2026-07-15)

- 시뮬레이터에서 앱이 실행되지 않는 문제 수정 (v2.4.0)
- 어댑터 버전 업데이트
   - AdMixerMediationPangle v1.2.0 — 네이티브 광고 정보 아이콘이 미디어 영역이 아닌 광고 뷰 기준으로 표시

---

## v2.4.0 (2026-07-14)

- 광고 로드 API 개선 (`loadAd`)
- 안정성 개선
- 어댑터 버전 업데이트
   - AdMixerMediationGAM v1.2.0
   - AdMixerMediationNAM v1.3.0

---

## v2.3.7 (2026-06-22)

- 뷰형 광고(배너/네이티브/비디오) load API 추가
- AdFit 비즈보드 타입 지원
- 전면배너 표시 옵션(popup/countDown) 제거
- 버그 픽스
- 어댑터 버전 업데이트
   - AdMixerMediationGAM v1.1.0
   - AdMixerMediationAdFit v1.1.0
   - AdMixerMediationNAM v1.1.0
   - AdMixerMediationPangle v1.1.0
   - AdMixerMediationAppLovin v1.1.0
   - AdMixerMediationUnityAds v1.1.0
   - AdMixerMediationTeads v1.0.0 (신규)
   - AdMixerMediationNAM v1.1.1 (2026-07-02)
   - AdMixerMediationNAM v1.2.0 (2026-07-10)
- 코어 버전 업데이트
   - AdMixer v1.2.1 (2026-07-08)

---

## v2.3.6 (2026-06-02)

- 버그 픽스

---

## v2.3.5 (2026-05-22)

- 어댑터 버전 전송 기능 추가

---

## v2.3.4 (2026-05-15)

- 미디에이션 최적화 기능 추가
- 메모리 누수 코드 개선

---

## v2.3.3 (2026-04-30)

- Privacy Manifest 추가

---

## v2.3.2 (2026-04-20)

- 버그 픽스

---

## v2.3.1 (2026-04-16)

- 전면비디오 — load&show 기능 추가

---

## v2.3.0 (2026-04-14)

- interstitial, reward — load&show 기능 추가

---

## v2.2.1 (2026-03-23)

- native 지면 `loadAD()` 시 `removeView` 추가

---

## v2.2.0 (2026-03-17)

- 버그 픽스

---

## v2.1.9 (2026-03-11)

- 버그 픽스

---

## v2.1.8 (2026-02-20)

- 미디에이션 기능 업데이트

---

## v2.1.7 (2026-01-20)

- 배너 320x100 — 네트워크 추가

---

## v2.1.6 (2026-01-16)

- 매체 커스텀 파람 수정

---

## v2.1.5 (2026-01-16)

- 리워드 비디오 콜백 로직 수정

---

## v2.1.4 (2026-01-09)

- 네트워크 버전 업데이트
- 리워드 이벤트 추가 (`onRewardVideoEarned`)

---

## v2.1.3 (2025-12-22)

- 하우스애드 수정

---

## v2.1.2 (2025-12-09)

- `AMMBannerView` — adapter 전역으로 수정

---

## v2.1.1 (2025-12-04)

- objc 추가

---

## v2.1.0 (2025-09-09)

- `AMMBannerView` isLoading flag 추가
- `failProcess()`에서 `loadNetwork` async로 수정

---

## v2.0.9 (2025-08-27)

- 버그 픽스

---

## v2.0.8 (2025-08-27)

- 버그 픽스

---

## v2.0.7 (2025-08-26)

- Applovin 추가

---

## v2.0.7 (2025-08-22)

- `AMMVideoView` init 시 `rootViewController` 주입하도록 수정

---

## v2.0.6 (2025-08-22)

- 버그 픽스, 미디에이션 로직 개선

---

## v2.0.5 (2025-08-04)

- 전면 배너 닫기버튼 기능 추가
- 클릭 이벤트 Delegate 메서드 추가

---

## v2.0.4 (2025-07-25)

- Pangle 추가

---

## v2.0.4 (2025-07-08)

- Mobwith 추가, 버그 픽스

---

## v2.0.3 (2025-04-25)

- 이벤트 추가

---

## v2.0.2 (2025-04-14)

- Adfit 추가, 버그 픽스

---

## v2.0.1 (2025-03-24)

- 버그 픽스

---

## v2.0.0 (2025-02-28)

- 버그 픽스

---

## v1.0.1 (2024-12-27)

- 네이티브 에셋 옵션 변경

---

## v1.0.0 (2024-10-29)

- 최초 릴리즈
