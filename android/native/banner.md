# 배너 광고

> ℹ️ 배너 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

배너 광고는 `AdView`를 사용하여 화면 내에 고정 크기의 광고를 표시합니다.

---

## 노출 방식

배너 광고는 두 가지 방식으로 노출할 수 있습니다.

| 방식 | 설명 | 권장 사용 시나리오 |
|------|------|--------------------|
| **즉시 노출** | `addView()` 후 광고 수신 시 자동 표시 | 광고 영역이 항상 고정된 경우 |
| **지연 노출** | `loadAd()` → `onReceivedAd` 후 `showAd()` 호출 | 특정 시점에 광고를 노출해야 하는 경우 |

---

## 방법 1: 코드로 추가 (즉시 노출)

#### Java
```java
public class BannerActivity extends AppCompatActivity {

    private RelativeLayout container;
    private AdView adView;
    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            // 광고 수신 성공 — addView 이후 자동 표시됨
        }

        @Override
        public void onFailedToReceiveAd(@Nullable Object adView,
                @NonNull String adapterName, int errorCode, @Nullable String errorMsg) {
            // 광고 수신 실패
        }

        @Override
        public void onEventAd(@NonNull Object adView, @NonNull AdEvent event) {
            switch (event) {
                case DISPLAYED:
                    // 광고 화면에 표시됨
                    break;
                case CLICK:
                    // 사용자가 광고 클릭
                    break;
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_banner);

        container = findViewById(R.id.container_banner);

        // AdInfo 구성
        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
            .build();

        // AdView 생성 및 설정
        adView = new AdView(this);
        adView.setLayoutParams(new RelativeLayout.LayoutParams(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        ));
        adView.setAdInfo(adInfo);
        adView.setAdViewListener(adListener);

        // 레이아웃에 추가 후 광고 로드 시작
        container.addView(adView);
        adView.loadAd();
    }

    @Override protected void onResume() { super.onResume(); if (adView != null) adView.onResume(); }
    @Override protected void onPause() { if (adView != null) adView.onPause(); super.onPause(); }
    @Override
    protected void onDestroy() {
        if (adView != null) { adView.destroy(); adView = null; }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class BannerActivity : AppCompatActivity() {

    private lateinit var container: RelativeLayout
    private var adView: AdView? = null

    private val adListener = object : AdListener {
        override fun onReceivedAd(adapterName: String, adView: Any) {
            // 광고 수신 성공
        }
        override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                          errorCode: Int, errorMsg: String?) {
            // 광고 수신 실패
        }
        override fun onEventAd(adView: Any, event: AdEvent) {
            when (event) {
                AdEvent.DISPLAYED -> { /* 광고 표시됨 */ }
                AdEvent.CLICK -> { /* 광고 클릭 */ }
                else -> {}
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_banner)

        container = findViewById(R.id.container_banner)

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER).build()

        adView = AdView(this).apply {
            layoutParams = RelativeLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT
            )
            setAdInfo(adInfo)
            setAdViewListener(adListener)
        }

        container.addView(adView)
        adView?.loadAd()
    }

    override fun onResume() { super.onResume(); adView?.onResume() }
    override fun onPause() { adView?.onPause(); super.onPause() }
    override fun onDestroy() {
        adView?.destroy(); adView = null
        super.onDestroy()
    }
}
```

---

## 방법 2: XML 레이아웃으로 추가

#### activity_banner.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.nasmedia.admixerssp.ads.AdView
        android:id="@+id/ad_view_banner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true" />

</RelativeLayout>
```

#### Java
```java
public class BannerXmlActivity extends AppCompatActivity {

    private AdView adView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_banner);

        adView = findViewById(R.id.ad_view_banner);

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER).build();
        adView.setAdInfo(adInfo);
        adView.setAdViewListener(adListener);
        adView.loadAd(); // 광고 로드 시작 (XML로 추가된 View이므로 addView 불필요)
    }
    // ... onResume / onPause / onDestroy 동일
}
```

#### Kotlin
```kotlin
class BannerXmlActivity : AppCompatActivity() {

