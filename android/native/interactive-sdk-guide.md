# AdMixer Interactive SDK 매체 앱 연동 가이드 (실험실)

AdMixer Interactive SDK는 30종 미니게임을 매체 Android 앱 안에서 실행하고, 게임 결과와
예상 보상, 티켓 및 리워드 광고 결과를 매체 앱에 전달하는 선택형 모듈입니다.

> 🧪 **실험실(Labs) · Beta 테스트 제공**
>
> 현재 버전은 `1.0.0-beta01`이며 **Maven Central에 배포되어 있습니다**
> (`io.github.nasmedia-tech:admixer-interactive`, BOM `2026.08.01`부터 멤버 포함).
> 정식 GA가 아니며 제한된 매체 테스트에만 사용합니다. 전체 사용자 대상 상용 배포 전
> 담당자와 지원 버전, 광고 지면, 테스트 범위를 다시 확인하세요.

이 문서가 매체 앱 공개 연동 절차의 단일 기준(SSOT)입니다. 게임별 규칙과 실험실 제약 문서는
담당자([nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr))에게 요청하세요.

## 1. SDK와 매체 앱의 책임

| 구분 | Interactive SDK | 매체 앱 또는 매체 서버 |
|---|---|---|
| 게임 | UI, 규칙, 성공·실패, 점수, 결과 화면 | 사용할 게임과 문구·난이도 선택 |
| 보상 | 설정에 따른 예상 보상 확정 및 결과 전달 | 실제 지급, 잔액, 원장, 지급 성공 판단 |
| 중복 방지 | 실행별 `eventId`, `sessionId` 제공 | 처리한 `eventId` 영속화 및 중복 지급 차단 |
| 티켓 | 화면 세션 안의 기회 차감·충전 | 계정 단위 기회나 일일 한도의 신뢰 가능한 집행 |
| 광고 | Gateway 호출과 성공·취소·실패 이벤트 처리 | 실제 광고 SDK 설정, 지면 운영, 테스트 광고 사용 |
| 계정·보안 | 자동 수집하지 않음 | 사용자 인증, 부정 방지, 서버 검증 |

`RewardResult`는 **지급 요청 또는 지급 참고값**입니다. SDK는 포인트·코인·젬을 실제로
지급하거나 잔액을 변경하지 않습니다. `eventId` 역시 중복 처리 키일 뿐 보안 원장이 아니며,
앱 데이터 삭제·변조·재설치까지 포함한 exactly-once 지급을 보장하지 않습니다.

## 2. 연동 흐름 한눈에 보기

1. AdMixer 코어를 기존 방식으로 초기화합니다.
2. `MiniGameSdkConfig`에 활성 게임, 보상, 티켓, UI와 광고 Gateway를 설정합니다.
3. `MiniGameSdk.initializeDetailed()` 결과의 경고·오류를 확인합니다.
4. `MiniGameSdk.getAvailableGames()`로 실제 실행 가능한 게임만 노출합니다.
5. 화면 Activity에서 `MiniGameSdk.start()`를 호출합니다.
6. `onMiniGameEvent()`는 화면·분석 로그에만 사용합니다.
7. 지급 판단은 `onMiniGameFinished()`에서만 수행합니다.
8. 매체 저장소에 `eventId` 처리 성공 기록을 남깁니다.

## 3. 요구 환경과 설치

| 항목 | Beta 후보 기준 |
|---|---|
| Interactive 버전 | `1.0.0-beta01` |
| minSdk | 21 |
| targetSdk / compileSdk | 35 / 35 (AdMixer SDK 전 모듈과 동일 — 매체 앱에 상향을 요구하지 않습니다) |
| 언어 | Java 공개 API, Kotlin 호출 가능 |
| 필수 권한 | 라이브러리는 `VIBRATE`만 병합, 햅틱 비활성화 가능 |
| 화면 컴포넌트 | `MiniGameLauncherActivity`, `exported=false` |

### 3.1 Maven Central 방식 (권장)

`1.0.0-beta01`부터 Maven Central에서 바로 받을 수 있습니다. BOM(`2026.08.01`+)을 쓰면 버전 생략이 가능합니다.

```groovy
dependencies {
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.08.01')
    implementation 'io.github.nasmedia-tech:admixer-ssp'
    implementation 'io.github.nasmedia-tech:admixer-interactive'   // 버전 생략 = BOM이 1.0.0-beta01 고정
}
```

### 3.2 AAR 직접 연동

Nasmedia가 전달한 `admixer-interactive-release.aar`을 앱의 `libs`에 복사한 뒤 추가합니다.

```groovy
dependencies {
    implementation files('libs/admixer-interactive-release.aar')
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.08.01')
    implementation 'io.github.nasmedia-tech:admixer-ssp'
}
```

AAR 직접 연동은 전이 의존성을 자동 설치하지 않으므로 `admixer-ssp`를 별도로 선언해야 합니다.

## 4. 권장 지급 방식 선택

한 앱에서 실제 지급 경로를 두 개 동시에 사용하면 안 됩니다.

