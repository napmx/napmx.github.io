# 전면 배너 광고

> **📌 참고** 전면 광고 추가 전, [SDK 시작하기](getting-started.md)의 Step 1~4 설정이 완료되었는지 확인하세요.

전면 광고는 `InterstitialAd`를 사용하여 화면 전체를 덮는 형태의 광고를 표시합니다.

---

## 전면 광고 형태 (3종)

| 형식 | `InterstitialAdType` | 설명 |
|------|---------------------|------|
| **기본형 (Basic)** | `InterstitialAdType.Basic` | 우측 상단에 "X" 아이콘 형태 닫기 버튼 노출 |
| **팝업형 (Popup)** | `InterstitialAdType.Popup` | 광고 하단에 텍스트 형태 닫기 버튼 노출. 배경색 커스터마이징 가능 |
| **카운트다운형 (CountDown)** | `InterstitialAdType.CountDown` | 설정된 시간 후 닫기 버튼 노출 (`gauge` / `text` 타입 선택) |

---

## 기본 사용법

### 1단계: AdInfo 구성

**기본형 (Basic)**

```java
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .interstitialAdType(AdInfo.InterstitialAdType.Basic)
    .build();
```

**팝업형 (Popup)**

```java
PopupInterstitialAdOption adOption = new PopupInterstitialAdOption();
adOption.setDisableBackKey(false);              // 뒤로가기 버튼으로 닫기 허용 여부 (false: 허용)
adOption.setButtonLeft("닫기", "#333333");       // 닫기(왼쪽) 버튼 텍스트, 색상
adOption.setButtonRight("앱 종료", null);        // 앱 종료(오른쪽) 버튼 (null: 사용 안 함)
adOption.setButtonFrameColor("#FFFFFF");         // 버튼 영역 배경색 (null: 기본값)

AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .interstitialAdType(AdInfo.InterstitialAdType.Popup)
    .popupAdOption(adOption)
    .setUseBackgroundAlpha(true)                 // 배경 반투명 처리 여부
    .build();
```

**카운트다운형 (CountDown)**

```java
PopupInterstitialAdOption adOption = new PopupInterstitialAdOption();
adOption.setCountDown(AdMixer.AX_COUNT_TYPE_GAUGE, 5); // AX_COUNT_TYPE_GAUGE=0(게이지), AX_COUNT_TYPE_TEXT=1(텍스트) / time: 2~5초

AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
    .interstitialAdType(AdInfo.InterstitialAdType.CountDown)
    .popupAdOption(adOption)
    .build();
```

### 2단계: 전면 광고 로드 및 노출

**Java**

```java
public class InterstitialActivity extends AppCompatActivity {

    private InterstitialAd interstitialAd;

    private final AdListener adListener = new AdListener() {
        @Override
        public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
            // 광고 수신 성공 — hasInterstitial 플래그가 true로 설정됨
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
                case CLOSE:
                    // 광고 닫힘 (닫기 버튼 클릭 또는 뒤로가기)
                    break;
                case LEFT_CLICK:
                    // 팝업형: 왼쪽(닫기) 버튼 클릭
                    break;
                case RIGHT_CLICK:
                    // 팝업형: 오른쪽(앱 종료) 버튼 클릭
                    finish(); // 앱 종료 처리
                    break;
                case CLICK:
                    // 광고 소재 클릭
                    break;
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_interstitial);

        // AdInfo 구성 (팝업형 예시)
        PopupInterstitialAdOption adOption = new PopupInterstitialAdOption();
        adOption.setButtonLeft("닫기", "#333333");
        adOption.setDisableBackKey(false);

        AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
            .interstitialAdType(AdInfo.InterstitialAdType.Popup)
            .popupAdOption(adOption)
            .setUseBackgroundAlpha(true)
            .build();

        // InterstitialAd 생성
        interstitialAd = new InterstitialAd(this);
        interstitialAd.setAdInfo(adInfo);
        interstitialAd.setAdListener(adListener);
        interstitialAd.loadAd(); // 광고 로드 시작

        // 광고 표시 버튼 (원하는 시점에 노출)
        Button btnShow = findViewById(R.id.btn_show_interstitial);
        btnShow.setOnClickListener(v -> {
            if (interstitialAd.hasInterstitial) {
                interstitialAd.showInterstitial();
            } else {
                // 아직 로드 중 — 필요 시 재요청
                interstitialAd.loadAd();
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

**Kotlin**

```kotlin
class InterstitialActivity : AppCompatActivity() {

    private var interstitialAd: InterstitialAd? = null

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
                AdEvent.CLOSE -> { /* 닫힘 */ }
                AdEvent.LEFT_CLICK -> { /* 왼쪽 버튼 */ }
                AdEvent.RIGHT_CLICK -> finish() // 앱 종료
                else -> {}
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_interstitial)

        val adOption = PopupInterstitialAdOption().apply {
            setButtonLeft("닫기", "#333333")
            setDisableBackKey(false)
        }

        val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_INTERSTITIAL)
            .interstitialAdType(AdInfo.InterstitialAdType.Popup)
            .popupAdOption(adOption)
            .setUseBackgroundAlpha(true)
            .build()

        interstitialAd = InterstitialAd(this).apply {
            setAdInfo(adInfo)
            setAdListener(adListener)
            loadAd()
        }

        findViewById<Button>(R.id.btn_show_interstitial).setOnClickListener {
            if (interstitialAd?.hasInterstitial == true) {
                interstitialAd?.showInterstitial()
            }
        }
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

광고 수신 즉시 화면에 표시하려면 `onReceivedAd` 콜백에서 `showInterstitial()`을 호출하세요.

```java
@Override
public void onReceivedAd(@NonNull String adapterName, @NonNull Object adView) {
    // 수신 즉시 표시
    if (interstitialAd != null && interstitialAd.hasInterstitial) {
        interstitialAd.showInterstitial();
    }
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
| `closeButtonDelay(int)` | `0` | 닫기 버튼 지연 노출 시간 (초) |

## PopupInterstitialAdOption 레퍼런스

| 메서드 | 설명 |
|--------|------|
| `setDisableBackKey(boolean)` | 뒤로가기 버튼으로 닫기 비활성화 여부 |
| `setButtonLeft(String text, String color)` | 왼쪽(닫기) 버튼 텍스트와 색상 (필수) |
| `setButtonRight(String text, String color)` | 오른쪽 버튼 텍스트와 색상 (선택, null: 미표시) |
| `setButtonFrameColor(String color)` | 버튼 영역 배경색 (null: 기본값) |
| `setCountDown(int type, int time)` | 카운트다운 타입 (`AX_COUNT_TYPE_GAUGE`=0 게이지, `AX_COUNT_TYPE_TEXT`=1 텍스트), 시간 (2~5초) |

---

## 라이프사이클 관리

| 시점 | 호출 메서드 | 역할 |
|------|------------|------|
| `Activity.onDestroy()` | `interstitialAd.stopInterstitial()` | 광고 객체 해제 (필수) |

> **⚠️ 주의** `interstitialAd.showInterstitial()`은 **Activity Context**가 필요합니다. Application Context만 있는 상태에서는 호출되지 않습니다.
