# 배너 광고

> ℹ️ 배너 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이
> 완료되었는지 확인하세요.

배너 광고는 `AMMBannerView`를 사용하여 화면 내 지정한 영역에 광고를 표시합니다.
광고 크기는 서버 설정과 미디에이션 네트워크에 따라 결정됩니다.

> 💡 **광고 크기**
>
> - 배너 컨테이너는 **너비 `match_parent` + 높이 `wrap_content`** 로 두는 것을
>   권장합니다(예제 동일). 높이를 고정하면 광고가 잘리거나 여백이 생길 수
>   있습니다.
> - **AdManager(GAM) 표준 배너**는 컨테이너 너비에 맞춘 **anchored adaptive**
>   크기로 요청되어 디바이스 너비에 따라 높이가 달라집니다. MREC(300×250) 등
>   고정 크기 슬롯은 종전대로 유지됩니다.

---

## 노출 방식

배너 광고는 뷰가 화면에 부착(`addView` 또는 XML 배치)되는 시점에 **자동으로
표시**됩니다. 별도의 `showAd()` 호출은 필요 없습니다.

| 방식          | 설명                                                               | 권장 사용 시나리오                    |
| ------------- | ------------------------------------------------------------------ | ------------------------------------- |
| **고정 영역** | 레이아웃(XML 또는 `addView`)에 미리 배치하고 `loadAd()`            | 광고 영역이 항상 고정된 경우          |
| **지연 노출** | `loadAd()`로 미리 로드 후, 원하는 시점에 `addView()`로 화면에 부착 | 특정 시점에 광고를 노출해야 하는 경우 |

> `showAd()`는 호출할 필요가 없습니다 — 뷰가 화면에 부착되는 시점에 자동
> 표시됩니다.

---

## 방법 1: 코드로 추가 (즉시 노출)

#### Java

```java
public class BannerActivity extends AppCompatActivity {

    private RelativeLayout container;
    private AMMBannerView adView;
    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull AdNetworkType networkType, @NonNull Object adView) {
            // 광고 수신 성공 — addView 이후 자동 표시됨
            // networkType로 switch: switch(networkType){ case PANGLE: ... }
        }

        @Override
        public void onFailedToReceiveAd(int errorCode, @Nullable String errorMsg) {
            // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
        }

        @Override
        public void onAdDisplayed() {
            // 광고 화면에 표시됨
        }

        @Override
        public void onAdClicked() {
            // 사용자가 광고 클릭
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

        // AMMBannerView 생성 및 설정
        adView = new AMMBannerView(this);
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
        if (adView != null) { adView.stop(); adView = null; }
        super.onDestroy();
    }
}
```

#### Kotlin

```kotlin
class BannerActivity : AppCompatActivity() {

    private lateinit var container: RelativeLayout
    private var adView: AMMBannerView? = null

    private val adListener = object : AdListener() {
        override fun onReceivedAd(networkType: AdNetworkType, adView: Any) {
            // 광고 수신 성공
        }
        override fun onFailedToReceiveAd(errorCode: Int, errorMsg: String?) {
            // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
        }
        override fun onAdDisplayed() { /* 광고 표시됨 */ }
        override fun onAdClicked() { /* 광고 클릭 */ }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_banner)

        container = findViewById(R.id.container_banner)

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER).build()

        adView = AMMBannerView(this).apply {
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
        adView?.stop(); adView = null
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

    <com.nasmedia.admixerssp.ads.AMMBannerView
        android:id="@+id/ad_view_banner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true" />

</RelativeLayout>
```

#### Java

```java
public class BannerXmlActivity extends AppCompatActivity {

    private AMMBannerView adView;

    // ⚠️ 리스너는 반드시 멤버 변수로 — 익명 클래스로 넘기면 GC에 수집될 수 있습니다.
    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull AdNetworkType networkType, @NonNull Object adView) { }

        @Override
        public void onFailedToReceiveAd(int errorCode, @Nullable String errorMsg) { }
    };

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

    private lateinit var adView: AMMBannerView

    // ⚠️ 리스너는 반드시 프로퍼티로 — 익명 객체를 그대로 넘기면 GC에 수집될 수 있습니다.
    private val adListener = object : AdListener() {
        override fun onReceivedAd(networkType: AdNetworkType, adView: Any) { }
        override fun onFailedToReceiveAd(errorCode: Int, errorMsg: String?) { }
    }

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

광고를 미리 로드한 후 특정 이벤트(예: 콘텐츠 로딩 완료) 시점에 표시하는
방식입니다.

#### Java

```java
// 1. 광고 미리 로드 (아직 레이아웃에 추가하지 않음)
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
    .build();

