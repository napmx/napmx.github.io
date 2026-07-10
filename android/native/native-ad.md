# 네이티브 광고

> ℹ️ 네이티브 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

네이티브 광고는 `AMMNativeAdView`를 사용하여 앱 UI에 자연스럽게 통합된 형태의 광고를 표시합니다. 제공된 asset을 이용해 자유롭게 UI를 구성할 수 있습니다.

---

## 구성 Asset

네이티브 광고는 아래 6가지 asset으로 구성됩니다.

| Asset | 레이아웃 `android:id` | 설명 | 필수 여부 |
|-------|---------------------|------|----------|
| 제목 | `nap_mx_tv_title` | 광고 제목 (TextView) | AdMixer 단독: 1개 이상 필수 |
| 아이콘 | `nap_mx_iv_icon` | 광고주 아이콘 이미지 (ImageView) | 선택 |
| 광고주 | `nap_mx_tv_adv` | 광고주명 (TextView) | 선택 |
| 설명 | `nap_mx_tv_desc` | 광고 설명 텍스트 (TextView) | 선택 |
| 메인 | `nap_mx_iv_main` | 메인 이미지 또는 동영상 슬롯 (빈 `FrameLayout` 등 ViewGroup 컨테이너) | AdMixer 단독: 1개 이상 필수 |
| CTA 버튼 | `nap_mx_btn_cta` | 행동 유도 버튼 (Button) | 선택 |
| 광고 정보 고지 | `nap_mx_privacy_container` | 정보 고지 / AdChoices (View/ViewGroup/ImageView) | 선택 (미지정 시 우측 상단 자동 오버레이) |

> ⚠️ **필수 규칙**
> - AdMixer 단독 사용 시: `title`, `icon`, `mainView` 중 **최소 1개**는 반드시 사용해야 합니다.
> - Google AdManager 사용 시: Google이 요구하는 최소 View를 반드시 설정해야 합니다.

> ℹ️ **선택 asset 미수신 시 자동 숨김 — [현재 버전]**
> 서버가 내려준 광고 소재에 선택(optional) 텍스트 asset(제목·광고주·설명 등)이 포함되지 않은 경우, SDK/어댑터(Naver·Pangle·AdManager 등)는 해당 View를 **빈칸이나 placeholder 텍스트로 노출하지 않고 자동으로 `GONE` 처리**합니다. 따라서 레이아웃에 모든 asset View를 선언해 두어도 실제 소재에 없는 항목은 표시되지 않습니다.
> - 일부 소재는 제목/설명 등이 없을 수 있으므로, **asset이 누락된 경우에도 레이아웃이 자연스럽게 보이도록** 상대 위치 제약(`layout_below` 등)을 사용해 배치하고, 누락 케이스로 한 번 테스트하는 것을 권장합니다.

---

## 레이아웃 XML 작성

네이티브 광고 레이아웃을 XML로 먼저 정의합니다.

**res/layout/item_native_ad.xml**
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

    <!-- 메인 이미지 / 동영상 슬롯 -->
    <!-- 빈 FrameLayout 등 ViewGroup 컨테이너면 됩니다. 내부에 자식 뷰를 넣지 마세요(SDK가 채웁니다). -->
    <FrameLayout
        android:id="@+id/nap_mx_iv_main"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:layout_below="@id/nap_mx_tv_desc"
        android:layout_marginTop="8dp" />

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

    <!-- 광고 정보 고지 (AdChoices) 컨테이너 -->
    <FrameLayout
        android:id="@+id/nap_mx_privacy_container"
        android:layout_width="20dp"
        android:layout_height="20dp"
        android:layout_alignParentTop="true"
        android:layout_alignParentEnd="true"
        android:layout_marginTop="4dp"
        android:layout_marginEnd="4dp" />

