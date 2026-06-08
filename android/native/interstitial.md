# 전면 배너 광고

> ℹ️ 전면 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

전면 광고는 `AMMInterstitial`을 사용하여 화면 전체를 덮는 형태의 광고를 표시합니다.

---

## 전면 광고 형태 (3종)

| 형식 | `InterstitialAdType` | 설명 |
|------|---------------------|------|
| **기본형 (Basic)** | `InterstitialAdType.Basic` | 우측 상단에 "X" 아이콘 형태 닫기 버튼 노출 |
| **팝업형 (Popup)** | `InterstitialAdType.Popup` | 광고 하단에 텍스트 형태 닫기 버튼 노출. 배경색 커스터마이징 가능 |
| **카운트다운형 (CountDown)** | `InterstitialAdType.CountDown` | 설정된 시간 후 닫기 버튼 노출 (`gauge` / `text` 타입 선택) |

---

## ⚠️ 호출 방식 — 정적 `load()` + `FullScreenContentCallback` (v2.0.0)

> 🚨 **v2.0.0부터 전면 광고는 정적 `AMMInterstitial.load()`로 로드합니다.** 로드가 완료되면 콜백(`onSuccessLoadInterstitial`)으로 로드 완료된 광고 객체가 전달됩니다. 이 객체에 `FullScreenContentCallback`을 등록한 뒤 `show(activity)`로 노출하세요.
>
> | 단계 | API | 설명 |
> |---|---|---|
> | 로드 | `AMMInterstitial.load(context, adInfo, callback)` | 정적 메서드. 로드 완료 시 콜백으로 광고 객체 전달 |
> | 노출 | `ad.show(activity)` | 콜백에서 받은 광고 객체로 노출 (Activity Context 필요) |
> | 이벤트 | `ad.setFullScreenContentCallback(...)` | 노출/클릭/닫힘/표시실패 수신 |
>
> **로드 즉시 노출**하려면 `onSuccessLoadInterstitial` 안에서 바로 `ad.show(activity)`를 호출하고, **미리 로드 후 원하는 시점에 노출**하려면 광고 객체를 멤버 변수에 보관했다가 버튼 클릭 시 `show()`를 호출하세요. 정적 `load()`는 1회성 호출이므로 v1.x의 무한 워터폴 루프(수신 콜백 내 재로드) 문제가 구조적으로 발생하지 않습니다.

---

## 뒤로가기(BACK) 키 정책

> ⚠️ **v2.0.0 동작 변경**: 전면 광고는 시스템 **뒤로가기(BACK) 키를 기본 차단**합니다. 광고는 'X' 닫기 버튼으로만 닫히며, 뒤로가기로 임의 종료되지 않습니다(비디오·리워드와 동일 정책).
>
> 뒤로가기로 닫기를 **허용**하려면 광고 유형에 따라 아래처럼 명시적으로 해제하세요.
>
> **Basic 전면** — `AdInfo.Builder.setDisableBackKey(false)` (공통 옵션):
> ```java
> AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
>         .setInterstitialAdType(AdInfo.InterstitialAdType.Basic)
>         .setDisableBackKey(false) // 명시적 false → 뒤로가기로 닫기 허용
>         .build();
> ```
>
> **Popup / CountDown 전면** — `PopupInterstitialAdOption.setDisableBackKey(false)` (팝업 옵션이 우선):
> ```java
> PopupInterstitialAdOption opt = new PopupInterstitialAdOption();
> opt.setDisableBackKey(false); // 명시적 false → 뒤로가기로 닫기 허용
> ```
>
> 기존(v1.x)에 뒤로가기 닫기에 의존하던 매체는 위와 같이 `setDisableBackKey(false)`를 명시해야 종전 동작이 유지됩니다.
>
> ℹ️ Android 13(API 33)+ 예측형 뒤로가기(predictive back)가 켜진 앱(예: `targetSdk 35`)에서도 위 차단이 정상 적용됩니다.

---

## 기본 사용법

### 1단계: AdInfo 구성

#### 기본형 (Basic)
```java
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .interstitialAdType(AdInfo.InterstitialAdType.Basic)
    .build();
```

#### 팝업형 (Popup)
```java
PopupInterstitialAdOption adOption = new PopupInterstitialAdOption();
adOption.setButtonLeft("닫기", "#333333");       // 닫기(왼쪽) 버튼 텍스트, 색상
adOption.setButtonRight("앱 종료", null);        // 앱 종료(오른쪽) 버튼 (null: 사용 안 함)
adOption.setButtonFrameColor("#FFFFFF");         // 버튼 영역 배경색 (null: 기본값)

AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .interstitialAdType(AdInfo.InterstitialAdType.Popup)
    .popupAdOption(adOption)
    .setUseBackgroundAlpha(true)                 // 배경 반투명 처리 여부
    .build();
```

