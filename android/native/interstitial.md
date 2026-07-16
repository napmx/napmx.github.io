# 전면 배너 광고

> ℹ️ 전면 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

전면 광고는 `AMMInterstitial`을 사용하여 화면 전체를 덮는 형태의 광고를 표시합니다.

> ℹ️ 전면 광고는 **정적 `loadAd()` → 로드된 광고 객체 반환 → `FullScreenContentCallback` 노출 콜백** 구조를 사용합니다.
>
> 구 `InterstitialAd` 클래스는 제거되었습니다 — `AMMInterstitial` 정적 `loadAd()`로 전환하세요.
> v1.x에서 업그레이드하는 경우 [마이그레이션 가이드](migration.md)와 이 페이지 하단의 "구 API에서 전환" 표를 참고하세요.

---

## 전면 광고 형태

전면 광고는 **기본형(Basic)** 을 제공합니다 — 우측 상단 "X" 아이콘으로 닫는 전체 화면 형태입니다. (별도 타입 설정 없이 기본 동작)

---

## 호출 흐름

```
AMMInterstitial.loadAd(context, adInfo, callback)
    → onSuccessLoadInterstitial(networkType, ad)   ← 로드된 광고 객체 전달
        → ad.setFullScreenContentCallback(...)     ← 노출/클릭/닫힘/표시실패 콜백 등록
        → ad.show(activity)                        ← 원하는 시점에 노출 (Activity 필요)
    → onFailLoadInterstitial(errorCode, errorMsg)  ← 로드 실패
```

- **로드**는 정적 `AMMInterstitial.loadAd(...)`로 시작합니다(인스턴스 생성 불필요).
- **노출**은 콜백으로 받은 `ad` 객체의 `show(activity)`로 수행합니다. 콜백 안에서 즉시 호출하면 **즉시 노출**, 객체를 보관했다가 나중에 호출하면 **지연 노출(Load-Only)** 입니다.
- 노출 단계 이벤트(노출/클릭/닫힘/표시 실패)는 `FullScreenContentCallback`으로 수신합니다.


> ℹ️ **커스텀 어댑터를 쓰는 경우** — `AdMixer.registerAdapter(String)`로 등록한 어댑터는 `AdNetworkType`에 등재되어 있지 않아, 로드 **성공** 콜백이 `String adapterName` 오버로드로만 통지됩니다. 커스텀 어댑터를 사용한다면 그 오버로드도 함께 구현하세요. (로드 **실패** `onFailLoad*`는 어댑터와 무관하게 항상 호출되므로 별도 조치가 필요 없습니다.)

---

## 기본 사용법

### 1단계: AdInfo 구성

```java
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .build();
```

### 2단계: 전면 광고 로드 및 노출

#### Java
```java
public class InterstitialActivity extends AppCompatActivity {

    // 정적 load() 콜백으로 받은 로드 완료 광고 객체 (지연 노출/정리를 위해 보관)
    private AMMInterstitial loadedAd;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitial);

        // 광고 표시 버튼 (원하는 시점에 로드+노출)
        Button btnShow = findViewById(R.id.btn_show_interstitial);
        btnShow.setOnClickListener(v -> loadAndShowInterstitial());
    }

    private void loadAndShowInterstitial() {
        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
            .build();

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체를 받는다
        AMMInterstitial.loadAd(this, adInfo, new AMMInterstitialLoadCallback() {
            @Override
            public void onSuccessLoadInterstitial(@NonNull AdNetworkType networkType, @NonNull AMMInterstitial ad) {
                // 광고 수신 성공 (networkType: 응답한 광고 네트워크)
                loadedAd = ad;

                // 노출 단계 콜백 등록 (GAM FullScreenContentCallback과 동일 구조)
                ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                    @Override public void onAdShowedFullScreenContent() {
                        // 광고가 화면에 표시됨 (= 임프레션)
                    }
                    @Override public void onAdClicked() {
                        // 광고 소재 클릭
                    }
                    @Override public void onAdDismissedFullScreenContent() {
                        // 광고 닫힘 — 참조 해제
                        loadedAd = null;
                    }
                    @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                        // 노출 실패 (code/message 확인 가능)
                        loadedAd = null;
                    }
                });

                // 노출 (Activity Context 필요)
                ad.show(InterstitialActivity.this);
            }

            @Override
            public void onFailLoadInterstitial(int errorCode, @Nullable String errorMsg) {
                // 광고 수신 실패
            }
        });
    }

    @Override
    protected void onDestroy() {
        // 객체 해제 (필수)
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
class InterstitialActivity : AppCompatActivity() {

    private var loadedAd: AMMInterstitial? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_interstitial)

        findViewById<Button>(R.id.btn_show_interstitial).setOnClickListener {
            loadAndShowInterstitial()
        }
    }

    private fun loadAndShowInterstitial() {
        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
            .build()

        AMMInterstitial.loadAd(this, adInfo, object : AMMInterstitialLoadCallback() {
            override fun onSuccessLoadInterstitial(networkType: AdNetworkType, ad: AMMInterstitial) {
                loadedAd = ad
                // Kotlin은 프로퍼티 접근(ad.fullScreenContentCallback = ...)도 가능
                ad.fullScreenContentCallback = object : FullScreenContentCallback() {
                    override fun onAdShowedFullScreenContent() { /* 노출됨 */ }
                    override fun onAdClicked() { /* 클릭 */ }
                    override fun onAdDismissedFullScreenContent() { loadedAd = null }
                    override fun onAdFailedToShowFullScreenContent(adError: AdError) { loadedAd = null }
                }
                ad.show(this@InterstitialActivity)
            }

            override fun onFailLoadInterstitial(errorCode: Int, errorMsg: String?) {
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

## 지연 노출 (Load-Only)

광고를 미리 로드해 두었다가 원하는 시점에 노출하려면, `onSuccessLoadInterstitial`에서 `show()`를 호출하지 말고 **광고 객체만 보관**한 뒤 나중에 `show(activity)`를 호출하세요. `hasInterstitial` 필드로 노출 가능 여부를 확인할 수 있습니다.

```java
// 1) 미리 로드: 콜백에서 show() 하지 않고 보관만
AMMInterstitial.loadAd(this, adInfo, new AMMInterstitialLoadCallback() {
    @Override public void onSuccessLoadInterstitial(@NonNull AdNetworkType networkType, @NonNull AMMInterstitial ad) {
        loadedAd = ad;
        loadedAd.setFullScreenContentCallback(fullScreenCallback);
        // show() 호출하지 않음 → 지연 노출 대기
    }
    @Override public void onFailLoadInterstitial(int errorCode, @Nullable String errorMsg) { }
});

