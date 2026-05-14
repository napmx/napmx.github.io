# 동영상 - Unity (Android)

---

## 동영상 광고 로드

```csharp
using UnityEngine;

public class VideoAd : MonoBehaviour
{
    private string adUnitId = "발급받은_ADUNIT_ID";

    void Start()
    {
        NapSspAndroid.LoadVideoAd(adUnitId, OnVideoLoaded, OnVideoFailed);
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
        NapSspAndroid.DestroyVideoAd();
    }
}
```
