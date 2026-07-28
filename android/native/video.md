# 동영상 광고

> ℹ️ 동영상 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

> ℹ️ 동영상이 아닌 일반 전면 광고(Interstitial Ad)를 연동하려면 [전면 배너 광고](interstitial.md)를 참고하세요.

| 포맷 | 클래스 | 설명 |
|------|--------|------|
| 인라인 동영상 | `AMMVideoView` | 앱 화면 내에 인라인으로 재생 |
| 전면 동영상 | `AMMVideoInterstitial` | 화면 전체를 덮는 전면 동영상 |

> ℹ️ 리워드 지급이 필요한 전면 동영상은 [리워드 동영상 광고](rewarded-video.md)를 참고하세요.

> ℹ️ **전면 동영상**은 `AMMVideoInterstitial`의 정적 `loadAd()` + `FullScreenContentCallback` 구조를, **인라인 동영상**은 화면 내 View인 `AMMVideoView`의 `AdListener` 모델을 사용합니다.
>
> v1.x에서 업그레이드하는 경우 [마이그레이션 가이드](migration.md)를 먼저 참고하세요. 포맷별 상세 매핑은 이 페이지 하단의 "구 API에서 전환" 표에 있습니다.

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
            // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
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
            // 수신 실패(최종) — 내부 No-Ad 포함 모든 실패가 이 콜백 하나로 옵니다
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

## 수신 실패 콜백은 **표준 하나만** 구현하면 됩니다

로드 실패의 표준 콜백은 **`onFailedToReceiveAd(int errorCode, String errorMsg)`** 입니다. 이것 하나만 구현하면 전 네트워크 No-Ad("All adapters failed."), SDK 미초기화·AdUnit 누락 등 **내부 실패를 포함한 모든 수신 실패**를 받습니다. 실패 경로의 네트워크 식별자는 항상 내부 합성값(`"SDK"`/`"Mediation"`)이라 유의미하지 않아 전달하지 않습니다.

> ℹ️ **구버전 호환** — 기존 4-인자 오버로드(`String adapterName` / `AdNetworkType networkType`)는 둘 다 `@Deprecated`(3.0 제거 예정)이며, 기본 구현이 표준 콜백으로 위임하므로 이미 구현해 둔 코드도 동작은 그대로입니다. 신규 코드는 표준 콜백만 구현하세요.

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

### `setMute(boolean mute)`

| 항목 | 내용 |
|------|------|
| **의미** | 동영상 시작 시 음소거 여부. `true` = 무음으로 시작. |
| **기본값** | `false` (소리 켜짐) |
| **권장값** | 사용자가 예상하지 못한 소리를 싫어하는 지면(피드·목록 안의 인라인 동영상)에서는 `true` 를 권장합니다. |
| **언제 사용** | 자동 재생되는 인라인 동영상, 또는 앱이 이미 오디오를 재생 중인 화면. |
| **주의사항** | **요청이며 보장이 아닙니다.** 음소거 지원 여부와 사용자 조작 우선순위는 네트워크 정책에 따라 달라질 수 있습니다. 무음 재생을 전제로 한 UX(예: 자막 필수)는 피하세요. |

### `setVideoType(String)` · `setVideoPlacement(String)`

| 항목 | 내용 |
|------|------|
| **의미** | 동영상 요청 시 서버에 전달하는 **재생 유형·지면 위치** 힌트입니다. |
| **기본값** | 미지정(`null`) — 서버가 애드유닛 설정으로 결정 |
| **권장값** | **미지정(호출하지 않음)을 권장합니다.** |
| **언제 사용** | 파트너 사이트 설정과 다른 값을 보내야 한다고 별도로 안내받은 경우에만. |
| **주의사항** | 애드유닛 설정과 어긋나면 재고 매칭이 줄어 노필이 늘 수 있습니다. 값 체계는 서버 정책을 따르므로 임의로 지정하지 마세요. |

