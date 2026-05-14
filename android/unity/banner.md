# 배너 - Unity (Android)

---

## 배너 광고 로드 및 표시

```csharp
using UnityEngine;

public class BannerAd : MonoBehaviour
{
    private string adUnitId = "발급받은_ADUNIT_ID";

    void Start()
    {
        LoadBanner();
    }

    void LoadBanner()
    {
        NapSspAndroid.LoadBanner(adUnitId, OnBannerLoaded, OnBannerFailed);
    }

    void OnBannerLoaded()
    {
        NapSspAndroid.ShowBanner();
    }

    void OnBannerFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Banner failed: {errorCode} - {errorMsg}");
    }

    void OnDestroy()
    {
        NapSspAndroid.DestroyBanner();
    }
}
```

---

## 배너 위치 설정

| 위치 | 상수 |
|------|------|
| 상단 | `BannerPosition.TOP` |
| 하단 | `BannerPosition.BOTTOM` |

```csharp
NapSspAndroid.ShowBanner(BannerPosition.BOTTOM);
```
