# 동영상 광고

> ℹ️ 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

nap ssp SDK는 두 가지 동영상 광고 포맷을 지원합니다.

| 포맷 | 클래스 | 설명 |
|------|--------|------|
| 인라인 동영상 | `VideoAdView` | 앱 화면 내에 인라인으로 재생 |
| 전면 동영상 | `InterstitialVideoAd` | 화면 전체를 덮는 전면 동영상 |

> ℹ️ 리워드 지급이 필요한 전면 동영상은 [리워드 동영상 광고](rewarded-video.md)를 참고하세요.

---

## 인라인 동영상 광고 (VideoAdView)

앱 피드나 콘텐츠 사이에 인라인으로 동영상 광고를 표시합니다.

### 레이아웃 XML

**res/layout/activity_video.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- 동영상 광고가 삽입될 컨테이너 -->
    <RelativeLayout
        android:id="@+id/container_video"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000000">

        <!-- 재생 완료 후 표시할 UI (선택사항) -->
        <TextView
            android:id="@+id/tv_complete"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="광고 시청 완료"
            android:textColor="#FFFFFF"
            android:visibility="gone" />

    </RelativeLayout>

</RelativeLayout>
```

### 코드 구현

#### Java
```java
public class VideoAdActivity extends AppCompatActivity {

    private VideoAdView videoAdView;
    private RelativeLayout container;
    private TextView tvComplete;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            // 광고 수신 성공 — 컨테이너에 추가
            RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT
            );
            params.addRule(RelativeLayout.CENTER_IN_PARENT);
            container.removeView(videoAdView);
            container.addView(videoAdView, params);
        }

        @Override
        public void onFailedToReceiveAd(@Nullable Object adView,
                @NonNull String adapterName, int errorCode, @Nullable String errorMsg) {
            // 광고 수신 실패
        }

        @Override
        public void onEventAd(@NonNull Object adView, @NonNull AdEvent event) {
            switch (event) {
                case COMPLETION:
                    // 동영상 재생 완료
                    tvComplete.setVisibility(View.VISIBLE);
                    break;
                case SKIPPED:
                    // 사용자가 Skip 클릭
                    break;
                case CLICK:
                    // 더보기 링크 클릭
                    break;
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video);

        container = findViewById(R.id.container_video);
        tvComplete = findViewById(R.id.tv_complete);

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .isRetry(false) // 광고 없을 때 재요청 여부 (false: 1회 요청 후 바로 실패 콜백)
            .build();

        videoAdView = new VideoAdView(this);
        videoAdView.setAdInfo(adInfo);
        videoAdView.setAdViewListener(adListener);
        videoAdView.loadAd(); // 광고 로드 시작
    }

    @Override protected void onResume() { super.onResume(); if (videoAdView != null) videoAdView.onResume(); }
    @Override protected void onPause() { if (videoAdView != null) videoAdView.onPause(); super.onPause(); }
    @Override
    protected void onDestroy() {
        if (videoAdView != null) { videoAdView.destroy(); videoAdView = null; }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class VideoAdActivity : AppCompatActivity() {

    private var videoAdView: VideoAdView? = null
    private lateinit var container: RelativeLayout
    private lateinit var tvComplete: TextView

    private val adListener = object : AdListener {
        override fun onReceivedAd(adapterName: String, adView: Any) {
            val params = RelativeLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT
            ).apply { addRule(RelativeLayout.CENTER_IN_PARENT) }
            container.removeView(videoAdView)
            container.addView(videoAdView, params)
        }
        override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                          errorCode: Int, errorMsg: String?) { }
        override fun onEventAd(adView: Any, event: AdEvent) {
            when (event) {
                AdEvent.COMPLETION -> tvComplete.visibility = View.VISIBLE
                AdEvent.SKIPPED -> { /* Skip됨 */ }
                else -> {}
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_video)

        container = findViewById(R.id.container_video)
        tvComplete = findViewById(R.id.tv_complete)

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .isRetry(false)
            .build()

        videoAdView = VideoAdView(this).apply {
            setAdInfo(adInfo)
            setAdViewListener(adListener)
            loadAd()
        }
    }

    override fun onResume() { super.onResume(); videoAdView?.onResume() }
    override fun onPause() { videoAdView?.onPause(); super.onPause() }
    override fun onDestroy() {
        videoAdView?.destroy(); videoAdView = null
        super.onDestroy()
    }
}
```

---

## 전면 동영상 광고 (InterstitialVideoAd)

화면 전체를 덮는 전면 동영상 광고를 표시합니다.

#### Java
```java
public class InterstitialVideoActivity extends AppCompatActivity {