#### 카운트다운형 (CountDown)
```java
PopupInterstitialAdOption adOption = new PopupInterstitialAdOption();
adOption.setCountDown(AdMixer.AX_COUNT_TYPE_GAUGE, 5); // AX_COUNT_TYPE_GAUGE=0(게이지), AX_COUNT_TYPE_TEXT=1(텍스트) / time: 2~5초

AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .interstitialAdType(AdInfo.InterstitialAdType.CountDown)
    .popupAdOption(adOption)
    .build();
```

### 2단계: 전면 광고 로드 및 노출

#### Java
```java
public class InterstitialActivity extends AppCompatActivity {

    private AMMInterstitial interstitialAd; // load() 콜백으로 받은 로드 완료 광고

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitial);

        // 화면 진입 시 미리 로드
        loadInterstitial();

        // 광고 표시 버튼 (원하는 시점에 노출)
        Button btnShow = findViewById(R.id.btn_show_interstitial);
        btnShow.setOnClickListener(v -> {
            if (interstitialAd != null && interstitialAd.hasInterstitial) {
                interstitialAd.show(this);   // 로드 완료된 광고 노출
            } else {
                loadInterstitial();          // 아직 로드 전이면 재요청
            }
        });
    }

    private void loadInterstitial() {
        // AdInfo 구성 (팝업형 예시)
        PopupInterstitialAdOption adOption = new PopupInterstitialAdOption();
        adOption.setButtonLeft("닫기", "#333333");
        adOption.setButtonRight("앱 종료", null); // null: 오른쪽 버튼 미사용

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
            .interstitialAdType(AdInfo.InterstitialAdType.Popup)
            .popupAdOption(adOption)
            .setUseBackgroundAlpha(true)
            .build();

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체 전달
        AMMInterstitial.load(this, adInfo, new AMMInterstitialLoadCallback() {
            @Override
            public void onSuccessLoadInterstitial(@NonNull String adapterName,
                    @NonNull AMMInterstitial ad) {
                // 광고 수신 성공 — 노출/클릭/닫힘은 FullScreenContentCallback로 수신
                interstitialAd = ad;
                ad.setFullScreenContentCallback(new FullScreenContentCallback() {
                    @Override public void onAdShowedFullScreenContent() { /* 노출됨 */ }
                    @Override public void onAdClicked() { /* 광고 소재 클릭 */ }
                    @Override public void onAdDismissedFullScreenContent() {
                        // 광고 닫힘 — 객체 정리
                        interstitialAd = null;
                    }
                    @Override public void onAdFailedToShowFullScreenContent(@NonNull AdError adError) {
                        // 노출 단계 실패
                        interstitialAd = null;
                    }
                    // 팝업형 좌/우 버튼은 SDK가 자동으로 닫지 않으므로 앱이 직접 처리
                    @Override public void onAdLeftClicked() {
                        if (interstitialAd != null) interstitialAd.stopInterstitial(); // 닫기
                    }
                    @Override public void onAdRightClicked() {
                        finish(); // 앱 종료
                    }
                });
                // 로드 즉시 노출하려면 여기서 ad.show(InterstitialActivity.this); 를 호출하세요.
            }

            @Override
            public void onFailLoadInterstitial(int errorCode, String errorMsg) {
                // 광고 수신 실패
                interstitialAd = null;
            }
        });
    }

    @Override
    protected void onDestroy() {
        if (interstitialAd != null) {
            interstitialAd.stopInterstitial();
            interstitialAd = null;
        }
        super.onDestroy();
    }
}
```

