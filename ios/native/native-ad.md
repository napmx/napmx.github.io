# 네이티브 광고 (iOS Native)

네이티브 광고는 앱 UI에 자연스럽게 녹아드는 광고 형식입니다.  
`NapNativeAdLoader`로 광고를 로드하고, 에셋을 앱의 커스텀 뷰에 직접 바인딩합니다.

---

## 1. 광고 로드

```swift
import NapSSP

class NativeAdViewController: UIViewController {

    private var nativeAdLoader: NapNativeAdLoader?

    override func viewDidLoad() {
        super.viewDidLoad()
        loadNativeAd()
    }

    func loadNativeAd() {
        nativeAdLoader = NapNativeAdLoader(adUnitId: "발급받은_ADUNIT_ID")
        nativeAdLoader?.delegate = self
        nativeAdLoader?.load()
    }
}

extension NativeAdViewController: NapNativeAdLoaderDelegate {

    func nativeAdLoader(_ loader: NapNativeAdLoader,
                        didReceive nativeAd: NapNativeAd, adapterName: String) {
        // 에셋 바인딩
        titleLabel.text = nativeAd.headline
        bodyLabel.text = nativeAd.body
        ctaButton.setTitle(nativeAd.callToAction, for: .normal)
        if let iconImage = nativeAd.icon?.image {
            iconImageView.image = iconImage
        }
        // 광고 뷰 등록 (노출 트래킹)
        nativeAd.register(adContainer: adContainerView, clickableViews: [ctaButton])
    }

    func nativeAdLoader(_ loader: NapNativeAdLoader,
                        didFailToReceiveAdWithError error: NapAdError) {
        print("Native Ad error: \(error.code) - \(error.message)")
    }
}
```

---

## 2. 광고 에셋

| 프로퍼티 | 타입 | 설명 |
|---------|------|------|
| `headline` | `String?` | 광고 제목 |
| `body` | `String?` | 광고 본문 |
| `callToAction` | `String?` | CTA 버튼 텍스트 |
| `icon` | `NapNativeAdImage?` | 아이콘 이미지 |
| `mediaView` | `UIView?` | 미디어(이미지/영상) 뷰 |
| `advertiser` | `String?` | 광고주 이름 |

---

## 3. 정리

```swift
deinit {
    nativeAdLoader?.destroy()
}
```