| 방식 | 설정 | 실제 지급 위치 | `onMiniGameFinished()` 역할 |
|---|---|---|---|
| 권장 기본 | `RewardMode.NONE` | `onMiniGameFinished()`에서 매체가 지급 | 유일한 지급 판단 콜백 |
| 기존 호환 | `RewardMode.LEGACY_CALLBACK` | 기존 `InteractiveRewardListener` 경로 | 신규 API 결과 관찰 |
| 고급 Claim | `CALLBACK_ONLY` / `CALLBACK_WITH_ACK` | `RewardSettlementHandler`에서 매체가 지급 | 정산 결과 확인만 수행, 재지급 금지 |

이 문서의 기본 예제는 `RewardMode.NONE`을 사용합니다. 이 모드는 SDK 내부 Claim 정산을
사용하지 않는다는 뜻이며, 성공 결과의 `RewardResult`는 그대로 전달됩니다.

## 5. Java 빠른 시작

### 5.1 Application에서 초기화

AdMixer 코어 초기화는 기존 앱 코드에서 한 번만 수행합니다. `MiniGameSdk.initializeDetailed()`는
AdMixer 코어를 다시 초기화하지 않습니다.

```java
Set<InteractiveContentType> enabledGames = EnumSet.of(
        InteractiveContentType.OX_QUIZ,
        InteractiveContentType.WHEEL_SPIN,
        InteractiveContentType.SCRATCH_LOTTERY
);

RewardResult quizReward = new RewardResult(100L, "coin", "코인", null);
MiniGameOptions quizOptions = MiniGameOptions.builder()
        .enabled(true)
        .displayName("오늘의 OX 퀴즈")
        .initialTicketCount(3)
        .requiredTicketCount(1)
        .rewardAddTicketCount(2)
        .maxSessionPlays(3)
        .timeLimitMillis(10_000L)
        .rewardConfig(RewardConfig.fixed(quizReward))
        .resultDisplayDurationMillis(0L)
        .retryButtonVisible(true)
        .build();

MiniGameSdkConfig config = new MiniGameSdkConfig.Builder()
        .setMediaKey("MEDIA_KEY")
        .setRewardAdUnitId("REWARD_AD_UNIT")
        .setBannerAdUnit("BANNER_AD_UNIT", 320, 50)
        .setEnabledGames(enabledGames)
        .setDefaultTicketCount(3)
        .setDefaultRewardAddTicketCount(2)
        .setTicketConsumePolicy(TicketConsumePolicy.ON_GAME_START)
        .setGameConfig(InteractiveContentType.OX_QUIZ, quizOptions)
        .setRewardMode(RewardMode.NONE)
        .setHapticsEnabled(true)
        .setSoundEnabled(true)
        .setAnimationsEnabled(true)
        .setResultDisplayDurationMillis(0L)
        .setRetryButtonVisible(true)
        .setEventListener(event -> {
            // Application 범위 리스너입니다. Activity나 View를 캡처하지 마세요.
            analytics.logMiniGameState(event.getType().name(), event.getGameId());
        })
        .build();

MiniGameInitializationResult init =
        MiniGameSdk.initializeDetailed(application, config);

for (MiniGameConfigIssue issue : init.getIssues()) {
    appLogger.warn("Interactive config: " + issue.getSeverity()
            + " / " + issue.getGameType() + " / " + issue.getMessage());
}

if (!init.isApplied()) {
    appLogger.error("Interactive SDK configuration was rejected");
}
```

초기화 결과는 다음과 같습니다.

- `APPLIED`: 설정이 그대로 적용됨
- `APPLIED_WITH_WARNINGS`: 잘못된 선택 항목만 안전한 기본값으로 복구됨
- `REJECTED`: `Application` 또는 전체 설정이 없어 적용되지 않음

### 5.2 Activity에서 게임 실행

```java
MiniGameRequest request = new MiniGameRequest.Builder(InteractiveContentType.OX_QUIZ.name())
        .setUserId(hostUserId)
        .setClientRequestId(hostRequestId)
        .build();

MiniGameSdk.start(this, request, new MiniGameListener() {
    @Override
    public void onMiniGameEvent(@NonNull MiniGameEvent event) {
        // 상태 관찰과 UI 로그 전용입니다. GAME_COMPLETED에서도 지급하지 않습니다.
        renderDebugEvent(event.getType(), event.getSessionId());
    }

    @Override
    public void onMiniGameFinished(@NonNull MiniGameResult result) {
        handleMiniGameFinished(result);
    }
});
```

### 5.3 매체 지급 방어 코드

아래 순서를 바꾸지 않는 것을 권장합니다.

1. `eventId` 존재 확인
2. 매체 저장소에서 이미 처리한 이벤트인지 확인
3. `Status`와 `Outcome` 확인
4. `RewardResult` 존재 확인
5. `amount`와 `rewardType` 검증
6. 매체 자체 지급 API 호출
7. 지급 성공 결과를 매체 저장소에 영속화

