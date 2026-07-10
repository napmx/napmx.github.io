# 리워드 동영상 광고

> ℹ️ 리워드 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

리워드 동영상 광고는 `AMMRewardVideo`를 사용합니다. 사용자가 동영상을 끝까지 시청하면 **`OnUserEarnedRewardListener.onUserEarnedReward()`** 콜백이 호출되며, 이 시점에 리워드를 지급하세요.

> ℹ️ 리워드 광고는 **정적 `loadAd()` → 로드된 광고 객체 반환 → `show(activity, OnUserEarnedRewardListener)`** 구조를 사용합니다. 노출/클릭/닫힘은 `FullScreenContentCallback`으로, 보상 적립은 `OnUserEarnedRewardListener`로 수신합니다.
>
> 구 `RewardInterstitialVideoAd` 클래스는 제거되었습니다 — `AMMRewardVideo` 정적 `loadAd()`로 전환하세요.

---

## 기본 흐름

```
AMMRewardVideo.loadAd(context, adInfo, callback)
    → onSuccessLoadReward(adapterName, ad)             ← 로드된 광고 객체 전달
        → ad.setFullScreenContentCallback(...)         ← 노출/클릭/재생완료/닫힘 콜백
        → ad.show(activity, onUserEarnedRewardListener) ← 노출 + 보상 리스너 등록
            → onUserEarnedReward()                     ← 🎁 리워드 지급 시점 (시청 완료)
    → onFailLoadReward(errorCode, errorMsg)            ← 로드 실패
```

> ℹ️ **보상 적립(`onUserEarnedReward`)** 과 **영상 재생 완료(`onAdCompleted`)** 는 별개의 이벤트입니다. 리워드 지급은 반드시 `onUserEarnedReward()` 시점에 처리하세요.

---

## 코드 구현

#### Java
```java
public class RewardVideoActivity extends AppCompatActivity {

    // 정적 load() 콜백으로 받은 로드 완료 광고 객체
    private AMMRewardVideo loadedAd;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_reward_video);

        Button btnShow = findViewById(R.id.btn_show_reward);
        btnShow.setOnClickListener(v -> loadAndShowReward());
    }

    private void loadAndShowReward() {
        // S2S Reward Callback용 커스텀 파라미터 (선택사항)
        Map<String, String> customParams = new HashMap<>();
        customParams.put("user_id", "user123");     // 리워드 지급 대상 사용자 ID
        customParams.put("reward_type", "coin");     // 리워드 종류

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)  // S2S Callback 시 파라미터로 전달됨
            .setMute(false)                 // 동영상 음소거 여부 (false: 소리 켬)
            .build();

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체를 받는다
        AMMRewardVideo.loadAd(this, adInfo, new AMMRewardVideoLoadCallback() {
            @Override
            public void onSuccessLoadReward(@NonNull String adapterName, @NonNull AMMRewardVideo ad) {
                loadedAd = ad;

                // 노출/클릭/재생완료/닫힘은 FullScreenContentCallback으로 수신
                ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                    @Override public void onAdShowedFullScreenContent() { /* 노출됨 */ }
                    @Override public void onAdClicked() { /* 클릭 */ }
                    @Override public void onAdCompleted() { /* 동영상 재생 완료 (보상과 별개) */ }
                    @Override public void onAdDismissedFullScreenContent() {
                        loadedAd = null; // 닫힘 — 참조 해제
                    }
                    @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                        loadedAd = null;
                    }
                });

                // 노출 + 보상 리스너 등록 (GAM의 show(activity, OnUserEarnedRewardListener)와 동일)
                ad.show(RewardVideoActivity.this, new OnUserEarnedRewardListener() {
                    @Override public void onUserEarnedReward() {
                        // ✅ 리워드 지급 처리 — 사용자가 광고를 끝까지 시청함
                        giveRewardToUser();
                    }
                });
            }

            @Override
            public void onFailLoadReward(int errorCode, @Nullable String errorMsg) {
                // 광고 수신 실패
            }
        });
    }

    private void giveRewardToUser() {
        // 리워드 지급 로직 구현 (예: 코인 +100, 프리미엄 콘텐츠 해제 등)
    }

    @Override
    protected void onDestroy() {
        if (loadedAd != null) {
            loadedAd.stop();
            loadedAd = null;
        }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class RewardVideoActivity : AppCompatActivity() {

    private var loadedAd: AMMRewardVideo? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_reward_video)

        findViewById<Button>(R.id.btn_show_reward).setOnClickListener {
            loadAndShowReward()
        }
    }

    private fun loadAndShowReward() {
        val customParams = mapOf(
            "user_id" to "user123",
            "reward_type" to "coin"
        )

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)
            .setMute(false)
            .build()

        AMMRewardVideo.loadAd(this, adInfo, object : AMMRewardVideoLoadCallback() {
            override fun onSuccessLoadReward(adapterName: String, ad: AMMRewardVideo) {
                loadedAd = ad
                ad.fullScreenContentCallback = object : FullScreenContentCallback() {
                    override fun onAdShowedFullScreenContent() { /* 노출됨 */ }
                    override fun onAdClicked() { /* 클릭 */ }
                    override fun onAdCompleted() { /* 재생 완료 */ }
                    override fun onAdDismissedFullScreenContent() { loadedAd = null }
                    override fun onAdFailedToShowFullScreenContent(adError: AdError) { loadedAd = null }
                }
                // 노출 + 보상 리스너 등록
                ad.show(this@RewardVideoActivity, OnUserEarnedRewardListener {
                    giveRewardToUser() // 🎁 리워드 지급
                })
            }

            override fun onFailLoadReward(errorCode: Int, errorMsg: String?) {
                // 수신 실패
            }
        })
    }

    private fun giveRewardToUser() {
        // 리워드 지급 로직
    }

    override fun onDestroy() {
        loadedAd?.stop()
        loadedAd = null
        super.onDestroy()
    }
}
```

