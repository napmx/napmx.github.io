# 동영상 - Unity (Android) (beta)

> 🧪 **beta.** C# 플러그인은 별도 제공되며, 아래 예제의 클래스·메서드명은 **흐름 참고용**입니다. 실제 시그니처는 제공되는 플러그인 배포본을 따릅니다. 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정은 [시작하기 - Unity](/android/unity/getting-started) 참고.
>
> ℹ️ 네이티브 SDK는 **인라인 동영상**과 **전면 동영상**을 별도 포맷으로 제공합니다. 아래 예제는 인라인 기준이며, 전면 동영상은 플러그인 배포본을 확인하세요. (네이티브 기준 동작은 [동영상 광고](/android/native/video) 참고)

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
