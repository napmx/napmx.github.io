# 에러 코드

실패 콜백(`onFailedToReceiveAd` / `onAdShowFailed`)의 `errorCode` 파라미터로 전달되는 에러 코드 목록입니다.

---

## SDK 에러 코드

`errorCode`는 `AdMixer` 클래스에 정의된 `int` 상수입니다. **전달 경로** 열은 각 코드가 실제로 어느 콜백을 통해 앱에 도달하는지를 나타냅니다.

| 상수명 | 값 | 전달 경로 | 설명 | 권장 조치 |
|--------|-----|-----------|------|----------|
| `AdMixer.AX_ERR_INIT` | `0x80000001` | `onFailedToReceiveAd` | SDK가 초기화되지 않은 상태에서 로드를 요청 | `AdMixer.getInstance().initialize()` 호출 여부 확인 |
| `AdMixer.AX_ERR_ADUNIT` | `0x80000002` | `onFailedToReceiveAd` | AdInfo/AdUnit ID 누락, 또는 전면 광고에 Activity Context가 아닌 Context 전달 | AdUnit ID·Media Key 및 전달한 Context 확인 |
| `AdMixer.AX_ERR_TIMEOUT` | `0x80000004` | `onAdShowFailed` | `show()` 이후 노출/실패 신호가 제한 시간 내에 오지 않음 | 재요청. 로드 단계 타임아웃은 이 코드가 아닌 `AX_ERR_NETWORK`입니다 |
| `AdMixer.AX_ERR_NO_ADAPTER` | `0x80000005` | `onFailedToReceiveAd` | 미디에이션 실행 컴포넌트를 준비하지 못했거나, 선택된 어댑터가 요청 포맷을 지원하지 않음 | `errorMsg` 확인. 어댑터 모듈이 빌드에 없는 경우는 이 코드가 아니라 해당 네트워크가 워터폴에서 건너뛰어져 최종 `AX_ERR_NO_ADS`로 이어집니다 |
| `AdMixer.AX_ERR_ADAPTER` | `0x80000006` | `onFailedToReceiveAd` / `onAdShowFailed` | 어댑터 내부 오류(필수 키 누락, 지원하지 않는 포맷, 표시 실패 등) | `errorMsg`의 상세 문구와 해당 네트워크 SDK 상태 확인 |
| `AdMixer.AX_ERR_CONFIG_FAIL` | `0x80000007` | `onFailedToReceiveAd` | 서버 config 미수신·타임아웃, 또는 config에 해당 AdUnit이 없음 | Media Key·AdUnit 프로비저닝 및 네트워크 상태 확인 |
| `AdMixer.AX_ERR_NO_ADS` | `0x80000008` | `onFailedToReceiveAd` / `onAdShowFailed` | 워터폴의 모든 네트워크가 실패/no-fill로 소진됨 (앱이 가장 자주 받는 코드). **no-fill은 전부 이 코드로 통지됩니다.** 준비되지 않은 상태에서 `show()`를 호출한 경우에도 같은 코드가 `onAdShowFailed`(`onAdFailedToShowFullScreenContent`)로 전달됩니다 | 일정 시간 후 재요청 |
| `AdMixer.AX_ERR_INVALID_REQUEST` | `0x8000000A` | 주로 어댑터 단계 | 잘못된 요청 파라미터(예: 형식이 맞지 않는 placement 값) | AdInfo 및 서버 키 설정 값 확인 |
| `AdMixer.AX_ERR_NETWORK` | `0x8000000B` | 주로 어댑터 단계 | 네트워크 오류 또는 어댑터 로드 타임아웃 | 네트워크 연결 상태 확인 |

> 🚫 **`AdMixer.AX_ERR_NO_FILL`(`0x80000009`)로 분기하지 마세요.** 상수는 존재하지만 **SDK가 이 코드를 전달하는 경로가 없습니다.** 모든 네트워크의 no-fill(재고 없음)은 위 표의 **`AX_ERR_NO_ADS`(`0x80000008`)** 로 통지됩니다. `AX_ERR_NO_FILL`을 조건으로 쓰면 **절대 실행되지 않는 분기**가 되어 no-fill 처리가 통째로 누락됩니다. 차기 메이저 버전에서 상수 자체가 삭제될 예정이므로 신규 코드에서는 사용하지 마세요.

