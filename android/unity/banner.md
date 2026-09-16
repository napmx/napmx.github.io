# 배너 - Unity (Android) (beta)

> 🧪 **beta.** 플러그인은 별도 배포되며, 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정과 이벤트 수신 구조는 [시작하기 - Unity](/android/unity/getting-started) 참고.

---

## 배너 광고 로드 및 표시

Adunit ID는 `AdMixer` Inspector의 `Android Banner AdUnitId`를 사용합니다.

배너는 `LoadBanner()`로 로드만 하고, `ShowBanner()`를 호출한 시점에 화면 **하단**에 부착되어 노출됩니다. 로드가 끝나기 전에 `ShowBanner()`를 호출하면 무시되므로 `OnReceivedAd` 수신 후 호출하세요.

```csharp
using UnityEngine;

public class BannerAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadBanner();
    }

    // AdMixerAdListener.OnReceivedAd 수신 후 호출
    public void ShowBanner()
    {
        AdMixer.Instance.ShowBanner();
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyBanner();
    }
}
```

---

## 일시정지 / 재개

배너의 광고 갱신 타이머를 Activity 생명주기에 맞춰 제어하려면 다음을 호출합니다.

```csharp
void OnApplicationPause(bool pause)
{
    if (pause) AdMixer.Instance.BannerOnPause();
    else       AdMixer.Instance.BannerOnResume();
}
```

---

## 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 로드 성공 |
| `OnFailedToReceiveAd` | 로드 · 표시 실패 |
| `OnEventAd("DISPLAYED")` | 노출 |
| `OnEventAd("CLICK")` | 클릭 |
| `OnEventAd("BANNER_DESTROYED")` | 제거 완료 |
