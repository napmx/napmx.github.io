# 네이티브 광고 (iOS Native)

네이티브 광고를 추가하기 전 [iOS SDK 시작하기 - Native](/ios/native/getting-started) 설정을 완료해주세요.

네이티브 광고는 광고 요청 후 즉시 노출하는 방식을 지원합니다.

---

## 1. 구성

네이티브 광고는 6가지 asset으로 구성되어 있으며, 각 asset을 사용하여 자유롭게 UI를 구성할 수 있습니다.

![네이티브 광고 구성](/resources/ios_native_assets.png)

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

## 2. 네이티브 인스턴스 생성 및 설정

**Step 1. AMMNativeAdView.xib 파일 추가**

아래의 `AMMNativeAdView.xib` 파일을 프로젝트에 추가합니다. (사이즈별 권장 xib 파일을 사용해주세요.)

<a href="/resources/AMMNativeAdView.xib" download>📎 AMMNativeAdView.xib 다운로드</a>

**Step 2. 인스턴스 변수 생성**

광고를 노출할 ViewController에 `AMMNativeAdViewContainer` 인스턴스 변수를 생성합니다.

```swift
import AdMixerMediation

class NativeAdViewController: UIViewController {

    var nativeAd: AMMNativeAdViewContainer!

    override func viewDidLoad() {
        super.viewDidLoad()

        let nibView = Bundle.main.loadNibNamed("AMMNativeAdView", owner: nil, options: nil)?.first
        let nativeAdView = nibView as? AMMNativeAdView

        nativeAd = AMMNativeAdViewContainer(rootViewController: self)
        nativeAd.nativeAdView = nativeAdView
        nativeAd.adUnitID = "ADUNIT_ID"
        nativeAd.delegate = self
    }
}
```

---

## 3. 광고 요청

`load()`를 호출하여 네이티브 광고를 로드하고 보여줍니다.

```swift
nativeAd.load()
```

---

## 4. 리소스 해제

`stop()`을 사용하여 사용된 리소스를 해제하고 메모리 누수를 방지합니다.

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        nativeAd.stop()
        nativeAd = nil
    }
}
```

---

## 5. Delegate

네이티브 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.  
델리게이트를 사용하려면 `AMMNativeDelegate`를 추가해야 합니다.

| 메서드 | 설명 |
|--------|------|
| `onSuccessNative()` | 네이티브 광고 호출 성공 |
| `onFailNative()` | 네이티브 광고 호출 실패 |
| `onTapNative()` | 네이티브 광고 탭 |

```swift
extension NativeAdViewController: AMMNativeDelegate {

    func onSuccessNative() {
        // 광고 로드 성공
    }

    func onFailNative() {
        // 광고 로드 실패
    }

    func onTapNative() {
        // 광고 클릭
    }
}
```
