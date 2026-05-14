# 리워드 동영상 (Android Native)

리워드 동영상 광고는 `RewardInterstitialVideoAd`를 사용합니다.  
사용자가 동영상을 끝까지 시청하면 보상을 지급할 수 있습니다.

---

## 1. 광고 로드 및 표시

```java
// Java
import com.nasmedia.admixerssp.ads.RewardInterstitialVideoAd;
import com.nasmedia.admixerssp.ads.AdListener;
import com.nasmedia.admixerssp.ads.AdEvent;

public class MainActivity extends AppCompatActivity {

    private RewardInterstitialVideoAd rewardAd;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        rewardAd = new RewardInterstitialVideoAd(this); // Activity context 필수
        rewardAd.setAdUnitId("발급받은_ADUNIT_ID");
        rewardAd.setAdListener(new AdListener() {
            @Override
            public void onReceivedAd(String adapterName, Object adView) {
                // 광고 준비 완료 → 표시
                rewardAd.show();
            }

            @Override
            public void onFailedToReceiveAd(Object adView, String adapterName,
                                             int errorCode, String errorMsg) { }

            @Override
            public void onEventAd(Object adView, AdEvent event) {
                if (event == AdEvent.VIDEO_COMPLETE) {
                    // 리워드 지급 처리
                    grantReward();
                }
            }
        });
        rewardAd.loadAd();
    }

    private void grantReward() {
        // 앱 내 보상 지급 로직
    }
}
```

```kotlin
// Kotlin
import com.nasmedia.admixerssp.ads.RewardInterstitialVideoAd
import com.nasmedia.admixerssp.ads.AdEvent

class MainActivity : AppCompatActivity() {

    private lateinit var rewardAd: RewardInterstitialVideoAd

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        rewardAd = RewardInterstitialVideoAd(this)
        rewardAd.setAdUnitId("발급받은_ADUNIT_ID")
        rewardAd.setAdListener(object : AdListener {
            override fun onReceivedAd(adapterName: String, adView: Any) {
                rewardAd.show()
            }

            override fun onFailedToReceiveAd(adView: Any, adapterName: String,
                                              errorCode: Int, errorMsg: String) { }

            override fun onEventAd(adView: Any, event: AdEvent) {
                if (event == AdEvent.VIDEO_COMPLETE) {
                    grantReward()
                }
            }
        })
        rewardAd.loadAd()
    }

    private fun grantReward() { }
}
```

---

## 2. 리워드 이벤트 처리

`onEventAd`의 `AdEvent`를 통해 동영상 상태를 추적합니다.

| AdEvent | 설명 |
|---------|------|
| `VIDEO_START` | 동영상 재생 시작 |
| `VIDEO_COMPLETE` | 동영상 완료 (리워드 지급 시점) |
| `AD_CLOSED` | 광고 닫힘 |

---

## 3. 생명주기 처리

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    if (rewardAd != null) rewardAd.destroy();
}
```

---

## 주의사항

- `RewardInterstitialVideoAd` 생성 시 반드시 `Activity` Context를 전달해야 합니다.
- 리워드는 `VIDEO_COMPLETE` 이벤트 수신 시 지급합니다. `AD_CLOSED` 만으로 판단하지 마세요.
