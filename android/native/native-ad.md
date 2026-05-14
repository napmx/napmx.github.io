# 네이티브 광고 (Android Native)

네이티브 광고는 `NativeAdView`를 사용합니다.  
앱의 UI 스타일에 맞게 자유롭게 레이아웃을 구성할 수 있습니다.

---

## 1. 광고 로드

```java
// Java
import com.nasmedia.admixerssp.ads.NativeAdView;
import com.nasmedia.admixerssp.ads.AdListener;
import com.nasmedia.admixerssp.ads.AdEvent;

public class MainActivity extends AppCompatActivity {

    private NativeAdView nativeAdView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        nativeAdView = new NativeAdView(this);
        nativeAdView.setAdUnitId("발급받은_ADUNIT_ID");
        nativeAdView.setAdListener(new AdListener() {
            @Override
            public void onReceivedAd(String adapterName, Object adView) {
                // adView를 원하는 컨테이너에 추가
                ViewGroup container = findViewById(R.id.nativeAdContainer);
                container.removeAllViews();
                if (adView instanceof View) {
                    container.addView((View) adView);
                }
            }

            @Override
            public void onFailedToReceiveAd(Object adView, String adapterName,
                                             int errorCode, String errorMsg) { }

            @Override
            public void onEventAd(Object adView, AdEvent event) { }
        });
        nativeAdView.loadAd();
    }
}
```

```kotlin
// Kotlin
import com.nasmedia.admixerssp.ads.NativeAdView

class MainActivity : AppCompatActivity() {

    private lateinit var nativeAdView: NativeAdView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        nativeAdView = NativeAdView(this)
        nativeAdView.setAdUnitId("발급받은_ADUNIT_ID")
        nativeAdView.setAdListener(object : AdListener {
            override fun onReceivedAd(adapterName: String, adView: Any) {
                val container = findViewById<ViewGroup>(R.id.nativeAdContainer)
                container.removeAllViews()
                if (adView is View) container.addView(adView)
            }

            override fun onFailedToReceiveAd(adView: Any, adapterName: String,
                                              errorCode: Int, errorMsg: String) { }

            override fun onEventAd(adView: Any, event: AdEvent) { }
        })
        nativeAdView.loadAd()
    }
}
```

---

## 2. 레이아웃 컨테이너

```xml
<!-- activity_main.xml -->
<FrameLayout
    android:id="@+id/nativeAdContainer"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

---

## 3. 생명주기 처리

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    if (nativeAdView != null) nativeAdView.destroy();
}
```
