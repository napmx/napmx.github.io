# 리워드 동영상 광고

> ℹ️ 리워드 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

리워드 동영상 광고는 `AMMRewardVideo`를 사용합니다. 정적 `load()`로 로드한 뒤, `show(activity, OnUserEarnedRewardListener)`로 노출합니다. 사용자가 동영상을 끝까지 시청하면 등록한 리스너의 `onUserEarnedReward()`가 호출되며, 이 시점에 리워드를 지급하세요.

---

## 기본 흐름

```
AMMRewardVideo.load(context, adInfo, callback)
    → onSuccessLoadReward(adapterName, ad) 콜백
    → ad.setFullScreenContentCallback(...)   // 노출/클릭/완료/닫힘 수신
    → ad.show(activity, OnUserEarnedRewardListener)
    → 사용자 시청 완료
    → onUserEarnedReward()                    ← 🎁 리워드 지급 시점
    → onAdDismissedFullScreenContent()        ← 광고 닫힘
```

---

## 코드 구현

#### Java
```java
public class RewardVideoActivity extends AppCompatActivity {

    private AMMRewardVideo rewardAd; // load() 콜백으로 받은 로드 완료 광고

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_reward_video);

        // 화면 진입 시 미리 로드
        loadRewardVideo();

        // 광고 표시 버튼
        Button btnShow = findViewById(R.id.btn_show_reward);
        btnShow.setOnClickListener(v -> {
            if (rewardAd != null && rewardAd.hasInterstitial) {
                showRewardAd(rewardAd);
            } else {
                loadRewardVideo(); // 아직 로드 전이면 재요청
            }
        });
    }

    private void loadRewardVideo() {
        // S2S Reward Callback용 커스텀 파라미터 (선택사항)
        Map<String, String> customParams = new HashMap<>();
        customParams.put("user_id", "user123");     // 리워드 지급 대상 사용자 ID
        customParams.put("reward_type", "coin");     // 리워드 종류

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)  // S2S Callback 시 파라미터로 전달됨
            .setMute(false)                 // 동영상 음소거 여부 (false: 소리 켬)
            .build();

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체 전달
        AMMRewardVideo.load(this, adInfo, new AMMRewardVideoLoadCallback() {
            @Override
            public void onSuccessLoadReward(@NonNull String adapterName, @NonNull AMMRewardVideo ad) {
                // 광고 수신 성공 — 노출/클릭/완료/닫힘은 FullScreenContentCallback로 수신
                rewardAd = ad;
                ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                    @Override public void onAdShowedFullScreenContent() { /* 노출됨 */ }
                    @Override public void onAdClicked() { /* 더보기/링크 클릭 */ }
                    @Override public void onAdCompleted() { /* 동영상 재생 완료 */ }
                    @Override public void onAdDismissedFullScreenContent() {
                        rewardAd = null; // 광고 닫힘 — 객체 정리
                    }
                    @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                        rewardAd = null; // 노출 단계 실패
                    }
                });
            }

            @Override
            public void onFailLoadReward(int errorCode, String errorMsg) {
                rewardAd = null; // 광고 수신 실패
            }
        });
    }

    // show() 시점에 OnUserEarnedRewardListener를 등록 → 시청 완료 시 보상 지급
    private void showRewardAd(AMMRewardVideo ad) {
        ad.show(this, new OnUserEarnedRewardListener() {
            @Override
            public void onUserEarnedReward() {
                // ✅ 리워드 지급 처리 — 사용자가 광고를 끝까지 시청함
                giveRewardToUser();
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

#### Kotlin
```kotlin
class RewardVideoActivity : AppCompatActivity() {

