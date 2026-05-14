# 배너 - Unity (iOS)

```csharp
public class BannerAd : MonoBehaviour
{
    void Start()
    {
        NapSspIOS.LoadBanner("발급받은_ADUNIT_ID", OnBannerLoaded, OnBannerFailed);
    }

    void OnBannerLoaded()
    {
        NapSspIOS.ShowBanner(BannerPosition.BOTTOM);
    }

    void OnBannerFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Banner failed: {errorCode} - {errorMsg}");
    }

    void OnDestroy()
    {
        NapSspIOS.DestroyBanner();
    }
}
```