```java
private void handleMiniGameFinished(@NonNull MiniGameResult result) {
    String eventId = result.getEventId();
    if (eventId == null || eventId.trim().isEmpty()) {
        return;
    }
    if (hostReceiptStore.contains(eventId)) {
        return;
    }
    if (result.getStatus() != MiniGameResult.Status.COMPLETED) {
        return;
    }
    if (result.getOutcome() != MiniGameResult.Outcome.SUCCESS) {
        return;
    }

    RewardResult reward = result.getRewardResult();
    if (reward == null || !reward.isValid()) {
        return;
    }
    if (reward.getAmount() <= 0L || reward.getAmount() > RewardResult.MAX_AMOUNT) {
        return;
    }
    if (!hostRewardTypes.contains(reward.getRewardType())) {
        return;
    }

    boolean granted = hostWallet.grant(
            reward.getRewardType(),
            reward.getAmount(),
            eventId
    );
    if (granted) {
        hostReceiptStore.save(eventId, result.getSessionId());
    }
}
```

실제 앱에서는 지급과 처리 기록 사이의 원자성, 실패 재시도, 네트워크 타임아웃을 매체 시스템
정책으로 결정해야 합니다. SDK 내부에 지급 원장이나 포인트 잔액을 만들지 마세요.

실패 참여 보상을 명시적으로 설정한 경우에만 `Outcome.FAILURE`를 별도 정책으로 허용할 수
있습니다. 취소와 오류는 항상 지급 대상에서 제외하세요.

## 6. Kotlin 빠른 시작

```kotlin
val coin = RewardResult(
    100L,
    "coin",
    "코인",
    null,
)

val oxOptions = MiniGameOptions.builder()
    .displayName("오늘의 OX 퀴즈")
    .requiredTicketCount(1)
    .rewardAddTicketCount(2)
    .rewardConfig(RewardConfig.fixed(coin))
    .retryButtonVisible(true)
    .build()

val config = MiniGameSdkConfig.Builder()
    .setMediaKey("MEDIA_KEY")
    .setRewardAdUnitId("REWARD_AD_UNIT")
    .setEnabledGames(
        EnumSet.of(
            InteractiveContentType.OX_QUIZ,
            InteractiveContentType.WHEEL_SPIN,
        ),
    )
    .setGameConfig(InteractiveContentType.OX_QUIZ, oxOptions)
    .setRewardMode(RewardMode.NONE)
    .build()

val init = MiniGameSdk.initializeDetailed(application, config)
check(init.isApplied) { "Interactive configuration was rejected" }

MiniGameSdk.start(
    activity,
    MiniGameRequest.Builder(InteractiveContentType.OX_QUIZ.name)
        .setUserId(hostUserId)
        .setClientRequestId(hostRequestId)
        .build(),
) { result ->
    handleMiniGameFinished(result)
}
```

`onMiniGameFinished()`는 Android Main Thread에서 호출됩니다. 네트워크나 DB 지급 작업은
매체 앱의 비동기 계층으로 넘기고 UI 스레드를 차단하지 마세요.

## 7. 설정 병합과 지원 항목

실행 시점에 불변 Snapshot을 만들기 때문에 실행 도중 외부 설정 객체가 바뀌어도 현재 게임은
영향받지 않습니다.

```text
MiniGameRequest 명시값 > MiniGameOptions > MiniGameSdkConfig > 게임 내장 기본값
```

콘텐츠 설정은 별도로 다음 순서를 사용합니다.

```text
BUILT_IN < SERVER_REMOTE < MEDIA_GLOBAL < MEDIA_GAME < LAUNCH_OVERRIDE
```

| 범위 | 주요 설정 |
|---|---|
| 전체 | 활성 게임, 기본 티켓, 광고 충전 티켓, 티켓 차감 정책 |
| 게임별 | enabled, 표시 이름, 초기·필요 티켓, 세션 플레이 상한 |
| 보상 | 고정, 범위, 가중치, 점수 비례, 성공·실패, 사용자 계산기 |
| 게임 진행 | 제한 시간 250~300,000ms, 결과 노출 0~60,000ms, 다시 하기 |
| UI | 햅틱, 효과음, 애니메이션, 테마·문구·콘텐츠 override |
| 광고 | 전역 또는 요청별 배너·리워드 지면, 커스텀 Gateway |

0 이하 티켓, 음수 보상, 뒤집힌 범위, 가중치 합 오류, 빈 문제 목록, 제한 범위 밖 시간은
조용히 적용되지 않습니다. `MiniGameInitializationResult` 또는 콘텐츠 `ConfigApplyResult`에서
진단을 확인하세요.

## 8. 보상 설정 예제

`rewardType`은 매체 지급 시스템의 키이고 `displayUnit`은 사용자에게 표시하는 단위입니다.
두 값을 분리하면 `coin/코인`, `gem/젬`처럼 매체별 명칭을 안전하게 사용할 수 있습니다.

### 8.1 고정 보상