    private var rewardAd: AMMRewardVideo? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_reward_video)

        loadRewardVideo()

        findViewById<Button>(R.id.btn_show_reward).setOnClickListener {
            val ad = rewardAd
            if (ad != null && ad.hasInterstitial) {
                showRewardAd(ad)
            } else {
                loadRewardVideo() // 재요청
            }
        }
    }

    private fun loadRewardVideo() {
        val customParams = mapOf(
            "user_id" to "user123",
            "reward_type" to "coin"
        )

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_REWARD_VIDEO)
            .setCustomParams(customParams)
            .setMute(false)
            .build()

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체 전달
        AMMRewardVideo.load(this, adInfo, object : AMMRewardVideoLoadCallback() {
            override fun onSuccessLoadReward(adapterName: String, ad: AMMRewardVideo) {
                rewardAd = ad
                ad.setFullScreenContentCallback(object : FullScreenContentCallback() {
                    override fun onAdShowedFullScreenContent() { /* 노출됨 */ }
                    override fun onAdClicked() { /* 클릭 */ }
                    override fun onAdCompleted() { /* 재생 완료 */ }
                    override fun onAdDismissedFullScreenContent() { rewardAd = null }
                    override fun onAdFailedToShowFullScreenContent(adError: AdError) { rewardAd = null }
                })
            }

            override fun onFailLoadReward(errorCode: Int, errorMsg: String?) {
                rewardAd = null
            }
        })
    }

    // show() 시점에 OnUserEarnedRewardListener 등록 → 시청 완료 시 보상 지급
    private fun showRewardAd(ad: AMMRewardVideo) {
        ad.show(this, OnUserEarnedRewardListener {
            // 🎁 리워드 지급 — 사용자가 광고를 끝까지 시청함
            giveRewardToUser()
        })
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

> ℹ️ 광고 네트워크별로 보상 적립(`onUserEarnedReward`) 시점이 다를 수 있습니다. 서버 콜백이 클라이언트 콜백보다 먼저 또는 나중에 도달할 수 있으니, 서버에서 양쪽을 모두 처리하는 방식을 권장합니다.

---

## 뒤로가기(BACK) 키 정책

> ⚠️ **v2.0.0**: 리워드 광고는 시스템 **뒤로가기(BACK) 키를 기본 차단**합니다(시청 도중 스킵·조기 종료 방지, 닫기는 닫기 버튼 전용). 보상은 `onUserEarnedReward()` 시점에만 지급되므로, BACK 차단 여부와 무관하게 보상 무결성은 항상 유지됩니다.
>
> 뒤로가기로 닫기를 허용하려면 `AdInfo`에서 명시적으로 해제하세요:
> ```java
> AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
>         .setDisableBackKey(false) // 명시적 false → 뒤로가기로 닫기 허용
>         .build();
> ```
>
> ℹ️ Android 13(API 33)+ 예측형 뒤로가기(predictive back)가 켜진 앱(예: `targetSdk 35`)에서도 위 차단이 정상 적용됩니다.

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `setMute(boolean)` | `false` | 동영상 음소거 여부 |
| `setCustomParams(Map)` | `{}` | S2S Reward Callback 커스텀 파라미터 |
| `interstitialTimeout(int)` | `0` (서버 지정) | 로딩 타임아웃 (초) |
| `setDisableBackKey(boolean)` | `true` (차단) | 리워드 광고 뒤로가기 닫기 차단 여부. `false` 설정 시에만 BACK으로 닫기 허용 |

---

## 이벤트 콜백 레퍼런스

**보상 적립** — `show(activity, OnUserEarnedRewardListener)`로 등록한 리스너로 수신

| 콜백 | 설명 | 리워드 지급 |
|------|------|------------|
| `onUserEarnedReward()` | 동영상 시청 완료 → **리워드 지급 시점** | ✅ 지급 |

**노출/클릭/완료/닫힘** — `FullScreenContentCallback`로 수신

| 콜백 | 발생 시점 |
|------|----------|
| `onAdShowedFullScreenContent()` | 광고가 풀스크린으로 표시됨 |
| `onAdClicked()` | 광고 내 링크/더보기 클릭 |
| `onAdCompleted()` | 동영상 재생 완료 |
| `onAdDismissedFullScreenContent()` | 광고 닫힘 |
| `onAdFailedToShowFullScreenContent(AdError)` | 노출 단계 실패 |

> ℹ️ 보상은 `onUserEarnedReward()`에서만 지급하세요. `onAdCompleted()`(재생 완료)는 보상 적립과 별개의 이벤트입니다.

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| 화면 전환·백그라운드 (표시 광고 유지) | `rewardAd.cancelLoad()` | 진행 중 **로드만 취소** (표시 중이면 no-op) |
| `Activity.onDestroy()` | `rewardAd.stopRewardVideoAd()` | 광고 정지 및 리소스 해제 (리스너 참조도 함께 해제됨) |

> ℹ️ 광고 닫힘은 `FullScreenContentCallback.onAdDismissedFullScreenContent()`로 통지되며 SDK가 자동 처리합니다. v1.x의 `closeInterstitialVideoAd()` 수동 호출은 더 이상 필요하지 않습니다.
