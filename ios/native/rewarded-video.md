# 리워드 동영상 (iOS Native)

리워드 동영상은 `NapRewardedAd`를 사용합니다.

---

## 광고 로드 및 표시

```swift
import NapSSP

class RewardedAdViewController: UIViewController {

    private var rewardedAd: NapRewardedAd?

    override func viewDidLoad() {
        super.viewDidLoad()
        loadRewardedAd()
    }

    func loadRewardedAd() {
        NapRewardedAd.load(adUnitId: "발급받은_ADUNIT_ID") { [weak self] ad, error in
            if let error = error {
                print("Rewarded failed: \(error)")
                return
            }
            self?.rewardedAd = ad
            self?.rewardedAd?.delegate = self
        }
    }

    func showRewardedAd() {
        rewardedAd?.present(from: self)
    }
}

extension RewardedAdViewController: NapRewardedAdDelegate {

    func rewardedAdDidPresent(_ ad: NapRewardedAd) {
        // 광고 표시됨
    }

    func rewardedAd(_ ad: NapRewardedAd, didRewardUser reward: NapAdReward) {
        // 리워드 지급
        let amount = reward.amount
        let type = reward.type
        print("Reward: \(amount) \(type)")
        grantReward(amount: amount)
    }

    func rewardedAdDidDismiss(_ ad: NapRewardedAd) {
        // 광고 닫힘 → 다음 광고 미리 로드
        loadRewardedAd()
    }

    func rewardedAd(_ ad: NapRewardedAd, didFailToPresentWithError error: Error) { }

    func grantReward(amount: Int) {
        // 앱 내 보상 지급 로직
    }
}
```

---

## 주의사항

- 리워드는 `didRewardUser` 콜백에서 지급합니다.
- `didDismiss` 만으로 리워드를 지급하지 마세요.
