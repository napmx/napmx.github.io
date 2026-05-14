# 전면 광고 (Android Native)

전면 광고는 `InterstitialAd`를 사용합니다.  
`Activity` Context가 필요합니다.

---

## 1. 광고 로드 및 표시

```java
// Java
import com.nasmedia.admixerssp.ads.InterstitialAd;
import com.nasmedia.admixerssp.ads.AdListener;
import com.nasmedia.admixerssp.ads.AdEvent;

public class MainActivity extends AppCompatActivity {

    private InterstitialAd interstitialAd;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        interstitialAd = new InterstitialAd(this); // Activity context 필수
        interstitialAd.setAdUnitId("발급받은_ADUNIT_ID");
        interstitialAd.setAdListener(new AdListener() {
            @Override
            public void onReceivedAd(String adapterName, Object adView) {
                // 광고 수신 완료 → 표시
                interstitialAd.show();
            }

            @Override
            public void onFailedToReceiveAd(Object adView, String adapterName,
                                             int errorCode, String errorMsg) {
                // 광고 없음 또는 오류
            }

            @Override
            public void onEventAd(Object adView, AdEvent event) {
                // 광고 닫힘, 클릭 등
            }
        });
        interstitialAd.loadAd();
    }
}
```

```kotlin
// Kotlin
import com.nasmedia.admixerssp.ads.InterstitialAd

class MainActivity : AppCompatActivity() {

    private lateinit var interstitialAd: InterstitialAd

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        interstitialAd = InterstitialAd(this)
        interstitialAd.setAdUnitId("발급받은_ADUNIT_ID")
        interstitialAd.setAdListener(object : AdListener {
            override fun onReceivedAd(adapterName: String, adView: Any) {
                interstitialAd.show()
            }

            override fun onFailedToReceiveAd(adView: Any, adapterName: String,
                                              errorCode: Int, errorMsg: String) { }

            override fun onEventAd(adView: Any, event: AdEvent) { }
        })
        interstitialAd.loadAd()
    }
}
```

---

## 2. 미리 로드 후 원하는 시점에 표시

```java
// 앱 진입 시 미리 로드
interstitialAd = new InterstitialAd(this);
interstitialAd.setAdUnitId("발급받은_ADUNIT_ID");
interstitialAd.loadOnly(); // 표시하지 않고 로드만

// 원하는 시점에 표시
if (interstitialAd.hasInterstitial) {
    interstitialAd.show();
}
```

---

## 3. 생명주기 처리

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    if (interstitialAd != null) interstitialAd.destroy();
}
```

---

## 주의사항

- `InterstitialAd` 생성 시 반드시 `Activity` Context를 전달해야 합니다.
- `show()`는 `onReceivedAd` 콜백 이후 호출하거나, `hasInterstitial == true` 확인 후 호출합니다.
