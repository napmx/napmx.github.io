# 동영상 - Unity (Android)

광고를 추가하기 전 [Android SDK 시작하기 - Unity](/android/unity/getting-started) 설정을 완료해주세요.

> ℹ️ 네이티브 SDK는 **인라인 동영상**과 **전면 동영상**을 별도 포맷으로 제공합니다. Android 플러그인은 현재 **인라인 동영상**만 지원합니다. (네이티브 기준 동작은 [동영상 광고](/android/native/video) 참고)

---

## 1. 동영상(Video) 광고

### 1-1. 비디오 인스턴스 설정

Adunit ID는 `AdMixer` Inspector의 `Android Video AdUnitId`를 사용합니다. 로드 성공 시 화면 **중앙**에 자동으로 부착됩니다.

### 1-2. 비디오 광고 요청

```csharp
AdMixer.Instance.LoadVideoAd();   // 로드 성공 시 자동 표시
```

### 1-3. 비디오 제거

```csharp
AdMixer.Instance.DestroyVideoAd();
```

### 1-4. 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 비디오 광고 로드 성공 (자동 표시) |
| `OnFailedToReceiveAd` | 비디오 광고 로드 · 표시 실패 |
| `OnEventAd("DISPLAYED")` | 비디오 광고 노출 |
| `OnEventAd("CLICK")` | 비디오 광고 내 더보기 버튼 클릭 |
| `OnEventAd("SKIPPED")` | 비디오 광고 내 skip 버튼 클릭 |
| `OnEventAd("COMPLETION")` | 비디오 광고 재생 완료 |
| `OnEventAd("VIDEO_DESTROYED")` | 제거 완료 |

```csharp
using UnityEngine;

public class VideoAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadVideoAd();
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyVideoAd();
    }
}
```
