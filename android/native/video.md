# 동영상 광고

> ℹ️ 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

> Interstitial Ad 광고 연동을 희망하시는 경우, [배너 - 전면 배너]를 연동해주세요.

| 포맷 | 클래스 | 설명 |
|------|--------|------|
| 인라인 동영상 | `AMMVideoView` | 앱 화면 내에 인라인으로 재생 |
| 전면 동영상 | `AMMVideoInterstitial` | 화면 전체를 덮는 전면 동영상 |

> ℹ️ 리워드 지급이 필요한 전면 동영상은 [리워드 동영상 광고](rewarded-video.md)를 참고하세요.

> ℹ️ **전면 동영상**은 `AMMVideoInterstitial`의 정적 `loadAd()` + `FullScreenContentCallback` 구조를 사용합니다. 기존 `InterstitialVideoAd` 클래스는 제거되었습니다 — `AMMVideoInterstitial`로 전환하세요.
>
> **인라인 동영상**은 `AMMVideoView`(구 `VideoAdView`, 제거됨)를 사용하며, 화면 내 View이므로 기존 `AdListener` 모델을 그대로 사용합니다.

---

## 인라인 동영상 광고 (AMMVideoView)

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

    private AMMVideoView videoAdView;
    private RelativeLayout container;
    private TextView tvComplete;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull AdNetworkType networkType, @NonNull Object adView) {
            // 광고 수신 성공 — 컨테이너에 추가
            // networkType로 switch: switch(networkType){ case PANGLE: ... }
            RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT
            );
            params.addRule(RelativeLayout.CENTER_IN_PARENT);
            container.removeView(videoAdView);
            container.addView(videoAdView, params);
        }

        @Override
        public void onFailedToReceiveAd(int errorCode, @Nullable String errorMsg) {
            // 광고 수신 실패
        }

        @Override
        public void onAdDisplayed() {
            // 광고 화면에 표시됨
        }

        @Override
        public void onAdCompleted() {
            // 동영상 재생 완료
            tvComplete.setVisibility(View.VISIBLE);
        }

        @Override
        public void onAdSkipped() {
            // 사용자가 Skip 클릭
        }

        @Override
        public void onAdClicked() {
            // 더보기 링크 클릭
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video);

        container = findViewById(R.id.container_video);
        tvComplete = findViewById(R.id.tv_complete);

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .build();

        videoAdView = new AMMVideoView(this);
        videoAdView.setAdInfo(adInfo);
        videoAdView.setAdViewListener(adListener);
        videoAdView.loadAd(); // 광고 로드 시작
    }

    @Override protected void onResume() { super.onResume(); if (videoAdView != null) videoAdView.onResume(); }
    @Override protected void onPause() { if (videoAdView != null) videoAdView.onPause(); super.onPause(); }
    @Override
    protected void onDestroy() {
        if (videoAdView != null) { videoAdView.stop(); videoAdView = null; }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class VideoAdActivity : AppCompatActivity() {

    private var videoAdView: AMMVideoView? = null
    private lateinit var container: RelativeLayout
    private lateinit var tvComplete: TextView

    private val adListener = object : AdListener() {
        override fun onReceivedAd(networkType: AdNetworkType, adView: Any) {
            val params = RelativeLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT
            ).apply { addRule(RelativeLayout.CENTER_IN_PARENT) }
            container.removeView(videoAdView)
            container.addView(videoAdView, params)
        }
        override fun onFailedToReceiveAd(errorCode: Int, errorMsg: String?) {
            // 광고 수신 실패
        }
        override fun onAdDisplayed() { /* 광고 표시됨 */ }
        override fun onAdCompleted() { tvComplete.visibility = View.VISIBLE }
        override fun onAdSkipped() { /* Skip됨 */ }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_video)

        container = findViewById(R.id.container_video)
        tvComplete = findViewById(R.id.tv_complete)

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .build()

        videoAdView = AMMVideoView(this).apply {
            setAdInfo(adInfo)
            setAdViewListener(adListener)
            loadAd()
        }
    }

    override fun onResume() { super.onResume(); videoAdView?.onResume() }
    override fun onPause() { videoAdView?.onPause(); super.onPause() }
    override fun onDestroy() {
        videoAdView?.stop(); videoAdView = null
        super.onDestroy()
    }
}
```

> ℹ️ 인라인 동영상(`AMMVideoView`)은 화면 내 View로 동작하므로 `AdListener`의 이름 있는 이벤트 콜백(`onAdDisplayed`/`onAdClicked`/`onAdCompleted`/`onAdSkipped` 등)을 사용합니다. 정적 `loadAd()` / `FullScreenContentCallback`은 전면(풀스크린) 포맷에만 적용됩니다.

---

## 수신 실패 콜백

로드 실패는 **`onFailedToReceiveAd(int errorCode, String errorMsg)`** 하나로 통지됩니다. 전 네트워크 No-Fill·SDK 미초기화·AdUnit 누락을 포함한 **모든 수신 실패**가 이 콜백으로 옵니다. 에러 코드 목록은 [에러 코드](error-codes.md)를 참고하세요.

---

## 전면 동영상 광고 (AMMVideoInterstitial)

화면 전체를 덮는 전면 동영상 광고를 표시합니다. 전면 광고와 동일한 정적 `loadAd()` + `FullScreenContentCallback` 구조입니다.

### 호출 흐름

```
AMMVideoInterstitial.loadAd(context, adInfo, callback)
    → onSuccessLoadVideoInterstitial(networkType, ad)  ← 로드된 광고 객체 전달
        → ad.setFullScreenContentCallback(...)         ← 노출/클릭/재생완료/닫힘 콜백
        → ad.show(activity)                            ← 노출 (Activity 필요)
    → onFailLoadVideoInterstitial(errorCode, errorMsg) ← 로드 실패