```java
RewardConfig fixed = RewardConfig.fixed(
        new RewardResult(100L, "coin", "코인", null)
);
```

### 8.2 범위 보상

```java
RewardConfig range = RewardConfig.range("gem", 1L, 5L, "젬");
```

### 8.3 가중치 보상

가중치는 basis point 합이 정확히 10,000이어야 합니다.

```java
List<RewardConfig.WeightedOption> options = Arrays.asList(
        new RewardConfig.WeightedOption(
                new RewardResult(10L, "coin", "코인", null),
                7_000
        ),
        new RewardConfig.WeightedOption(
                new RewardResult(50L, "coin", "코인", null),
                2_500
        ),
        new RewardConfig.WeightedOption(
                new RewardResult(100L, "coin", "코인", null),
                500
        )
);
RewardConfig weighted = RewardConfig.weighted(options);
```

### 8.4 점수 비례 보상

아래 설정은 `점수 × 1 ÷ 10`을 계산하고 1~1,000 사이로 제한합니다.

```java
RewardConfig scoreReward = RewardConfig.scoreProportional(
        "coin",
        1L,
        10L,
        1L,
        1_000L,
        "코인"
);
```

### 8.5 성공·실패 보상

```java
RewardConfig successFailure = RewardConfig.successFailure(
        new RewardResult(100L, "coin", "코인", "성공 보상"),
        new RewardResult(5L, "coin", "코인", "참여 보상")
);
```

### 8.6 사용자 계산기

```java
RewardConfig custom = RewardConfig.custom((context, random) -> {
    long amount = context.isSuccess() ? Math.min(context.getScore(), 500L) : 0L;
    return new RewardResult(amount, "coin", "코인", null);
});
```

사용자 계산기가 예외를 던지거나 잘못된 결과를 반환하면 SDK 내장 보상으로 안전하게
복구됩니다. 한 세션의 보상 난수는 한 번만 확정되며 결과 UI와 콜백이 같은 값을 사용합니다.

## 9. 콘텐츠와 문제 데이터 재정의

타입 안전 객체, JSON, `Map<String, ?>`은 같은 Canonical Config 검증기를 사용합니다.
변경은 다음 실행부터 적용됩니다.

```java
QuizItem item = new QuizItem(
        "media-ox-001",
        "대한민국의 수도는 서울이다.",
        true,
        "대한민국의 수도는 서울입니다.",
        1
);

GameDataOverride gameData = GameDataOverride.builder()
        .mergeMode(MergeMode.APPEND)
        .quizItems(Collections.singletonList(item))
        .questionCount(1)
        .build();

MiniGameContentConfig contentConfig = MiniGameContentConfig.builder()
        .schemaVersion(2)
        .configId("media-labs-20260810")
        .game(
                InteractiveContentType.OX_QUIZ.name(),
                GameContentOverride.builder()
                        .gameData(gameData)
                        .build()
        )
        .build();

ConfigApplyResult applyResult =
        MiniGameSdk.setContentConfigDetailed(contentConfig);

if (!applyResult.isApplied()) {
    appLogger.error("Interactive content configuration was rejected");
}
```

앱 자산이나 매체 서버 응답을 JSON으로 적용할 때는 `schemaVersion`과 `configId`를 포함합니다.

```json
{
  "schemaVersion": 2,
  "configId": "media-labs-20260810",
  "global": {
    "content": {
      "retryButtonText": "한 번 더",
      "closeButtonText": "게임 종료"
    }
  },
  "games": {
    "OX_QUIZ": {
      "enabled": true,
      "content": {
        "title": "오늘의 OX 퀴즈",
        "description": "정답을 골라 주세요"
      },
      "gameplay": {
        "timeLimitSeconds": 10
      },
      "gameData": {
        "mergeMode": "APPEND",
        "questionCount": 1,
        "quizItems": [
          {
            "id": "media-ox-001",
            "question": "대한민국의 수도는 서울이다.",
            "answer": true,
            "explanation": "대한민국의 수도는 서울입니다.",
            "difficulty": 1
          }
        ]
      },
      "reward": {
        "rewardType": "coin",
        "amount": 100,
        "rewardName": "OX 퀴즈 코인",
        "minimumScore": 1
      }
    }
  }
}
```

```java
ConfigApplyResult jsonResult =
        MiniGameSdk.setContentConfigJsonDetailed(jsonString);

if (!jsonResult.isApplied()) {
    appLogger.error("Interactive JSON configuration was rejected");
}
```

`Map<String, ?>`을 사용할 때는 같은 key 구조를 만든 뒤
`MiniGameSdk.setContentConfigMap(configMap)`을 호출합니다.

JSON/Map 스키마 v2의 주요 제한은 다음과 같습니다.

- 최대 입력 256KiB, 중첩 깊이 8, 배열 200, 게임 항목 64
- `REPLACE`: 유효한 매체 데이터로 기본 데이터 교체
- `APPEND`: 기본 데이터 뒤에 추가
- `PREPEND`: 기본 데이터 앞에 추가
- 전부 거부되면 해당 게임의 안전한 내장 데이터 사용
- `mediaKey`, 광고 지면, callback URL, class/method/script/intent/exec 필드 설정 금지
- 임의 POJO, `Serializable`, 문자열이 아닌 Map key 거부