</RelativeLayout>
```

---

## 코드 구현

가장 기본적인 방식으로, 광고 수신에 성공하면 바로 화면에 붙여 표시합니다. 연동은 아래 4단계로 이루어집니다.

| 단계 | 하는 일 |
|------|---------|
| Step 1 | `NativeAdViewBinder`로 레이아웃 ↔ asset View 매핑 |
| Step 2 | `AMMNativeAdView` 생성 후 `loadAd()`로 로드 |
| Step 3 | `AdListener`로 수신 콜백을 받아 화면에 부착 |
| Step 4 | Activity 라이프사이클에 연결 |

### Step 1. NativeAdViewBinder — 레이아웃 바인딩

레이아웃 XML의 View ID와 광고 asset을 연결합니다. 이 설정은 **모든 어댑터(AdMixer·AdManager·Adfit·Pangle·NaverAd)에 공통 적용**되므로 한 번만 만들면 됩니다.

```java
// Java
NativeAdViewBinder viewBinder = new NativeAdViewBinder.Builder(R.layout.item_native_ad)
    .setIconImageId(R.id.nap_mx_iv_icon)
    .setTitleId(R.id.nap_mx_tv_title)
    .setAdvertiserId(R.id.nap_mx_tv_adv)
    .setDescriptionId(R.id.nap_mx_tv_desc)
    .setMainViewId(R.id.nap_mx_iv_main)
    .setCtaId(R.id.nap_mx_btn_cta)
    .setPrivacyViewId(R.id.nap_mx_privacy_container) // 선택: 미지정 시 우측 상단 자동 오버레이
    .build();
```
```kotlin
// Kotlin
val viewBinder = NativeAdViewBinder.Builder(R.layout.item_native_ad)
    .setIconImageId(R.id.nap_mx_iv_icon)
    .setTitleId(R.id.nap_mx_tv_title)
    .setAdvertiserId(R.id.nap_mx_tv_adv)
    .setDescriptionId(R.id.nap_mx_tv_desc)
    .setMainViewId(R.id.nap_mx_iv_main)
    .setCtaId(R.id.nap_mx_btn_cta)
    .setPrivacyViewId(R.id.nap_mx_privacy_container) // 선택: 미지정 시 우측 상단 자동 오버레이
    .build()
```
> ℹ️ 생성자에 넘기는 `R.layout.item_native_ad`가 [위에서 작성한 레이아웃](#레이아웃-xml-작성)입니다. 각 `setXxxId()`는 해당 asset을 표시할 View ID를 지정하며, 필요한 asset만 골라 설정하면 됩니다.

### Step 2. AMMNativeAdView 생성 및 로드

`AdInfo`(광고 단위 ID)를 담아 `AMMNativeAdView`를 만들고 `loadAd()`로 요청합니다.

```java
// Java
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build();

nativeAdView = new AMMNativeAdView(this); // ⚠️ Activity Context 사용 (Adfit 필수)
nativeAdView.setAdInfo(adInfo);
nativeAdView.setViewBinder(viewBinder);   // ✅ 필수 — 없으면 렌더링되지 않음
nativeAdView.setAdViewListener(adListener); // Step 3에서 정의
nativeAdView.loadAd();
```
```kotlin
// Kotlin
val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build()

