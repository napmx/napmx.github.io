# 배너 구현 (iOS Native)

배너 광고는 `NapAdView`를 사용합니다.

---

## 1. 코드로 배너 추가

```swift
import NapSSP

class ViewController: UIViewController {

    private var adView: NapAdView?

    override func viewDidLoad() {
        super.viewDidLoad()
        loadBannerAd()
    }

    func loadBannerAd() {
        adView = NapAdView(frame: CGRect(x: 0, y: 0, width: 320, height: 50))
        adView?.adUnitId = "발급받은_ADUNIT_ID"
        adView?.delegate = self
        view.addSubview(adView!)
        adView?.loadAd()
    }
}

extension ViewController: NapAdViewDelegate {
    func adViewDidReceiveAd(_ adView: NapAdView, adapterName: String) {
        // 광고 수신 성공
    }

    func adView(_ adView: NapAdView, didFailToReceiveAdWithError error: NapAdError) {
        // 광고 수신 실패
        print("Banner error: \(error.code) - \(error.message)")
    }

    func adViewDidRecordClick(_ adView: NapAdView) {
        // 광고 클릭
    }
}
```

```objc
// Objective-C
#import <NapSSP/NapSSP.h>

@interface ViewController () <NapAdViewDelegate>
@property (nonatomic, strong) NapAdView *adView;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.adView = [[NapAdView alloc] initWithFrame:CGRectMake(0, 0, 320, 50)];
    self.adView.adUnitId = @"발급받은_ADUNIT_ID";
    self.adView.delegate = self;
    [self.view addSubview:self.adView];
    [self.adView loadAd];
}

- (void)adViewDidReceiveAd:(NapAdView *)adView adapterName:(NSString *)adapterName { }
- (void)adView:(NapAdView *)adView didFailToReceiveAdWithError:(NapAdError *)error { }

@end
```

---

## 2. Storyboard / XIB에서 사용

Interface Builder에서 `UIView`를 추가하고 Custom Class를 `NapAdView`로 설정한 후, `@IBOutlet`으로 연결합니다.

```swift
@IBOutlet weak var adView: NapAdView!

override func viewDidLoad() {
    super.viewDidLoad()
    adView.adUnitId = "발급받은_ADUNIT_ID"
    adView.delegate = self
    adView.loadAd()
}
```

---

## 3. 생명주기 처리

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    adView?.pause()
}

override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    adView?.resume()
}

deinit {
    adView?.destroy()
}
```

---

## 에러 코드

| 코드 | 설명 |
|------|------|
| `AX_ERR_INIT` | 초기화 오류 |
| `AX_ERR_ADUNIT` | Adunit 오류 |
| `AX_ERR_HTTP` | 네트워크 오류 |
| `AX_ERR_TIMEOUT` | 타임아웃 |
| `AX_ERR_NO_ADS` | 광고 없음 (No Fill) |
