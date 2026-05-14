# 동영상 광고 (iOS Native)

동영상 광고는 `NapVideoAdView`를 사용합니다.

---

## 광고 로드

```swift
import NapSSP

class VideoAdViewController: UIViewController {

    private var videoAdView: NapVideoAdView?

    override func viewDidLoad() {
        super.viewDidLoad()

        videoAdView = NapVideoAdView(frame: CGRect(x: 0, y: 0, width: view.bounds.width, height: 200))
        videoAdView?.adUnitId = "발급받은_ADUNIT_ID"
        videoAdView?.delegate = self
        view.addSubview(videoAdView!)
        videoAdView?.loadAd()
    }
}

extension VideoAdViewController: NapVideoAdViewDelegate {

    func videoAdViewDidReceiveAd(_ videoAdView: NapVideoAdView, adapterName: String) {
        // 동영상 수신 완료
    }

    func videoAdView(_ videoAdView: NapVideoAdView, didFailToReceiveAdWithError error: NapAdError) {
        print("Video error: \(error.code)")
    }

    func videoAdViewDidStartPlaying(_ videoAdView: NapVideoAdView) {
        // 재생 시작
    }

    func videoAdViewDidCompletePlayback(_ videoAdView: NapVideoAdView) {
        // 재생 완료
    }
}
```

---

## 생명주기 처리

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    videoAdView?.pause()
}

deinit {
    videoAdView?.destroy()
}
```