nativeAdView = AMMNativeAdView(this).apply { // ⚠️ Activity Context 사용 (Adfit 필수)
    setAdInfo(adInfo)
    setViewBinder(viewBinder)   // ✅ 필수 — 없으면 렌더링되지 않음
    setAdViewListener(adListener) // Step 3에서 정의
    loadAd()
}
```
> ⚠️ `getApplicationContext()`가 아닌 **Activity Context**로 생성해야 합니다(Adfit 미지원). `setViewBinder()`는 **필수**이며, 빠뜨리면 광고가 그려지지 않습니다.

### Step 3. AdListener — 수신 콜백에서 화면에 부착

로드 결과를 콜백으로 받습니다. 수신에 성공(`onReceivedAd`)하면 컨테이너에 `addView()`로 붙이며, **부착 시점에 자동 렌더링**되므로 별도의 `showAd()` 호출은 필요 없습니다.

```java
// Java
private final AdListener adListener = new AdListener() {
    @Override
    public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
        // 수신 성공 → 화면에 부착 (부착 시점에 자동 렌더링)
        if (nativeAdView != null && nativeAdView.hasAd) {
            container.removeAllViews();
            container.addView(nativeAdView);
        }
    }
    @Override
    public void onFailedToReceiveAd(@Nullable Object adView,
            @NonNull String adapterName, int errorCode, @Nullable String errorMsg) {
        // 수신 실패 (errorCode/errorMsg로 원인 확인)
    }
    @Override
    public void onAdDisplayed() { /* 렌더링 완료 — 화면에 표시됨 */ }
    @Override
    public void onAdClicked()   { /* 광고 클릭 */ }
};
```
```kotlin
// Kotlin
private val adListener = object : AdListener() {
    override fun onReceivedAd(adapterName: String, adView: Any) {
        // 수신 성공 → 화면에 부착 (부착 시점에 자동 렌더링)
        if (nativeAdView?.hasAd == true) {
            container.removeAllViews()
            container.addView(nativeAdView)
        }
    }
    override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                      errorCode: Int, errorMsg: String?) {
        // 수신 실패 (errorCode/errorMsg로 원인 확인)
    }
    override fun onAdDisplayed() { /* 렌더링 완료 — 화면에 표시됨 */ }
    override fun onAdClicked()   { /* 광고 클릭 */ }
}
```
> ℹ️ `AdListener`는 내부적으로 WeakReference로 보유되므로, 익명 객체를 지역 변수가 아닌 **멤버 변수**로 선언하세요.

### Step 4. 라이프사이클 연결

Activity 생명주기에 맞춰 아래 메서드를 호출합니다. `onDestroy()`의 `stop()`은 리소스 누수 방지를 위해 **필수**입니다.

```java
// Java
@Override protected void onResume()  { super.onResume(); if (nativeAdView != null) nativeAdView.onResume(); }
@Override protected void onPause()   { if (nativeAdView != null) nativeAdView.onPause(); super.onPause(); }
@Override protected void onDestroy() {
    if (nativeAdView != null) { nativeAdView.stop(); nativeAdView = null; }
    super.onDestroy();
}
```
```kotlin
// Kotlin
override fun onResume()  { super.onResume(); nativeAdView?.onResume() }
override fun onPause()   { nativeAdView?.onPause(); super.onPause() }
override fun onDestroy() {
    nativeAdView?.stop(); nativeAdView = null
    super.onDestroy()
}
```

<details>
<summary>📋 전체 코드 (복사용) — Java / Kotlin</summary>

#### Java
```java
public class NativeAdActivity extends AppCompatActivity {

