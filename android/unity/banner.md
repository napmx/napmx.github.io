# 배너 - Unity (Android) (beta)

> 🧪 **beta.** C# 플러그인은 별도 제공되며, 아래 예제의 클래스·메서드명은 **흐름 참고용**입니다. 실제 시그니처는 제공되는 플러그인 배포본을 따릅니다. 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정은 [시작하기 - Unity](/android/unity/getting-started) 참고.

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