광고 지면과 보안 식별자는 타입 안전 앱 코드에서만 지정하세요.

## 10. 게임 목록과 실행 ID

매체 화면에는 `getAvailableGames()` 결과만 노출하는 것을 권장합니다.

```java
List<GameMetadata> games = MiniGameSdk.getAvailableGames();
```

| 카테고리 | 공개 ID |
|---|---|
| 행운·추첨 | `SCRATCH_LOTTERY`, `DAILY_LOTTO`, `WHEEL_SPIN`, `GOLDEN_LADDER`, `FORTUNE_COOKIE`, `LUCKY_DRAW`, `MINI_SLOT`, `TREASURE_CHEST` |
| 터치·액션 | `POP_BALLOONS`, `MEMORY_CARD`, `STACK_BURGER`, `COIN_FLIP`, `HEART_CLICKER`, `DICE_ROLL`, `CHAM_CHAM_CHAM`, `MONEY_TREE_SHAKE` |
| 퀴즈·퍼즐 | `FLASH_QUIZ`, `OX_QUIZ`, `SPOT_DIFFERENCE`, `WORD_PUZZLE`, `REWARD_BINGO`, `NUMBER_MEMORY`, `COLOR_DIFFERENCE` |
| 힐링·운세 | `DAILY_HOROSCOPE`, `TARO_CARD`, `LUCKY_FOOD`, `LUCKY_COLOR`, `HEALING_QUOTE`, `ATTENDANCE_STAMP`, `LUCKY_WEATHER` |

게임 ID와 enum 순서는 공개 계약입니다. 문자열을 별도 변환하거나 `REWARD_BINGO`를 다른
이름으로 치환하지 마세요.

## 11. 콜백 계약

`onMiniGameEvent()`는 관찰·분석용이고, `onMiniGameFinished()`만 지급 판단용입니다.

### 성공

```text
SCREEN_SHOWN → GAME_STARTED → TICKET_CONSUMED → GAME_SUCCEEDED
→ GAME_COMPLETED → onMiniGameFinished → SCREEN_CLOSED
```

### 게임 규칙상 실패

```text
SCREEN_SHOWN → GAME_STARTED → TICKET_CONSUMED → GAME_FAILED
→ GAME_COMPLETED → onMiniGameFinished → SCREEN_CLOSED
```

### 결과 확정 전 취소

```text
SCREEN_SHOWN → GAME_STARTED → GAME_CANCELLED
→ onMiniGameFinished → SCREEN_CLOSED
```

### 실행 오류

```text
SCREEN_SHOWN → GAME_ERROR → onMiniGameFinished → SCREEN_CLOSED
```

기존 공개 API 호환을 위해 취소·오류도 `onMiniGameFinished()`로 한 번 전달됩니다. 두 결과는
`rewardResult == null`이며 지급 대상이 아닙니다.

| `Outcome` | 의미 | 기본 지급 판단 |
|---|---|---|
| `SUCCESS` | 게임 성공 또는 정상 추첨 결과 | 보상 검증 후 가능 |
| `FAILURE` | 게임 규칙상 실패 | 명시적 참여 보상 설정 때만 검토 |
| `CANCELLED` | 사용자 중도 종료 | 지급 금지 |
| `ERROR` | SDK 실행 오류 | 지급 금지 |

완료 버튼 연타, 뒤로가기 연타, 화면 회전, 광고 중복 콜백이 있어도 동일 세션의 최종 결과는
최대 한 번만 전달됩니다. 다시 하기는 새 `sessionId`와 `eventId`를 발급합니다. 동시에 두 게임을
실행하면 두 번째 요청은 `ALREADY_RUNNING` 오류로 거부됩니다.

## 12. 티켓 정책

| 정책 | 차감 시점 | 취소 처리 |
|---|---|---|
| `ON_GAME_START` | 실제 게임 판이 시작될 때 | 시작 후 취소해도 이미 차감됨 |
| `ON_GAME_RESULT` | 성공 또는 게임 실패 결과가 확정될 때 | 결과 확정 전 취소·오류는 차감하지 않음 |

광고 시청 완료는 게임 보상 지급과 별개입니다. 광고 Gateway가 `earnedReward=true`를 전달한
경우에만 설정된 티켓을 한 번 충전합니다.

## 13. 리워드 광고 Gateway

특정 광고 네트워크 SDK를 Interactive 코어 공개 API에 직접 노출하지 않습니다.

### 13.1 AdMixer 리워드 광고 사용

별도 Factory를 지정하지 않으면 SDK가 기본 `AdMixerRewardedAdGateway`를 사용합니다.
`mediaKey`와 리워드 지면만 설정하면 됩니다.