adView = new AMMBannerView(this);
adView.setAdInfo(adInfo);
// ⚠️ adListener 는 멤버 변수여야 합니다(익명 클래스를 인라인으로 넘기면 GC 대상).
//    선언 예시는 위 "방법 1" 참고.
adView.setAdViewListener(adListener);
adView.loadAd(); // 백그라운드 로드 시작

// 2. 원하는 시점에 화면에 추가 → 부착되는 즉시 자동 표시 (showAd 호출 불필요)
showAdButton.setOnClickListener(v -> {
    if (adView.hasAd) {
        container.removeAllViews();
        container.addView(adView); // addView 시점에 자동 노출
    }
});
```

#### Kotlin

```kotlin
// 1. 광고 미리 로드 (아직 레이아웃에 추가하지 않음)
val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
    .build()

adView = AMMBannerView(this).apply {
    setAdInfo(adInfo)
    // ⚠️ adListener 는 프로퍼티여야 합니다(익명 객체를 인라인으로 넘기면 GC 대상).
    //    선언 예시는 위 "방법 1" 참고.
    setAdViewListener(adListener)
    loadAd() // 백그라운드 로드 시작
}

// 2. 원하는 시점에 화면에 추가 → 부착되는 즉시 자동 표시 (showAd 호출 불필요)
showAdButton.setOnClickListener {
    if (adView?.hasAd == true) {
        container.removeAllViews()
        container.addView(adView) // addView 시점에 자동 노출
    }
}
```

> ⚠️ **지연 노출 유의사항**
>
> - 광고 수신 후 너무 오랜 시간이 지나면 표시 시 정상 노출되지 않을 수 있습니다.
> - `adView.loadAd()`만 호출하고 레이아웃에 `addView()`를 하지 않으면 광고가
>   화면에 표시되지 않습니다.
> - `addView()`로 화면에 부착하면 자동으로 표시됩니다(`showAd()` 호출 불필요).

---

> **[v2.0.0]** 배너 자동 갱신/실패 재시도는 서버(media-conf) 광고 단위
> `interval`(초)이 **0보다 클 때만** 동작합니다(서버 0/미설정 → 단발성, 자동
> 재로드 없음). 기존 클라이언트 `isRetry` 옵션은 제거되었습니다.

---

## 수신 실패 콜백은 **표준 하나만** 구현하면 됩니다

로드 실패의 표준 콜백은
**`onFailedToReceiveAd(int errorCode, String errorMsg)`** 입니다. 이것 하나만
구현하면 전 네트워크 No-Ad("All adapters failed."), SDK 미초기화·AdUnit 누락 등
**내부 실패를 포함한 모든 수신 실패**를 받습니다. 실패 경로의 네트워크 식별자는
항상 내부 합성값(`"SDK"`/`"Mediation"`)이라 유의미하지 않아 전달하지 않습니다.

> ℹ️ **구버전 호환** — 기존 4-인자 오버로드(`String adapterName` /
> `AdNetworkType networkType`)는 둘 다 `@Deprecated`(3.0 제거 예정)이며, 기본
> 구현이 표준 콜백으로 위임하므로 이미 구현해 둔 코드도 동작은 그대로입니다.
> 신규 코드는 표준 콜백만 구현하세요.

---

## 네트워크에 따라 달라질 수 있는 항목

nap mx는 미디에이션 SDK입니다. 실제 광고를 만들고 이벤트를 발생시키는 주체는 **낙찰된 광고 네트워크의 SDK**이며, nap mx는 이를 공통 인터페이스로 중계합니다. 따라서 아래 항목은 **어떤 네트워크가 낙찰되었는지에 따라 동작이 달라질 수 있습니다.** 특정 동작을 전제로 화면을 설계하지 마세요.

| 항목 | 설명 | 앱에서 권장하는 대응 |
|------|------|---------------------|
| **광고 크기** | 요청 크기와 실제 반환 크기가 다를 수 있습니다. 네트워크에 따라 고정 규격만 제공하기도 합니다. | 배너 컨테이너는 **고정 높이 대신 `wrap_content`** 를 쓰고, 주변 레이아웃이 밀리지 않게 최소 높이만 확보하세요. |
| **Adaptive / Collapsible 배너** | 지원 여부가 네트워크마다 다르며, 지원하더라도 조건이 다를 수 있습니다. | 특정 형태를 전제하지 말고, 어떤 크기가 와도 깨지지 않는 레이아웃을 사용하세요. 지원 여부는 해당 네트워크 공식 문서를 확인하세요. |
| **자동 갱신(Auto Refresh)** | 갱신 주기와 갱신 주체(네트워크 SDK / nap mx 서버 설정)가 네트워크마다 다를 수 있습니다. | 앱에서 별도 타이머로 재요청하지 마세요. 갱신 정책은 파트너 사이트의 애드유닛 설정을 따릅니다. |
| **노출(임프레션) 시점** | "화면에 붙는 시점"과 "네트워크가 노출로 집계하는 시점"이 다를 수 있습니다. | `onAdDisplayed()` 를 정산 근거로 삼지 말고, 리포트는 파트너 사이트 기준으로 확인하세요. |
| **클릭 동작** | 외부 브라우저 / 인앱 브라우저 / 앱 이동 등 랜딩 방식이 네트워크 정책에 따라 다릅니다. | 클릭 후 앱이 백그라운드로 갈 수 있다는 전제로 `onPause()`/`onResume()` 를 처리하세요. |
| **Context 요구사항** | 일부 네트워크 SDK는 `Application Context` 로 동작하지 않습니다. | 배너는 항상 **Activity Context** 로 생성하세요(아래 참고). |

> ℹ️ 지원 여부·정책은 각 네트워크 SDK 버전에 따라 변경될 수 있습니다. 최신 내용은 해당 네트워크의 공식 문서를 참고하세요.

---

## 라이프사이클 관리

> 🚨 `AMMBannerView`의 라이프사이클 메서드를 반드시 연결하세요. 누락 시 메모리
> 누수가 발생하거나 광고가 정상 동작하지 않습니다.

| Activity 메서드 | AMMBannerView 메서드 | 역할                       |
| --------------- | -------------------- | -------------------------- |
| `onResume()`    | `adView.onResume()`  | 광고 갱신 타이머 재개      |
| `onPause()`     | `adView.onPause()`   | 광고 갱신 타이머 일시 정지 |
| `onDestroy()`   | `adView.stop()`      | 모든 리소스 해제 (필수)    |

> ⚠️ `AdListener`는 내부적으로 `WeakReference`로 보유됩니다. **익명
> 클래스(anonymous class)**로 구현하면 GC에 의해 수집될 수 있으므로, 반드시
> **멤버 변수**로 선언하세요.

---

## Adfit 사용 시 주의사항

> ⚠️ Kakao Adfit을 미디에이션으로 사용하는 경우, `AMMBannerView`는 반드시
> **Activity Context**로 생성해야 합니다. `getApplicationContext()`는 Adfit에서
> 지원하지 않습니다.
>
> ```java
> // ✅ 올바른 방법
> adView = new AMMBannerView(this); // Activity context
>
> // ❌ 잘못된 방법
> adView = new AMMBannerView(getApplicationContext()); // Application context — Adfit 동작 안 함
> ```

---

## 구 API에서 전환 (배너 · v1.x.x → v2)

구 `AdView` 클래스는 v2에서 제거되었습니다. `AMMBannerView`로 전환하세요. 전체
마이그레이션 절차는 [마이그레이션 가이드](migration.md)를 참고하세요.

| v1.x.x (제거됨)                        | v2.0.0                                 |
| -------------------------------------- | -------------------------------------- |
| `AdView`                               | `AMMBannerView` (메서드 시그니처 동일) |
| `onEventAd(AdEvent.DISPLAYED / CLICK)` | `onAdDisplayed()` / `onAdClicked()`    |
| `destroy()` / `onDestroy()`            | `stop()`                               |