    private AMMNativeAdView nativeAdView;
    private ViewGroup container;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            if (nativeAdView != null && nativeAdView.hasAd) {
                container.removeAllViews();
                container.addView(nativeAdView); // 부착 시점에 자동 렌더링
            }
        }
        @Override
        public void onFailedToReceiveAd(@Nullable Object adView,
                @NonNull String adapterName, int errorCode, @Nullable String errorMsg) { }
        @Override public void onAdDisplayed() { }
        @Override public void onAdClicked() { }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_native);
        container = findViewById(R.id.container_native);

        NativeAdViewBinder viewBinder = new NativeAdViewBinder.Builder(R.layout.item_native_ad)
            .setIconImageId(R.id.nap_mx_iv_icon)
            .setTitleId(R.id.nap_mx_tv_title)
            .setAdvertiserId(R.id.nap_mx_tv_adv)
            .setDescriptionId(R.id.nap_mx_tv_desc)
            .setMainViewId(R.id.nap_mx_iv_main)
            .setCtaId(R.id.nap_mx_btn_cta)
            .setPrivacyViewId(R.id.nap_mx_privacy_container)
            .build();

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build();

        nativeAdView = new AMMNativeAdView(this); // Activity Context (Adfit 필수)
        nativeAdView.setAdInfo(adInfo);
        nativeAdView.setViewBinder(viewBinder);   // 필수
        nativeAdView.setAdViewListener(adListener);
        nativeAdView.loadAd();
    }

    @Override protected void onResume()  { super.onResume(); if (nativeAdView != null) nativeAdView.onResume(); }
    @Override protected void onPause()   { if (nativeAdView != null) nativeAdView.onPause(); super.onPause(); }
    @Override protected void onDestroy() {
        if (nativeAdView != null) { nativeAdView.stop(); nativeAdView = null; }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class NativeAdActivity : AppCompatActivity() {

    private var nativeAdView: AMMNativeAdView? = null
    private lateinit var container: ViewGroup

    private val adListener = object : AdListener() {
        override fun onReceivedAd(adapterName: String, adView: Any) {
            if (nativeAdView?.hasAd == true) {
                container.removeAllViews()
                container.addView(nativeAdView) // 부착 시점에 자동 렌더링
            }
        }
        override fun onFailedToReceiveAd(adView: Any?, adapterName: String,
                                          errorCode: Int, errorMsg: String?) { }
        override fun onAdDisplayed() { }
        override fun onAdClicked() { }
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
            .setPrivacyViewId(R.id.nap_mx_privacy_container)
            .build()

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build()

        nativeAdView = AMMNativeAdView(this).apply { // Activity Context (Adfit 필수)
            setAdInfo(adInfo)
            setViewBinder(viewBinder)   // 필수
            setAdViewListener(adListener)
            loadAd()
        }
    }

    override fun onResume()  { super.onResume(); nativeAdView?.onResume() }
    override fun onPause()   { nativeAdView?.onPause(); super.onPause() }
    override fun onDestroy() {
        nativeAdView?.stop(); nativeAdView = null
        super.onDestroy()
    }
}
```

</details>

### 응용: 원하는 시점에 노출하기 (프리로드)

위 예제는 `onReceivedAd` 콜백에서 **즉시** `addView()`를 호출합니다. 광고를 미리 로드해 두고 특정 이벤트(콘텐츠 진입, 버튼 클릭 등) 시점에 표시하고 싶다면, **`addView()` 호출만 그 시점으로 옮기면 됩니다.** ViewBinder·`loadAd()` 등 나머지는 위와 동일합니다.

```java
// Java — 콜백에서는 부착하지 않고, 원하는 시점에 부착
adListener = new AdListener() {
    @Override
    public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
        // 수신 완료(hasAd = true)까지만. 아직 부착하지 않았으므로 표시되지 않음
    }
};

showAdButton.setOnClickListener(v -> {
    if (nativeAdView.hasAd) {
        container.removeAllViews();
        container.addView(nativeAdView); // 이 시점에 노출
    }
});
```
```kotlin
// Kotlin — 콜백에서는 부착하지 않고, 원하는 시점에 부착
adListener = object : AdListener() {
    override fun onReceivedAd(adapterName: String, adView: Any) {
        // 수신 완료(hasAd = true)까지만. 아직 부착하지 않았으므로 표시되지 않음
    }
}

