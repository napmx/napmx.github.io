# 네이티브 - Unity (iOS)

```csharp
public class NativeAd : MonoBehaviour
{
    void Start()
    {
        NapSspIOS.LoadNativeAd("발급받은_ADUNIT_ID", OnNativeLoaded, OnNativeFailed);
    }

    void OnNativeLoaded()
    {
        string title = NapSspIOS.GetNativeAdTitle();
        string body = NapSspIOS.GetNativeAdBody();
        string cta = NapSspIOS.GetNativeAdCTA();
        // UI에 바인딩
    }

    void OnNativeFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Native failed: {errorCode} - {errorMsg}");
    }

    void OnDestroy()
    {
        NapSspIOS.DestroyNativeAd();
    }
}
```
