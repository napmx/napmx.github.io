# AdMixer Interactive SDK 연동 가이드 (실험실)

AdMixer Interactive SDK는 AdMixer 미디에이션과 함께 사용할 수 있는 선택형 미니게임
모듈입니다. 행운·액션·퀴즈·퍼즐·힐링 카테고리의 게임 30종을 단일 API로 실행하고,
광고를 통한 게임 기회 충전과 매체 앱의 자체 보상 처리를 연결할 수 있습니다.

> 🧪 **실험실(Labs) · 테스트 제공**
>
> 이 모듈은 정식 GA(General Availability)가 아닌 제한된 매체 테스트용 실험실 기능입니다.
> 현재 Maven Central과 `admixer-bom`에는 포함되지 않으며, Nasmedia가 제공한 Maven
> Local 아티팩트로만 연동합니다. 실험실 운영 기간에는 Interactive 전용 API·설정 스키마와
> 게임별 동작이 변경될 수 있으므로 전체 사용자 대상 상용 배포 전 담당자와 버전 및
> 지원 범위를 확인하세요. 기존 AdMixer 광고 API와 광고 이벤트 계약은 변경하지 않습니다.

## 1. 책임 경계

- SDK는 게임 결과를 검증하고 지급 가능한 `RewardClaim`을 매체 앱에 전달합니다.
- SDK는 포인트 잔액을 저장하거나 실제 포인트를 지급하지 않습니다.
- 실제 적립과 일일 한도, 사용자 검증, 중복 차단은 매체 앱 또는 매체 서버가 담당합니다.
- 매체 저장소는 `RewardClaim.idempotencyKey`를 unique key로 사용해야 합니다.
- 콜백 URL, 검증 토큰 생성 규칙, `mediaKey`, 광고 지면 ID는 JSON/Map 설정으로 변경할 수 없습니다.

포인트가 없는 앱은 `RewardMode.NONE`으로 게임 결과만 받거나,
`CALLBACK_ONLY`에서 Claim을 자체 혜택·로컬 재화로 변환할 수 있습니다.

## 2. 실험실 모듈 설치

### 2.1 SDK 저장소에서 Maven Local 발행

Nasmedia가 제공한 SDK 소스/패키지에서 다음 명령으로 실험실 아티팩트를 발행합니다.

```bash
./gradlew :admixer-interactive:publishToMavenLocal \
  -PInteractiveSDKVersion=1.0.0-SNAPSHOT
```

Windows PowerShell에서는 `./gradlew` 대신 `.\gradlew.bat`를 사용합니다.

### 2.2 매체 앱 저장소 설정

`settings.gradle`에서 Interactive 모듈과 코어 전이 의존성에만 Maven Local을
허용합니다. 다른 광고 어댑터는 계속 Maven Central/BOM 버전을 사용합니다.

```groovy
dependencyResolutionManagement {
    repositories {
        mavenLocal {
            content {
                includeModule 'io.github.nasmedia-tech', 'admixer-interactive'
                includeModule 'io.github.nasmedia-tech', 'admixer-ssp'
            }
        }
        google()
        mavenCentral()
    }
}
```

앱 모듈의 `build.gradle`에 의존성을 추가합니다.

```groovy
dependencies {
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.07.03')
    implementation 'io.github.nasmedia-tech:admixer-ssp'
    implementation 'io.github.nasmedia-tech:admixer-interactive:1.0.0-SNAPSHOT'
}
```

> Maven Local 아티팩트가 없는 개발 PC와 CI에서는 빌드할 수 없습니다. 팀 테스트 또는
> 스토어 실험실 빌드를 재현하려면 동일 버전의 로컬 아티팩트를 먼저 배포해야 합니다.

## 3. SDK 초기화

기존 AdMixer 초기화는 앱에서 기존 방식대로 최초 1회만 수행합니다.
`MiniGameSdk.initialize()`는 AdMixer 코어를 다시 초기화하지 않습니다.

