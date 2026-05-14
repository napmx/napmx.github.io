# 배너 광고 (Android Native)

배너 광고는 `AdView`를 사용합니다.

---

## 1. 레이아웃 XML에 AdView 추가

```xml
<!-- activity_main.xml -->
<com.nasmedia.admixerssp.ads.AdView
    android:id="@+id/adView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

---

## 2. 광고 로드

```java
// Java
import com.nasmedia.admixerssp.ads.AdView;
import com.nasmedia.admixerssp.ads.AdListener;
import com.nasmedia.admixerssp.ads.AdEvent;

public class MainActivity extends AppCompatActivity {

    private AdView adView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        adView = findViewById(R.id.adView);
        adView.setAdUnitId("발급받은_ADUNIT_ID");
        adView.setAdListener(new AdListener() {
            @Override
            public void onReceivedAd(String adapterName, Object adView) {
                // 광고 수신 성공
            }

            @Override
            public void onFailedToReceiveAd(Object adView, String adapterName,
                                             int errorCode, String errorMsg) {
                // 광고 수신 실패
                // errorCode: AX_ERR_NO_ADS, AX_ERR_TIMEOUT 등
            }

            @Override
            public void onEventAd(Object adView, AdEvent event) {
                // 광고 클릭 등 이벤트
            }
        });
        adView.loadAd();
    }
}
```

```kotlin
// Kotlin
import com.nasmedia.admixerssp.ads.AdView
import com.nasmedia.admixerssp.ads.AdListener
import com.nasmedia.admixerssp.ads.AdEvent

class MainActivity : AppCompatActivity() {

    private lateinit var adView: AdView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        adView = findViewById(R.id.adView)
        adView.setAdUnitId("발급받은_ADUNIT_ID")
        adView.setAdListener(object : AdListener {
            override fun onReceivedAd(adapterName: String, adView: Any) {
                // 광고 수신 성공
            }

            override fun onFailedToReceiveAd(adView: Any, adapterName: String,
                                              errorCode: Int, errorMsg: String) {
                // 광고 수신 실패
            }

            override fun onEventAd(adView: Any, event: AdEvent) {
                // 광고 이벤트
            }
        })
        adView.loadAd()
    }
}
```

---

## 3. 생명주기 처리

Activity/Fragment 생명주기에 맞게 AdView를 관리합니다.

```java
@Override
protected void onResume() {
    super.onResume();
    if (adView != null) adView.onResume();
}

@Override
protected void onPause() {
    super.onPause();
    if (adView != null) adView.onPause();
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (adView != null) adView.destroy();
}
```

---

## 4. 자동 새로고침

파트너 사이트에서 Adunit 설정 시 새로고침 주기를 설정하면 자동으로 적용됩니다.

---

## 에러 코드

| 상수 | 값 | 설명 |
|------|----|------|
| `AX_ERR_INIT` | `0x80000001` | 초기화 오류 |
| `AX_ERR_ADUNIT` | `0x80000002` | Adunit 오류 |
| `AX_ERR_HTTP` | `0x80000003` | 네트워크 통신 오류 |
| `AX_ERR_TIMEOUT` | `0x80000004` | 타임아웃 |
| `AX_ERR_NO_ADAPTER` | `0x80000005` | 어댑터 없음 |
| `AX_ERR_ADAPTER` | `0x80000006` | 어댑터 오류 |
| `AX_ERR_CONFIG_FAIL` | `0x80000007` | 설정 파일 로드 실패 |
| `AX_ERR_NO_ADS` | `0x80000008` | 광고 없음 (No Fill) |
