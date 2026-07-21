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
    → onSuccessLoadReward(networkType, ad)             ← 로드된 광고 객체 전달
        → ad.setFullScreenContentCallback(...)         ← 노출/클릭/재생완료/닫힘 콜백
        → ad.show(activity, onUserEarnedRewardListener) ← 노출 + 보상 리스너 등록
            → onUserEarnedReward()                     ← 🎁 리워드 지급 시점 (시청 완료)
    → onFailLoadReward(errorCode, errorMsg)            ← 로드 실패
```

> ℹ️ **보상 적립(`onUserEarnedReward`)** 과 **영상 재생 완료(`onAdCompleted`)** 는 별개의 이벤트입니다. 리워드 지급은 반드시 `onUserEarnedReward()` 시점에 처리하세요.

> ℹ️ **커스텀 어댑터를 쓰는 경우** — `AdMixer.registerAdapter(String)`로 등록한 어댑터는 `AdNetworkType`에 등재되어 있지 않아, 로드 **성공** 콜백이 `String adapterName` 오버로드로만 통지됩니다. 커스텀 어댑터를 사용한다면 그 오버로드도 함께 구현하세요. (로드 **실패** `onFailLoadReward`는 어댑터와 무관하게 항상 호출되므로 별도 조치가 필요 없습니다.)

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
        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setMute(false)                 // 동영상 음소거 여부 (false: 소리 켬)
            .build();

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체를 받는다
        AMMRewardVideo.loadAd(this, adInfo, new AMMRewardVideoLoadCallback() {
            @Override
            public void onSuccessLoadReward(@NonNull AdNetworkType networkType, @NonNull AMMRewardVideo ad) {
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
        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setMute(false)
            .build()

        AMMRewardVideo.loadAd(this, adInfo, object : AMMRewardVideoLoadCallback() {
            override fun onSuccessLoadReward(networkType: AdNetworkType, ad: AMMRewardVideo) {
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

> ℹ️ **표시 결과는 광고당 정확히 1회만 전달됩니다.** `show()` 이후 `onAdShowedFullScreenContent()`(표시됨) 또는 `onAdFailedToShowFullScreenContent()`(표시 실패) 중 하나만 최종 결과로 수신하며, 네트워크가 표시 결과를 통지하지 않으면 SDK가 표시 실패를 백스톱으로 전달합니다. 표시 실패가 전달된 뒤의 상반된 콜백(닫힘/스킵/표시됨/중복 실패)은 억제되어 이중·모순 콜백을 방어할 필요가 없습니다. **단, 보상 적립(`onUserEarnedReward`)은 유실 방지를 위해 예외적으로 항상 전달됩니다.**

> ℹ️ **스킵(`onAdSkipped`)이 필요하면 `setAdListener`를 쓰세요.** `FullScreenContentCallback`은 GAM 표준 서브셋이라 스킵 콜백이 없습니다. 표시·클릭·완료·닫힘(`onAdDisplayed`/`onAdClicked`/`onAdCompleted`/`onAdClosed`)은 `FullScreenContentCallback`으로도 받을 수 있으며, `setAdListener(AdListener)`로 등록하면 여기에 더해 **`onAdSkipped`(스킵)까지** 받습니다.
> ```java
> ad.setAdListener(new AdListener() {
>     @Override public void onAdCompleted() { /* 재생 완료 (보상과 별개) */ }
>     @Override public void onAdSkipped()   { /* 사용자가 Skip 클릭 (보상 미적립) */ }
>     @Override public void onAdClosed()    { /* 닫힘 */ }
> });
> ```
> `setFullScreenContentCallback`과 `setAdListener`는 **동일 슬롯을 공유하므로 둘 중 하나만** 등록하세요. 보상 적립(`onUserEarnedReward`)은 어느 경로를 쓰든 `show(activity, OnUserEarnedRewardListener)`로 별도 수신합니다.

> ℹ️ **지급 채널 상호배타** — `show(activity, OnUserEarnedRewardListener)`로 전용 보상 리스너를 등록하면 `AdListener.onAdRewarded()`는 호출되지 않습니다(전용 리스너 우선). 지급 통지는 어느 경로로든 **정확히 1회**입니다.

> ℹ️ **이벤트 순서 보장** — 네트워크마다 달랐던 보상/닫힘 순서를 SDK가 정규화하여, 앱은 항상 **`onUserEarnedReward` → `onAdDismissedFullScreenContent`(닫힘)** 순서로 수신합니다. 닫힘 콜백에서 보상 수신 여부를 안전하게 판정할 수 있습니다.

---

## 리워드 지급 식별자 RewardInfo (transaction_id)

보상 지급 1건마다 고유 `transaction_id`(UUID)가 발급되어 앱 콜백으로 전달됩니다. 지급 이력 기록·중복 지급 방지의 기준 키로 사용하세요.

앱에서 받으려면 `RewardInfo`를 인자로 받는 오버로드를 구현합니다. **오버로드는 선택 사항**이며, 구현하지 않으면 기존 무인자 `onUserEarnedReward()`가 그대로 호출됩니다(기존 코드 수정 불필요).

```java
ad.show(this, new OnUserEarnedRewardListener() {
    @Override
    public void onUserEarnedReward(@NonNull RewardInfo rewardInfo) {
        // 🎁 지급 처리 — transaction_id를 지급 이력 키로 기록
        giveRewardToUser(rewardInfo.getTransactionId());
    }
});
```

> ℹ️ 오버로드를 구현하면 무인자 `onUserEarnedReward()`는 호출되지 않습니다(지급 통지는 정확히 1회). `RewardInfo`는 보상 금액/타입을 담지 않습니다 — 네트워크별로 단위·의미가 달라 미디에이션에서 신뢰할 수 없기 때문입니다. 보상 내용은 애드유닛 정책으로 관리하세요.

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `setMute(boolean)` | `false` | 동영상 음소거 여부 |
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
| `setListener(AdListener)` + `onReceivedAd` | `AMMRewardVideoLoadCallback.onSuccessLoadReward(networkType, ad)` |
| `onFailedToReceiveAd(...)` | `onFailLoadReward(errorCode, errorMsg)` |
| `loadRewardVideoAd()` / `startRewardVideoAd()` | `AMMRewardVideo.loadAd(...)` (노출은 `ad.show(activity, listener)`) |
| `showRewardVideoAd()` | `ad.show(activity, OnUserEarnedRewardListener)` |
| `onEventAd(AdEvent.EARNEDREWARD)` | `OnUserEarnedRewardListener.onUserEarnedReward()` |
| `onEventAd(AdEvent.COMPLETION)` | `FullScreenContentCallback.onAdCompleted()` |
| `onEventAd(AdEvent.DISPLAYED / CLICK / CLOSE)` | `onAdShowedFullScreenContent()` / `onAdClicked()` / `onAdDismissedFullScreenContent()` |
| `stopRewardVideoAd()` | `stop()` |
