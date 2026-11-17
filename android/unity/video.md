# 동영상 - Unity (Android)

동영상 광고를 추가하기 전에 [Android SDK 시작하기 - Unity](/android/unity/getting-started) 설정을 완료해주세요.

> 전면(Interstitial) 동영상 광고를 연동하려면 아래 **전면 동영상** 섹션을 참고해주세요(네이티브 기준 동작은 [동영상 광고](/android/native/video) 참고).

---

## 1. 동영상(Video) 광고

### 1-1. 동영상 인스턴스 설정

Adunit ID는 `AdMixer` Inspector의 `Android Video AdUnitId`를 사용합니다. 로드에 성공한 뒤 `ShowVideoAd()`를 호출하면 화면 **중앙**에 붙어 노출됩니다.

### 1-2. 동영상 광고 요청

```csharp
AdMixer.Instance.LoadVideoAd();
```

### 1-3. 동영상 광고 노출

`OnReceivedAd`를 받은 뒤 원하는 시점에 호출합니다. 로드 전이거나 이미 노출 중이면 무시됩니다.

```csharp
AdMixer.Instance.ShowVideoAd();
```

### 1-4. 동영상 제거

```csharp
AdMixer.Instance.DestroyVideoAd();
```

### 1-5. 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 동영상 광고 로드 성공 — 이후 `ShowVideoAd()` 호출 |
| `OnFailedToReceiveAd` | 동영상 광고 로드 실패 |
| `OnFailedToShowAd` | 동영상 광고 노출 실패 — 다시 노출하려면 `DestroyVideoAd()` → `LoadVideoAd()` → `ShowVideoAd()` |
| `OnEventAd("DISPLAYED")` | 동영상 광고 노출 |
| `OnEventAd("CLICK")` | 동영상 광고 내 더보기 버튼 클릭 |
| `OnEventAd("SKIPPED")` | 동영상 광고 내 skip 버튼 클릭 |
| `OnEventAd("COMPLETION")` | 동영상 광고 재생 완료 |
| `OnEventAd("VIDEO_DESTROYED")` | 제거 완료 |

```csharp
using UnityEngine;

public class VideoAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadVideoAd();
    }

    // AdMixerAdListener.OnReceivedAd 수신 후 호출
    public void ShowVideoAd()
    {
        AdMixer.Instance.ShowVideoAd();
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyVideoAd();
    }
}
```

---

## 2. 전면 동영상(Interstitial Video) 광고

### 2-1. 전면 동영상 광고 요청

전면 동영상은 별도 인스턴스 설정 없이 `LoadVideoInterstitial()`로 바로 요청합니다. Adunit ID는 `AdMixer` Inspector의 `Android Video Interstitial AdUnitId`를 사용합니다.

```csharp
AdMixer.Instance.LoadVideoInterstitial();
```

### 2-2. 전면 동영상 광고 노출

`OnReceivedAd`를 받은 뒤 원하는 시점에 호출합니다. 로드 전에 호출하면 무시됩니다.

```csharp
AdMixer.Instance.ShowVideoInterstitial();
```

### 2-3. 전면 동영상 제거

```csharp
AdMixer.Instance.DestroyVideoInterstitial();
```

### 2-4. 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 전면 동영상 광고 로드 성공 |
| `OnFailedToReceiveAd` | 전면 동영상 광고 로드 실패 |
| `OnFailedToShowAd` | 전면 동영상 광고 노출 실패 (다시 노출하려면 다시 로드) |
| `OnEventAd("DISPLAYED")` | 전면 동영상 광고 노출 |
| `OnEventAd("CLICK")` | 전면 동영상 광고 내 더보기 버튼 클릭 |
| `OnEventAd("COMPLETION")` | 전면 동영상 광고 재생 완료 |
| `OnEventAd("CLOSE")` | 전면 동영상 광고 닫기 |
| `OnEventAd("VIDEO_INTERSTITIAL_DESTROYED")` | 제거 완료 |

```csharp
using UnityEngine;

public class VideoInterstitialAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadVideoInterstitial();
    }

    // AdMixerAdListener.OnReceivedAd 수신 후 호출
    public void ShowVideoInterstitial()
    {
        AdMixer.Instance.ShowVideoInterstitial();
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyVideoInterstitial();
    }
}
```

> ⚠️ **노출 실패(`OnFailedToShowAd`)도 반드시 처리하세요.** 성공·닫힘 이벤트만으로 흐름을 짜면 노출에 실패했을 때 앱이 계속 대기할 수 있습니다.
