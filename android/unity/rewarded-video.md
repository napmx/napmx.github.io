# 리워드 동영상 - Unity (Android) (beta)

> 🧪 **beta.** C# 플러그인은 별도 제공되며, 아래 예제의 클래스·메서드명은 **흐름 참고용**입니다. 실제 시그니처는 제공되는 플러그인 배포본을 따릅니다. 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정은 [시작하기 - Unity](/android/unity/getting-started) 참고.
>
> ⚠️ **표시 실패 처리를 반드시 준비하세요.** 네트워크 SDK가 표시에 실패할 수 있으므로, 성공·닫힘 콜백만으로 흐름을 구성하면 앱이 대기 상태에 빠질 수 있습니다. 실패 통지 방법은 플러그인 배포본을 확인하세요. (네이티브 기준 동작은 [리워드 동영상](/android/native/rewarded-video) 참고)

---

## 리워드 동영상 로드 및 표시

```csharp
using UnityEngine;

public class RewardedAd : MonoBehaviour
{
    private string adUnitId = "발급받은_ADUNIT_ID";

    void Start()
    {
        LoadRewardVideo();
    }

    void LoadRewardVideo()
    {
        NapSspAndroid.LoadRewardVideo(adUnitId, OnRewardLoaded, OnRewardFailed);
    }

    void OnRewardLoaded()
    {
        // 로드 완료 후 적절한 시점에 표시
        NapSspAndroid.ShowRewardVideo(OnRewardComplete, OnRewardClosed);
    }

    void OnRewardFailed(int errorCode, string errorMsg)
    {
        Debug.Log($"Reward failed: {errorCode} - {errorMsg}");
    }

    void OnRewardComplete()
    {
        // 동영상 완료 → 리워드 지급
        GrantReward();
    }

    void OnRewardClosed()
    {
        // 광고 닫힘 → 다음 광고 미리 로드
        LoadRewardVideo();
    }

    void GrantReward()
    {
        // 앱 내 보상 지급 로직
    }
}
```

---

## 주의사항

- 리워드는 `OnRewardComplete` 콜백에서 지급합니다.
- `OnRewardClosed`만으로 리워드를 지급하면 동영상을 끝까지 보지 않은 경우에도 지급될 수 있습니다.
