# 네이티브 광고

네이티브 광고를 추가하기 전 [시작하기](/ios/native/getting-started) 설정을 완료해주세요.

네이티브 광고는 광고 요청 후 즉시 노출하는 방식을 지원합니다.

---

## 구성 Asset

네이티브 광고는 6가지 asset으로 구성되어 있으며, 각 asset을 사용하여 자유롭게 UI를 구성할 수 있습니다.

<img src="/resources/ios_native_assets.png" alt="네이티브 광고 구성" width="480">

| Asset | 설명 |
|-------|------|
| `icon` | 아이콘 이미지 |
| `headline` | 광고 제목 |
| `advertiser` | 광고주 이름 |
| `description` | 광고 설명 |
| `media` | 미디어 (이미지 또는 동영상) |
| `cta` | 행동 유도 버튼 |

> `headline`(제목), `icon`(아이콘), `media`(미디어) 중 **최소 1개**는 반드시 사용해야 합니다.

---

## 네이티브 인스턴스 생성 및 설정

**Step 1. AMMNativeAdView.xib 파일 추가**

아래의 `AMMNativeAdView.xib` 파일을 프로젝트에 추가합니다. 광고 사이즈에 맞는 권장 xib 파일을 사용해주세요.

| 사이즈 | 다운로드 |
|--------|----------|
| 기본 | <a href="/resources/AMMNativeAdView.xib" download>📎 AMMNativeAdView.xib</a> |
| 320x50 | <a href="/resources/AMMNativeAdView320x50.xib" download>📎 AMMNativeAdView320x50.xib</a> |
| 320x100 | <a href="/resources/AMMNativeAdView320x100.xib" download>📎 AMMNativeAdView320x100.xib</a> |
| 300x250 | <a href="/resources/AMMNativeAdView300x250.xib" download>📎 AMMNativeAdView300x250.xib</a> |
| 320x480 | <a href="/resources/AMMNativeAdView320x480.xib" download>📎 AMMNativeAdView320x480.xib</a> |

> xib 파일의 루트 뷰 클래스는 `AMMNativeAdView`로 동일하므로, 코드에서는 `loadNibNamed("AMMNativeAdView", ...)` 대신 추가한 파일명(예: `"AMMNativeAdView300x250"`)으로 로드해주세요.

**Step 2. 광고 요청 및 노출**

`AMMNativeAdView.xib`를 로드해 `nativeAdView`를 만들고, `AMMNativeAdViewContainer.loadAd()`에 전달합니다. 로드가 완료되면 completion으로 전달받은 컨테이너를 `addSubview`하면 노출됩니다.

```swift
import AdMixerMediation

class NativeAdViewController: UIViewController {

    var nativeAd: AMMNativeAdViewContainer?

    override func viewDidLoad() {
        super.viewDidLoad()

        let nibView = Bundle.main.loadNibNamed("AMMNativeAdView", owner: nil, options: nil)?.first
        guard let nativeAdView = nibView as? AMMNativeAdView else { return }

        // ADUNIT_ID: 발급받은 광고 단위 ID (Int)
        AMMNativeAdViewContainer.loadAd(adUnitID: ADUNIT_ID, rootViewController: self, nativeAdView: nativeAdView) { [weak self] container, adapterType, error in
            guard let self = self else { return }

            if let error = error {
                // 광고 로드 실패
                return
            }

            guard let container = container else { return }
            self.nativeAd = container
            container.delegate = self
            self.view.addSubview(container)
        }
    }
}
```

---

## 리소스 해제

`stop()`을 사용하여 사용된 리소스를 해제하고 메모리 누수를 방지합니다.

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        nativeAd?.stop()
        nativeAd = nil
    }
}
```

---

## Delegate

네이티브 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.  
델리게이트를 사용하려면 `AMMNativeDelegate`를 추가해야 합니다.

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowNative()` | 네이티브 광고 노출 성공 |
| `onClickNative()` | 네이티브 광고 클릭 |

> 광고 로드 성공/실패는 `loadAd()`의 completion(`container`, `adapterType`, `error`)으로 전달됩니다. `adapterType`은 로드된 네트워크(`AdNetworkType`)입니다.

```swift
extension NativeAdViewController: AMMNativeDelegate {

    func onSuccessShowNative() {
        // 광고 노출 성공
    }

    func onClickNative() {
        // 광고 클릭
    }
}
```