> ℹ️ **`setCustomParams(Map)`** 는 리워드 S2S 콜백용입니다 — [리워드 동영상 가이드](rewarded-video.md) 참고.
> **`setAdapterConfig(String, Map)`** 는 서버에 네트워크 키가 없을 때 앱에서 주입하는 용도입니다 — [Q&A](qna.md) 참고.


---

## 이벤트 레퍼런스

**인라인 동영상 (`AMMVideoView` — AdListener 콜백)**

| 콜백 | 발생 시점 |
|--------|----------|
| `onAdDisplayed()` | 광고가 화면에 표시됨 |
| `onAdCompleted()` | 동영상이 끝까지 재생됨 (네트워크에 따라 발화하지 않을 수 있음 — 아래 참고) |
| `onAdSkipped()` | 사용자가 Skip 버튼 클릭 |
| `onAdClosed()` | 광고 창 닫힘 |
| `onAdClicked()` | 광고 내 링크(더보기 등) 클릭 |

**전면 동영상 (`AMMVideoInterstitial` — FullScreenContentCallback)**

| 콜백 | 발생 시점 |
|------|----------|
| `onAdShowedFullScreenContent()` | 광고가 화면에 표시됨 (임프레션) |
| `onAdClicked()` | 광고 내 링크 클릭 |
| `onAdCompleted()` | 동영상 재생 완료 (네트워크에 따라 발화하지 않을 수 있음 — 아래 참고) |
| `onAdDismissedFullScreenContent()` | 광고 창 닫힘 |
| `onAdFailedToShowFullScreenContent(AdError)` | 노출 실패 |

> ⚠️ **재생 완료(`onAdCompleted`)에 의존하는 로직은 구성하지 마세요.** 재생 완료를 별도 이벤트로 통지하는 네트워크가 있는 반면, 적립·종료 신호만 전달하고 완료 콜백을 독립적으로 발화하지 않는 네트워크도 있습니다. 완료 여부로 화면 전환·보상 처리 등 비즈니스 로직을 분기하지 마시고, **종료 처리는 닫힘 콜백**(인라인 `onAdClosed()` / 전면 `onAdDismissedFullScreenContent()`)을 단일 복귀 지점으로 삼으세요.

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

## 네트워크에 따라 달라질 수 있는 항목

동영상 광고는 재생 제어 권한이 대부분 **네트워크 SDK 쪽에 있습니다.** 앱이 재생을 직접 제어할 수 있다고 가정하지 마세요.

| 항목 | 설명 | 앱에서 권장하는 대응 |
|------|------|---------------------|
| **음소거(mute)** | `setMute()` 는 **요청**이며, 네트워크가 자체 정책으로 무시하거나 사용자 조작을 우선할 수 있습니다. | 무음 재생을 전제로 한 UX(예: 자막 필수)를 만들지 마세요. |
| **스킵(skip)** | 스킵 버튼 제공 여부와 노출 시점이 네트워크 정책을 따릅니다. 스킵 콜백이 오지 않을 수도 있습니다. | 스킵 여부로 앱 로직을 분기하지 말고, 닫힘 콜백을 종료 신호로 사용하세요. |
| **재생 완료 통지** | 완료를 별도 이벤트로 통지할지 여부가 다릅니다. **발화하지 않을 수 있습니다.** | 완주 판정을 `onAdCompleted()` 에만 의존하지 마세요. |
| **화면 방향(orientation)** | 지원 방향과 회전 시 동작이 다릅니다. 회전 중 재생이 중단될 수 있습니다. | 재생 구간 방향 고정을 검토하세요. |
| **인라인 동영상 크기** | 소재 비율에 따라 높이가 달라질 수 있습니다. | 컨테이너 높이를 고정하지 말고 비율 기반으로 잡으세요. |
| **백그라운드 전환** | 앱이 백그라운드로 갈 때 일시정지/중단/계속 중 무엇을 하는지가 다릅니다. | `onPause()`/`onResume()` 를 반드시 연결하세요(아래 라이프사이클 참고). |

> ℹ️ 지원 여부와 정책은 네트워크 SDK 버전에 따라 변경될 수 있습니다. 최신 내용은 해당 네트워크 공식 문서를 참고하세요.

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
