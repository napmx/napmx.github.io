# 네이티브 - Unity (Android)

---

## 네이티브 광고 로드

```csharp
using UnityEngine;

public class NativeAd : MonoBehaviour
{
    private string adUnitId = "발급받은_ADUNIT_ID";

    void Start()
    {
        NapSspAndroid.LoadNativeAd(adUnitId, OnNativeLoaded, OnNativeFailed);
    }

    void OnNativeLoaded()
    {
        // 네이티브 광고 에셋을 UI에 바인딩
        string title = NapSspAndroid.GetNativeAdTitle();
        string body = NapSspAndroid.GetNativeAdBody();
        // ...
    }

    void OnNativeFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Native failed: {errorCode} - {errorMsg}");
    }

    void OnDestroy()
    {
        NapSspAndroid.DestroyNativeAd();
    }
}
```
