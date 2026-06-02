# 네이티브 광고

{% hint style="info" %}
네이티브 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.
{% endhint %}

네이티브 광고는 `NativeAdView`를 사용하여 앱 UI에 자연스럽게 통합된 형태의 광고를 표시합니다. 제공된 asset을 이용해 자유롭게 UI를 구성할 수 있습니다.

---

## 구성 Asset

네이티브 광고는 아래 6가지 asset으로 구성됩니다.

| Asset | 레이아웃 `android:id` | 설명 | 필수 여부 |
|-------|---------------------|------|----------|
| 제목 | `nap_mx_tv_title` | 광고 제목 (TextView) | AdMixer 단독: 1개 이상 필수 |
| 아이콘 | `nap_mx_iv_icon` | 광고주 아이콘 이미지 (ImageView) | 선택 |
| 광고주 | `nap_mx_tv_adv` | 광고주명 (TextView) | 선택 |
| 설명 | `nap_mx_tv_desc` | 광고 설명 텍스트 (TextView) | 선택 |
| 메인 | `nap_mx_iv_main` | 메인 이미지 또는 동영상 (NativeMainAdView) | AdMixer 단독: 1개 이상 필수 |
| CTA 버튼 | `nap_mx_btn_cta` | 행동 유도 버튼 (Button) | 선택 |

{% hint style="warning" %}
**필수 규칙**
- AdMixer 단독 사용 시: `title`, `icon`, `mainView` 중 **최소 1개**는 반드시 사용해야 합니다.
- Google AdManager 사용 시: Google이 요구하는 최소 View를 반드시 설정해야 합니다.
{% endhint %}

---

## 레이아웃 XML 작성

네이티브 광고 레이아웃을 XML로 먼저 정의합니다.

{% code title="res/layout/item_native_ad.xml" %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="#FFFFFF"
    android:padding="12dp">

    <!-- 아이콘 이미지 -->
    <ImageView
        android:id="@+id/nap_mx_iv_icon"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true"
        android:scaleType="centerCrop" />

    <!-- 광고 제목 -->
    <TextView
        android:id="@+id/nap_mx_tv_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_toEndOf="@id/nap_mx_iv_icon"
        android:textSize="14sp"
        android:textStyle="bold"
        android:textColor="#222222"
        android:maxLines="2"
        android:ellipsize="end" />

    <!-- 광고주명 -->
    <TextView
        android:id="@+id/nap_mx_tv_adv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/nap_mx_tv_title"
        android:layout_marginStart="8dp"
        android:layout_toEndOf="@id/nap_mx_iv_icon"
        android:textSize="11sp"
        android:textColor="#999999" />

    <!-- 광고 설명 -->
    <TextView
        android:id="@+id/nap_mx_tv_desc"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/nap_mx_iv_icon"
        android:layout_marginTop="8dp"
        android:textSize="12sp"
        android:textColor="#555555"
        android:maxLines="3"
        android:ellipsize="end" />

    <!-- 메인 이미지 / 동영상 (NativeMainAdView 필수) -->
    <com.nasmedia.admixerssp.common.nativeads.NativeMainAdView
        android:id="@+id/nap_mx_iv_main"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:layout_below="@id/nap_mx_tv_desc"
        android:layout_marginTop="8dp">

        <!-- 내부에 ImageView 배치 (메인 이미지용) -->
        <ImageView
            android:id="@+id/nap_mx_iv_main_image"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="centerCrop" />

    </com.nasmedia.admixerssp.common.nativeads.NativeMainAdView>

    <!-- CTA(행동 유도) 버튼 -->
    <Button
        android:id="@+id/nap_mx_btn_cta"
        android:layout_width="match_parent"
        android:layout_height="44dp"
        android:layout_below="@id/nap_mx_iv_main"
        android:layout_marginTop="8dp"
        android:background="#3A86FF"
        android:textColor="#FFFFFF"
        android:textSize="13sp" />

</RelativeLayout>
```
{% endcode %}

---

## 코드 구현

{% tabs %}
{% tab title="Java" %}
```java
public class NativeAdActivity extends AppCompatActivity {

    private NativeAdView nativeAdView;
    private ViewGroup container;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            // 광고 수신 성공 — 레이아웃에 추가 후 반드시 showAd() 호출
            if (nativeAdView != null && nativeAdView.hasAd) {
                container.removeAllViews();
                container.addView(nativeAdView);
                nativeAdView.showAd(); // 광고 소재 렌더링 및 노출 처리 (필수)
            }
        }

        @Override
        public void onFailedToReceiveAd(@Nullable Object adView,
                @NonNull String adapterName, int errorCode, @Nullable String errorMsg) {
            // 광고 수신 실패
        }

        @Override
        public void onEventAd(@NonNull Object adView, @NonNull AdEvent event) {
            if (event == AdEvent.CLICK) {
                // 광고 클릭
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_native);

        container = findViewById(R.id.container_native);

        // ① NativeAdViewBinder — 모든 어댑터가 공유하는 레이아웃 바인딩 설정
        NativeAdViewBinder viewBinder = new NativeAdViewBinder.Builder(R.layout.item_native_ad)
            .setIconImageId(R.id.nap_mx_iv_icon)
            .setTitleId(R.id.nap_mx_tv_title)
            .setAdvertiserId(R.id.nap_mx_tv_adv)
            .setDescriptionId(R.id.nap_mx_tv_desc)
            .setMainViewId(R.id.nap_mx_iv_main)
            .setCtaId(R.id.nap_mx_btn_cta)
            .build();

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build();

        // ② NativeAdView 생성 및 로드
        nativeAdView = new NativeAdView(this); // Activity context 사용 (Adfit 필수)
        nativeAdView.setAdInfo(adInfo);
        nativeAdView.setViewBinder(viewBinder); // ✅ 필수 — AdMixer·AdManager·Adfit·Pangle·Mobwith·NaverAd 전체 적용
        nativeAdView.setAdViewListener(adListener);
        nativeAdView.loadNativeAd();
    }

    @Override protected void onResume() { super.onResume(); if (nativeAdView != null) nativeAdView.onResume(); }
    @Override protected void onPause() { if (nativeAdView != null) nativeAdView.onPause(); super.onPause(); }
    @Override
    protected void onDestroy() {
        if (nativeAdView != null) { nativeAdView.destroy(); nativeAdView = null; }
        super.onDestroy();
    }
}
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
class NativeAdActivity : AppCompatActivity() {