#### Kotlin
```kotlin
class InterstitialActivity : AppCompatActivity() {

    private var interstitialAd: AMMInterstitial? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_interstitial)

        loadInterstitial()

        findViewById<Button>(R.id.btn_show_interstitial).setOnClickListener {
            val ad = interstitialAd
            if (ad != null && ad.hasInterstitial) {
                ad.show(this)           // 로드 완료된 광고 노출
            } else {
                loadInterstitial()      // 재요청
            }
        }
    }

    private fun loadInterstitial() {
        val adOption = PopupInterstitialAdOption().apply {
            setButtonLeft("닫기", "#333333")
            setButtonRight("앱 종료", null)
        }

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
            .interstitialAdType(AdInfo.InterstitialAdType.Popup)
            .popupAdOption(adOption)
            .setUseBackgroundAlpha(true)
            .build()

        // 정적 load() — 로드 완료 시 콜백으로 광고 객체 전달
        AMMInterstitial.load(this, adInfo, object : AMMInterstitialLoadCallback() {
            override fun onSuccessLoadInterstitial(adapterName: String, ad: AMMInterstitial) {
                interstitialAd = ad
                ad.setFullScreenContentCallback(object : FullScreenContentCallback() {
                    override fun onAdShowedFullScreenContent() { /* 노출됨 */ }
                    override fun onAdClicked() { /* 클릭 */ }
                    override fun onAdDismissedFullScreenContent() { interstitialAd = null }
                    override fun onAdFailedToShowFullScreenContent(adError: AdError) { interstitialAd = null }
                    override fun onAdLeftClicked() { interstitialAd?.stopInterstitial() } // 닫기
                    override fun onAdRightClicked() { finish() }                          // 앱 종료
                })
                // 로드 즉시 노출하려면 여기서 ad.show(this@InterstitialActivity) 호출
            }

            override fun onFailLoadInterstitial(errorCode: Int, errorMsg: String?) {
                interstitialAd = null
            }
        })
    }

    override fun onDestroy() {
        interstitialAd?.stopInterstitial()
        interstitialAd = null
        super.onDestroy()
    }
}
```

---

## 즉시 노출 방식

광고 수신 즉시 화면에 표시하려면 `onSuccessLoadInterstitial` 콜백에서 바로 `show(activity)`를 호출하세요.

```java
@Override
public void onSuccessLoadInterstitial(@NonNull String adapterName, @NonNull AMMInterstitial ad) {
    interstitialAd = ad;
    ad.setFullScreenContentCallback(fullScreenContentCallback);
    ad.show(InterstitialActivity.this); // 수신 즉시 노출
}
```

---

## AdInfo 옵션 레퍼런스

| 메서드 | 기본값 | 설명 |
|--------|--------|------|
| `interstitialAdType(InterstitialAdType)` | `Basic` | 전면 광고 형태 선택 |
| `popupAdOption(PopupInterstitialAdOption)` | `null` | 팝업/카운트다운형 옵션 설정 |
| `setUseBackgroundAlpha(boolean)` | `true` | 배경 반투명 처리 여부 |
| `interstitialTimeout(int)` | `0` (서버 지정, 기본 20초) | 광고 로딩 타임아웃 (초) |
| `showCloseButton(boolean)` | `true` | 닫기(X) 버튼 표시 여부 |
| `setCloseButtonBound(int)` | `100` | Basic/CountDown 전면 닫기(X) 버튼 터치 영역 비율 (20~100%) |

## PopupInterstitialAdOption 레퍼런스

| 메서드 | 설명 |
|--------|------|
| `setDisableBackKey(boolean)` | **Popup/CountDown** 뒤로가기 닫기 차단 여부. **기본값 `true`(차단)** — `false` 설정 시에만 허용. (Basic 전면은 `AdInfo.Builder.setDisableBackKey` 사용) |
| `setButtonLeft(String text, String color)` | 왼쪽(닫기) 버튼 텍스트와 색상 (필수) |
| `setButtonRight(String text, String color)` | 오른쪽 버튼 텍스트와 색상 (선택, null: 미표시) |
| `setButtonFrameColor(String color)` | 버튼 영역 배경색 (null: 기본값) |
| `setCountDown(int type, int time)` | 카운트다운 타입 (`AX_COUNT_TYPE_GAUGE`=0 게이지, `AX_COUNT_TYPE_TEXT`=1 텍스트), 시간 (2~5초) |

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| 화면 전환·백그라운드 (표시 광고 유지) | `interstitialAd.cancelLoad()` | 진행 중 **로드만 취소** (표시 중이면 no-op) |
| `Activity.onDestroy()` | `interstitialAd.stopInterstitial()` | 광고 객체 해제 (필수) |

> ℹ️ `cancelLoad()`는 "로드만 취소", `stopInterstitial()`은 "전체 정리(리스너 해제 포함)"입니다. 표시 중인 광고를 끊지 않고 미완료 로드만 중단할 때 `cancelLoad()`를 사용하세요. (리워드·전면 동영상도 동일하게 `cancelLoad()` 제공)

> ⚠️ `interstitialAd.show(activity)`는 **Activity Context**가 필요합니다. Application Context만 있는 상태에서는 호출되지 않습니다.
