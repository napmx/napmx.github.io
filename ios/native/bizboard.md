# 비즈보드 (iOS Native)

비즈보드는 네이버 비즈보드 광고 형식을 지원합니다.  
`NapBizboardAdView`를 사용합니다.

---

## 광고 로드

```swift
import NapSSP

class BizboardViewController: UIViewController {

    private var bizboardAdView: NapBizboardAdView?

    override func viewDidLoad() {
        super.viewDidLoad()
        loadBizboardAd()
    }

    func loadBizboardAd() {
        bizboardAdView = NapBizboardAdView(frame: CGRect(x: 0, y: 0,
                                                         width: view.bounds.width, height: 100))
        bizboardAdView?.adUnitId = "발급받은_ADUNIT_ID"
        bizboardAdView?.delegate = self
        view.addSubview(bizboardAdView!)
        bizboardAdView?.loadAd()
    }
}

extension BizboardViewController: NapBizboardAdViewDelegate {

    func bizboardAdViewDidReceiveAd(_ adView: NapBizboardAdView, adapterName: String) {
        // 수신 완료
    }

    func bizboardAdView(_ adView: NapBizboardAdView,
                        didFailToReceiveAdWithError error: NapAdError) {
        print("Bizboard error: \(error.code)")
    }
}
```

---

## 주의사항

- 비즈보드는 **NaverAdManager** 어댑터가 필요합니다.
- 네이버 성과형DA 계정 및 별도 계약이 필요합니다.
- 문의: [nap_adx@nasmedia.co.kr](mailto:nap_adx@nasmedia.co.kr)
