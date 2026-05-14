# 리워드 동영상 - Unity (iOS)

```csharp
public class RewardedAd : MonoBehaviour
{
    void Start()
    {
        LoadRewardVideo();
    }

    void LoadRewardVideo()
    {
        NapSspIOS.LoadRewardVideo("발급받은_ADUNIT_ID", OnRewardLoaded, OnRewardFailed);
    }

    void OnRewardLoaded()
    {
        NapSspIOS.ShowRewardVideo(OnRewardComplete, OnRewardClosed);
    }

    void OnRewardFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Reward failed: {errorCode} - {errorMsg}");
    }

    void OnRewardComplete()
    {
        // 리워드 지급
        GrantReward();
    }

    void OnRewardClosed()
    {
        LoadRewardVideo();
    }

    void GrantReward() { }
}
```
