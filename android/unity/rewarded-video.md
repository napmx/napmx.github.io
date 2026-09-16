# 리워드 동영상 - Unity (Android) (beta)

> 🧪 **beta.** 플러그인은 별도 배포되며, 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정과 이벤트 수신 구조는 [시작하기 - Unity](/android/unity/getting-started) 참고.
>
> ⚠️ **표시 실패 처리를 반드시 준비하세요.** 네트워크 SDK가 표시에 실패하면 `OnFailedToReceiveAd`가 호출됩니다. 성공·닫힘 이벤트만으로 흐름을 구성하면 앱이 대기 상태에 빠질 수 있습니다. (네이티브 기준 동작은 [리워드 동영상](/android/native/rewarded-video) 참고)

---

## 리워드 동영상 로드 및 표시

Adunit ID는 `AdMixer` Inspector의 `Android Reward AdUnitId`를 사용합니다.

`LoadRewardVideo()`로 로드하고, `OnReceivedAd` 수신 후 원하는 시점에 `ShowRewardVideo()`를 호출합니다. 로드 전에 `ShowRewardVideo()`를 호출하면 무시됩니다.

```csharp
using UnityEngine;

public class RewardedAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadRewardVideo();
    }

    // AdMixerAdListener.OnReceivedAd 수신 후 호출
    public void ShowRewardVideo()
    {
        AdMixer.Instance.ShowRewardVideo();
    }
}
```

---

## 이벤트 및 보상 지급

`AdMixerAdListener.OnEventAd`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 로드 성공 |
| `OnFailedToReceiveAd` | 로드 · 표시 실패 |
| `OnEventAd("DISPLAYED")` | 노출 |
| `OnEventAd("CLICK")` | 클릭 |
| `OnEventAd("COMPLETION")` | 재생 완료 — **보상과 별개**이며 네트워크에 따라 발화하지 않을 수 있음 |
| `OnEventAd("EARNEDREWARD\|<transactionId>")` | **보상 적립 — 이 이벤트에서 보상을 지급.** `transactionId`는 서버 포스트백 대조용 |
| `OnEventAd("CLOSE")` | 광고 닫힘 |

```csharp
[Preserve]
public void OnEventAd(string param)
{
    if (param.StartsWith("EARNEDREWARD"))
    {
        // 형식: "EARNEDREWARD|<transactionId>"
        string transactionId = param.Length > "EARNEDREWARD|".Length ? param.Substring("EARNEDREWARD|".Length) : "";
        GrantReward(transactionId);              // ✅ 보상 지급은 여기서
    }
    else if (param == "CLOSE")
    {
        AdMixer.Instance.LoadRewardVideo();      // 다음 광고 미리 로드
    }
}
```

> ⚠️ **`COMPLETION`이나 `CLOSE`로 보상을 지급하지 마세요.** `EARNEDREWARD`와 `CLOSE`의 도착 순서는 네트워크 정책에 따라 달라질 수 있습니다. 보상 지급은 `EARNEDREWARD`에서 즉시 처리하고, 사용자 알림(Toast 등)은 `CLOSE` 이후로 미루는 것을 권장합니다.

---

## Reward Callback (선택사항)

매체사가 정의한 외부 서버로 리워드 지급 완료를 전달하는 기능입니다. 파트너 사이트에서 콜백 URL을 설정한 뒤, 추가 파라미터가 필요하면 **`LoadRewardVideo()` 호출 전에** `RewardAdSetCustomParam()`으로 설정합니다.

```csharp
var customParam = new Dictionary<string, string>
{
    { "user_id", "user123" },
    { "reward_type", "coin" }
};
AdMixer.Instance.RewardAdSetCustomParam(customParam);
AdMixer.Instance.LoadRewardVideo();
```
