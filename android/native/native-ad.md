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

> ℹ️ **선택 asset 미수신 시 동작 — [현재 버전]**
> 미디에이션 네트워크(Naver·Pangle·AdManager 등) 소재에 선택(optional) asset(제목·본문·광고주·CTA·아이콘 등)이 포함되지 않은 경우, 해당 어댑터가 그 View를 **빈칸이나 placeholder 텍스트로 노출하지 않고 자동으로 `GONE` 처리**합니다.
> - **AdMixer 자체 소재는 자동 `GONE` 처리를 하지 않습니다.** 빈 값이 그대로 설정되고(CTA는 문구가 없으면 기본 문구 "더보기"로 대체), 아이콘 이미지가 없으면 `ImageView`가 비어 있는 채로 자리를 차지합니다.
> - 따라서 **asset이 누락된 경우에도 레이아웃이 자연스럽게 보이도록** 상대 위치 제약(`layout_below` 등)을 사용해 배치하고, 누락 케이스로 한 번 테스트하는 것을 권장합니다.

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

**선언한 크기가 그대로 렌더링됩니다.** 슬롯에 고정 크기(`250dp`, `match_parent` 등)를 지정하면 SDK는 그 크기를 그대로 채웁니다. 슬롯 크기 자체는 어떤 네트워크가 채우든 선언한 값이 유지됩니다. 다만 슬롯 안에서 소재가 어떻게 배치·재생되는지는 네트워크 정책에 따라 달라질 수 있습니다.

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
            // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
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
            // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
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

### 자산 매핑

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `new Builder(int layoutResId)` | — | 네이티브 광고 레이아웃 리소스 ID (**필수**) |
| `setIconImageId(int viewId)` | 미지정 | 아이콘 이미지 View ID |
| `setTitleId(int viewId)` | 미지정 | 제목 TextView ID |
| `setAdvertiserId(int viewId)` | 미지정 | 광고주명 TextView ID |
| `setDescriptionId(int viewId)` | 미지정 | 설명 TextView ID |
| `setMainViewId(int viewId)` | 미지정 | 메인 이미지/동영상 슬롯(**빈 ViewGroup 컨테이너**) View ID |
| `setCtaId(int viewId)` | 미지정 | CTA 버튼 View ID |

> ℹ️ 매핑하지 않은 자산은 렌더링되지 않습니다. 반대로 매핑했더라도 소재에 해당 자산이 없으면 **미디에이션 어댑터(Naver·Pangle·AdManager 등)는 해당 View를 자동으로 `GONE` 처리**합니다. AdMixer 자체 소재는 자동 숨김 대상이 아니므로(위 [구성 Asset] 참고) 누락 케이스 레이아웃을 함께 확인하세요.

### `setAdChoicesPosition(AdChoicesPosition position)`

| 항목 | 내용 |
|------|------|
| **의미** | 광고 정보 고지(AdChoices) 아이콘을 광고 뷰의 **어느 모서리**에 놓을지 지정합니다. |
| **기본값** | `AdChoicesPosition.DEFAULT` = **`RIGHT_TOP`**(우측 상단) |
| **허용 값** | `LEFT_TOP` · `RIGHT_TOP` · `LEFT_BOTTOM` · `RIGHT_BOTTOM` |
| **권장값** | 기본값 `RIGHT_TOP`. 해당 모서리에 앱 UI(닫기 버튼 등)가 겹칠 때만 다른 모서리로 옮기세요. |
| **언제 사용** | 광고 카드 우측 상단에 이미 앱 요소가 있는 레이아웃. |
| **주의사항** | **요청이며 보장이 아닙니다.** 일부 네트워크는 지정 위치를 무시하고 자체 규칙으로 배치합니다. 어느 모서리에 와도 다른 UI를 가리지 않도록 네 모서리 모두에 여백을 확보하세요. `null` 을 넘기면 기본값으로 대체됩니다. |

### `setAddGAMAdAttribute(boolean show)`

| 항목 | 내용 |
|------|------|
| **의미** | 광고임을 알리는 **"Ad" 표시(Ad attribution) 배지**를 SDK가 광고 위에 오버레이할지 여부. |
| **기본값** | `false` (SDK가 배지를 그리지 않음) |
| **권장값** | **레이아웃에 자체 "광고" 라벨이 없다면 `true`** 로 설정하세요. |
| **언제 사용** | 광고 카드에 "AD"·"광고" 같은 표기를 직접 넣지 않은 경우. |
| **주의사항** | 광고 표시는 **네트워크 정책상 요구되는 사항**입니다. 기본값(`false`) 상태에서는 그 의무를 **앱 레이아웃이 부담**합니다. 자체 라벨도 없고 이 옵션도 끄면 정책 위반이 될 수 있습니다. 반대로 자체 라벨이 있는데 `true` 로 켜면 표기가 중복됩니다. 적용 대상 네트워크는 SDK 버전에 따라 달라질 수 있습니다. |

---

## 수신 실패 콜백은 **표준 하나만** 구현하면 됩니다

로드 실패의 표준 콜백은 **`onFailedToReceiveAd(int errorCode, String errorMsg)`** 입니다. 이것 하나만 구현하면 전 네트워크 No-Ad("All adapters failed."), SDK 미초기화·AdUnit 누락 등 **내부 실패를 포함한 모든 수신 실패**를 받습니다. 실패 경로의 네트워크 식별자는 항상 내부 합성값(`"SDK"`/`"Mediation"`)이라 유의미하지 않아 전달하지 않습니다.

