# 에러 코드

`onFailedToReceiveAd` 콜백의 `errorCode` 파라미터로 전달되는 에러 코드 목록입니다.

---

## SDK 에러 코드

`onFailedToReceiveAd` 콜백의 `errorCode`는 `AdMixer` 클래스에 정의된 `int` 상수입니다.

| 상수명 | 값 | 설명 | 권장 조치 |
|--------|-----|------|----------|
| `AdMixer.AX_ERR_INIT` | `0x80000001` | SDK 초기화 오류 | `AdMixer.initialize()` 호출 여부 확인 |
| `AdMixer.AX_ERR_ADUNIT` | `0x80000002` | 유효하지 않은 AdUnit ID | AdUnit ID 및 Media Key 확인 |
| `AdMixer.AX_ERR_TIMEOUT` | `0x80000004` | 광고 로딩 타임아웃 | 타임아웃 설정 조정 또는 재요청 |
| `AdMixer.AX_ERR_NO_ADAPTER` | `0x80000005` | 어댑터 미등록 | `build.gradle`에 해당 어댑터 모듈 의존성 추가 여부 확인 |
| `AdMixer.AX_ERR_ADAPTER` | `0x80000006` | 어댑터 내부 오류 | 해당 네트워크 SDK 초기화 상태 확인 |
| `AdMixer.AX_ERR_CONFIG_FAIL` | `0x80000007` | 서버 Config 파싱 실패 | Media Key 및 네트워크 상태 확인 |
| `AdMixer.AX_ERR_NO_ADS` | `0x80000008` | 광고 없음 (재고 부족) | 일정 시간 후 재요청 |
| `AdMixer.AX_ERR_NO_FILL` | `0x80000009` | 노출 가능한 광고 없음 | 미디에이션 네트워크 설정 확인 |
| `AdMixer.AX_ERR_INVALID_REQUEST` | `0x8000000A` | 잘못된 요청 파라미터 | AdInfo 설정 값 확인 |
| `AdMixer.AX_ERR_NETWORK` | `0x8000000B` | 네트워크 오류 | 네트워크 연결 상태 확인 |

```java
// 에러 코드 비교 예시
@Override
public void onFailedToReceiveAd(@Nullable Object adView, @NonNull AdNetworkType networkType,
                                  int errorCode, @Nullable String errorMsg) {
    if (errorCode == AdMixer.AX_ERR_NO_ADS || errorCode == AdMixer.AX_ERR_NO_FILL) {
        // 광고 재고 없음 — 일정 시간 후 재요청
    } else if (errorCode == AdMixer.AX_ERR_NETWORK) {
        // 네트워크 오류 — 연결 상태 확인
    }
}
```

## 미디에이션 에러 코드

네트워크 어댑터에서 발생하는 에러의 경우, 해당 네트워크 SDK의 에러 코드가 그대로 전달될 수 있습니다.

> ℹ️ LogCat에서 `AdMixer` 태그로 필터링하면 상세 에러 로그를 확인할 수 있습니다.
> ```
> adb logcat -s AdMixer
> ```

---

## 문제 해결 가이드

### 광고가 전혀 노출되지 않는 경우

1. `AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE)` 설정 후 LogCat 확인
2. `AdMixer.initialize()` 호출 여부 및 올바른 Media Key 확인
3. 올바른 AdUnit ID 사용 여부 확인
4. 네트워크 연결 상태 확인
5. 파트너 사이트에서 해당 AdUnit 광고 설정 활성화 여부 확인

### 특정 네트워크 광고만 나오지 않는 경우

1. `build.gradle`에 해당 네트워크 어댑터 모듈 의존성 추가 여부 확인
2. Google AdManager: `AndroidManifest.xml`의 `APPLICATION_ID` 설정 확인
3. Pangle: `app_id`/`placement_id`가 media-conf 또는 `setAdapterConfig`로 전달되는지 확인 (SDK init은 어댑터가 자동 처리)
4. NaverAdManager: 운영 Ad Unit ID 발급 여부 확인 (`PUBLISHER_CD`는 SDK가 제공)
5. Adfit: Activity Context 사용 여부 확인

### 리워드가 지급되지 않는 경우

1. `onAdRewarded()` 콜백 수신 여부 LogCat으로 확인
2. `onAdRewarded()` 내 리워드 지급 로직 구현 여부 확인
3. S2S Callback URL 설정 여부 확인 (서버 기반 리워드의 경우)