    private lateinit var adView: AdView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_banner)

        adView = findViewById(R.id.ad_view_banner)

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER).build()
        adView.setAdInfo(adInfo)
        adView.setAdViewListener(adListener)
        adView.loadAd() // 광고 로드 시작
    }
}
```

---

## 방법 3: 지연 노출 (원하는 시점에 표시)

광고를 미리 로드한 후 특정 이벤트(예: 콘텐츠 로딩 완료) 시점에 표시하는 방식입니다.

#### Java
```java
// 1. 광고 미리 로드 (화면 표시 없이 백그라운드 로드)
// isLoadOnly(true): 광고 수신 후 자동 노출하지 않고 loadAd() 완료만 처리
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
    .isLoadOnly(true) // ← 지연 노출 필수 설정
    .build();

adView = new AdView(this);
adView.setAdInfo(adInfo);
adView.setAdViewListener(new AdListener() {
    @Override
    public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
        // 광고 수신 완료 — hasAd 플래그가 true로 설정됨
        // isLoadOnly(true)이므로 아직 화면에 표시되지 않음
    }
    // ...
});
adView.loadAd(); // 백그라운드 로드 시작 (자동 노출 안 함)

// 2. 원하는 시점에 표시
showAdButton.setOnClickListener(v -> {
    if (adView.hasAd) {
        container.removeAllViews();
        container.addView(adView);
        adView.showAd(); // 실제 화면에 표시
    }
});
```

#### Kotlin
```kotlin
// 1. 광고 미리 로드
val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
    .isLoadOnly(true) // ← 지연 노출 필수 설정
    .build()

adView = AdView(this).apply {
    setAdInfo(adInfo)
    setAdViewListener(object : AdListener {
        override fun onReceivedAd(adapterName: String, adView: Any) {
            // 수신 완료 — isLoadOnly(true)이므로 자동 노출 안 됨
        }
        // ...
    })
    loadAd() // 자동 노출 없이 로드만 수행
}

// 2. 원하는 시점에 표시
showAdButton.setOnClickListener {
    if (adView?.hasAd == true) {
        container.removeAllViews()
        container.addView(adView)
        adView?.showAd()
    }
}
```

> ⚠️ **지연 노출 유의사항**
> - 광고 수신 후 너무 오랜 시간이 지나면 `showAd()` 호출 시 정상 표시되지 않을 수 있습니다.
> - `adView.loadAd()`만 호출하고 레이아웃에 `addView()`를 하지 않으면 광고가 화면에 표시되지 않습니다.
> - 반드시 `showAd()` 를 호출해야 광고가 화면에 표시됩니다.

---

## AdInfo 옵션 레퍼런스

`AdInfo.Builder`에서 설정 가능한 배너 관련 주요 옵션입니다.

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `isRetry(boolean)` | `true` | 광고 수신 실패 시 자동 재시도 여부 |
| `maxRetryCountInSlot(int)` | `-1` | 단일 슬롯 내 최대 재시도 횟수 (`-1` 또는 `0`: 무제한, 양수: 해당 횟수까지) |
| `showReportIcon(boolean)` | `false` | 광고 소재 위에 신고 아이콘 표시 여부 |

---

## 라이프사이클 관리

> 🚨 `AdView`의 라이프사이클 메서드를 반드시 연결하세요. 누락 시 메모리 누수가 발생하거나 광고가 정상 동작하지 않습니다.

| Activity 메서드 | AdView 메서드 | 역할 |
|----------------|--------------|------|
| `onResume()` | `adView.onResume()` | 광고 갱신 타이머 재개 |
| `onPause()` | `adView.onPause()` | 광고 갱신 타이머 일시 정지 |
| `onDestroy()` | `adView.destroy()` | 모든 리소스 해제 (필수) |

> ⚠️ `AdListener`는 내부적으로 `WeakReference`로 보유됩니다. **익명 클래스(anonymous class)**로 구현하면 GC에 의해 수집될 수 있으므로, 반드시 **멤버 변수**로 선언하세요.

---

## Adfit 사용 시 주의사항

> ⚠️ Kakao Adfit을 미디에이션으로 사용하는 경우, `AdView`는 반드시 **Activity Context**로 생성해야 합니다. `getApplicationContext()`는 Adfit에서 지원하지 않습니다.
>
> ```java
> // ✅ 올바른 방법
> adView = new AdView(this); // Activity context
>
> // ❌ 잘못된 방법
> adView = new AdView(getApplicationContext()); // Application context — Adfit 동작 안 함
> ```
