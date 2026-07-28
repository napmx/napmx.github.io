# 리워드 동영상 광고

> ℹ️ 리워드 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

리워드 동영상 광고는 `AMMRewardVideo`를 사용합니다. 네트워크가 정한 보상 조건을 충족하면 **`OnUserEarnedRewardListener.onUserEarnedReward()`** 콜백이 호출되며, 이 시점에 리워드를 지급하세요. (보통 시청 완료 시점이지만, 네트워크에 따라 완료 전에 도착할 수도 있습니다 — [§2-1](#reward-flag-pattern) 참조.)

> ℹ️ 리워드 광고는 **정적 `loadAd()` → 로드된 광고 객체 반환 → `show(activity, OnUserEarnedRewardListener)`** 구조를 사용합니다. 노출/클릭/닫힘은 `FullScreenContentCallback`으로, 보상 적립은 `OnUserEarnedRewardListener`로 수신합니다.

---

## 기본 흐름

```
AMMRewardVideo.loadAd(context, adInfo, callback)
    → onSuccessLoadReward(networkType, ad)             ← 로드된 광고 객체 전달
        → ad.setFullScreenContentCallback(...)         ← 노출/클릭/재생완료/닫힘 콜백
        → ad.show(activity, onUserEarnedRewardListener) ← 노출 + 보상 리스너 등록
            → onUserEarnedReward()                     ← 🎁 리워드 지급 시점 (보상 조건 충족)
    → onFailLoadReward(errorCode, errorMsg)            ← 로드 실패
```

> ℹ️ **보상 적립(`onUserEarnedReward`)** 과 **영상 재생 완료(`onAdCompleted`)** 는 의미가 다른 별개의 신호입니다. 리워드 지급은 반드시 `onUserEarnedReward()` 시점에 처리하세요.
>
> ⚠️ **`onAdCompleted()`는 네트워크에 따라 발화하지 않을 수 있습니다.** 재생 완료를 별도 신호로 통지할지 여부는 각 네트워크 SDK가 결정하며, SDK는 이를 합성하지 않습니다([§3-1](#reward-close-order)). 지급 여부나 완주 판정을 `onAdCompleted()` 에 의존하면 낙찰 네트워크에 따라 동작이 달라지므로, **지급은 `onUserEarnedReward()` 단일 채널로만** 배선하세요.

---

## 코드 구현

#### Java
```java
public class RewardVideoActivity extends AppCompatActivity {

    // 정적 load() 콜백으로 받은 로드 완료 광고 객체
    private AMMRewardVideo loadedAd;

    // ── 노출 1회당 상태를 담는 플래그 2개 ─────────────────────────
    // 보상과 닫힘 콜백이 어떤 순서로 도착할지는 네트워크 정책에 따라 달라질 수 있습니다.
    // 순서를 가정하지 않고, 이 두 플래그로 지급·알림 시점을 판정합니다. (§2-1 참조)
    private boolean rewardGranted = false; // 보상이 지급되었는가
    private boolean adDismissed   = false; // 광고가 닫혀 앱으로 돌아왔는가

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
        customParams.put("user_id", "user123");      // 리워드 지급 대상 사용자 ID
        customParams.put("reward_type", "coin");     // 리워드 종류

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)  // S2S Callback 시 파라미터로 전달됨
            .setMute(false)                 // 음소거 요청 힌트 (false: 소리 켬) — 네트워크에 따라 미적용될 수 있음
            .build();

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체를 받는다
        AMMRewardVideo.loadAd(this, adInfo, new AMMRewardVideoLoadCallback() {
            @Override
            public void onSuccessLoadReward(@NonNull AdNetworkType networkType, @NonNull AMMRewardVideo ad) {
                loadedAd = ad;
                showRewardAd(ad);
            }

            @Override
            public void onFailLoadReward(int errorCode, @Nullable String errorMsg) {
                // 광고 수신 실패
            }
        });
    }

    /** 로드된 광고를 노출한다. */
    private void showRewardAd(@NonNull AMMRewardVideo ad) {
        // ✅ show() 직전에 반드시 초기화 — 노출 1회 = 지급 1회
        rewardGranted = false;
        adDismissed   = false;

        // 노출/클릭/재생완료/닫힘은 FullScreenContentCallback으로 수신
        ad.setFullScreenContentCallback(new FullScreenContentCallback() {
            @Override public void onAdShowedFullScreenContent() { /* 노출됨 */ }
            @Override public void onAdClicked() { /* 클릭 */ }
            @Override public void onAdCompleted() {
                // 재생 완료 — 보상과 별개의 신호이며, 네트워크 정책에 따라 발화하지 않을 수 있습니다.
                // 지급 판정에 사용하지 마세요.
            }

            @Override public void onAdDismissedFullScreenContent() {
                adDismissed = true;
                // 보상이 이미 도착해 있으면 지금 알림
                if (rewardGranted) showRewardToast();
                loadedAd = null; // 닫힘 — 참조 해제
            }

            @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                loadedAd = null;
            }
        });

        // 노출 + 보상 리스너 등록 (GAM의 show(activity, OnUserEarnedRewardListener)와 동일)
        ad.show(this, new OnUserEarnedRewardListener() {
            @Override public void onUserEarnedReward() {
                // 무인자 버전이 추상 메서드라 구현이 필요합니다.
                // SDK는 RewardInfo 오버로드만 호출하므로 이 경로는 실행되지 않습니다.
            }

            @Override public void onUserEarnedReward(@NonNull RewardInfo info) {
                if (rewardGranted) return;   // 중복 도착 방어 — 지급은 1회만
                rewardGranted = true;

                // ✅ 지급은 지금 즉시 (유실 방지)
                giveRewardToUser(info.getTransactionId());

                // 닫힘이 이미 지나갔다면(= 보상이 늦게 도착) 지금 알림
                if (adDismissed) showRewardToast();
                // ❌ 여기서 무조건 Toast 를 띄우지 마세요 — 광고 화면에 가려질 수 있습니다.
            }
        });
    }

    private void giveRewardToUser(String transactionId) {
        // 리워드 지급 로직 구현 (예: 코인 +100, 프리미엄 콘텐츠 해제 등)
        // transactionId 를 지급 이력 키로 기록해 두면 서버 포스트백과 대조할 수 있습니다.
    }

    private void showRewardToast() {
        Toast.makeText(this, "리워드가 지급되었습니다.", Toast.LENGTH_SHORT).show();
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

    // ── 노출 1회당 상태를 담는 플래그 2개 ─────────────────────────
    // 보상과 닫힘 콜백이 어떤 순서로 도착할지는 네트워크 정책에 따라 달라질 수 있습니다.
    // 순서를 가정하지 않고, 이 두 플래그로 지급·알림 시점을 판정합니다. (§2-1 참조)
    private var rewardGranted = false // 보상이 지급되었는가
    private var adDismissed   = false // 광고가 닫혀 앱으로 돌아왔는가

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
            override fun onSuccessLoadReward(networkType: AdNetworkType, ad: AMMRewardVideo) {
                loadedAd = ad
                showRewardAd(ad)
            }

            override fun onFailLoadReward(errorCode: Int, errorMsg: String?) {
                // 수신 실패
            }
        })
    }

    /** 로드된 광고를 노출한다. */
    private fun showRewardAd(ad: AMMRewardVideo) {
        // ✅ show() 직전에 반드시 초기화 — 노출 1회 = 지급 1회
        rewardGranted = false
        adDismissed   = false

        ad.fullScreenContentCallback = object : FullScreenContentCallback() {
            override fun onAdShowedFullScreenContent() { /* 노출됨 */ }
            override fun onAdClicked() { /* 클릭 */ }
            override fun onAdCompleted() {
                // 재생 완료 — 보상과 별개의 신호이며, 네트워크 정책에 따라 발화하지 않을 수 있습니다.
                // 지급 판정에 사용하지 마세요.
            }

            override fun onAdDismissedFullScreenContent() {
                adDismissed = true
                if (rewardGranted) showRewardToast()  // 보상이 이미 도착 → 지금 알림
                loadedAd = null
            }

            override fun onAdFailedToShowFullScreenContent(adError: AdError) {
                loadedAd = null
            }
        }

        // 노출 + 보상 리스너 등록
        // ℹ️ transactionId 가 필요 없으면 SAM 람다로 간단히 쓸 수 있습니다:
        //    ad.show(this) { /* 지급 */ }   ← 무인자 onUserEarnedReward() 에 바인딩됩니다.
        ad.show(this, object : OnUserEarnedRewardListener {
            override fun onUserEarnedReward() {
                // RewardInfo 오버로드를 구현했으므로 이 경로는 호출되지 않습니다.
            }

            override fun onUserEarnedReward(info: RewardInfo) {
                if (rewardGranted) return   // 중복 도착 방어 — 지급은 1회만
                rewardGranted = true

                giveRewardToUser(info.transactionId)  // ✅ 지급은 지금 즉시 (유실 방지)

                if (adDismissed) showRewardToast()    // 닫힘이 이미 지나갔다면 지금 알림
                // ❌ 여기서 무조건 Toast 를 띄우지 마세요 — 광고 화면에 가려질 수 있습니다.
            }
        })
    }

    private fun giveRewardToUser(transactionId: String) {
        // 리워드 지급 로직 (transactionId 를 지급 이력 키로 기록하면 서버 포스트백과 대조 가능)
    }

    private fun showRewardToast() {
        Toast.makeText(this, "리워드가 지급되었습니다.", Toast.LENGTH_SHORT).show()
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
| `onAdCompleted()` | 동영상이 끝까지 재생됨 (**보상과 별개** · 네트워크에 따라 발화하지 않을 수 있음) |
| `onAdDismissedFullScreenContent()` | 광고 창 닫힘 |
| `onAdFailedToShowFullScreenContent(AdError)` | 노출 실패 |

> 보상 적립은 `FullScreenContentCallback`이 아니라 **`show(activity, OnUserEarnedRewardListener)`** 의 `onUserEarnedReward()`로 수신합니다.

> ℹ️ **표시 결과는 광고당 정확히 1회만 전달됩니다.** `show()` 이후 `onAdShowedFullScreenContent()`(표시됨) 또는 `onAdFailedToShowFullScreenContent()`(표시 실패) 중 하나만 최종 결과로 수신하며, 네트워크가 표시 결과를 통지하지 않으면 SDK가 표시 실패를 백스톱으로 전달합니다. 표시 실패가 전달된 뒤의 상반된 콜백(닫힘/스킵/표시됨/중복 실패)은 억제되어 이중·모순 콜백을 방어할 필요가 없습니다. **단, 보상 적립(`onUserEarnedReward`)은 유실 방지를 위해 예외적으로 항상 전달됩니다.**

> ℹ️ **스킵(`onAdSkipped`)이 필요하면 `setAdListener`를 쓰세요.** `FullScreenContentCallback`은 GAM 표준 서브셋이라 스킵 콜백이 없습니다. 표시·클릭·완료·닫힘은 `FullScreenContentCallback`으로도 받을 수 있으며(메서드명이 다릅니다 — `onAdShowedFullScreenContent`/`onAdClicked`/`onAdCompleted`/`onAdDismissedFullScreenContent`), `setAdListener(AdListener)`로 등록하면 여기에 더해 **`onAdSkipped`(스킵)까지** 받습니다.
> ```java
> ad.setAdListener(new AdListener() {
>     @Override public void onAdCompleted() { /* 재생 완료 — 보상과 별개, 네트워크에 따라 미발화 */ }
>     @Override public void onAdSkipped()   { /* 사용자가 Skip 클릭 (보상 미적립) */ }
>     @Override public void onAdClosed()    { /* 닫힘 */ }
> });
> ```
> `setFullScreenContentCallback`과 `setAdListener`는 **동일 슬롯을 공유하므로 둘 중 하나만** 등록하세요. 보상 적립(`onUserEarnedReward`)은 어느 경로를 쓰든 `show(activity, OnUserEarnedRewardListener)`로 별도 수신합니다.

---

## 보상 지급 안전 수칙

리워드 지급 누락·중복은 대부분 아래 다섯 가지에서 발생합니다. 지급 로직을 배선하기 전에 반드시 확인하세요.

<a id="reward-single-channel"></a>
### 1. 지급 채널은 하나만 사용

보상 적립은 `OnUserEarnedRewardListener.onUserEarnedReward()` 와 `AdListener.onAdRewarded()` 두 경로로 수신할 수 있지만, **지급 처리는 반드시 한 채널에서만** 하세요. 권장 채널은 `show(activity, OnUserEarnedRewardListener)`의 `onUserEarnedReward()`입니다.

> ℹ️ **SDK 2.1.1부터** 전용 리스너(`OnUserEarnedRewardListener`)를 등록하면 `onAdRewarded()`는 호출되지 않습니다(전용 리스너 우선, 지급 콜백은 정확히 1회). **2.1.0 이하 버전에서는 두 채널이 동시에 호출되어** 양쪽에 지급 로직을 배선하면 같은 보상이 두 번 지급될 수 있었으므로, 구버전 지원 기간에는 단일 채널 원칙을 반드시 지키세요.

### 2. 앱측 one-shot 가드

네트워크·타이밍에 따라 보상 콜백이 예상보다 늦게 또는 중복으로 도달하는 상황을 방어하려면, 노출 1회당 지급 1회를 앱에서 보장하세요.

> 💡 아래는 **지급 중복만 막는 최소 형태**입니다. 사용자 알림(Toast/다이얼로그)까지 다루려면 플래그를 하나 더 두어야 합니다 — 위 [코드 구현](#코드-구현) 예제와 [§2-1](#reward-flag-pattern)이 완성형입니다.

```java
private boolean rewardGranted = false; // show() 호출 직전에 false로 초기화

ad.show(this, new OnUserEarnedRewardListener() {
    @Override public void onUserEarnedReward() {
        if (rewardGranted) return; // 이미 지급됨 — 무시
        rewardGranted = true;
        giveRewardToUser();
    }
});
```

<a id="reward-flag-pattern"></a>
### 2-1. 지급 시점과 사용자 알림(UI) 시점을 분리하세요 — 순서 대신 **플래그로 판정**

> **핵심**: 보상 **지급**은 `onUserEarnedReward()`에서 즉시 하고, 사용자에게 보여주는 **알림(Toast/다이얼로그)** 은 광고가 닫혀 앱으로 돌아온 뒤에 표시합니다.
> 이때 두 콜백의 **도착 순서를 가정하지 말고, 공유 플래그로 판정**하세요.

**이유** — 보상 콜백은 **영상이 끝나기 전에**(또는 광고가 여러 개 이어지는 경우 중간에) 도착할 수 있습니다. 네트워크가 "보상 조건 충족"으로 판단한 시점에 `onUserEarnedReward()`가 오기 때문입니다. 이때는 아직 **네트워크의 전체 화면 광고가 떠 있어**, `onUserEarnedReward()`에서 곧바로 Toast를 띄우면 **광고 화면에 가려져 사용자가 보지 못합니다.**

한편 **보상과 닫힘 중 무엇이 먼저 도착할지는 네트워크와 상황에 따라 달라질 수 있습니다**([§3-1](#reward-close-order)). 따라서 "닫힘 시점에는 보상이 이미 도착해 있다"고 전제하면 알림이 누락될 수 있습니다. 아래처럼 **두 플래그를 두고, 나중에 도착한 쪽이 알림을 띄우도록** 작성하면 순서와 무관하게 성립합니다.

```java
private boolean rewardGranted = false; // 보상 지급 완료 여부
private boolean adDismissed   = false; // 광고 닫힘(앱 복귀) 여부

/** 로드된 광고를 노출한다. onSuccessLoadReward() 등에서 호출. */
private void showRewardAd(@NonNull AMMRewardVideo ad) {
    // ✅ show() 직전에 반드시 초기화 — 노출 1회 = 지급 1회
    rewardGranted = false;
    adDismissed   = false;

    ad.setFullScreenContentCallback(new FullScreenContentCallback() {
        @Override public void onAdDismissedFullScreenContent() {
            adDismissed = true;
            // 보상이 이미 도착해 있으면 지금 알림
            if (rewardGranted) showRewardToast();
            loadedAd = null;
        }
    });

    ad.show(this, new OnUserEarnedRewardListener() {
        @Override public void onUserEarnedReward() {
            // 무인자 버전이 추상 메서드라 구현이 필요합니다. SDK는 RewardInfo 오버로드만 호출합니다.
        }

        @Override public void onUserEarnedReward(@NonNull RewardInfo info) {
            if (rewardGranted) return;                    // 중복 도착 방어
            rewardGranted = true;
            giveRewardToUser(info.getTransactionId());    // ✅ 지급은 즉시 (유실 방지)
            // 닫힘이 이미 지나갔다면(= 보상이 늦게 도착) 지금 알림
            if (adDismissed) showRewardToast();
            // ❌ 광고가 아직 떠 있는 상태에서의 Toast 는 가려지므로 여기서 무조건 띄우지 않습니다
        }
    });
}
```

> ℹ️ **이 패턴을 쓰는 이유** — 광고 이벤트를 언제·어떤 순서로 보낼지는 **각 광고 네트워크 SDK가 결정**하며, 미디에이션은 이를 전달받아 중계하는 위치에 있습니다([§3-1](#reward-close-order)). **순서를 전제하지 않는 플래그 방식**이면 어느 네트워크가 낙찰되든 동일하게 동작합니다.
>
> ⚠️ **광고 화면 위에는 알림을 띄울 수 없습니다.** 전체 화면 광고는 네트워크 SDK가 소유하므로, 그 위에 앱이 UI를 겹칠 방법이 없습니다. "지급됐다는 안내가 광고에 묻힌다"는 현상은 결함이 아니라 이 구조 때문이며, 위처럼 알림을 닫힘 이후로 옮기면 해소됩니다.
>
> 🔎 "영상을 끝까지 봐야만 지급"처럼 지급 시점 자체를 완주로 강제하려면, 이는 앱 코드가 아니라 **네트워크 대시보드의 리워드 정책(reward on completion)** 으로 설정합니다.

### 3. 리소스 해제(`stop()`)는 최종 종료 시점에

리소스 해제(`stop()`)의 **가장 안전한 위치는 `Activity.onDestroy()`** 입니다. 닫힘 콜백에서는 참조만 정리(`= null`)하고, 실제 `stop()`은 화면 종료 시점에 맡기세요.

```java
@Override public void onAdDismissedFullScreenContent() {
    loadedAd = null;        // 참조만 해제 (권장)
}

@Override protected void onDestroy() {
    if (loadedAd != null) {
        loadedAd.stop();    // 최종 해제 — stop() 먼저, null 나중 (순서 중요: 반대로 하면 NPE)
        loadedAd = null;
    }
    super.onDestroy();
}
```

> ⚠️ **닫힘 콜백 안에서 `stop()`을 호출하면, 뒤이어 도착하는 보상 콜백을 받지 못할 수 있습니다.**
> 보상과 닫힘의 도착 순서는 네트워크에 따라 다르므로(아래 [보상·닫힘 이벤트 순서](#reward-close-order) 참조), 닫힘 시점에 광고를 파기하면 아직 도착하지 않은 보상을 놓칠 수 있습니다.
> 위처럼 `stop()`을 `onDestroy()`로 미루면 이 구간 자체가 사라집니다.

<a id="reward-close-order"></a>
### 3-1. 보상·닫힘 이벤트 순서

보상(earn)·재생 완료(complete)·닫힘(close) 이벤트를 **언제, 어떤 순서로 보낼지는 각 광고 네트워크 SDK가 결정**합니다. 미디에이션 SDK는 이를 전달받아 앱에 중계하는 위치에 있습니다.

따라서 **보상과 닫힘 중 어느 쪽이 먼저 도착할지는 네트워크와 상황에 따라 달라질 수 있습니다.** 특정 순서를 전제하지 마세요.

| 상황 | 앱이 받는 콜백 |
|---|---|
| 보상 조건을 충족한 경우 | 보상·닫힘 모두 도착 (도착 순서는 네트워크에 따라 다름) |
| 보상 조건을 충족하지 못한 경우 (미완주·스킵·표시 실패) | 닫힘만 도착 — 보상 콜백 없음 |

**SDK가 보장하는 것**

- 지급 통지는 노출 1회당 **정확히 1회**입니다([§1](#reward-single-channel)).
- 네트워크가 보낸 보상 콜백을 SDK가 무기한 지연하거나 유실하지 않습니다.
- **네트워크가 보내지 않은 이벤트를 SDK가 합성하지 않습니다.** 닫힘·재생 완료를 지어내지 않습니다. 특히 재생 완료는 '완주'라는 사실이고 보상은 '보상 조건 충족'이라 의미가 달라, 합성하면 매체의 완주율 지표가 왜곡됩니다.

> ⚠️ **콜백 도착 순서를 전제로 지급·UI 로직을 작성하지 마세요.**
> 지급 여부는 **순서가 아니라 플래그로 판정**하세요([§2-1](#reward-flag-pattern)).

> ⚠️ 닫힘 이후에 보상이 도착하는 동안 앱이 광고를 파기(`stop()`/`onDestroy`)하면 **클라이언트 보상 콜백을 받지 못할 수 있습니다.** 서버 지급까지 보장해야 한다면 서버 검증(S2S)을 병행하세요(§5).

### 4. 화면 회전 대응

화면 회전 시 Activity가 재생성되면서 로드해 둔 광고 객체(`loadedAd`)가 유실될 수 있습니다. 리워드 화면에서는 아래 중 하나로 대응하세요.

- `AndroidManifest.xml`의 해당 Activity에 `android:configChanges="orientation|screenSize|screenLayout|keyboardHidden"`을 선언해 재생성을 막거나,
- 재생성을 허용한다면 `onCreate()`에서 광고를 다시 로드하도록 구현하세요. (로드 완료 전 노출 버튼은 비활성화 권장)

### 5. 서버 검증은 선택 사항

아래의 [S2S Reward Callback](#s2s-reward-callback-서버-간-리워드-검증)(`callback_url` 포스트백)은 **선택 기능**입니다.

- **사용하는 매체**: 매체 서버에서 `transaction_id` 기반 멱등(idempotent) 처리를 권장합니다 — `(adunit_id, transaction_id)` 중복 수신 시 무적립 처리.
- **사용하지 않는 매체**: 클라이언트 보상 콜백(`onUserEarnedReward`) + 앱 서버 자체 지급 원장으로 중복·누락을 관리하세요.

---

## S2S Reward Callback (서버 간 리워드 검증)

사용자가 리워드를 획득했을 때, 매체사 서버로 직접 콜백(포스트백)을 전송하는 기능입니다.

> ℹ️ 클라이언트 보상 콜백(`onUserEarnedReward`)은 **참고용 UI 트리거**로 활용하고, 실제 재화 지급은 네트워크 사 서버와 연동되는 **S2S 포스트백 검증과 함께 처리하는 방식을 권장**합니다. 앱과 서버 중 어느 경로를 지급 기준으로 삼을지는 매체 정책에 따라 선택하세요.

### 설정 방법

#### Step 1: 파트너 사이트에서 콜백 URL 등록

`파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정`에서 콜백 서버 URL을 입력합니다. 리워드 획득 시 SDK가 등록된 URL에 아래 파라미터를 쿼리 스트링으로 자동으로 붙여 호출합니다.

**① 항상 포함되는 파라미터**

| 파라미터 | 설명 | 예시 값 |
|---------|------|---------|
| `media_key` | 미디어 키 | `12345678` |
| `adunit_id` | 애드유닛 ID | `87654321` |
| `timestamp` | 포스트백 전송 시각, Unix 타임스탬프 (**밀리초**) | `1704067200000` |
| `transaction_id` | 리워드 지급 1건당 고유값(지급당 UUID). 매체 서버는 `(adunit_id, transaction_id)` 기준 중복 수신 시 무적립 처리 권장 | `f47ac10b-58cc-...` |

**② 조건에 따라 포함되는 파라미터**

`ifa`와 `ifa_use`는 **오지 않을 수 있습니다.** 매체 서버는 두 값이 **없는 경우에도 정상 동작**하도록 구현하세요.

| 파라미터 | 설명 | 포함 조건 | 예시 값 |
|---------|------|-----------|---------|
| `ifa` | 광고 ID (Android: GAID) | 광고 ID를 획득했고, **아동 대상(COPPA)으로 설정되지 않은** 경우에만 포함 | `860635ea-...` |
| `ifa_use` | 광고 추적 허용 여부. `1`=허용, `0`=제한(opt-out) | **`ifa`가 포함될 때만** 함께 포함 | `1` |

`ifa`가 **생략되는 경우**는 두 가지입니다.

1. 광고 ID를 얻지 못한 경우 (Google Play 서비스 미탑재·조회 실패 등)
2. `AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE)` 로 **아동 대상 설정**을 켠 경우

**③ 매체가 넣은 커스텀 파라미터**

`AdInfo.Builder.setCustomParams()` 에 담은 항목이 그대로 쿼리 스트링에 붙습니다(아래 **Step 2** 참조).

> ℹ️ `transaction_id`는 **SDK 2.1.1부터** 전송되는 파라미터입니다. 그 이전 버전은 전송하지 않으므로, 매체 서버는 파라미터 부재 시에도 동작하도록 처리하세요.

> 🔁 **포스트백은 실패 시 자동 재시도됩니다** — 최초 1회 + 2초·8초 후 최대 2회. **재시도는 동일한 `transaction_id`로 전송**되므로, 매체 서버가 `transaction_id`를 UNIQUE 키로 멱등 처리하면 중복 적립이 발생하지 않습니다. (앱 프로세스가 종료되면 대기 중이던 재시도는 유실됩니다.)

### 클라이언트 콜백과 포스트백 대조

`transaction_id`는 **앱이 받는 클라이언트 콜백에도 동일한 값이 전달**되므로, 두 경로를 짝지을 수 있습니다.

```java
rewardVideo.show(activity, new OnUserEarnedRewardListener() {
    @Override
    public void onUserEarnedReward() {
        // 기존 방식 — 그대로 동작합니다.
    }

    @Override
    public void onUserEarnedReward(@NonNull RewardInfo info) {
        // SDK 2.1.1+ — 서버 포스트백의 transaction_id 와 동일한 값
        String txId = info.getTransactionId();
        myAppServer.grantReward(txId);   // 서버에서 txId 로 중복 제거
    }
});
```

> ℹ️ **두 메서드는 함께 호출되지 않습니다.** SDK는 `RewardInfo` 오버로드만 호출하며, 재정의하지 않은 경우 기본 구현이 무인자 버전으로 위임합니다. 따라서 **기존 코드는 수정 없이 그대로 동작**하고, 지급 콜백은 어느 경우든 정확히 1회입니다.
>
> `AdListener`를 쓰는 경우에도 동일한 `onAdRewarded(RewardInfo)` 오버로드가 제공됩니다.
>
> ⚠️ `RewardInfo`는 **보상 금액/타입을 담지 않습니다.** 네트워크마다 amount의 단위·의미가 달라 미디에이션 계층에서 신뢰할 수 있는 값이 아니기 때문입니다. 실제 보상량은 매체/서버 정책으로 결정하세요.
>
> ⚠️ `transaction_id`는 **SDK 예약 키**입니다. `setCustomParams()`에 동일한 키를 넣어도 SDK 값으로 대체됩니다.
> 사용자가 광고 추적을 제한(opt-out)한 경우, `ifa` 자리에 0으로 채워진 값(`00000000-0000-0000-0000-000000000000`)이 올 수 있습니다. 이때 `ifa_use=0` 이 함께 전송되므로, 수신 서버는 **`ifa` 값 자체가 아니라 `ifa_use` 로 제한 여부를 판별**하세요.
>
> ⚠️ `setCustomParams()`의 항목 중 **값이 빈 문자열인 것은 전송되지 않습니다.** 키만 있고 값이 없는 파라미터는 콜백 URL에 포함되지 않으니 주의하세요.

**매체 서버가 수신하는 콜백 URL 예시:**

광고 ID를 정상적으로 얻은 경우 —
```
https://your-server.com/reward?media_key=12345678&adunit_id=87654321&ifa=860635ea-...&ifa_use=1&timestamp=1704067200000&transaction_id=f47ac10b-58cc-...
```

광고 추적을 제한(opt-out)한 사용자 — `ifa`는 0으로 채워진 값이 오고 `ifa_use=0` —
```
https://your-server.com/reward?media_key=12345678&adunit_id=87654321&ifa=00000000-0000-0000-0000-000000000000&ifa_use=0&timestamp=1704067200000&transaction_id=f47ac10b-58cc-...
```

광고 ID를 얻지 못했거나 아동 대상(COPPA) 설정인 경우 — **`ifa`·`ifa_use` 자체가 없습니다** —
```
https://your-server.com/reward?media_key=12345678&adunit_id=87654321&timestamp=1704067200000&transaction_id=f47ac10b-58cc-...
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

**커스텀 파라미터 포함 콜백 URL 예시:**
```
https://your-server.com/reward?media_key=12345678&adunit_id=87654321&ifa=860635ea-...&ifa_use=1&timestamp=1704067200000&transaction_id=f47ac10b-58cc-...&user_id=user123&session_token=abc123
```

> ℹ️ `setCustomParams(Map)`에 넣은 키·값은 위와 같이 기본 파라미터 뒤에 그대로 추가됩니다(키 또는 값이 비어 있는 항목은 제외).

> ℹ️ 광고 네트워크별로 리워드 획득(`onUserEarnedReward`) 이벤트 발생 시점이 다를 수 있습니다. 서버 콜백이 클라이언트 콜백보다 먼저 또는 나중에 도달할 수 있으니, 서버에서 양쪽을 모두 처리하는 방식을 권장합니다.

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `setMute(boolean)` | `false` | 동영상을 음소거로 시작할지에 대한 **요청 힌트**. 네트워크 사 SDK의 지원 여부와 대시보드 정책에 따라 적용되지 않을 수 있습니다 |
| `setCustomParams(Map)` | `{}` | S2S Reward Callback 커스텀 파라미터 |

> ℹ️ **닫기 버튼·시청 시간·뒤로 가기 처리는 낙찰된 네트워크 SDK가 담당합니다.** 뒤로 가기 차단 옵션 `setDisableBackKey(boolean)`(기본 `true` = 차단)은 **AdMixer가 직접 렌더링하는 리워드 광고에만** 적용되며, 다른 네트워크가 낙찰되면 해당 네트워크 SDK 정책을 따릅니다. 전면 광고용 `setCloseButtonBound(int)`는 리워드 동영상에 반영되지 않습니다. 옵션 전체 목록은 [API 레퍼런스](api-reference.md)를 참고하세요.

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| 화면 전환·백그라운드 (표시 광고 유지) | `loadedAd.cancelLoad()` | 진행 중 **로드만 취소** (표시 중이면 no-op) |
| `Activity.onDestroy()` | `loadedAd.stop()` | 광고 정지 및 리소스 해제 (리스너 참조도 함께 해제됨) |

---

## 지연 노출 (Load-Only)

미리 로드해 두었다가 원하는 시점에 노출하려면, `onSuccessLoadReward`에서 `show()`를 호출하지 말고 광고 객체만 보관한 뒤 나중에 `show(activity, listener)`를 호출하세요. `isReady()`로 노출 가능 여부를 확인할 수 있습니다(**v2.1.3+**).

> ℹ️ **`isReady()` / `isLoading()`** — `loadAd()`는 이미 준비된 광고를 보호하고 중복 요청을 막기 위해 재요청을 **콜백 없이 무시**할 수 있습니다. 요청 전에 상태를 확인하세요 — `isReady()`가 `true`면 재로드 대신 `show()`, `isLoading()`이 `true`면 진행 중인 로드의 콜백을 기다리면 됩니다. `stop()` 이후에는 둘 다 `false`입니다.
>
> 구버전 호환 필드 `hasInterstitial`도 남아 있으나 신규 코드는 `isReady()`를 사용하세요.

```java
// 원하는 시점에 노출
if (loadedAd != null && loadedAd.isReady()) {
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
