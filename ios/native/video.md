# 동영상 광고

동영상 광고를 추가하기 전 [시작하기](/ios/native/getting-started) 설정을 완료해주세요.

> Interstitial Ad 광고 연동을 희망하시는 경우, [배너 - 전면 배너](/ios/native/banner#2-전면-배너interstitial-광고)를 연동해주세요.

---

## 동영상(Video) 광고

동영상 광고는 광고 요청 후 즉시 노출하는 방식을 지원합니다.

### 광고 요청 및 노출

`AMMVideoView.load()`로 동영상 광고를 요청합니다. 로드가 완료되면 completion으로 동영상 뷰를 전달받으며, 이를 화면에 `addSubview`하면 노출됩니다.

```swift
import AdMixerMediation

class VideoAdViewController: UIViewController {

    var ammVideoView: AMMVideoView?

    override func viewDidLoad() {
        super.viewDidLoad()

        // ADUNIT_ID: 발급받은 광고 단위 ID (Int)
        AMMVideoView.load(adUnitID: ADUNIT_ID, rootViewController: self) { [weak self] video, adapterName, error in
            guard let self = self else { return }

            if let error = error {
                // 광고 로드 실패
                return
            }

            guard let video = video else { return }
            self.ammVideoView = video
            video.delegate = self
            // addSubview 시점(화면 진입)에 자동으로 노출됩니다
            self.addVideoViewToView(video)
        }
    }

    func addVideoViewToView(_ videoView: UIView) {
        videoView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(videoView)
        NSLayoutConstraint.activate([
            videoView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            videoView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            videoView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -50),
            videoView.heightAnchor.constraint(equalToConstant: 200)
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
        ammVideoView?.stop()
        ammVideoView = nil
    }
}
```

### Delegate

동영상 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.  
델리게이트를 사용하려면 `AMMVideoViewDelegate`를 추가해야 합니다.

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowVideo` | 동영상 광고 노출 성공 |
| `onClickVideo` | 동영상 광고 클릭(더보기) |
| `onSkipVideo` | 동영상 광고 내 skip 버튼 클릭 |
| `onCompleteVideo` | 동영상 광고 재생 완료 |

> 광고 로드 성공/실패는 `load()`의 completion(`video`, `error`)으로 전달됩니다.

```swift
extension VideoAdViewController: AMMVideoViewDelegate {

    func onSuccessShowVideo() {
        // 광고 노출 성공
    }

    func onClickVideo() {
        // 더보기 클릭
    }

    func onSkipVideo() {
        // skip 버튼 클릭
    }

    func onCompleteVideo() {
        // 재생 완료
    }
}
```

---

## 전면 동영상(Interstitial Video) 광고

전면 동영상 광고는 광고 요청 후 받은 뒤, 원하는 시점에 노출하는 방식을 지원합니다.

> 전면 동영상 재생 시 Skip 버튼은 제공되지 않으며, Skip 가능한 시점에 닫기 버튼이 노출됩니다.

### 전면 동영상 뷰 인스턴스 생성 및 설정

전면 동영상 광고를 노출할 ViewController에 nap mx Mediation을 import하여 `AMMVideoInterstitial` 인스턴스 변수를 생성합니다.

```swift
import AdMixerMediation

class VideoInterstitialViewController: UIViewController {

    var videoInterstitial: AMMVideoInterstitial?
}
```

### 광고 요청

`load()`를 사용하여 전면 동영상 광고를 로드합니다.

```swift
class VideoInterstitialViewController: UIViewController {

    var videoInterstitial: AMMVideoInterstitial?

    override func viewDidLoad() {
        super.viewDidLoad()

        // ADUNIT_ID: 발급받은 광고 단위 ID (Int)
        AMMVideoInterstitial.load(adUnitID: ADUNIT_ID) { [weak self] videoInterstitial, adapterName, error in
            guard let self = self else { return }

            if let error = error {
                print("AMMVideoInterstitial error: \(error)")
            }

            if let videoInterstitial = videoInterstitial {
                self.videoInterstitial = videoInterstitial
                self.videoInterstitial?.delegate = self
            }
        }
    }
}
```

### 광고 노출

`show()`를 사용하여 로드된 전면 동영상 광고를 보여줍니다.

```swift
videoInterstitial?.show(rootViewController: self)
```

### 리소스 해제

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        videoInterstitial?.stop()
        videoInterstitial = nil
    }
}
```

### Delegate

델리게이트를 사용하려면 `AMMVideoInterstitialDelegate`를 추가해야 합니다.

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowVideoInterstitial` | 전면 동영상 광고 노출 성공 |
| `onFailShowVideoInterstitial` | 전면 동영상 광고 노출 실패 |
| `onCloseVideoInterstitial` | 전면 동영상 광고 닫기 |
| `onClickVideoInterstitial` | 전면 동영상 광고 내 더보기 버튼 클릭 |
| `onCompleteVideoInterstitial` | 전면 동영상 광고 재생 완료 |

```swift
extension VideoInterstitialViewController: AMMVideoInterstitialDelegate {

    func onSuccessShowVideoInterstitial() {
        // 광고 노출 성공
    }

    func onFailShowVideoInterstitial(error: Error?) {
        // 광고 노출 실패
    }

    func onCloseVideoInterstitial() {
        // 광고 닫힘
    }

    func onClickVideoInterstitial() {
        // 더보기 버튼 클릭
    }

    func onCompleteVideoInterstitial() {
        // 재생 완료
    }
}
```