    private var nativeAdView: NativeAdView? = null
    private lateinit var container: ViewGroup

    private val adListener = object : AdListener {
        override fun onReceivedAd(adapterName: String, adView: Any) {
            if (nativeAdView?.hasAd == true) {
                container.removeAllViews()
                container.addView(nativeAdView)
                nativeAdView?.showAd() // 광고 소재 렌더링 및 노출 처리 (필수)
            }
        }
        override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                          errorCode: Int, errorMsg: String?) { }
        override fun onEventAd(adView: Any, event: AdEvent) {
            if (event == AdEvent.CLICK) { /* 클릭 처리 */ }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_native)

        container = findViewById(R.id.container_native)

        val viewBinder = NativeAdViewBinder.Builder(R.layout.item_native_ad)
            .setIconImageId(R.id.nap_mx_iv_icon)
            .setTitleId(R.id.nap_mx_tv_title)
            .setAdvertiserId(R.id.nap_mx_tv_adv)
            .setDescriptionId(R.id.nap_mx_tv_desc)
            .setMainViewId(R.id.nap_mx_iv_main)
            .setCtaId(R.id.nap_mx_btn_cta)
            .build()

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build()

        nativeAdView = NativeAdView(this).apply {
            setAdInfo(adInfo)
            setViewBinder(viewBinder) // ✅ 필수 — AdMixer·AdManager·Adfit·Pangle·Mobwith·NaverAd 전체 적용
            setAdViewListener(adListener)
            loadNativeAd()
        }
    }

    override fun onResume() { super.onResume(); nativeAdView?.onResume() }
    override fun onPause() { nativeAdView?.onPause(); super.onPause() }
    override fun onDestroy() {
        nativeAdView?.destroy(); nativeAdView = null
        super.onDestroy()
    }
}
```
{% endtab %}
{% endtabs %}

---

## NativeAdViewBinder 옵션

`NativeAdViewBinder.Builder`에서 설정 가능한 메서드입니다.

| 메서드 | 설명 |
|--------|------|
| `new Builder(int layoutResId)` | 네이티브 광고 레이아웃 리소스 ID (필수) |
| `setIconImageId(int viewId)` | 아이콘 이미지 View ID |
| `setTitleId(int viewId)` | 제목 TextView ID |
| `setAdvertiserId(int viewId)` | 광고주명 TextView ID |
| `setDescriptionId(int viewId)` | 설명 TextView ID |
| `setMainViewId(int viewId)` | 메인 이미지/동영상 NativeMainAdView ID |
| `setCtaId(int viewId)` | CTA 버튼 View ID |

---

## 주의사항

{% hint style="warning" %}
**Adfit 사용 시**: `NativeAdView`는 반드시 **Activity Context**로 생성하세요. `getApplicationContext()`는 Adfit에서 지원하지 않습니다.
{% endhint %}

{% hint style="info" %}
**레이아웃 구조**: 네이티브 광고 레이아웃에는 `RelativeLayout` 사용을 권장합니다. 다른 레이아웃을 사용해야 하는 경우, 해당 레이아웃을 `RelativeLayout` 안에 넣는 방식으로 구현할 수 있습니다.
{% endhint %}

{% hint style="info" %}
**`setViewBinder()`는 필수입니다.** `setViewBinder()` 없이는 네이티브 광고가 렌더링되지 않습니다.
{% endhint %}

---

## 라이프사이클 관리

| Activity 메서드 | NativeAdView 메서드 | 역할 |
|----------------|---------------------|------|
| `onResume()` | `nativeAdView.onResume()` | 동영상 재생 재개 |
| `onPause()` | `nativeAdView.onPause()` | 동영상 재생 일시 정지 |
| `onDestroy()` | `nativeAdView.destroy()` | 모든 리소스 해제 (필수) |