```java
MiniGameSdkConfig config = new MiniGameSdkConfig.Builder()
        .setMediaKey("MEDIA_KEY")
        .setRewardAdUnitId("REWARD_AD_UNIT")
        .build();
```

AdMixer 초기화 시 게임에서 사용할 모든 광고 지면을 등록해야 합니다.

### 13.2 매체의 다른 광고 SDK 연결

매체 광고 SDK를 감싸는 `RewardedAdGateway`를 구현하고 실행마다 새 인스턴스를 반환하세요.

```java
MiniGameSdkConfig config = new MiniGameSdkConfig.Builder()
        .setMediaKey("MEDIA_KEY")
        .setRewardAdUnitId("HOST_REWARDED_UNIT")
        .setRewardedAdGatewayFactory((activity, mediaKey, rewardAdUnitId) ->
                new HostRewardedAdGateway(
                        hostRewardedAdClientFactory.create(rewardAdUnitId)
                )
        )
        .build();
```

`HostRewardedAdGateway`는 매체 광고 SDK의 타입을 이 모듈 밖으로 숨기는 앱 소유 클래스입니다.
아래처럼 매체 광고 계층에서 먼저 로드·노출 결과를 단일 계약으로 정규화한 뒤 Gateway에
연결할 수 있습니다.

```java
interface HostRewardedClient {
    interface LoadResult {
        void onLoaded();
        void onFailed(int sourceCode, @Nullable String message);
    }

    interface ShowResult {
        void onDisplayed();
        void onFinished(boolean earnedReward);
        void onFailed(int sourceCode, @Nullable String message);
    }

    void load(@NonNull LoadResult result);
    boolean isReady();
    void show(@NonNull Activity activity, @NonNull ShowResult result);
    void destroy();
}

final class HostRewardedAdGateway implements RewardedAdGateway {
    private final HostRewardedClient client;
    private boolean released;

    HostRewardedAdGateway(@NonNull HostRewardedClient client) {
        this.client = client;
    }

    @Override
    public void preload(@NonNull AdLoadCallback callback) {
        if (released) {
            callback.onAdLoadFailed(new AdError(
                    AdError.ERROR_AD_LOAD_FAILED,
                    "Gateway released"
            ));
            return;
        }
        AtomicBoolean terminal = new AtomicBoolean();
        client.load(new HostRewardedClient.LoadResult() {
            @Override
            public void onLoaded() {
                if (!released && terminal.compareAndSet(false, true)) {
                    callback.onAdLoaded();
                }
            }

            @Override
            public void onFailed(int sourceCode, @Nullable String message) {
                if (!released && terminal.compareAndSet(false, true)) {
                    callback.onAdLoadFailed(new AdError(
                            AdError.ERROR_AD_LOAD_FAILED,
                            message,
                            sourceCode
                    ));
                }
            }
        });
    }

    @Override
    public boolean isReady() {
        return !released && client.isReady();
    }

    @Override
    public void show(@NonNull Activity activity, @NonNull AdShowCallback callback) {
        if (released || activity.isFinishing() || activity.isDestroyed()) {
            callback.onAdShowFailed(new AdError(
                    AdError.ERROR_INVALID_ACTIVITY,
                    "Activity is unavailable"
            ));
            return;
        }
        AtomicBoolean terminal = new AtomicBoolean();
        client.show(activity, new HostRewardedClient.ShowResult() {
            @Override
            public void onDisplayed() {
                if (!released && !terminal.get()) {
                    callback.onAdDisplayed();
                }
            }

            @Override
            public void onFinished(boolean earnedReward) {
                if (!released && terminal.compareAndSet(false, true)) {
                    callback.onAdFinished(earnedReward);
                }
            }

            @Override
            public void onFailed(int sourceCode, @Nullable String message) {
                if (!released && terminal.compareAndSet(false, true)) {
                    callback.onAdShowFailed(new AdError(
                            AdError.ERROR_AD_SHOW_FAILED,
                            message,
                            sourceCode
                    ));
                }
            }
        });
    }

    @Override
    public void release() {
        if (released) {
            return;
        }
        released = true;
        client.destroy();
    }
}
```

위 예제의 `HostRewardedClient` 구현에서 광고 네트워크별 Reward/Close 역순, show timeout과
중복 콜백을 먼저 정규화해야 합니다. `HostRewardedAdGateway`에는 매체 네트워크의 광고 객체나
listener 타입을 공개하지 마세요.

매체 구현체는 다음 계약을 지켜야 합니다.

- `preload`: `onAdLoaded` 또는 `onAdLoadFailed` 중 하나만 한 번 호출
- `show`: `onAdFinished` 또는 `onAdShowFailed` 중 하나만 한 번 호출
- 광고가 표시됐다는 이유만으로 `earnedReward=true`를 전달하지 않음
- 보상 획득이 확인된 경우에만 `onAdFinished(true)` 호출
- 보상 없이 닫힘은 `onAdFinished(false)` 호출
- `release()` 이후 모든 늦은 콜백 무시
- Factory에 Activity/View를 정적으로 저장하지 않음
- 네트워크의 Reward/Close 순서가 달라도 종단 결과는 한 번만 확정

