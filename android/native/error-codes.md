# 에러 코드

실패 콜백(`onFailedToReceiveAd` / `onAdShowFailed`)의 `errorCode` 파라미터로 전달되는 에러 코드 목록입니다.

---

## SDK 에러 코드

`errorCode`는 `AdMixer` 클래스에 정의된 `int` 상수입니다. 2.5.0부터 iOS·WebBridge와 같은 **공통 코드**를 씁니다. **전달 경로** 열은 각 코드가 실제로 어느 콜백을 통해 앱에 도달하는지를 나타냅니다.

| 상수명 | 값 | 전달 경로 | 설명 | 권장 조치 |
|--------|-----|-----------|------|----------|
| `AdMixer.AX_ERR_ADUNIT` | `-1` | `onFailedToReceiveAd` | 잘못된 요청: AdInfo/AdUnit ID 누락, `destroy()` 한 전면 광고 인스턴스로 `loadAd()` | AdUnit ID 확인, 전면 광고는 새 인스턴스 생성 |
| `AdMixer.AX_ERR_INIT` | `-2` | `onFailedToReceiveAd` | 초기화 안 됨: Media Key 없이 로드 요청 | `AdMixer.getInstance().initialize()` 호출 여부와 Media Key 확인 |
| `AdMixer.AX_ERR_NOT_READY` | `-3` | `onAdShowFailed` | 로드가 끝나기 전에 `show()` 호출. 메시지 "Ad is not ready to show." | `onAdReceived` 이후 `show()` |
| `AdMixer.AX_ERR_PRESENTER_UNAVAILABLE` | `-4` | `onAdShowFailed`(인라인 비디오) | 광고를 표시할 Activity를 찾지 못함 | Activity Context로 뷰 생성 |
| `AdMixer.AX_ERR_NO_ADAPTER` | `-10` | `onFailedToReceiveAd` | 미디에이션 실행 컴포넌트를 준비하지 못함(예: SDK 초기화 전에 `setAdInfo` 한 뷰) | 초기화 이후 `setAdInfo`. 어댑터 모듈이 빌드에 없는 경우는 이 코드가 아니라 해당 네트워크가 건너뛰어져 최종 `AX_ERR_NO_ADS`로 이어집니다 |
| `AdMixer.AX_ERR_NO_ADS` | `-20` | `onFailedToReceiveAd` | 워터폴의 모든 네트워크가 실패/no-fill로 소진됨 (앱이 가장 자주 받는 코드). **no-fill은 전부 이 코드로 통지됩니다** | 일정 시간 후 재요청 |
| `AdMixer.AX_ERR_CONFIG_FAIL` | `-21` | `onFailedToReceiveAd` | 서버 config 미수신·타임아웃, 또는 config에 해당 AdUnit이 없음 | Media Key·AdUnit 프로비저닝 및 네트워크 상태 확인 |
| `AdMixer.AX_ERR_TIMEOUT` | `-22` | `onFailedToReceiveAd` | `loadAd()` 호출부터 30초 안에 결과가 없음 | 재요청 |
| `AdMixer.AX_ERR_SHOW_FAILED` | `-23` | `onAdShowFailed` | `show()` 이후의 모든 실패: 어댑터 show 실패·예외, 제한 시간 안에 노출/실패 신호 없음, 비디오 재생 실패. **네트워크 원인은 `errorMsg`에 남습니다** | `errorMsg` 로깅 후 재요청 |
| `AdMixer.AX_ERR_BACKGROUND` | `-24` | `onFailedToReceiveAd` | 앱이 백그라운드라 로드를 막음(media-conf `bg_req_yn=0`). 메시지 "Load blocked in background." | 포그라운드 복귀 후 재요청 |
| `AdMixer.AX_ERR_ADAPTER` | `-30` | 거의 없음 | SDK 내부 오류 | `errorMsg`와 LogCat 확인 |
| `AdMixer.AX_ERR_CANCELLED` | `-31` | `onFailedToReceiveAd` | 로드 중 `cancelLoad()` 호출. 배너·네이티브·인라인 비디오가 이전 광고를 표시하는 중(자동 갱신 로드 중)이면 광고가 그대로 남으므로 보내지 않습니다 | 필요 시 다시 `loadAd()` |

`AX_ERR_SUCCESS`(`0`)는 실패 콜백으로 오지 않으며 결과 코드 비교용입니다.

> 🚫 **`AX_ERR_NO_FILL`·`AX_ERR_INVALID_REQUEST`·`AX_ERR_NETWORK`로 분기하지 마세요.** SDK가 이 코드를 앱에 보내는 경로가 없습니다. no-fill은 **`AX_ERR_NO_ADS`**, 표시 단계의 네트워크 오류는 **`AX_ERR_SHOW_FAILED`** 로 옵니다. 세 상수는 어댑터 내부 분류용으로 남아 있고, 앱 `switch`에서 공통 코드와 겹치지 않도록 2.4.x 이하 값을 유지합니다. `AX_ERR_NO_FILL`은 다음 major에서 제거됩니다.