```java
AdMixer.getInstance().initialize(application, "MEDIA_KEY", adUnitIds);

MiniGameSdk.initialize(
        application,
        new MiniGameSdkConfig.Builder()
                .setMediaKey("MEDIA_KEY")
                .setRewardAdUnitId("REWARD_AD_UNIT")
                .setBannerAdUnit("BANNER_AD_UNIT", 320, 50)
                .setRewardMode(RewardMode.CALLBACK_WITH_ACK)
                .setRewardTypeWhitelist(Collections.singleton("POINT"))
                .setRewardSettlementHandler((claim, callback) -> {
                    // 앱 DB 또는 매체 서버에서 idempotencyKey로 멱등 처리합니다.
                    pointRepository.insertIfAbsent(claim, inserted -> {
                        if (inserted) {
                            callback.confirm("APP_TX_ID", claim.getProposedAmount());
                        } else {
                            // 이미 같은 Claim이 적립된 경우에도 기존 거래 결과로 응답합니다.
                            callback.confirm("EXISTING_TX_ID", claim.getProposedAmount());
                        }
                    });
                })
                .build());
```

`RewardSettlementHandler`를 사용하지 않는 기존 30종 Builder와
`InteractiveRewardListener` 연동은 그대로 동작합니다. 초기화가 필수가 아닌 기존
요청별 `mediaKey`/광고 지면 설정 방식도 유지됩니다.

## 4. 콘텐츠 설정

타입 안전 객체, JSON 문자열, `Map<String, ?>` 세 방식은 동일한 Canonical Config와
검증기를 사용합니다. 실행 중인 게임에는 변경이 반영되지 않으며, 다음 게임 실행 시
불변 Snapshot으로 적용됩니다.

```java
MiniGameSdk.setContentConfig(typedConfig);

ConfigApplyResult jsonResult =
        MiniGameSdk.setContentConfigJsonDetailed(jsonString);

ConfigApplyResult mapResult =
        MiniGameSdk.setContentConfigMap(configMap);

if (!jsonResult.isApplied()) {
    // 앱을 종료하지 않고 SDK 기본값으로 안전하게 복구됩니다.
    renderConfigIssues(jsonResult.getIssues());
}
```

설정 우선순위는 다음과 같습니다.

```text
BUILT_IN < SERVER_REMOTE < MEDIA_GLOBAL < MEDIA_GAME < LAUNCH_OVERRIDE
```

JSON/Map에는 `schemaVersion`이 필요하며, 현재 예제 스키마는 v2입니다. JSON·Map의
크기/중첩/배열/문자열 제한을 초과하거나 알 수 없는 게임·필드가 들어오면 해당 값은
거부되고 `ConfigApplyResult`의 Issue에 기록됩니다.

```json
{
  "schemaVersion": 2,
  "configId": "media-labs-20260805",
  "games": {
    "OX_QUIZ": {
      "enabled": true,
      "content": { "title": "오늘의 OX 퀴즈" },
      "gameData": {
        "mergeMode": "APPEND",
        "quizItems": [
          {
            "id": "media-ox-001",
            "question": "대한민국의 수도는 서울이다.",
            "answer": true
          }
        ]
      },
      "reward": {
        "rewardType": "POINT",
        "amount": 100,
        "minimumScore": 0
      }
    }
  }
}
```

실험실 상세 스키마와 게임 데이터 제한값은 배포 아티팩트와 함께 제공되는 버전별
명세를 기준으로 확인하세요. 스키마 버전이 다르면 설정을 적용하지 마세요.

## 5. 게임 목록 조회 및 실행

```java
List<GameMetadata> games = MiniGameSdk.getAvailableGames();

MiniGameSdk.start(
        activity,
        new MiniGameRequest.Builder("OX_QUIZ")
                .setUserId(userId)
                .setClientRequestId(clientRequestId)
                .build(),
        result -> {
            switch (result.getStatus()) {
                case COMPLETED:
                    renderCompleted(result);
                    break;
                case CANCELLED:
                    renderCancelled();
                    break;
                case FAILED:
                    renderError(result.getError());
                    break;
            }
        });
```

모든 공개 콜백은 Main Thread에서 호출됩니다. 종료 중인 Activity, 초기화 전 호출,
동시 중복 실행, 잘못된 설정은 앱 크래시 대신 실패 결과 또는 Issue로 전달됩니다.

### 5.1 게임별 광고 지면

