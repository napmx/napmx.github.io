# 배너 광고

배너 광고를 추가하기 전 [시작하기](/ios/native/getting-started) 설정을 완료해주세요.

---

## 배너(Banner) 광고

배너 광고는 광고 요청 후 즉시 노출하는 방식을 지원합니다.

### 광고 요청 및 노출

`AMMBannerView.load()`로 배너 광고를 요청합니다. 로드가 완료되면 completion으로 배너 뷰를 전달받으며, 이를 화면에 `addSubview`하면 노출됩니다.

```swift
import AdMixerMediation

class ViewController: UIViewController {

    var bannerView: AMMBannerView?

    override func viewDidLoad() {
        super.viewDidLoad()

        // ADUNIT_ID: 발급받은 광고 단위 ID (Int)
        AMMBannerView.load(adUnitID: ADUNIT_ID, rootViewController: self) { [weak self] banner, adapterName, error in
            guard let self = self else { return }

            if let error = error {
                // 광고 로드 실패
                return
            }

            guard let banner = banner else { return }
            self.bannerView = banner
            banner.delegate = self
            // addSubview 시점(화면 진입)에 자동으로 노출됩니다
            self.addBannerViewToView(banner)
        }
    }

    func addBannerViewToView(_ bannerView: UIView) {
        bannerView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(bannerView)
        NSLayoutConstraint.activate([
            bannerView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            bannerView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            bannerView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -50),
        ])
    }
}
```

### 리소스 해제

`stop()`을 사용하여 사용된 리소스를 해제하고 메모리 누수를 방지합니다.

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        bannerView?.stop()
        bannerView = nil
    }
}
```

### Delegate

배너 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.  
델리게이트를 사용하려면 `AMMBannerViewDelegate`를 추가해야 합니다.

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowBanner` | 배너 광고 노출 성공 |
| `onClickBanner` | 배너 광고 클릭 |

> 광고 로드 성공/실패는 `load()`의 completion(`banner`, `error`)으로 전달됩니다.

```swift
extension ViewController: AMMBannerViewDelegate {

    func onSuccessShowBanner() {
        // 광고 노출 성공
    }

    func onClickBanner() {
        // 광고 클릭
    }
}
```

---

## 전면 배너(Interstitial) 광고

전면 배너는 광고 요청 후 받은 뒤, 원하는 시점에 노출하는 방식을 지원합니다.

### 네트워크에 따른 전면 광고 형태

| 네트워크 | 소재 | 노출 형태 |
|---------|------|----------|
| Admixer | 배너 | 레이어 팝업 형태로 배너 노출 |
| Adfit | 배너 | 레이어 팝업 형태로 배너 노출 |
| Google | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 |
| Applovin | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 |
| Pangle | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 |
| Unity Ads | 짧은 초수의 영상 또는 배너 | 풀스크린 노출 |

### 전면 배너 인스턴스 생성 및 설정

배너 소재로 응답되는 Admixer, Adfit 네트워크를 사용하는 경우, 닫기 버튼의 터치 영역 비율을 설정할 수 있습니다.

```swift
import AdMixerMediation

class InterstitialViewController: UIViewController {

    var interstitial: AMMInterstitial?

    override func viewDidLoad() {
        super.viewDidLoad()

        let config = AMMInterstitialConfig()

        // 닫기 버튼 터치 영역 비율 (0.0 ~ 1.0, default: 0.6)
        config.closeButtonTouchAreaRatio = 0.6
    }
}
```

### 광고 요청

`load()`를 사용하여 전면 배너 광고를 로드합니다.

```swift
AMMInterstitial.load(adUnitID: "ADUNIT_ID", config: config) { [weak self] interstitial, error in
    guard let self = self else { return }

    if let error = error {
        print("AMMInterstitial error: \(error)")
    }

    if let interstitial = interstitial {
        self.interstitial = interstitial
        self.interstitial?.delegate = self
    }
}
```

### 광고 노출

`show()`를 사용하여 로드된 광고를 보여줍니다.

```swift
interstitial?.show(rootViewController: self)
```

### 리소스 해제

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        interstitial?.stop()
        interstitial = nil
    }
}
```

### Delegate

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowInterstitial` | 전면 광고 노출 성공 |
| `onFailShowInterstitial` | 전면 광고 노출 실패 |
| `onTapInterstitial` | 전면 광고 탭 |
| `onCloseInterstitial` | 전면 광고 닫기 |

```swift
extension InterstitialViewController: AMMInterstitialDelegate {

    func onSuccessShowInterstitial() {
        // 광고 노출 성공
    }

    func onFailShowInterstitial(error: Error?) {
        // 광고 노출 실패
    }

    func onTapInterstitial() {
        // 광고 클릭
    }

    func onCloseInterstitial() {
        // 광고 닫힘
    }
}
```