광고 이벤트 순서는 다음과 같습니다.

```text
완주: REWARDED_AD_REQUESTED → REWARDED_AD_COMPLETED → TICKETS_RECHARGED
취소: REWARDED_AD_REQUESTED → REWARDED_AD_CANCELLED
실패: REWARDED_AD_REQUESTED → REWARDED_AD_FAILED
```

## 14. 선택적 RewardClaim 승인 연동

이미 `RewardSettlementHandler` 기반 구조를 사용하는 매체만 이 절을 적용하세요.

```java
MiniGameSdkConfig config = new MiniGameSdkConfig.Builder()
        .setMediaKey("MEDIA_KEY")
        .setRewardAdUnitId("REWARD_AD_UNIT")
        .setRewardMode(RewardMode.CALLBACK_WITH_ACK)
        .setRewardTypeWhitelist(Collections.singleton("coin"))
        .setSettlementTimeoutMillis(10_000L)
        .setRewardSettlementHandler((claim, callback) -> {
            mediaRewardApi.grant(
                    claim.getIdempotencyKey(),
                    claim.getUserId(),
                    claim.getRewardType(),
                    claim.getProposedAmountLong(),
                    response -> {
                        if (response.isSuccessful()) {
                            callback.confirm(
                                    response.getTransactionId(),
                                    response.getGrantedAmount()
                            );
                        } else {
                            callback.reject(new RewardError(
                                    response.getErrorCode(),
                                    response.getErrorMessage()
                            ));
                        }
                    }
            );
        })
        .build();
```

이 방식에서는 실제 지급이 `RewardSettlementHandler`에서 이미 수행됩니다.
`onMiniGameFinished()`에서 다시 `hostWallet.grant()`를 호출하면 중복 지급이므로 금지합니다.
매체 서버는 `RewardClaim.idempotencyKey`에 unique 제약을 두고 같은 요청에는 기존 거래 결과를
반환해야 합니다.

`RewardClaim.claimToken`은 같은 입력에서 같은 값이 나오는 키 없는 SHA-256 지문입니다.
중복 진단에는 사용할 수 있지만 서명·인증·무결성 증명이 아니므로 서버가 지급 증빙으로
신뢰하면 안 됩니다. 지급 상한과 계정 단위 일일 한도는 인증된 매체 서버가 집행해야 합니다.

`CALLBACK_ONLY`는 승인 응답을 기다리지 않고 `PENDING` 결과를 전달합니다.
`CALLBACK_WITH_ACK`는 confirm·reject·timeout 중 하나가 확정될 때까지 최종 결과를 기다립니다.

## 15. 오류와 안전한 실패

| 영역 | 코드 | 의미와 매체 처리 |
|---|---|---|
| 실행 | `INVALID_ACTIVITY` | Activity가 null·종료 중·파괴됨. 새 화면을 열지 않음 |
| 실행 | `INVALID_REQUEST` | 필수 ID·티켓·시간 값 오류. 요청 수정 |
| 실행 | `UNKNOWN_GAME` | ID 오류 또는 host/content 설정에서 비활성 |
| 실행 | `ALREADY_RUNNING` | 다른 게임 실행 중. 현재 게임 종료 후 재시도 |
| 실행 | `RESOURCE_LOAD_FAILED` | View·리소스 생성 실패. 지급 금지 |
| 실행 | `INTERNAL` | SDK 내부 오류. 지급 금지 및 비민감 진단 기록 |
| 정산 | `HANDLER_MISSING` | Claim 모드에 Handler가 없음 |
| 정산 | `REWARD_TYPE_NOT_ALLOWED` | whitelist 밖의 지급 키 |
| 정산 | `INVALID_REWARD_AMOUNT` | 보상 범위 오류 |
| 정산 | `HANDLER_EXCEPTION` | 매체 Handler 예외가 안전한 실패로 변환됨 |
| 정산 | `INVALID_SETTLEMENT_ACK` | 거래 ID 또는 승인 수량 오류 |
| 정산 | `SETTLEMENT_TIMEOUT` | 제한 시간 안에 승인되지 않음 |

오류 로그에 userId, 광고 ID, claim token, 기기 식별자, 광고 원본 응답을 자동으로 넣지 마세요.

## 16. 생명주기와 스레드

- 모든 공개 이벤트와 최종 결과 콜백은 Main Thread에서 발화합니다.
- 리스너나 정산 Handler가 `Throwable`을 던져도 SDK는 화면 종료와 내부 정리를 계속합니다.
- Factory와 전역 이벤트 리스너는 Application 범위이므로 Activity/View를 캡처하면 안 됩니다.
- 화면 회전에서는 동일 플레이의 `sessionId`와 확정 결과를 보존합니다.
- 정산 승인 대기 중 사용자가 화면을 닫아도 confirm/reject/timeout 최종 콜백은 계속 전달됩니다.
- 프로세스가 종료되면 메모리의 리스너를 복원할 수 없습니다. 복원된 orphan 런처는 자동 지급이나
  콜백 재발행 없이 안전한 재진입 안내를 표시합니다. 매체 앱은 서버 원장과 영속 `eventId` 기록을
  기준으로 최종 상태를 조회해야 합니다.
