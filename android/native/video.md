# 동영상 광고 (Android Native)

동영상 광고는 `VideoAdView`를 사용합니다.

---

## 1. 레이아웃 XML에 VideoAdView 추가

```xml
<com.nasmedia.admixerssp.ads.VideoAdView
    android:id="@+id/videoAdView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

---

## 2. 광고 로드

```java
// Java
import com.nasmedia.admixerssp.ads.VideoAdView;
import com.nasmedia.admixerssp.ads.AdListener;
import com.nasmedia.admixerssp.ads.AdEvent;

public class MainActivity extends AppCompatActivity {

    private VideoAdView videoAdView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        videoAdView = findViewById(R.id.videoAdView);
        videoAdView.setAdUnitId("발급받은_ADUNIT_ID");
        videoAdView.setAdListener(new AdListener() {
            @Override
            public void onReceivedAd(String adapterName, Object adView) {
                // 동영상 광고 수신 성공
            }

            @Override
            public void onFailedToReceiveAd(Object adView, String adapterName,
                                             int errorCode, String errorMsg) { }

            @Override
            public void onEventAd(Object adView, AdEvent event) {
                // AdEvent.VIDEO_START, AdEvent.VIDEO_COMPLETE 등
            }
        });
        videoAdView.loadAd();
    }
}
```

---

## 3. 생명주기 처리

```java
@Override
protected void onResume() {
    super.onResume();
    if (videoAdView != null) videoAdView.onResume();
}

@Override
protected void onPause() {
    super.onPause();
    if (videoAdView != null) videoAdView.onPause();
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (videoAdView != null) videoAdView.destroy();
}
```
