# 배너 구현 (iOS Native)

배너 광고를 추가하기 전 [iOS SDK 시작하기 - Native](/ios/native/getting-started) 설정을 완료해주세요.

---

## 1. 배너(Banner) 광고

배너 광고는 광고 요청 후 즉시 노출하는 방식을 지원합니다.

### 1-1. Banner 뷰 인스턴스 생성 및 설정

배너 광고를 노출할 ViewController에 nap ssp Mediation을 import하여 `AMMBannerAdView` 인스턴스 변수를 생성합니다.

```swift
import NapSSP

class ViewController: UIViewController {

    private var bannerView: AMMBannerAdView?

    override func viewDidLoad() {
        super.viewDidLoad()

        bannerView = AMMBannerAdView(frame: CGRect(x: 0, y: 0, width: 320, height: 50))
        bannerView?.adUnitId = "발급받은_ADUNIT_ID"
        bannerView?.delegate = self
        view.addSubview(bannerView!)
    }
}
```

### 1-2. 광고 요청 및 노출

`load()`를 사용하여 배너 광고를 로드하고 bannerView 영역 내에서 보여줍니다.

```swift
bannerView?.load()
```

### 1-3. 리소스 해제

`stop()`을 사용하여 사용된 리소스를 해제하고 메모리 누수를 방지합니다.

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    bannerView?.stop()
}
```

### 1-4. Delegate

배너 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.  
델리게이트를 사용하려면 `AMMBannerViewDelegate`를 추가해야 합니다.

| 메서드 | 설명 |
|--------|------|
| `onSuccessBanner` | 배너 광고 로드 성공 |
| `onFailBanner` | 배너 광고 로드 실패 |
| `onTapBanner` | 배너 광고 탭 |

```swift
extension ViewController: AMMBannerViewDelegate {

    func onSuccessBanner(_ bannerView: AMMBannerAdView) {
        // 광고 로드 성공
    }

    func onFailBanner(_ bannerView: AMMBannerAdView, error: AMMAdError) {
        print("Banner error: \(error.code) - \(error.message)")
    }

    func onTapBanner(_ bannerView: AMMBannerAdView) {
        // 광고 클릭
    }
}
```

---

## 2. 전면 배너(Interstitial) 광고

전면 배너는 광고 요청 후 받은 뒤, 원하는 시점에 노출하는 방식을 지원합니다.

### 2-1. 네트워크에 따른 전면 광고 형태

| 네트워크 | 소재 | 노출 형태 | 전면 형태 옵션 |
|---------|------|----------|--------------|
| Admixer | 배너 | 레이어 팝업 형태로 배너 노출 | basic, popup, countDown |
| Adfit | 배너 | 레이어 팝업 형태로 배너 노출 | basic, popup, countDown |
| Google | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 | 네트워크사 제공 설정 값 (커스텀 불가) |
| Applovin | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 | 네트워크사 제공 설정 값 (커스텀 불가) |
| Pangle | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 | 네트워크사 제공 설정 값 (커스텀 불가) |
| Unity Ads | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 | 네트워크사 제공 설정 값 (커스텀 불가) |

### 2-2. 전면 배너 인스턴스 생성 및 설정

배너 소재로 응답되는 Admixer, Adfit 네트워크를 사용하는 경우 설정이 필요합니다.

`AMMInterstitial`에서는 전면 배너 형식으로 `basic`, `popup`, `countDown` 세 가지 형태를 제공하며, 일부 네트워크에만 적용됩니다.

| 형식 | 설명 |
|------|------|
| `basic` | 우측 상단에 "X" 이미지 형태의 닫기 버튼 노출 |
| `popup` | 광고 소재 하단에 텍스트 형태의 닫기 버튼 노출 |
| `countDown` | 설정된 시간이 지난 후 닫기 버튼 노출 |

```swift
import NapSSP

class InterstitialViewController: UIViewController {

    private var interstitial: AMMInterstitial?

    override func viewDidLoad() {
        super.viewDidLoad()

        interstitial = AMMInterstitial()
        interstitial?.adUnitId = "발급받은_ADUNIT_ID"
        interstitial?.delegate = self

        // popup 형식 설정 (닫기 버튼 텍스트, 타이틀 색상, 버튼 배경색)
        interstitial?.popupOption = AMMInterstitialPopupOption(
            closeText: "닫기",
            titleColor: .black,
            buttonBackgroundColor: .white
        )

        // countDown 형식 설정
        // countDownTime: 카운트다운 시간 (최소 2초 ~ 최대 5초)
        // countDownType: .gauge (게이지 형태), .text (텍스트 형태)
        interstitial?.countDownOption = AMMInterstitialCountDownOption(
            countDownTime: 3,
            countDownType: .gauge
        )

        // 닫기 버튼 터치 영역 비율 (0.2 ~ 1.0, default: 1.0)
        // basic, countDown 형에만 적용됩니다.
        interstitial?.closeButtonTouchAreaRatio = 1.0
    }
}
```

### 2-3. 광고 요청

`load()`를 사용하여 전면 배너 광고를 로드합니다.

```swift
interstitial?.load()
```

### 2-4. 광고 노출

`show()`를 사용하여 로드된 광고를 보여줍니다.

```swift
interstitial?.show(from: self)
```

### 2-5. 리소스 해제

```swift
interstitial?.stop()
```

### 2-6. Delegate

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowInterstitial` | 전면 광고 노출 성공 |
| `onFailShowInterstitial` | 전면 광고 노출 실패 |
| `onTapInterstitial` | 전면 광고 탭 |
| `onCloseInterstitial` | 전면 광고 닫기 |

```swift
extension InterstitialViewController: AMMInterstitialDelegate {

    func onSuccessShowInterstitial(_ interstitial: AMMInterstitial) {
        // 광고 노출 성공
    }

    func onFailShowInterstitial(_ interstitial: AMMInterstitial, error: AMMAdError) {
        print("Interstitial error: \(error.code)")
    }

    func onTapInterstitial(_ interstitial: AMMInterstitial) {
        // 광고 클릭
    }

    func onCloseInterstitial(_ interstitial: AMMInterstitial) {
        // 광고 닫힘
    }
}
```