> ⚠️ **미디에이션이 개별 어댑터 실패를 흡수합니다.**
> 워터폴이 동작 중일 때 개별 네트워크의 실패(`AX_ERR_NETWORK`, `AX_ERR_INVALID_REQUEST`, 어댑터 단계 `AX_ERR_ADAPTER` 등)는 다음 네트워크 시도로 승계되며, 앱 리스너로 직접 전달되지 않는 것이 일반적입니다. 모든 네트워크가 소진되면 **`AX_ERR_NO_ADS`("All adapters failed.")** 한 번만 전달됩니다.
> 따라서 특정 코드가 반드시 온다고 가정하지 말고, **`AX_ERR_NO_ADS`를 기본 분기로 두고 나머지는 `else`로 처리**하는 것을 권장합니다. 어느 네트워크에서 무엇이 실패했는지는 `errorMsg` 문자열과 LogCat 로그로 확인하세요.

```java
// 에러 코드 비교 예시
@Override
public void onFailedToReceiveAd(int errorCode, @Nullable String errorMsg) {
    if (errorCode == AdMixer.AX_ERR_NO_ADS) {
        // 워터폴 소진(재고 없음) — 일정 시간 후 재요청
    } else if (errorCode == AdMixer.AX_ERR_CONFIG_FAIL) {
        // 서버 설정 미수신 — Media Key / AdUnit 프로비저닝 확인
    } else {
        // 그 외: errorMsg 로깅 후 재요청 정책에 따라 처리
    }
}
```

## 미디에이션 에러 코드

네트워크 어댑터는 각 네트워크 SDK의 원본 에러 코드를 위 표의 `AX_ERR_*` 값으로 **매핑해서** 전달합니다. 네트워크 SDK의 원본 코드·메시지는 `errorCode`가 아니라 **`errorMsg` 문자열**에 담깁니다(예: `"Pangle placement_id is null"`, `"NAM banner error: [3210] ..."`). 매핑 규칙은 네트워크마다 다르며, 자세한 원본 코드 의미는 해당 네트워크 공식 문서를 참고하십시오.

> ℹ️ LogCat 태그는 `AdMixerSDK::<버전>` 형태입니다(하위 태그가 붙으면 `AdMixerSDK::<버전>::<컴포넌트>`). 접두사로 필터링하세요.
> ```
> adb logcat | grep AdMixerSDK
> ```
> `adb logcat -s <tag>`는 태그 완전 일치 필터라 버전이 붙는 이 태그에는 사용할 수 없습니다.

---

## 문제 해결 가이드

### 광고가 전혀 노출되지 않는 경우

1. `AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE)` 설정 후 LogCat 확인
2. `AdMixer.getInstance().initialize(...)` 호출 여부 및 올바른 Media Key 확인
3. 올바른 AdUnit ID 사용 여부 확인
4. 네트워크 연결 상태 확인
5. 파트너 사이트에서 해당 AdUnit 광고 설정 활성화 여부 확인

### 특정 네트워크 광고만 나오지 않는 경우

LogCat에 `[SKIP] <네트워크> configuration is invalid (Missing Keys).`가 보이면, 해당 네트워크의 **필수 키가 없어 워터폴에서 건너뛴** 것입니다. 아래를 확인하세요.

1. `build.gradle`에 해당 네트워크 어댑터 모듈 의존성 추가 여부 확인 — 모듈이 없으면 LogCat에 `[SKIP] Adapter instantiation failed for: <네트워크>`가 남고 해당 네트워크는 건너뜁니다
2. Google AdManager: `AndroidManifest.xml`의 `com.google.android.gms.ads.APPLICATION_ID` 설정 확인
3. Pangle: `placement_id`(필수)와 `app_id`가 media-conf 또는 `setAdapterConfig`로 전달되는지 확인. Pangle SDK가 아직 초기화되지 않았다면 어댑터가 `app_id`로 초기화를 수행하므로 `app_id`도 필요합니다
4. AppLovin: `zone_id`(필수)가 전달되는지 확인
5. NaverAdManager: 운영 Ad Unit ID 발급 여부 확인 (`PUBLISHER_CD`는 SDK가 제공)
6. Adfit: Activity Context 사용 여부 확인

키 주입 방법은 [Q&A — 초기화 · 키 / 네트워크 설정](qna.md#초기화--키--네트워크-설정)을 참고하세요.

### 리워드가 지급되지 않는 경우

1. `onAdRewarded()` 콜백 수신 여부 LogCat으로 확인
2. `onAdRewarded()` 내 리워드 지급 로직 구현 여부 확인
3. 서버 기반 리워드 검증을 사용한다면 콜백 URL 등록 여부 확인 — [리워드 동영상 — S2S Reward Callback](rewarded-video.md#s2s-reward-callback-서버-간-리워드-검증)
