# 리워드 동영상 - Unity (Android)

---

## 리워드 동영상 로드 및 표시

```csharp
using UnityEngine;

public class RewardedAd : MonoBehaviour
{
    private string adUnitId = "발급받은_ADUNIT_ID";

    void Start()
    {
        LoadRewardVideo();
    }

    void LoadRewardVideo()
    {
        NapSspAndroid.LoadRewardVideo(adUnitId, OnRewardLoaded, OnRewardFailed);
    }

    void OnRewardLoaded()
    {
        // 로드 완료 후 적절한 시점에 표시
        NapSspAndroid.ShowRewardVideo(OnRewardComplete, OnRewardClosed);
    }

    void OnRewardFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Reward failed: {errorCode} - {errorMsg}");
    }

    void OnRewardComplete()
    {
        // 동영상 완료 → 리워드 지급
        GrantReward();
    }

    void OnRewardClosed()
    {
        // 광고 닫힘 → 다음 광고 미리 로드
        LoadRewardVideo();
    }

    void GrantReward()
    {
        // 앱 내 보상 지급 로직
    }
}
```

---

## 주의사항

- 리워드는 `OnRewardComplete` 콜백에서 지급합니다.
- `OnRewardClosed`만으로 리워드를 지급하면 동영상을 끝까지 보지 않은 경우에도 지급될 수 있습니다.