---

## FullScreenContentCallback 이벤트 (리워드)

| 콜백 | 발생 시점 |
|------|----------|
| `onAdShowedFullScreenContent()` | 광고가 화면에 표시됨 (임프레션) |
| `onAdClicked()` | 광고 내 링크/더보기 클릭 |
| `onAdCompleted()` | 동영상이 끝까지 재생됨 (**보상과 별개**) |
| `onAdDismissedFullScreenContent()` | 광고 창 닫힘 |
| `onAdFailedToShowFullScreenContent(AdError)` | 노출 실패 |

> 보상 적립은 `FullScreenContentCallback`이 아니라 **`show(activity, OnUserEarnedRewardListener)`** 의 `onUserEarnedReward()`로 수신합니다.

> ℹ️ **스킵(`onAdSkipped`)이 필요하면 `setAdListener`를 쓰세요.** `FullScreenContentCallback`은 GAM 표준 서브셋이라 스킵 콜백이 없습니다. 리워드 광고 객체는 `setAdListener(AdListener)`도 지원하며, 이 경로로 등록하면 `onAdSkipped`·`onAdCompleted`·`onAdDisplayed`·`onAdClicked`·`onAdClosed`를 받습니다.
> ```java
> ad.setAdListener(new AdListener() {
>     @Override public void onAdCompleted() { /* 재생 완료 (보상과 별개) */ }
>     @Override public void onAdSkipped()   { /* 사용자가 Skip 클릭 (보상 미적립) */ }
>     @Override public void onAdClosed()    { /* 닫힘 */ }
> });
> ```
> `setFullScreenContentCallback`과 `setAdListener`는 **동일 슬롯을 공유하므로 둘 중 하나만** 등록하세요. 보상 적립(`onUserEarnedReward`)은 어느 경로를 쓰든 `show(activity, OnUserEarnedRewardListener)`로 별도 수신합니다.

---

## S2S Reward Callback (서버 간 리워드 검증)

사용자가 리워드를 획득했을 때, 매체사 서버로 직접 콜백을 전송하는 기능입니다. 클라이언트 조작을 방지하고 리워드를 안전하게 지급할 때 사용합니다.

### 설정 방법

#### Step 1: 파트너 사이트에서 콜백 URL 등록

`파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정`에서 콜백 서버 URL을 입력합니다.

**기본 파라미터 (자동 포함):**

