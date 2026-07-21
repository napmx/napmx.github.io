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

> ℹ️ 광고 정보 고지(AdChoices) 아이콘은 SDK가 자동 오버레이하므로 **레이아웃 슬롯이 필요 없습니다**.
> 위치는 [`setAdChoicesPosition()`](#광고-정보-고지adchoices-위치-지정)으로 지정합니다.

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

    <!-- 광고 정보 고지(AdChoices)는 SDK가 자동 오버레이하므로 슬롯이 필요 없습니다. -->

</RelativeLayout>
```

> ℹ️ **메인 미디어 슬롯(`nap_mx_iv_main`) 규칙** — *중요*
> `nap_mx_iv_main`은 **빈 ViewGroup 컨테이너**(예: `FrameLayout`)여야 합니다. SDK가 이 슬롯 내부를 비우고(`removeAllViews`) 미디에이션 벤더(네이버·구글·팽글·애드핏)의 MediaView 또는 AdMixer 자체 이미지/동영상으로 채웁니다.
> - ✅ **빈 컨테이너로 두세요.** 안에 `ImageView` 등 자식 뷰를 직접 넣어도 SDK가 제거/무시합니다.
> - ✅ **높이(또는 비율)를 명시하세요.** `match_parent` 높이가 스크롤/`wrap_content` 부모 안에서 0으로 접히면 광고가 보이지 않을 수 있습니다.
> - ❌ **`ImageView`처럼 ViewGroup이 아닌 단일 View에 `nap_mx_iv_main` id만 부여하지 마세요.** 미디에이션 벤더 광고(네이버 등)에서 메인 미디어가 노출되지 않습니다.

### 슬롯 크기는 매체 레이아웃이 결정합니다 — [현재 버전]

**선언한 크기가 그대로 렌더링됩니다.** 슬롯에 고정 크기(`250dp`, `match_parent` 등)를 지정하면 SDK는 그 크기를 그대로 채웁니다. 어떤 네트워크가 채우든(AdMixer 자체 / AdManager / Pangle / NaverAd / Adfit) 동일합니다.

> ⚠️ **동작 변경** — 이전 버전에서는 AdMixer 자체 광고에 한해 SDK가 소재 비율에 맞춰 **슬롯보다 작게** 렌더링했고, 그래서 같은 레이아웃이라도 워터폴 승자에 따라 높이가 달라졌습니다(예: 144×96 카드에 1200×628 소재 → 144×75로 축소되어 카드 하단에 배경 노출). 이제 선언한 크기를 그대로 지킵니다.
>
> **영향받는 경우** — 슬롯 비율과 소재 비율이 다르면 여백(레터박스)의 **위치**가 바뀝니다. 기존에는 하단에 몰렸으나 이제 상·하로 나뉩니다(총량은 동일). 슬롯 비율과 소재 비율이 같으면 변화가 없습니다.

슬롯 안에서 소재를 어떻게 맞출지는 `ImageView`의 `scaleType`이 결정하며 기본값은 `FIT_CENTER`(잘림 없이 맞춤)입니다. 여백을 없애려면 **슬롯 비율을 소재 비율에 맞추세요** — 대부분의 네이티브 소재는 1.91:1입니다.

```xml
<!-- 권장: 소재 비율에 맞춘 슬롯 (1.91:1) -->
<FrameLayout
    android:id="@+id/nap_mx_iv_main"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="1.91:1" />  <!-- ConstraintLayout 사용 시 -->
```

---

## 코드 구현

#### Java
```java
public class NativeAdActivity extends AppCompatActivity {

    private AMMNativeAdView nativeAdView;
    private ViewGroup container;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull AdNetworkType networkType, @NonNull Object adView) {
            // 광고 수신 성공 — 레이아웃에 추가하면 부착 시점에 자동 렌더링 (showAd 호출 불필요)
            // networkType로 switch: switch(networkType){ case PANGLE: ... }
            if (nativeAdView != null && nativeAdView.hasAd) {
                container.removeAllViews();
                container.addView(nativeAdView); // addView 시점에 자동 렌더링
            }
        }

        @Override
        public void onFailedToReceiveAd(int errorCode, @Nullable String errorMsg) {
            // 광고 수신 실패
        }

        @Override
        public void onAdDisplayed() {
            // 광고 화면에 표시됨 (어댑터 렌더링 완료 시점)
        }

        @Override
        public void onAdClicked() {
            // 광고 클릭
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
            .setAdChoicesPosition(AdChoicesPosition.RIGHT_TOP) // ✅ 선택 — 광고 정보 고지(AdChoices) 모서리, 기본 RIGHT_TOP
            .build();

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build();

        // ② AMMNativeAdView 생성 및 로드
        nativeAdView = new AMMNativeAdView(this); // Activity context 사용 (Adfit 필수)
        nativeAdView.setAdInfo(adInfo);
        nativeAdView.setViewBinder(viewBinder); // ✅ 필수 — AdMixer·AdManager·Adfit·Pangle·NaverAd 전체 적용
        nativeAdView.setAdViewListener(adListener);
        nativeAdView.loadAd();
    }

    @Override protected void onResume() { super.onResume(); if (nativeAdView != null) nativeAdView.onResume(); }
    @Override protected void onPause() { if (nativeAdView != null) nativeAdView.onPause(); super.onPause(); }
    @Override
    protected void onDestroy() {
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
        override fun onReceivedAd(networkType: AdNetworkType, adView: Any) {
            if (nativeAdView?.hasAd == true) {
                container.removeAllViews()
                container.addView(nativeAdView) // addView 시점에 자동 렌더링 (showAd 호출 불필요)
            }
        }
        override fun onFailedToReceiveAd(errorCode: Int, errorMsg: String?) {
            // 광고 수신 실패
        }
        override fun onAdDisplayed() { /* 광고 표시됨 */ }
        override fun onAdClicked() { /* 클릭 처리 */ }
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
            .setAdChoicesPosition(AdChoicesPosition.RIGHT_TOP) // ✅ 선택 — 광고 정보 고지(AdChoices) 모서리, 기본 RIGHT_TOP
            .build()

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_NATIVE).build()

        nativeAdView = AMMNativeAdView(this).apply {
            setAdInfo(adInfo)
            setViewBinder(viewBinder) // ✅ 필수 — AdMixer·AdManager·Adfit·Pangle·NaverAd 전체 적용
            setAdViewListener(adListener)
            loadAd()
        }
    }

    override fun onResume() { super.onResume(); nativeAdView?.onResume() }
    override fun onPause() { nativeAdView?.onPause(); super.onPause() }
    override fun onDestroy() {
        nativeAdView?.stop(); nativeAdView = null
        super.onDestroy()
    }
}
```

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
| `setAdChoicesPosition(AdChoicesPosition p)` | 광고 정보 고지(AdChoices) 아이콘의 **모서리** 위치. 미지정 시 `RIGHT_TOP` |

---

## 광고 정보 고지(AdChoices) 위치 지정

광고 정보 고지 아이콘은 **SDK가 자동으로 오버레이**합니다. 레이아웃에 슬롯을 만들 필요가 없습니다.
위치를 바꾸고 싶으면 `setAdChoicesPosition()`으로 4개 모서리 중 하나를 고르세요.

```java
NativeAdViewBinder viewBinder = new NativeAdViewBinder.Builder(R.layout.item_native_ad)
    // ... 자산 슬롯 ...
    .setAdChoicesPosition(AdChoicesPosition.LEFT_TOP) // 선택 (미지정 시 RIGHT_TOP)
    .build();
```

| 값 | 위치 |
|---|---|
| `AdChoicesPosition.LEFT_TOP` | 왼쪽 상단 |
| `AdChoicesPosition.RIGHT_TOP` | 오른쪽 상단 **(기본값)** |
| `AdChoicesPosition.LEFT_BOTTOM` | 왼쪽 하단 |
| `AdChoicesPosition.RIGHT_BOTTOM` | 오른쪽 하단 |

모든 네이티브 네트워크(AdMixer 자체 / AdManager / GMA NextGen / NaverAd / Pangle / Adfit)에 동일하게
적용됩니다.

> ℹ️ **임의 위치는 지원하지 않습니다.** 4개 모서리만 지정할 수 있습니다. 네트워크 SDK마다 아이콘 소유권이
> 달라(일부는 SDK가 자기 뷰 계층에 직접 그림) 모서리 밖 위치는 네트워크 간 동작을 보장할 수 없기 때문입니다.

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
