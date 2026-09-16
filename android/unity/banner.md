# 배너 - Unity (Android)

광고를 추가하기 전 [Android SDK 시작하기 - Unity](/android/unity/getting-started) 설정을 완료해주세요.

---

## 1. 배너(Banner) 광고

### 1-1. 배너 인스턴스 설정

Adunit ID는 `AdMixer` Inspector의 `Android Banner AdUnitId`를 사용합니다. 배너는 화면 **하단**에 부착됩니다.

### 1-2. 배너 광고 요청

`LoadBanner()`는 로드만 수행합니다.

```csharp
AdMixer.Instance.LoadBanner();
```

### 1-3. 배너 광고 노출

`OnReceivedAd` 수신 후 호출하면 화면에 부착되어 노출됩니다. 로드가 끝나기 전에 호출하면 무시됩니다.

```csharp
AdMixer.Instance.ShowBanner();
```

### 1-4. 배너 제거

```csharp
AdMixer.Instance.DestroyBanner();
```

### 1-5. 일시정지 / 재개

배너의 광고 갱신 타이머를 앱 생명주기에 맞춰 제어하려면 다음을 호출합니다.

```csharp
void OnApplicationPause(bool pause)
{
    if (pause) AdMixer.Instance.BannerOnPause();
    else       AdMixer.Instance.BannerOnResume();
}
```

### 1-6. 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 배너 광고 로드 성공 |
| `OnFailedToReceiveAd` | 배너 광고 로드 · 표시 실패 |
| `OnEventAd("DISPLAYED")` | 배너 광고 노출 |
| `OnEventAd("CLICK")` | 배너 광고 클릭 |
| `OnEventAd("BANNER_DESTROYED")` | 제거 완료 |

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

## 2. 전면 배너(Interstitial) 광고

### 2-1. 전면 배너 광고 요청

전면 광고는 별도 인스턴스 설정 없이 `LoadInterstitial()`로 바로 요청합니다. Adunit ID는 `AdMixer` Inspector의 `Android Interstitial AdUnitId`를 사용합니다.

```csharp
AdMixer.Instance.LoadInterstitial();
```

### 2-2. 전면 배너 광고 노출

`OnReceivedAd` 수신 후 원하는 시점에 호출합니다. 로드 전에 호출하면 무시됩니다.

```csharp
AdMixer.Instance.ShowInterstitial();
```

### 2-3. 전면 배너 제거

```csharp
AdMixer.Instance.DestroyInterstitial();
```

### 2-4. 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 전면 광고 로드 성공 |
| `OnFailedToReceiveAd` | 전면 광고 로드 · 노출 실패 |
| `OnEventAd("DISPLAYED")` | 전면 광고 노출 |
| `OnEventAd("CLICK")` | 전면 광고 클릭 |
| `OnEventAd("COMPLETION")` | 동영상 소재 재생 완료 (네트워크에 따라 미발화) |
| `OnEventAd("CLOSE")` | 전면 광고 닫기 |
| `OnEventAd("INTERSTITIAL_DESTROYED")` | 제거 완료 |

```csharp
using UnityEngine;

public class InterstitialAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadInterstitial();
    }

    // AdMixerAdListener.OnReceivedAd 수신 후 호출
    public void ShowInterstitial()
    {
        AdMixer.Instance.ShowInterstitial();
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyInterstitial();
    }
}
```

> ⚠️ **노출 실패(`OnFailedToReceiveAd`) 처리를 반드시 준비하세요.** 성공·닫힘 이벤트만으로 흐름을 구성하면 앱이 대기 상태에 빠질 수 있습니다.