> ⚠️ **미디에이션이 개별 어댑터 실패를 흡수합니다.**
> 워터폴이 동작 중일 때 개별 네트워크의 로드 실패(네트워크 오류, 네트워크별 타임아웃, 잘못된 요청 등)는 다음 네트워크 시도로 승계되며 앱 리스너로 전달되지 않습니다. 모든 네트워크가 소진되면 **`AX_ERR_NO_ADS`("All adapters failed.")** 한 번만 전달됩니다.
> 따라서 특정 코드가 반드시 온다고 가정하지 말고, **`AX_ERR_NO_ADS`를 기본 분기로 두고 나머지는 `else`로 처리**하는 것을 권장합니다. 어느 네트워크에서 무엇이 실패했는지는 `errorMsg` 문자열과 LogCat 로그로 확인하세요.

## 2.5.0 마이그레이션

2.5.0에서 상수 **이름은 그대로**이고 **값만** 바뀌었습니다. `AX_ERR_*` 상수로 비교하는 코드는 **2.5.0으로 다시 빌드하면 그대로 동작**합니다. 상수는 컴파일 시 앱 바이너리에 값으로 들어가므로, 다시 빌드하지 않은 앱은 이전 값과 비교하게 됩니다. 숫자 값을 직접 쓰거나 서버·로그에 저장한 코드는 아래 표로 바꾸세요.

| 상수 | 2.4.x 이하 값 | 2.5.0 값 | 바뀐 점 |
|------|--------|--------|---------|
| `AX_ERR_ADUNIT` | `0x80000002` | `-1` | |
| `AX_ERR_INIT` | `0x80000001` | `-2` | |
| `AX_ERR_NO_ADAPTER` | `0x80000005` | `-10` | |
| `AX_ERR_NO_ADS` | `0x80000008` | `-20` | 준비 전 `show()`는 이제 `AX_ERR_NOT_READY` |
| `AX_ERR_CONFIG_FAIL` | `0x80000007` | `-21` | |
| `AX_ERR_TIMEOUT` | `0x80000004` | `-22` | 표시 가드 시간 초과(`onAdShowFailed`)는 이제 `AX_ERR_SHOW_FAILED` |
| `AX_ERR_BACKGROUND` | — | `-24` | 신설. 2.4.x 이하에서는 백그라운드 로드 차단 시 콜백이 없었습니다 |
| `AX_ERR_ADAPTER` | `0x80000006` | `-30` | 표시 실패는 이제 `AX_ERR_SHOW_FAILED`, 인라인 비디오 Activity 없음은 `AX_ERR_PRESENTER_UNAVAILABLE` |
| `AX_ERR_SUCCESS` | — | `0` | 신설 |
| `AX_ERR_NOT_READY` | — | `-3` | 신설 |
| `AX_ERR_PRESENTER_UNAVAILABLE` | — | `-4` | 신설 |
| `AX_ERR_SHOW_FAILED` | — | `-23` | 신설 |
| `AX_ERR_CANCELLED` | — | `-31` | 신설. 2.4.x 이하의 `cancelLoad()`는 콜백이 없었습니다 |
| `AX_ERR_NO_FILL`·`AX_ERR_INVALID_REQUEST`·`AX_ERR_NETWORK` | `0x80000009`·`0x8000000A`·`0x8000000B` | 같음 | 앱에 오지 않음 |

**`onAdShowFailed`는 원인 코드 대신 `AX_ERR_SHOW_FAILED`** 를 받습니다(준비 안 됨 `-3`, 화면 없음 `-4` 제외). 2.4.x 이하에서 `onAdShowFailed`의 `AX_ERR_ADAPTER`·`AX_ERR_NETWORK`·`AX_ERR_INVALID_REQUEST`·`AX_ERR_NO_ADS`·`AX_ERR_TIMEOUT`으로 분기했다면 `AX_ERR_SHOW_FAILED`로 합치고, 세부 원인은 `errorMsg`로 확인하세요.

**`com.nasmedia.admixerssp.common.error.ErrorCode` enum은 2.5.0에서 제거됐습니다.** SDK가 쓰지 않던 타입으로 값이 콜백 코드와 달랐습니다. 이 enum을 import했다면 `AdMixer.AX_ERR_*`로 바꾸세요.

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

네트워크 어댑터는 각 네트워크 SDK의 원본 에러 코드를 `AX_ERR_*` 값으로 **매핑해서** 미디에이션에 전달합니다(로드 실패는 흡수, 표시 실패는 앱에 `AX_ERR_SHOW_FAILED`). 네트워크 SDK의 원본 코드·메시지는 `errorCode`가 아니라 **`errorMsg` 문자열**에 담깁니다(예: `"Pangle placement_id is null"`, `"NAM banner error: [3210] ..."`). 매핑 규칙은 네트워크마다 다르며, 자세한 원본 코드 의미는 해당 네트워크 공식 문서를 참고하십시오.

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

키 주입 방법은 [Q&A — 초기화 · 키 / 네트워크 설정](qna.md#초기화-·-키-네트워크-설정)을 참고하세요.

### 리워드가 지급되지 않는 경우

1. `onAdRewarded()` 콜백 수신 여부 LogCat으로 확인
2. `onAdRewarded()` 내 리워드 지급 로직 구현 여부 확인
3. 서버 기반 리워드 검증을 사용한다면 콜백 URL 등록 여부 확인 — [리워드 동영상 — S2S Reward Callback](rewarded-video.md#s2s-reward-callback-서버-간-리워드-검증)