    private InterstitialVideoAd interstitialVideoAd;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            // 광고 수신 성공 — 즉시 노출 또는 버튼 활성화
        }

        @Override
        public void onFailedToReceiveAd(@Nullable Object adView,
                @NonNull String adapterName, int errorCode, @Nullable String errorMsg) { }

        @Override
        public void onEventAd(@NonNull Object adView, @NonNull AdEvent event) {
            switch (event) {
                case CLOSE:
                case SKIPPED:
                    // 광고 닫힘 또는 Skip — 반드시 closeInterstitialVideoAd() 호출
                    if (interstitialVideoAd != null) {
                        interstitialVideoAd.closeInterstitialVideoAd();
                    }
                    break;
                case COMPLETION:
                    // 동영상 재생 완료
                    break;
                case CLICK:
                    // 광고 클릭
                    break;
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitial_video);

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .interstitialTimeout(20)      // 타임아웃 (초, 0: 서버 지정)
            .maxRetryCountInSlot(-1)      // 재시도 횟수 (-1: 무제한)
            .build();

        interstitialVideoAd = new InterstitialVideoAd(this);
        interstitialVideoAd.setAdInfo(adInfo);
        interstitialVideoAd.setListener(adListener);
        interstitialVideoAd.loadInterstitialVideoAd();

        Button btnShow = findViewById(R.id.btn_show_video);
        btnShow.setOnClickListener(v -> {
            if (interstitialVideoAd.hasInterstitial) {
                interstitialVideoAd.showInterstitialVideoAd();
            } else {
                interstitialVideoAd.loadInterstitialVideoAd();
            }
        });
    }

    @Override
    protected void onDestroy() {
        if (interstitialVideoAd != null) {
            interstitialVideoAd.stopInterstitialVideoAd();
            interstitialVideoAd = null;
        }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class InterstitialVideoActivity : AppCompatActivity() {

    private var interstitialVideoAd: InterstitialVideoAd? = null

    private val adListener = object : AdListener {
        override fun onReceivedAd(adapterName: String, adView: Any) { }
        override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                          errorCode: Int, errorMsg: String?) { }
        override fun onEventAd(adView: Any, event: AdEvent) {
            when (event) {
                AdEvent.CLOSE, AdEvent.SKIPPED -> {
                    interstitialVideoAd?.closeInterstitialVideoAd()
                }
                AdEvent.COMPLETION -> { /* 재생 완료 */ }
                else -> {}
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_interstitial_video)

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .interstitialTimeout(20)
            .maxRetryCountInSlot(-1)
            .build()

        interstitialVideoAd = InterstitialVideoAd(this).apply {
            setAdInfo(adInfo)
            setListener(adListener)
            loadInterstitialVideoAd()
        }

        findViewById<Button>(R.id.btn_show_video).setOnClickListener {
            if (interstitialVideoAd?.hasInterstitial == true) {
                interstitialVideoAd?.showInterstitialVideoAd()
            } else {
                interstitialVideoAd?.loadInterstitialVideoAd()
            }
        }
    }

    override fun onDestroy() {
        interstitialVideoAd?.stopInterstitialVideoAd()
        interstitialVideoAd = null
        super.onDestroy()
    }
}
```

> ⚠️ `CLOSE` 또는 `SKIPPED` 이벤트 수신 시 반드시 `closeInterstitialVideoAd()`를 호출해야 합니다. 호출하지 않으면 광고 화면이 닫히지 않습니다.

---

## 뒤로가기(BACK) 키 정책 (전면 동영상)

> ⚠️ **v2.0.0**: 전면 동영상 광고는 시스템 **뒤로가기(BACK) 키를 기본 차단**합니다(스킵·조기 종료 방지, 닫기는 닫기 버튼 전용). 인라인 동영상(`VideoAdView`)은 화면 내 View이므로 해당하지 않습니다.
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
| `isRetry(boolean)` | `true` | 광고 없을 때 재요청 여부 (인라인 동영상) |
| `interstitialTimeout(int)` | `0` (서버 지정, 약 20초) | 로딩 타임아웃 (초) |
| `maxRetryCountInSlot(int)` | `-1` | 재시도 횟수 (`-1` 또는 `0`: 무제한, 양수: 해당 횟수까지) |
| `setDisableBackKey(boolean)` | `true` (차단) | **전면 동영상** 뒤로가기 닫기 차단 여부. `false` 설정 시에만 BACK으로 닫기 허용 |

---

## AdEvent 레퍼런스

| 이벤트 | 발생 시점 |
|--------|----------|
| `COMPLETION` | 동영상이 끝까지 재생됨 |
| `SKIPPED` | 사용자가 Skip 버튼 클릭 |
| `CLOSE` | 광고 창 닫힘 |
| `CLICK` | 광고 내 링크(더보기 등) 클릭 |

---

## 라이프사이클 관리

**인라인 동영상 (VideoAdView)**

| Activity 메서드 | VideoAdView 메서드 | 역할 |
|----------------|--------------------|------|
| `onResume()` | `videoAdView.onResume()` | 동영상 재생 재개 |
| `onPause()` | `videoAdView.onPause()` | 동영상 재생 일시 정지 |
| `onDestroy()` | `videoAdView.destroy()` | 리소스 해제 (필수) |

**전면 동영상 (InterstitialVideoAd)**

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| `Activity.onDestroy()` | `interstitialVideoAd.stopInterstitialVideoAd()` | 광고 정지 및 리소스 해제 |
| `CLOSE/SKIPPED 이벤트` | `interstitialVideoAd.closeInterstitialVideoAd()` | 광고 화면 닫기 (필수) |