showAdButton.setOnClickListener {
    if (nativeAdView?.hasAd == true) {
        container.removeAllViews()
        container.addView(nativeAdView) // 이 시점에 노출
    }
}
```

> ⚠️ **프리로드 유의사항**
> - 광고 수신 후 너무 오랜 시간이 지나면 정상 노출되지 않을 수 있습니다.
> - `loadAd()`만 호출하고 `addView()`를 하지 않으면 광고가 표시되지 않습니다.
> - 부착 전(프리로드 보유 구간)에는 [자동 갱신](#주의사항)이 발생하지 않으며, `addView()` 부착 이후부터 갱신 타이머가 동작합니다.

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
| `setMainViewId(int viewId)` | 메인 이미지/동영상 슬롯(컨테이너 ViewGroup) View ID |
| `setCtaId(int viewId)` | CTA 버튼 View ID |
| `setPrivacyViewId(int viewId)` | 광고 정보 고지 (AdChoices) 컨테이너 / 뷰 ID |

---

## 주의사항

> ⚠️ **Adfit 사용 시**: `AMMNativeAdView`는 반드시 **Activity Context**로 생성하세요. `getApplicationContext()`는 Adfit에서 지원하지 않습니다.

> ℹ️ **레이아웃 구조**: 네이티브 광고 레이아웃에는 `RelativeLayout` 사용을 권장합니다. 다른 레이아웃을 사용해야 하는 경우, 해당 레이아웃을 `RelativeLayout` 안에 넣는 방식으로 구현할 수 있습니다.

> ℹ️ **`setViewBinder()`는 필수입니다.** `setViewBinder()` 없이는 네이티브 광고가 렌더링되지 않습니다.

> ℹ️ **Naver Ad Manager — Native Simple(템플릿형) 광고 — [v2.0.0]**: NAM 통합형 지면에서 Native Simple 광고가 내려오는 경우, 소재 전체를 NAM SDK가 **템플릿으로 렌더링**합니다. 이때 `NativeAdViewBinder`의 레이아웃/뷰 ID 매핑(자산 바인딩)은 적용되지 않으며, 광고 높이는 소재 비율에 따라 자동 결정됩니다. 연동 방법은 일반 네이티브와 동일하며(`loadAd()` → `addView`), 응답 타입 분기는 SDK가 자동 처리합니다.

> ℹ️ **자동 갱신(롤링) — [v2.0.0]**: 배너와 **동일 로직**으로, 서버(media-conf) 광고 단위 `interval`(초)이 **0보다 클 때만** 노출 후 `interval`마다 자동 갱신되고 실패 시 재요청합니다(최소 5초 간격, 무한 루프는 #59 가드 차단). `interval`이 0(미설정)이면 단발성(자동 재로드 없음)입니다. 갱신 시에는 뷰가 이미 화면에 부착돼 있으므로 새 소재가 자동으로 다시 렌더링됩니다(`showAd()` 호출 불필요).

---

## 라이프사이클 관리

| Activity 메서드 | AMMNativeAdView 메서드 | 역할 |
|----------------|---------------------|------|
| `onResume()` | `nativeAdView.onResume()` | 동영상 재생 재개 |
| `onPause()` | `nativeAdView.onPause()` | 동영상 재생 일시 정지 |
| `onDestroy()` | `nativeAdView.stop()` | 모든 리소스 해제 (필수) |

---

## 구 API에서 전환 (네이티브 · v1.x.x → v2)

구 `NativeAdView` 클래스는 v2에서 제거되었습니다. `AMMNativeAdView`로 전환하세요. 네이티브 레이아웃 View ID도 `nap_mx_` prefix로 변경되었으니, 전체 절차는 [마이그레이션 가이드](migration.md)를 참고하세요.

| v1.x.x (제거됨) | v2.0.0 |
|---|---|
| `NativeAdView` | `AMMNativeAdView` (메서드 시그니처 동일) |
| `loadNativeAd()` | `loadAd()` |
| `setViewIds(...)` | `NativeAdViewBinder` 설정으로 통합 (모든 어댑터 공통) |
| `tv_title` / `iv_icon` / `tv_adv` / `tv_desc` / `iv_main` / `btn_cta` | `nap_mx_tv_title` / `nap_mx_iv_icon` / `nap_mx_tv_adv` / `nap_mx_tv_desc` / `nap_mx_iv_main` / `nap_mx_btn_cta` |
| `onEventAd(AdEvent.DISPLAYED / CLICK)` | `onAdDisplayed()` / `onAdClicked()` |
| `destroy()` / `onDestroy()` | `stop()` |
