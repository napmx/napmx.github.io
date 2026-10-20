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

`loadAd()`를 호출하여 리워드 동영상 광고를 로드합니다.

```swift
class RewardedAdViewController: UIViewController {

    var rewardVideo: AMMRewardVideo?

    override func viewDidLoad() {
        super.viewDidLoad()

        let customParam = ["userid": "nas", "name": "hdragon", "phone": "010-1111-1111"]

        // ADUNIT_ID: 발급받은 광고 단위 ID (Int)
        AMMRewardVideo.loadAd(adUnitID: ADUNIT_ID, customParam: customParam) { [weak self] reward, adapterType, error in
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
| `onClickRewardVideo()` | 리워드 동영상 광고 클릭 |
| `onRewardVideoComplete()` | 재생 완료 — 일부 네트워크는 미발화, **지급 판단에 사용 금지** |
| `onRewardVideoEarned(rewardInfo:)` | **리워드 지급 완료 (권장)** — `rewardInfo.transactionId` 로 지급 건별 고유 ID 수신 (v2.4.3+) |

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

    func onClickRewardVideo() {
        // 광고 클릭
    }

    func onRewardVideoComplete() {
        // 재생 완료 (일부 네트워크 미발화)
    }

    func onRewardVideoEarned(rewardInfo: RewardInfo) {
        // 리워드 지급 처리 — rewardInfo.transactionId 로 지급 건별 고유 ID 수신
    }
}
```

---

## 보상 지급 안전 수칙

리워드 지급 누락·중복은 대부분 아래 두 가지에서 발생합니다. 지급 로직을 배선하기 전에 반드시 확인하세요.

### 1. 지급 판단은 `onRewardVideoEarned(rewardInfo:)` 하나만 사용

- `rewardInfo.transactionId`(노출당 고유 UUID)를 자체 서버 지급의 멱등 키로 사용하면 중복 적립을 방지할 수 있습니다. 같은 값이 S2S Reward Callback 의 `transaction_id` 파라미터로도 전달됩니다.
- `onRewardVideoComplete()` 는 일부 네트워크에서 발화하지 않는 재생 완료 통지입니다 — **지급 판단에 사용하지 마세요.**

### 2. 보상·닫힘 이벤트 순서

- 보상(earn)·재생 완료(complete)·닫힘(close) 이벤트를 언제, 어떤 순서로 보낼지는 각 광고 네트워크 SDK가 결정합니다. 미디에이션 SDK는 이를 전달받아 앱에 중계하는 위치에 있습니다.
- 따라서 보상과 닫힘 중 어느 쪽이 먼저 도착할지는 네트워크와 상황에 따라 달라질 수 있습니다. 특정 순서를 전제하지 마세요.

| 상황 | 앱이 받는 콜백 |
|---|---|
| 보상 조건을 충족한 경우 | 보상·닫힘 모두 도착 (도착 순서는 네트워크에 따라 다름) |
| 보상 조건을 충족하지 못한 경우 (미완주·스킵) | 닫힘만 도착 — 보상 콜백 없음 |
| 표시에 실패한 경우 | `onFailShowReward(error:)` — 보상·닫힘 콜백 없음 |

**SDK가 보장하는 것**
- 지급 통지는 노출 1회당 **정확히 1회**입니다
- 네트워크가 보낸 보상 콜백을 SDK가 무기한 지연하거나 유실하지 않습니다.
- **네트워크가 보내지 않은 이벤트를 SDK가 합성하지 않습니다.** 닫힘·재생 완료를 지어내지 않습니다. 특히 재생 완료는 '완주'라는 사실이고 보상은 '보상 조건 충족'이라 의미가 달라, 합성하면 매체의 완주율 지표가 왜곡됩니다.

> ⚠️ **콜백 도착 순서를 전제로 지급·UI 로직을 작성하지 마세요.**
> 지급 여부는 **순서가 아니라 플래그로 판정**하세요

> ⚠️ 닫힘 이후에 보상이 도착하는 동안 앱이 광고를 파기(`stop()`)하면 **클라이언트 보상 콜백을 받지 못할 수 있습니다.** 서버 지급까지 보장해야 한다면 서버 검증(S2S)을 병행하세요.

### 3. 지급 시점과 사용자 알림(UI) 시점을 분리 — 순서 대신 플래그로 판정

보상 콜백은 영상이 끝나기 전에 도착할 수 있고(광고 화면 위에는 앱 UI 를 띄울 수 없음), 보상·닫힘 중 무엇이 먼저 도착할지는 네트워크에 따라 다릅니다. 두 플래그를 두고 **나중에 도착한 쪽이 알림을 띄우도록** 작성하면 순서와 무관하게 성립합니다.

```swift
private var rewardGranted = false  // 보상 지급 완료 여부
private var adDismissed   = false  // 광고 닫힘(앱 복귀) 여부

func showRewardAd() {
    rewardGranted = false          // ✅ show 직전에 반드시 초기화 — 노출 1회 = 지급 1회
    adDismissed   = false
    rewardVideo?.show(rootViewController: self)
}

func onRewardVideoEarned(rewardInfo: RewardInfo) {
    guard !rewardGranted else { return }              // 중복 도착 방어
    rewardGranted = true
    giveRewardToUser(rewardInfo.transactionId)        // ✅ 지급은 즉시 (유실 방지)
    if adDismissed { showRewardNotice() }             // 닫힘이 먼저였다면 지금 알림
    // ❌ 광고가 아직 떠 있는 상태의 알림은 가려지므로 여기서 무조건 띄우지 않습니다
}

func onCloseRewardVideo() {
    adDismissed = true
    if rewardGranted { showRewardNotice() }           // 보상이 먼저였다면 지금 알림
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
| `ifa_use` | 광고 식별자(IFA) 사용 가능 여부 — ATT 미승인 시 `0` (자동 포함, v2.4.3+) | `1` |
| `timestamp` | 리워드 지급 이벤트 발생 시간 (자동 포함) | `1546300800` |
| `transaction_id` | 지급 건별 고유 ID — 중복 지급 방지 멱등 키, `rewardInfo.transactionId` 와 동일 값 (자동 포함, v2.4.3+) | `11111111-2222-3333-4444-555555555555` |

### 설정 2: SDK CustomParm 추가

CustomParm을 통해 콜백에서 추가 데이터를 수집할 수 있습니다. Dictionary 형태로 추가해야 합니다.

```swift
let customParam = ["userid": "nas", "name": "hdragon", "phone": "010-1111-1111"]
AMMRewardVideo.loadAd(adUnitID: ADUNIT_ID, customParam: customParam) { reward, adapterType, error in
    // ...
}
```
