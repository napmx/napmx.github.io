# 동영상 - Unity (iOS)

```csharp
public class VideoAd : MonoBehaviour
{
    void Start()
    {
        NapSspIOS.LoadVideoAd("발급받은_ADUNIT_ID", OnVideoLoaded, OnVideoFailed);
    }

    void OnVideoLoaded()
    {
        // 동영상 광고 수신 완료
    }

    void OnVideoFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Video failed: {errorCode} - {errorMsg}");
    }

    void OnDestroy()
    {
        NapSspIOS.DestroyVideoAd();
    }
}
```