// 2) 원하는 시점에 노출
if (loadedAd != null && loadedAd.hasInterstitial) {
    loadedAd.show(this);
}
```

> ℹ️ 모든 로드는 "로드 후 콜백 반환"만 수행하며, 앱이 `show()`를 호출하기 전까지는 노출되지 않습니다.

---

## FullScreenContentCallback 이벤트

`setFullScreenContentCallback(...)`로 등록하며, 필요한 메서드만 오버라이드합니다(모두 기본 빈 구현). GAM(Google Mobile Ads)의 `FullScreenContentCallback`과 동일한 구조/네이밍입니다.

| 콜백 | 발생 시점 | 비고 |
|------|----------|------|
| `onAdShowedFullScreenContent()` | 광고가 풀스크린으로 표시됨 (임프레션) | |
| `onAdClicked()` | 광고 소재 클릭 | |
| `onAdDismissedFullScreenContent()` | 광고 닫힘 | |
| `onAdFailedToShowFullScreenContent(AdError)` | 노출 실패 | `AdError.getCode()` / `getMessage()` |

> ℹ️ **표시 결과는 광고당 정확히 1회만 전달됩니다.** `show()` 이후 `onAdShowedFullScreenContent()`(표시됨) 또는 `onAdFailedToShowFullScreenContent()`(표시 실패) 중 **하나만** 최종 결과로 수신합니다. 네트워크(어댑터)가 표시 결과를 통지하지 않아 일정 시간 내 아무 신호가 없으면 SDK가 표시 실패를 백스톱으로 전달합니다. 표시 실패가 전달된 뒤 뒤늦게 도착하는 상반된 콜백(닫힘/표시됨/중복 실패)은 억제되므로, **이중·모순 콜백을 방어할 필요가 없습니다.**

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `interstitialTimeout(int)` | `0` (서버 지정, 기본 20초) | 광고 로딩 타임아웃 (초) |
| `setCloseButtonBound(int)` | `100` | 닫기(X) 버튼 터치 영역 비율(20~100%) |

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| 화면 전환·백그라운드 (표시 광고 유지) | `loadedAd.cancelLoad()` | 진행 중 **로드만 취소** (표시 중이면 no-op) |
| `Activity.onDestroy()` | `loadedAd.stop()` | 광고 객체 해제 (필수) |

> ℹ️ `cancelLoad()`는 "로드만 취소", `stop()`은 "전체 정리(리스너 해제 포함)"입니다. 표시 중인 광고를 끊지 않고 미완료 로드만 중단할 때 `cancelLoad()`를 사용하세요. (리워드·전면 동영상도 동일하게 `cancelLoad()` 제공)

> ⚠️ `show(activity)`는 **Activity Context**가 필요합니다. `load()`는 Application Context로도 가능하나, 노출 시점에는 Activity가 필요합니다.

---

## 구 API에서 전환 (전면 광고 · v1.x.x → v2)

구 `InterstitialAd` 클래스는 v2에서 제거되었습니다. 아래 매핑을 참고해 `AMMInterstitial` 정적 `loadAd()`로 전환하세요. 전체 마이그레이션 절차는 [마이그레이션 가이드](migration.md)를 참고하세요.

| v1.x.x (제거됨) | v2.0.0 |
|---|---|
| `new InterstitialAd(context)` | (인스턴스 생성 불필요) `AMMInterstitial.loadAd(context, adInfo, callback)` |
| `setAdInfo(adInfo)` | `loadAd(...)`의 `adInfo` 인자로 전달 |
| `setAdListener(AdListener)` + `onReceivedAd` | `AMMInterstitialLoadCallback.onSuccessLoadInterstitial(networkType, ad)` |
| `onFailedToReceiveAd(...)` | `onFailLoadInterstitial(errorCode, errorMsg)` |
| `loadInterstitial()` / `startInterstitial()` | `AMMInterstitial.loadAd(...)` (노출은 `ad.show(activity)`) |
| `showInterstitial()` | `ad.show(activity)` |
| `onEventAd(AdEvent.DISPLAYED)` | `FullScreenContentCallback.onAdShowedFullScreenContent()` |
| `onEventAd(AdEvent.CLICK)` | `onAdClicked()` |
| `onEventAd(AdEvent.CLOSE)` | `onAdDismissedFullScreenContent()` |
| (노출 실패 신호 없음) | `onAdFailedToShowFullScreenContent(AdError)` |
| `stopInterstitial()` | `stop()` |
