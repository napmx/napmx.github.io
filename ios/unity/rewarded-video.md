# 리워드 동영상 - Unity (iOS)

리워드 동영상 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

---

## 1. 리워드 인스턴스 생성

리워드 광고 노출을 위해 `RewardAdInit` API를 호출하여 인스턴스를 생성합니다.

```csharp
RewardAdInit("발급받은_ADUNIT_ID");
```

---

## 2. 리워드 광고 요청

```csharp
LoadRewardVideo();
```

---

## 3. Delegate

리워드 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.

| 델리게이트 | 설명 |
|-----------|------|
| `OnSuccessRewardVideo` | 리워드 광고 로드 성공 |
| `OnFailRewardVideo` | 리워드 광고 로드 실패 |
| `OnCloseRewardVideo` | 리워드 광고 닫기 |
| `OnTapRewardVideo` | 리워드 광고 내 더보기 버튼 클릭 |
| `OnRewardVideoComplete` | 리워드 광고 재생 완료 |
| `OnRewardVideoEarned` | 리워드 광고 리워드 지급 완료 |

```csharp
public class RewardedAd : MonoBehaviour
{
    void Start()
    {
        RewardAdInit("발급받은_ADUNIT_ID");
        LoadRewardVideo();
    }

    void OnSuccessRewardVideo()
    {
        ShowRewardVideo();
    }

    void OnFailRewardVideo(string errorMsg)
    {
        Debug.Log($"리워드 광고 로드 실패: {errorMsg}");
    }

    void OnRewardVideoComplete()
    {
        Debug.Log("리워드 광고 재생 완료");
    }

    void OnRewardVideoEarned()
    {
        // 리워드 지급 처리
        GrantReward();
    }

    void OnCloseRewardVideo()
    {
        LoadRewardVideo();
    }

    void OnTapRewardVideo()
    {
        Debug.Log("더보기 버튼 클릭");
    }

    void GrantReward() { }
}
```

---

## 4. Reward Callback (선택사항)

매체사가 정의한 외부 서버로 해당 유저에게 리워드 지급이 완료되었음을 전달하는 기능입니다.  
콜백 수신까지 몇 분 정도 지연될 수 있습니다.

### 설정 1: 파트너 사이트에서 콜백 서버 URL 입력

**파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정**에서 매체사의 콜백 서버 URL을 입력합니다.

유저가 해당 애드유닛을 통해 리워드 지급이 완료되면, 입력된 콜백 서버 URL로 리워드 콜백 데이터를 전송합니다.

기본 파라미터는 아래와 같습니다.

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| `media_key` | 미디어 키 (자동 포함) | `12345678` |
| `adunit_id` | 애드유닛 ID (자동 포함) | `87654321` |
| `ifa` | iOS 기기 고유 식별자 (자동 포함) | `860635ea-65bc-eaed-d355-1b5283b30b94` |
| `timestamp` | 리워드 지급 이벤트 발생 시간 (자동 포함) | `1546300800` |

### 설정 2: CustomParm 추가

CustomParm을 통해 콜백에서 추가 데이터를 수집할 수 있습니다. Dictionary 형태로 추가해야 합니다.

SDK 내 CustomData 추가에서 세팅한 Custom 파라미터가 콜백 URL에 포함되어 전송됩니다.

```csharp
Dictionary<string, string> customParm = new Dictionary<string, string>
{
    { "userid", "nas" },
    { "name", "hdragon" },
    { "phone", "010-1111-1111" }
};
SetRewardCustomParm(customParm);
```
