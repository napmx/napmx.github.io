# 리워드 동영상 - Unity (Android)

광고를 추가하기 전 [Android SDK 시작하기 - Unity](/android/unity/getting-started) 설정을 완료해주세요.

---

## 1. 리워드 광고 요청

리워드 광고는 별도 인스턴스 설정 없이 `LoadRewardVideo()`로 바로 요청합니다. Adunit ID는 `AdMixer` Inspector의 `Android Reward AdUnitId`를 사용합니다.

```csharp
AdMixer.Instance.LoadRewardVideo();
```

---

## 2. 리워드 광고 노출

`OnReceivedAd` 수신 후 원하는 시점에 호출합니다. 로드 전에 호출하면 무시됩니다.

```csharp
AdMixer.Instance.ShowRewardVideo();
```

---

## 3. 이벤트 및 보상 지급

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 리워드 광고 로드 성공 |
| `OnFailedToReceiveAd` | 리워드 광고 로드 · 노출 실패 |
| `OnEventAd("DISPLAYED")` | 리워드 광고 노출 |
| `OnEventAd("CLICK")` | 리워드 광고 클릭 |
| `OnEventAd("COMPLETION")` | 재생 완료 — **보상과 별개**이며 네트워크에 따라 발화하지 않을 수 있음 |
| `OnEventAd("EARNEDREWARD\|<transactionId>")` | **보상 지급 — 이 이벤트에서 보상을 지급.** `transactionId`는 서버 포스트백 대조용 |
| `OnEventAd("CLOSE")` | 리워드 광고 닫기 |
| `OnEventAd("REWARD_DESTROYED")` | 제거 완료 |

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

```csharp
// AdMixerAdListener.cs
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

void GrantReward(string transactionId)
{
    // 앱 내 보상 지급 로직. transactionId 를 지급 이력 키로 기록해 두면 서버 포스트백과 대조할 수 있습니다.
}
```

> ⚠️ **`COMPLETION`이나 `CLOSE`로 보상을 지급하지 마세요.** `EARNEDREWARD`와 `CLOSE`의 도착 순서는 네트워크 정책에 따라 달라질 수 있습니다. 보상 지급은 `EARNEDREWARD`에서 즉시 처리하고, 사용자 알림(Toast 등)은 닫힘 이후로 미루는 것을 권장합니다.
>
> ⚠️ **노출 실패(`OnFailedToReceiveAd`) 처리를 반드시 준비하세요.** 성공·닫힘 이벤트만으로 흐름을 구성하면 앱이 대기 상태에 빠질 수 있습니다.

---

## 4. Reward Callback (선택사항)

매체사가 정의한 외부 서버로 해당 유저에게 리워드 지급이 완료되었음을 전달하는 기능입니다.  
콜백 수신까지 몇 분 정도 지연될 수 있습니다.

### 설정 1: 파트너 사이트에서 콜백 서버 URL 입력

**파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정**에서 매체사의 콜백 서버 URL을 입력합니다.

유저가 해당 애드유닛을 통해 리워드 지급이 완료되면, 입력된 콜백 서버 URL로 리워드 콜백 데이터를 전송합니다.

### 설정 2: CustomParam 추가

CustomParam을 통해 콜백에서 추가 데이터를 수집할 수 있습니다. **반드시 `LoadRewardVideo()` 호출 전에** 설정해야 합니다.

```csharp
var customParam = new Dictionary<string, string>
{
    { "user_id", "user123" },
    { "reward_type", "coin" }
};
AdMixer.Instance.RewardAdSetCustomParam(customParam);
AdMixer.Instance.LoadRewardVideo();
```