> ℹ️ **구버전 호환** — 기존 4-인자 오버로드(`String adapterName` / `AdNetworkType networkType`)는 둘 다 `@Deprecated`(3.0 제거 예정)이며, 기본 구현이 표준 콜백으로 위임하므로 이미 구현해 둔 코드도 동작은 그대로입니다. 신규 코드는 표준 콜백만 구현하세요.

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

SDK는 네이티브를 지원하는 네트워크(AdMixer 자체 / Google AdManager / GMA NextGen / Naver Ad Manager / Pangle / Adfit)
모두에 이 설정을 전달합니다. 다만 **반영되는 방식이 두 가지로 나뉩니다.**

| 반영 방식 | 네트워크 | 설명 |
|---|---|---|
| SDK가 아이콘을 직접 배치 | AdMixer 자체 · Pangle · Naver Ad Manager | 지정한 모서리에 SDK가 오버레이합니다. |
| 네트워크 SDK에 위치 옵션 전달 | Google AdManager · GMA NextGen · Adfit | 광고 요청 시 위치 옵션으로 전달하며, 최종 배치는 각 네트워크 SDK가 결정합니다. |

> ⚠️ **위치 지정은 요청이며 보장이 아닙니다.**
> 위 표에서 **위치 옵션을 네트워크 SDK에 전달하는 그룹**은 최종 배치를 각 네트워크가 결정하므로, 매체사 SDK
> 정책 및 템플릿 규격에 따라 지정한 위치와 다르게 노출될 수 있습니다. 어느 모서리에 아이콘이 놓여도 다른
> UI를 가리지 않도록 네 모서리 모두에 여백을 확보하세요.
>
> 특히 **템플릿형으로 내려오는 소재**(예: Naver Ad Manager의 Native Simple)는 네트워크 SDK가 소재 전체를
> 렌더링하므로 `NativeAdViewBinder` 설정 자체가 적용되지 않습니다.

> ℹ️ **임의 위치는 지원하지 않습니다.** 4개 모서리만 지정할 수 있습니다. 네트워크 SDK마다 아이콘 소유권이
> 달라(일부는 SDK가 자기 뷰 계층에 직접 그림) 모서리 밖 위치는 네트워크 간 동작을 보장할 수 없기 때문입니다.

---

## 네트워크에 따라 달라질 수 있는 항목

네이티브 광고는 **자산(asset)을 조합해 매체가 직접 그리는 포맷**이라, 포맷 중에서 네트워크별 편차가 가장 큽니다. 어떤 자산이 오는지, 무엇을 클릭 가능하게 해야 하는지는 낙찰 네트워크의 정책을 따릅니다. **모든 자산이 항상 온다고 가정하지 마세요.**

| 항목 | 설명 | 앱에서 권장하는 대응 |
|------|------|---------------------|
| **자산 제공 범위** | 제목·본문(body)·아이콘·메인 이미지·CTA·광고주명 중 **일부가 비어 올 수 있습니다.** 어떤 자산이 필수인지는 네트워크마다 다릅니다. | 자산이 비었을 때 해당 뷰를 `GONE` 처리하도록 레이아웃을 설계하세요. 빈 값을 그대로 그리면 빈 칸이 노출됩니다. |
| **아이콘 / 메인 이미지** | 아이콘만 오거나 메인 이미지만 오는 경우가 있습니다. 이미지 비율도 소재마다 다릅니다. | 고정 비율을 강제하지 말고, 없을 때의 대체 레이아웃을 준비하세요. |
| **MediaView(동영상 자산)** | 동영상 자산 지원 여부와 재생 정책(자동재생·음소거·사용자 조작)이 네트워크마다 다를 수 있습니다. | 메인 뷰 영역은 이미지와 동영상 **양쪽이 들어올 수 있는 크기**로 잡으세요. |
| **CTA 문구** | 제공 여부와 문구("자세히 보기", "설치" 등)가 다릅니다. | 앱에서 CTA 문구를 임의로 바꾸지 마세요. 네트워크 정책 위반이 될 수 있습니다. |
| **AdChoices / 광고 정보 고지** | 아이콘 모양·위치·클릭 동작이 네트워크마다 다르며, 일부는 **위치 지정을 무시**하고 자체 규칙으로 배치합니다. | `setAdChoicesPosition()` 은 **요청**이며 보장이 아닙니다. 네 모서리 어디에 와도 다른 UI를 가리지 않도록 여백을 두세요. |
| **클릭 처리 영역** | 어떤 뷰를 클릭 가능하게 등록해야 하는지가 다릅니다. 전체 영역 클릭을 요구하는 네트워크도, CTA만 허용하는 네트워크도 있습니다. | `NativeAdViewBinder` 에 자산을 정확히 매핑하고, 광고 뷰 위에 **자체 클릭 리스너를 덧씌우지 마세요.** 클릭이 집계되지 않거나 정책 위반이 될 수 있습니다. |
| **템플릿형 렌더링** | 일부 네트워크는 자산을 개별 제공하지 않고 **완성된 뷰를 통째로** 내려줍니다. 이 경우 뷰 바인딩이 적용되지 않습니다. | 광고 높이가 소재에 따라 달라질 수 있으므로 컨테이너 높이를 고정하지 마세요. |

> ℹ️ 위 항목은 네트워크 SDK 버전에 따라 변경될 수 있습니다. 지원 여부와 정책은 해당 네트워크 공식 문서를 참고하세요.

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
