# 네이티브 - Unity (Android) (beta)

> 🧪 **beta.** C# 플러그인은 별도 제공되며, 아래 예제의 클래스·메서드명은 **흐름 참고용**입니다. 실제 시그니처는 제공되는 플러그인 배포본을 따릅니다. 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정은 [시작하기 - Unity](/android/unity/getting-started) 참고.
>
> ⚠️ **에셋 텍스트만 꺼내 UI에 그리면 노출·클릭이 집계되지 않습니다.** 네이티브 광고는 SDK에 **뷰를 등록**해야 임프레션과 클릭이 잡히고, 그래야 수익으로 인정됩니다. Unity 레이아웃 연동에는 별도 헬퍼(`admixer-unity-nativeadlayout`)가 제공되므로 문의해 주세요. (네이티브 기준 동작은 [네이티브 광고](/android/native/native-ad) 참고)

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
