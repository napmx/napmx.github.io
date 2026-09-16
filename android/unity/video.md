# 동영상 - Unity (Android) (beta)

> 🧪 **beta.** 플러그인은 별도 배포되며, 연동 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요. 사전 설정과 이벤트 수신 구조는 [시작하기 - Unity](/android/unity/getting-started) 참고.
>
> ℹ️ 네이티브 SDK는 **인라인 동영상**과 **전면 동영상**을 별도 포맷으로 제공합니다. Android 플러그인은 현재 **인라인 동영상**만 지원합니다. (네이티브 기준 동작은 [동영상 광고](/android/native/video) 참고)

---

## 동영상 광고 로드

Adunit ID는 `AdMixer` Inspector의 `Android Video AdUnitId`를 사용합니다. 로드 성공 시 화면 **중앙**에 자동으로 부착됩니다.

```csharp
using UnityEngine;

public class VideoAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadVideoAd();   // 로드 성공 시 자동 표시
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyVideoAd();
    }
}
```

---

## 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 로드 성공 (자동 표시) |
| `OnFailedToReceiveAd` | 로드 · 표시 실패 |
| `OnEventAd("DISPLAYED")` | 노출 |
| `OnEventAd("CLICK")` | 더보기 클릭 |
| `OnEventAd("COMPLETION")` | 재생 완료 |
| `OnEventAd("SKIPPED")` | 사용자가 Skip 클릭 |
| `OnEventAd("VIDEO_DESTROYED")` | 제거 완료 |
