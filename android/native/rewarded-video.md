# 리워드 동영상 광고

{% hint style="info" %}
리워드 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.
{% endhint %}

리워드 동영상 광고는 `RewardInterstitialVideoAd`를 사용합니다. 사용자가 동영상을 끝까지 시청하면 `EARNEDREWARD` 이벤트가 발생하며, 이 시점에 리워드를 지급하세요.

---

## 기본 흐름

```
loadRewardVideoAd()
    → onReceivedAd() 콜백
    → hasInterstitial == true 확인
    → showRewardVideoAd()
    → 사용자 시청 완료
    → onEventAd(EARNEDREWARD) ← 🎁 리워드 지급 시점
    → onEventAd(CLOSE or COMPLETION)
```

---

## 코드 구현

{% tabs %}
{% tab title="Java" %}
```java
public class RewardVideoActivity extends AppCompatActivity {

    private RewardInterstitialVideoAd rewardAd;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            // 광고 수신 성공 — 즉시 노출하거나 버튼 활성화
            // 즉시 노출:
            // if (rewardAd.hasInterstitial) rewardAd.showRewardVideoAd();
        }

        @Override
        public void onFailedToReceiveAd(@Nullable Object adView,
                @NonNull String adapterName, int errorCode, @Nullable String errorMsg) {
            // 광고 수신 실패
        }

        @Override
        public void onEventAd(@NonNull Object adView, @NonNull AdEvent event) {
            switch (event) {
                case EARNEDREWARD:
                    // ✅ 리워드 지급 처리 — 사용자가 광고를 끝까지 시청함
                    giveRewardToUser();
                    break;
                case COMPLETION:
                    // 동영상 재생 완료
                    break;
                case SKIPPED:
                    // 사용자가 Skip 버튼 클릭 (리워드 지급 안 함)
                    // 반드시 closeInterstitialVideoAd() 호출 — 호출하지 않으면 광고 화면이 닫히지 않음
                    if (rewardAd != null) rewardAd.closeInterstitialVideoAd();
                    break;
                case CLOSE:
                    // 광고 창 닫힘 (Skip 또는 완료 후)
                    // 반드시 closeInterstitialVideoAd() 호출 — 호출하지 않으면 광고 화면이 닫히지 않음
                    if (rewardAd != null) rewardAd.closeInterstitialVideoAd();
                    break;
                case CLICK:
                    // 광고 내 더보기/링크 클릭
                    break;
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_reward_video);

        // S2S Reward Callback용 커스텀 파라미터 (선택사항)
        Map<String, String> customParams = new HashMap<>();
        customParams.put("user_id", "user123");     // 리워드 지급 대상 사용자 ID
        customParams.put("reward_type", "coin");     // 리워드 종류

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)  // S2S Callback 시 파라미터로 전달됨
            .setMute(false)                 // 동영상 음소거 여부 (false: 소리 켬)
            .build();

        rewardAd = new RewardInterstitialVideoAd(this);
        rewardAd.setAdInfo(adInfo);
        rewardAd.setListener(adListener);
        rewardAd.loadRewardVideoAd(); // 광고 로드 시작

        // 광고 표시 버튼
        Button btnShow = findViewById(R.id.btn_show_reward);
        btnShow.setOnClickListener(v -> {
            if (rewardAd.hasInterstitial) {
                rewardAd.showRewardVideoAd();
            } else {
                rewardAd.loadRewardVideoAd(); // 재요청
            }
        });
    }

    private void giveRewardToUser() {
        // 리워드 지급 로직 구현
        // 예: 코인 +100, 프리미엄 콘텐츠 해제 등
    }

    @Override
    protected void onDestroy() {
        if (rewardAd != null) {
            rewardAd.stopRewardVideoAd();
            rewardAd = null;
        }
        super.onDestroy();
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class RewardVideoActivity : AppCompatActivity() {

    private var rewardAd: RewardInterstitialVideoAd? = null

    private val adListener = object : AdListener {
        override fun onReceivedAd(adapterName: String, adView: Any) {
            // 광고 수신 성공
        }
        override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                          errorCode: Int, errorMsg: String?) { }
        override fun onEventAd(adView: Any, event: AdEvent) {
            when (event) {
                AdEvent.EARNEDREWARD -> giveRewardToUser() // 🎁 리워드 지급
                AdEvent.COMPLETION -> { /* 시청 완료 */ }
                AdEvent.SKIPPED -> rewardAd?.closeInterstitialVideoAd() // 광고 화면 닫기 (필수)
                AdEvent.CLOSE -> rewardAd?.closeInterstitialVideoAd()   // 광고 화면 닫기 (필수)
                else -> {}
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_reward_video)

        val customParams = mapOf(
            "user_id" to "user123",
            "reward_type" to "coin"
        )

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)
            .setMute(false)
            .build()

        rewardAd = RewardInterstitialVideoAd(this).apply {
            setAdInfo(adInfo)
            setListener(adListener)
            loadRewardVideoAd()
        }

        findViewById<Button>(R.id.btn_show_reward).setOnClickListener {
            if (rewardAd?.hasInterstitial == true) {
                rewardAd?.showRewardVideoAd()
            } else {
                rewardAd?.loadRewardVideoAd()
            }
        }
    }

    private fun giveRewardToUser() {
        // 리워드 지급 로직
    }

    override fun onDestroy() {
        rewardAd?.stopRewardVideoAd()
        rewardAd = null
        super.onDestroy()
    }
}
```
{% endtab %}
{% endtabs %}

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

{% hint style="info" %}
광고 네트워크별로 `EARNEDREWARD` 이벤트 발생 시점이 다를 수 있습니다. 서버 콜백이 클라이언트 콜백보다 먼저 또는 나중에 도달할 수 있으니, 서버에서 양쪽을 모두 처리하는 방식을 권장합니다.
{% endhint %}

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `setMute(boolean)` | `false` | 동영상 음소거 여부 |
| `setCustomParams(Map)` | `{}` | S2S Reward Callback 커스텀 파라미터 |
| `interstitialTimeout(int)` | `0` (서버 지정) | 로딩 타임아웃 (초) |

---

## AdEvent 레퍼런스

| 이벤트 | 설명 | 리워드 지급 여부 |
|--------|------|----------------|
| `EARNEDREWARD` | 동영상 시청 완료 → **리워드 지급 시점** | ✅ 지급 |
| `COMPLETION` | 재생이 끝까지 완료됨 | 네트워크에 따라 상이 |
| `SKIPPED` | 사용자가 Skip 버튼 클릭 | ❌ 미지급 |
| `CLOSE` | 광고 창 닫힘 | - |
| `CLICK` | 광고 내 링크 클릭 | - |

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| `Activity.onDestroy()` | `rewardAd.stopRewardVideoAd()` | 광고 정지 및 리소스 해제 (리스너 참조도 함께 해제됨) |
| `CLOSE/SKIPPED 이벤트` | `rewardAd.closeInterstitialVideoAd()` | 광고 화면 닫기 (필수) |

{% hint style="warning" %}
`CLOSE` 또는 `SKIPPED` 이벤트 수신 시 반드시 `closeInterstitialVideoAd()`를 호출해야 합니다. 호출하지 않으면 광고 화면이 닫히지 않습니다.
{% endhint %}