초기화의 배너·리워드 지면은 30종에 공통으로 적용되는 기본값입니다. 게임별로 다른
지면이 필요하면 실행 요청에 지면을 지정합니다. 실행 요청 값이 전역 기본값보다 우선하며,
지정하지 않은 항목만 전역값을 상속합니다.

```java
Map<String, String> bannerByGame = new HashMap<>();
bannerByGame.put("OX_QUIZ", "BANNER_OX_QUIZ");
bannerByGame.put("WHEEL_SPIN", "BANNER_WHEEL_SPIN");

String gameId = "OX_QUIZ";
MiniGameSdk.start(
        activity,
        new MiniGameRequest.Builder(gameId)
                .setBannerAdUnit(bannerByGame.get(gameId), 320, 50)
                .setRewardAdUnitId("REWARD_" + gameId)
                .build(),
        result -> renderResult(result));
```

```kotlin
val bannerByGame = mapOf(
    "OX_QUIZ" to "BANNER_OX_QUIZ",
    "WHEEL_SPIN" to "BANNER_WHEEL_SPIN",
)
val gameId = "OX_QUIZ"
MiniGameSdk.start(
    activity,
    MiniGameRequest.Builder(gameId)
        .setBannerAdUnit(bannerByGame[gameId], 320, 50)
        .setRewardAdUnitId("REWARD_$gameId")
        .build(),
) { result -> renderResult(result) }
```

`mediaKey`와 광고 지면 ID는 보안 설정이므로 JSON/Map 콘텐츠 Override로 바꿀 수
없습니다. AdMixer 초기화 시에는 위 요청에서 사용할 모든 지면 ID를 등록해야 합니다.
게임별 성과를 별도로 집계하려면 광고 서버에서 게임별 지면 ID를 발급받아 매핑하세요.

## 6. 보상 처리 모드

| 모드 | 동작 |
|---|---|
| `LEGACY_CALLBACK` | 기존 `InteractiveRewardListener` 계약 유지. 기존 연동 기본값 |
| `CALLBACK_ONLY` | `RewardClaim`만 전달하고 앱이 자체 처리 |
| `CALLBACK_WITH_ACK` | 앱/서버의 confirm·reject·timeout까지 기다린 뒤 최종 결과 전달 |
| `NONE` | 포인트 지급 없이 게임 결과만 전달 |

`onRewardRequested`는 Claim당 한 번만 호출됩니다. `confirm`/`reject` 중 먼저 도착한
응답만 인정하며, 중복·지연 응답과 Handler 예외는 무시 또는 안전한 실패로 처리합니다.
광고 SDK별 Reward/Close 순서는 다를 수 있으므로 매체 앱에서 Close를 지급 조건으로
사용하지 마세요.

상세 지급 시퀀스와 콜백·Thread 계약은 배포 아티팩트와 함께 제공되는 버전별
명세를 기준으로 확인하세요.

## 7. 지원 게임 30종

- **행운·추첨 8종**: ScratchLottery, DailyLotto, WheelSpin, GoldenLadder,
  TreasureChest, FortuneCookie, LuckyDraw, MiniSlot
- **터치·액션 8종**: PopBalloons, MemoryCard, StackBurger, CoinFlip,
  HeartClicker, DiceRoll, ChamChamCham, MoneyTreeShake
- **퀴즈·퍼즐 7종**: FlashQuiz, OXQuiz, SpotDifference, WordPuzzle,
  DailyBingo, NumberMemory, ColorDifference
- **힐링·운세 7종**: DailyHoroscope, TaroCard, LuckyFood, LuckyColor,
  HealingQuote, AttendanceStamp, LuckyWeather

## 8. 실험실 배포 체크리스트

- Nasmedia 담당자와 실험실 SDK 버전·대상 매체·테스트 기간 확인
- Debug와 Release/R8 빌드 모두 검증
- 30종 목록과 JSON 기본 설정 파싱 확인
- 실제 적립 저장소에 `idempotencyKey` unique constraint 적용
- 성공·실패·timeout·중복 confirm·Reward/Close 역순 테스트
- userId, Claim token, 광고 응답을 로그에 남기지 않음
- 앱 삭제 시 사라지는 로컬 포인트라면 사용자에게 명확히 고지
- GA 전 Maven Central/BOM 좌표와 최종 API로 재검증