| 파라미터 | 설명 | 예시 값 |
|---------|------|---------|
| `media_key` | 미디어 키 | `12345678` |
| `adunit_id` | 애드유닛 ID | `87654321` |
| `adid` | 광고 ID (Android: GAID) | `860635ea-...` |
| `earnedreward` | 리워드 획득 이벤트 식별 | - |
| `timestamp` | 이벤트 발생 Unix 타임스탬프 | `1704067200` |

**콜백 URL 형식 예시:**
```
https://your-server.com/reward?media_key={mediakey}&adunit_id={adunitid}&adid={adid}&complete={complete}&timestamp={timestamp}
```

#### Step 2: 커스텀 파라미터 추가 (선택사항)

```java
Map<String, String> params = new HashMap<>();
params.put("user_id", "user123");       // 리워드 지급 대상 사용자 식별자
params.put("session_token", "abc123");  // 세션 검증 토큰

AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
    .setCustomParams(params)
    .build();
```

**커스텀 파라미터 포함 콜백 URL:**
```
https://your-server.com/reward?media_key={mediakey}&adunit_id={adunitid}&adid={adid}&complete={complete}&timestamp={timestamp}&user_id=user123&session_token=abc123
```

> ℹ️ 광고 네트워크별로 리워드 획득(`onUserEarnedReward`) 이벤트 발생 시점이 다를 수 있습니다. 서버 콜백이 클라이언트 콜백보다 먼저 또는 나중에 도달할 수 있으니, 서버에서 양쪽을 모두 처리하는 방식을 권장합니다.

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `setMute(boolean)` | `false` | 동영상 음소거 여부 |
| `setCustomParams(Map)` | `{}` | S2S Reward Callback 커스텀 파라미터 |
| `interstitialTimeout(int)` | `0` (서버 지정) | 로딩 타임아웃 (초) |

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| 화면 전환·백그라운드 (표시 광고 유지) | `loadedAd.cancelLoad()` | 진행 중 **로드만 취소** (표시 중이면 no-op) |
| `Activity.onDestroy()` | `loadedAd.stop()` | 광고 정지 및 리소스 해제 (리스너 참조도 함께 해제됨) |

---

## 지연 노출 (Load-Only)

미리 로드해 두었다가 원하는 시점에 노출하려면, `onSuccessLoadReward`에서 `show()`를 호출하지 말고 광고 객체만 보관한 뒤 나중에 `show(activity, listener)`를 호출하세요. `hasInterstitial` 필드로 노출 가능 여부를 확인할 수 있습니다.

```java
// 원하는 시점에 노출
if (loadedAd != null && loadedAd.hasInterstitial) {
    loadedAd.show(this, () -> giveRewardToUser());
}
```

---

## 구 API에서 전환 (리워드 동영상 · v1.x.x → v2)

구 `RewardInterstitialVideoAd` 클래스는 v2에서 제거되었습니다. 아래 매핑을 참고해 `AMMRewardVideo` 정적 `loadAd()`로 전환하세요. 전체 마이그레이션 절차는 [마이그레이션 가이드](migration.md)를 참고하세요.

| v1.x.x (제거됨) | v2.0.0 |
|---|---|
| `new RewardInterstitialVideoAd(context)` | (인스턴스 생성 불필요) `AMMRewardVideo.loadAd(context, adInfo, callback)` |
| `setListener(AdListener)` + `onReceivedAd` | `AMMRewardVideoLoadCallback.onSuccessLoadReward(adapterName, ad)` |
| `onFailedToReceiveAd(...)` | `onFailLoadReward(errorCode, errorMsg)` |
| `loadRewardVideoAd()` / `startRewardVideoAd()` | `AMMRewardVideo.loadAd(...)` (노출은 `ad.show(activity, listener)`) |
| `showRewardVideoAd()` | `ad.show(activity, OnUserEarnedRewardListener)` |
| `onEventAd(AdEvent.EARNEDREWARD)` | `OnUserEarnedRewardListener.onUserEarnedReward()` |
| `onEventAd(AdEvent.COMPLETION)` | `FullScreenContentCallback.onAdCompleted()` |
| `onEventAd(AdEvent.DISPLAYED / CLICK / CLOSE)` | `onAdShowedFullScreenContent()` / `onAdClicked()` / `onAdDismissedFullScreenContent()` |
| `stopRewardVideoAd()` | `stop()` |