- Activity 종료 후 늦은 타이머·애니메이션·광고 콜백은 무시합니다.
- 네트워크·DB·파일 작업은 매체 앱의 백그라운드 계층에서 처리하세요.
- 향후 게임 실행을 완전히 중단할 때만 `MiniGameSdk.release()`로 전역 정책을 해제하세요.

## 17. 레거시 연동 호환

기존 30종 Builder, Dialog, `InteractiveRewardListener`는 즉시 제거되지 않았습니다.
기존 앱은 그대로 컴파일되며 신규 설정 API로 점진적으로 옮길 수 있습니다.

권장 이전 순서:

1. 기존 Builder 실행을 유지한 채 `MiniGameSdkConfig` 기본값만 도입
2. `getAvailableGames()`로 노출 목록 통합
3. 한 게임씩 `MiniGameSdk.start()`로 전환
4. `onMiniGameFinished()`의 `eventId` 저장소 적용
5. 기존 지급 콜백을 제거한 뒤 지급 경로를 하나로 통일
6. 마지막에 매체 광고 Gateway를 연결

세부 변경 정책은 담당자에게 문의하세요.

## 18. R8·Manifest·의존성 확인

- 라이브러리 consumer ProGuard 규칙은 enum 이름 계약만 제한적으로 보존하며, 공개 이름 계약은
  source `@Keep`를 정본으로 사용합니다. Interactive 전체 패키지 keep은 제공하지 않습니다.
- 매체 release 빌드에서 R8를 반드시 실제 실행하세요.
- 앱 테스트용 광고 Gateway 대역(Fake)은 매체 앱의 자체 test source에서 `RewardedAdGateway`를
  구현해 사용하고, 운영 빌드에 남지 않도록 debug 소스셋에만 두세요.
- 병합 Manifest에 불필요한 권한이나 exported 컴포넌트가 없는지 확인하세요.
- Sample 광고 그래프의 Google Mobile Ads 25.2.0은 Kotlin metadata 2.2.0 경고가 발생할 수
  있습니다. 매체의 Kotlin compiler, GMA, 어댑터 버전을 검증된 조합으로 정렬하세요.
- Interactive 코어 때문에 AGP/Kotlin 전체를 무조건 업그레이드하지 마세요.

## 19. 매체 QA 체크리스트

- 활성 게임 1종과 30종 전체 설정 확인
- 비활성 게임 실행 차단 확인
- `coin/코인`, `gem/젬` UI와 결과 콜백 값 일치 확인
- 성공·실패·취소·오류 구분 확인
- 완료·뒤로가기 버튼 연타 시 최종 콜백 1회 확인
- 결과 확정 전후 회전 시 중복 지급 없음 확인
- 다시 하기마다 새 `sessionId` 확인
- 동시 게임 실행이 `ALREADY_RUNNING`으로 거부되는지 확인
- 테스트용 Gateway 대역으로 광고 성공·취소·실패와 티켓 충전 확인
- 실제 테스트 광고의 Reward/Close 역순·no-fill·네트워크 단절 확인
- Reward 뒤 Close가 누락되면 10초 fallback으로 게임이 복귀하는지 확인
- 작은 화면, 가로, 큰 글자, 다크 모드, TalkBack 확인
- Debug·Release AAR, 앱 Release/R8 빌드 확인
- 처리한 `eventId`가 앱 재시작 뒤에도 중복 지급되지 않는지 확인

서버측 실행 상한·일일 한도·멱등 원장과 process-death 재조회 기준은 담당자가 제공하는
운영 보상 원장 체크리스트를 따르세요.

샘플 실행법과 30종 수동 검증표는 담당자에게 요청하세요.

## 20. 실험실에서 반드시 기억할 제한

- 실물 단말·OEM·TalkBack·실광고 조합은 매체 환경에서 다시 검증해야 합니다.
- `DAILY_HOROSCOPE`, `ATTENDANCE_STAMP`, `LUCKY_WEATHER`는 서버 원장이나 실제 날씨 API가
  아닙니다.
- 기기 시간 변경을 이용한 일일 제한 우회를 완벽히 방지하지 않습니다.
- 실제 포인트 지급 성공 여부는 SDK가 결정하지 않습니다.
- 운영 지급 상한과 일일 한도는 SDK 콘텐츠 설정이 아니라 매체 서버가 집행합니다.
- Beta에서 최종 GA로 이동할 때 Maven 좌표, BOM, 공개 API와 마이그레이션 문서를 다시
  확인해야 합니다.
- #119의 test fixture/AAR 경계와 실제 host R8 축소는 완료했습니다. 실광고·물리 단말·운영 원장
  staging 승인은 정식 GA 외부 게이트로 계속 추적합니다.