```

#### Java
```java
public class InterstitialVideoActivity extends AppCompatActivity {

    private AMMVideoInterstitial loadedAd;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitial_video);

        Button btnShow = findViewById(R.id.btn_show_video);
        btnShow.setOnClickListener(v -> loadAndShowVideo());
    }

    private void loadAndShowVideo() {
        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .interstitialTimeout(20)      // 타임아웃 (초, 0: 서버 지정)
            .build();

        AMMVideoInterstitial.loadAd(this, adInfo, new AMMVideoInterstitialLoadCallback() {
            @Override
            public void onSuccessLoadVideoInterstitial(@NonNull AdNetworkType networkType, @NonNull AMMVideoInterstitial ad) {
                loadedAd = ad;

                ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                    @Override public void onAdShowedFullScreenContent() { /* 노출됨 */ }
                    @Override public void onAdClicked() { /* 클릭 */ }
                    @Override public void onAdCompleted() { /* 동영상 재생 완료 */ }
                    @Override public void onAdDismissedFullScreenContent() {
                        loadedAd = null; // 닫힘
                    }
                    @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                        loadedAd = null;
                    }
                });

                ad.show(InterstitialVideoActivity.this);
            }

            @Override
            public void onFailLoadVideoInterstitial(int errorCode, @Nullable String errorMsg) {
                // 수신 실패
            }
        });
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
class InterstitialVideoActivity : AppCompatActivity() {

