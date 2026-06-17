# 리워드 동영상 광고

리워드 동영상 광고를 추가하기 전 [시작하기](/ios/native/getting-started) 설정을 완료해주세요.

리워드 동영상 광고는 광고 요청 후 받은 뒤, 원하는 시점에 노출하는 방식을 지원합니다.

---

## 리워드 동영상 인스턴스 생성 및 설정

광고를 노출할 ViewController에 `AMMRewardVideo` 인스턴스 변수를 생성합니다.

```swift
import AdMixerMediation

class RewardedAdViewController: UIViewController {

    var rewardVideo: AMMRewardVideo?
}
```

---

## 광고 요청

`load()`를 호출하여 리워드 동영상 광고를 로드합니다.

```swift
class RewardedAdViewController: UIViewController {

    var rewardVideo: AMMRewardVideo?

    override func viewDidLoad() {
        super.viewDidLoad()

        let customParam = ["userid": "nas", "name": "hdragon", "phone": "010-1111-1111"]

        AMMRewardVideo.load(adUnitID: "ADUNIT_ID", customParam: customParam) { [weak self] reward, error in
            guard let self = self else { return }

            if let error = error {
                print("AMMRewardVideo error: \(error)")
            }

            if let reward = reward {
                self.rewardVideo = reward
                self.rewardVideo?.delegate = self
            }
        }
    }
}
```

---

## 광고 노출

`show()`를 호출하여 로드된 리워드 동영상 광고를 보여줍니다.

```swift
rewardVideo?.show(rootViewController: self)
```

---

## 리소스 해제

`stop()`을 사용하여 사용된 리소스를 해제하고 메모리 누수를 방지합니다.

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        rewardVideo?.stop()
        rewardVideo = nil
    }
}
```

---

## Delegate

리워드 동영상 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.  
델리게이트를 사용하려면 `AMMRewardVideoDelegate`를 추가해야 합니다.

> 광고 네트워크 별로 이벤트가 상이할 수 있습니다. (일부 네트워크는 재생 완료 이벤트 제공 X)

| 메서드 | 설명 |
|--------|------|
| `onSuccessShowReward()` | 리워드 동영상 광고 노출 성공 |
| `onFailShowReward()` | 리워드 동영상 광고 노출 실패 |
| `onCloseRewardVideo()` | 리워드 동영상 광고 닫기 |
| `onTapRewardVideo()` | 리워드 동영상 광고 탭 |
| `onRewardVideoComplete()` | 리워드 동영상 재생 완료 |
| `onRewardVideoEarned()` | 리워드 동영상 리워드 지급 완료 |

```swift
extension RewardedAdViewController: AMMRewardVideoDelegate {

    func onSuccessShowReward() {
        // 광고 노출 성공
    }

    func onFailShowReward(error: Error?) {
        // 광고 노출 실패
    }

    func onCloseRewardVideo() {
        // 광고 닫힘
    }

    func onTapRewardVideo() {
        // 광고 클릭
    }

    func onRewardVideoComplete() {
        // 재생 완료
    }

    func onRewardVideoEarned() {
        // 리워드 지급 처리
    }
}
```

---

## S2S Reward Callback (선택사항)

매체사가 정의한 외부 서버로 해당 유저에게 리워드 지급이 완료되었음을 전달하는 기능입니다.  
콜백 수신까지 몇 분 정도 지연될 수 있습니다.

### 설정 1: 파트너 사이트에서 콜백 서버 URL 입력

**파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정**에서 매체사의 콜백 서버 URL을 입력합니다.

기본 파라미터는 아래와 같습니다.

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| `media_key` | 미디어 키 (자동 포함) | `12345678` |
| `adunit_id` | 애드유닛 ID (자동 포함) | `87654321` |
| `ifa` | iOS 기기 고유 식별자 (자동 포함) | `860635ea-65bc-eaed-d355-1b5283b30b94` |
| `timestamp` | 리워드 지급 이벤트 발생 시간 (자동 포함) | `1546300800` |

### 설정 2: SDK CustomParm 추가

CustomParm을 통해 콜백에서 추가 데이터를 수집할 수 있습니다. Dictionary 형태로 추가해야 합니다.

```swift
let customParam = ["userid": "nas", "name": "hdragon", "phone": "010-1111-1111"]
AMMRewardVideo.load(adUnitID: "ADUNIT_ID", customParam: customParam) { reward, error in
    // ...
}
```