    private var loadedAd: AMMVideoInterstitial? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_interstitial_video)

        findViewById<Button>(R.id.btn_show_video).setOnClickListener {
            loadAndShowVideo()
        }
    }

    private fun loadAndShowVideo() {
        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_VIDEO)
            .interstitialTimeout(20)
            .build()

        AMMVideoInterstitial.loadAd(this, adInfo, object : AMMVideoInterstitialLoadCallback() {
            override fun onSuccessLoadVideoInterstitial(networkType: AdNetworkType, ad: AMMVideoInterstitial) {
                loadedAd = ad
                ad.fullScreenContentCallback = object : FullScreenContentCallback() {
                    override fun onAdShowedFullScreenContent() { /* 노출됨 */ }
                    override fun onAdClicked() { /* 클릭 */ }
                    override fun onAdCompleted() { /* 재생 완료 */ }
                    override fun onAdDismissedFullScreenContent() { loadedAd = null }
                    override fun onAdFailedToShowFullScreenContent(adError: AdError) { loadedAd = null }
                }
                ad.show(this@InterstitialVideoActivity)
            }

            override fun onFailLoadVideoInterstitial(errorCode: Int, errorMsg: String?) {
                // 수신 실패
            }
        })
    }

    override fun onDestroy() {
        loadedAd?.stop()
        loadedAd = null
        super.onDestroy()
    }
}
```

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `interstitialTimeout(int)` | `0` (서버 지정, 약 20초) | 로딩 타임아웃 (초) |

---

## 이벤트 레퍼런스

**인라인 동영상 (`AMMVideoView` — AdListener 콜백)**

| 콜백 | 발생 시점 |
|--------|----------|
| `onAdDisplayed()` | 광고가 화면에 표시됨 |
| `onAdCompleted()` | 동영상이 끝까지 재생됨 |
| `onAdSkipped()` | 사용자가 Skip 버튼 클릭 |
| `onAdClosed()` | 광고 창 닫힘 |
| `onAdClicked()` | 광고 내 링크(더보기 등) 클릭 |

**전면 동영상 (`AMMVideoInterstitial` — FullScreenContentCallback)**

| 콜백 | 발생 시점 |
|------|----------|
| `onAdShowedFullScreenContent()` | 광고가 화면에 표시됨 (임프레션) |
| `onAdClicked()` | 광고 내 링크 클릭 |
| `onAdCompleted()` | 동영상 재생 완료 |
| `onAdDismissedFullScreenContent()` | 광고 창 닫힘 |
| `onAdFailedToShowFullScreenContent(AdError)` | 노출 실패 |

> ℹ️ **스킵(`onAdSkipped`)이 필요하면 `setAdListener`를 쓰세요.** `FullScreenContentCallback`은 GAM 표준 서브셋이라 스킵 콜백이 없습니다. 표시·클릭·완료·닫힘(`onAdDisplayed`/`onAdClicked`/`onAdCompleted`/`onAdClosed`)은 `FullScreenContentCallback`으로도 받을 수 있으며, `setAdListener(AdListener)`로 등록하면 여기에 더해 **`onAdSkipped`(스킵)까지** 받습니다.
> ```java
> ad.setAdListener(new AdListener() {
>     @Override public void onAdDisplayed() { /* 노출됨 */ }
>     @Override public void onAdCompleted() { /* 재생 완료 */ }
>     @Override public void onAdSkipped()   { /* 사용자가 Skip 클릭 */ }
>     @Override public void onAdClicked()   { /* 클릭 */ }
>     @Override public void onAdClosed()    { /* 닫힘 */ }
> });
> ```
> `setFullScreenContentCallback`과 `setAdListener`는 **동일 슬롯을 공유하므로 둘 중 하나만** 등록하세요(GAM 스타일 네이밍이 필요하면 FSCC, 스킵이 필요하면 AdListener).

---

## 라이프사이클 관리

**인라인 동영상 (AMMVideoView)**

| Activity 메서드 | AMMVideoView 메서드 | 역할 |
|----------------|--------------------|------|
| `onResume()` | `videoAdView.onResume()` | 동영상 재생 재개 |
| `onPause()` | `videoAdView.onPause()` | 동영상 재생 일시 정지 |
| `onDestroy()` | `videoAdView.stop()` | 리소스 해제 (필수) |

**전면 동영상 (AMMVideoInterstitial)**

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| 화면 전환·백그라운드 (표시 광고 유지) | `loadedAd.cancelLoad()` | 진행 중 **로드만 취소** (표시 중이면 no-op) |
| `Activity.onDestroy()` | `loadedAd.stop()` | 광고 정지 및 리소스 해제 (필수) |

---

## 구 API에서 전환 (전면 동영상 · v1.x.x → v2)

구 `InterstitialVideoAd` 클래스는 v2에서 제거되었습니다. 아래 매핑을 참고해 `AMMVideoInterstitial` 정적 `loadAd()`로 전환하세요. 전체 마이그레이션 절차는 [마이그레이션 가이드](migration.md)를 참고하세요.

| v1.x.x (제거됨) | v2.0.0 |
|---|---|
| `new InterstitialVideoAd(context)` | (인스턴스 생성 불필요) `AMMVideoInterstitial.loadAd(context, adInfo, callback)` |
| `setListener(AdListener)` + `onReceivedAd` | `AMMVideoInterstitialLoadCallback.onSuccessLoadVideoInterstitial(networkType, ad)` |
| `onFailedToReceiveAd(...)` | `onFailLoadVideoInterstitial(errorCode, errorMsg)` |
| `loadInterstitialVideoAd()` / `startInterstitialVideoAd()` | `AMMVideoInterstitial.loadAd(...)` (노출은 `ad.show(activity)`) |
| `showInterstitialVideoAd()` / `showInterstitialVideoAd(activity)` | `ad.show(activity)` |
| `onEventAd(AdEvent.DISPLAYED / CLICK / COMPLETION / CLOSE)` | `onAdShowedFullScreenContent()` / `onAdClicked()` / `onAdCompleted()` / `onAdDismissedFullScreenContent()` |
| `closeInterstitialVideoAd()` (CLOSE/SKIPPED 시 필수) | 불필요 — `onAdDismissedFullScreenContent()`로 닫힘 수신 |
| `stopInterstitialVideoAd()` | `stop()` |
