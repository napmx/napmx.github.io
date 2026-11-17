# 동영상 - Unity (iOS)

동영상 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

> 전면(Interstitial) 동영상 광고를 연동하려면 아래 **전면 비디오** 섹션을 참고해주세요.

---

## 1. 동영상(Video) 광고

### 1-1. 비디오 인스턴스 생성

`AdMixer` Inspector에 `iOS Video AdUnitId`를 입력하면 `Awake()`에서 좌표 (0, 400)에 400×200 크기의 비디오 인스턴스가 자동 생성됩니다. 위치·크기를 바꾸려면 `AdMixer.cs`의 `VideoViewInit` 호출을 수정하거나 아래처럼 직접 호출하세요.

```csharp
// 좌표 기반 생성 — adUnitId 는 int, 원점은 화면 왼쪽 상단
NAPSSPPluginIOS.VideoViewInit(adUnitId, x, y, width, height);
```

| 매개변수 | 타입 | 설명 |
|---------|------|------|
| `adUnitId` | int | 동영상 Adunit ID |
| `x`, `y` | float | 왼쪽 상단 위치 |
| `width`, `height` | float | 비디오 뷰 크기 |

### 1-2. 비디오 광고 요청

```csharp
AdMixer.Instance.LoadVideoAd();
```

### 1-3. 비디오 광고 노출

`OnSuccessVideo`를 받은 뒤 원하는 시점에 호출하면 1-1에서 지정한 위치에 붙어 노출됩니다. 로드 전이거나 이미 노출 중이면 무시됩니다.

```csharp
AdMixer.Instance.ShowVideoAd();
```

### 1-4. 비디오 제거

```csharp
AdMixer.Instance.DestroyVideoAd();
```

### 1-5. 이벤트

| 이벤트 | 시그니처 | 설명 |
|-----------|------|------|
| `OnSuccessVideo` | `Action` | 비디오 광고 로드 성공 — 이후 `ShowVideoAd()` 호출 |
| `OnFailVideo` | `Action` | 비디오 광고 로드 실패 |
| `OnFailShowVideo` | `Action<string>` | 비디오 광고 표시 실패 (에러 메시지) — SDK가 뷰를 숨기며, 다시 노출하려면 `DestroyVideoAd()` → `LoadVideoAd()` → `ShowVideoAd()` |
| `OnSkipVideo` | `Action` | 비디오 광고 내 skip 버튼 클릭 |
| `OnTapVideoViewMore` | `Action` | 비디오 광고 내 더보기 버튼 클릭 |
| `OnCompleteVideo` | `Action` | 비디오 광고 재생 완료 |

```csharp
using UnityEngine;

public class VideoAd : MonoBehaviour
{
    void OnEnable()
    {
        NAPSSPPluginIOS.OnSuccessVideo     += OnSuccessVideo;
        NAPSSPPluginIOS.OnFailVideo        += OnFailVideo;
        NAPSSPPluginIOS.OnFailShowVideo    += OnFailShowVideo;
        NAPSSPPluginIOS.OnSkipVideo        += OnSkipVideo;
        NAPSSPPluginIOS.OnTapVideoViewMore += OnTapVideoViewMore;
        NAPSSPPluginIOS.OnCompleteVideo    += OnCompleteVideo;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessVideo     -= OnSuccessVideo;
        NAPSSPPluginIOS.OnFailVideo        -= OnFailVideo;
        NAPSSPPluginIOS.OnFailShowVideo    -= OnFailShowVideo;
        NAPSSPPluginIOS.OnSkipVideo        -= OnSkipVideo;
        NAPSSPPluginIOS.OnTapVideoViewMore -= OnTapVideoViewMore;
        NAPSSPPluginIOS.OnCompleteVideo    -= OnCompleteVideo;
    }

    void Start()
    {
        AdMixer.Instance.LoadVideoAd();
    }

    void OnSuccessVideo()     { AdMixer.Instance.ShowVideoAd(); }
    void OnFailVideo()        { Debug.Log("비디오 광고 로드 실패"); }
    void OnFailShowVideo(string error) { Debug.Log($"비디오 광고 표시 실패: {error}"); }
    void OnSkipVideo()        { Debug.Log("비디오 광고 skip"); }
    void OnTapVideoViewMore() { Debug.Log("더보기 버튼 클릭"); }
    void OnCompleteVideo()    { Debug.Log("비디오 광고 재생 완료"); }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyVideoAd();
    }
}
```

---

## 2. 전면 비디오(Interstitial Video) 광고

### 2-1. 전면 비디오 광고 요청

전면 비디오는 별도 인스턴스 생성 없이 `LoadVideoInterstitial()`로 바로 요청합니다. Adunit ID는 `AdMixer` Inspector의 `iOS Video Interstitial AdUnitId`를 사용합니다.

```csharp
AdMixer.Instance.LoadVideoInterstitial();
```

### 2-2. 전면 비디오 광고 노출

`OnSuccessLoadVideoInterstitial`을 받은 뒤 원하는 시점에 호출합니다.

```csharp
AdMixer.Instance.ShowVideoInterstitial();
```

### 2-3. 전면 비디오 제거

```csharp
AdMixer.Instance.DestroyVideoInterstitial();
```

### 2-4. 이벤트

| 이벤트 | 시그니처 | 설명 |
|-----------|------|------|
| `OnSuccessLoadVideoInterstitial` | `Action` | 전면 비디오 광고 로드 성공 |
| `OnFailLoadVideoInterstitial` | `Action<string>` | 전면 비디오 광고 로드 실패 (에러 메시지) |
| `OnSuccessShowVideoInterstitial` | `Action` | 전면 비디오 광고 노출 성공 |
| `OnFailShowVideoInterstitial` | `Action<string>` | 전면 비디오 광고 노출 실패 (에러 메시지) |
| `OnCloseVideoInterstitial` | `Action` | 전면 비디오 광고 닫기 |
| `OnTapVideoInterstitialViewMore` | `Action` | 전면 비디오 광고 내 더보기 버튼 클릭 |
| `OnCompleteVideoInterstitial` | `Action` | 전면 비디오 광고 재생 완료 |

```csharp
using UnityEngine;

public class VideoInterstitialAd : MonoBehaviour
{
    void OnEnable()
    {
        NAPSSPPluginIOS.OnSuccessLoadVideoInterstitial += OnSuccessLoad;
        NAPSSPPluginIOS.OnFailLoadVideoInterstitial    += OnFailLoad;
        NAPSSPPluginIOS.OnFailShowVideoInterstitial    += OnFailShow;
        NAPSSPPluginIOS.OnCloseVideoInterstitial       += OnClose;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessLoadVideoInterstitial -= OnSuccessLoad;
        NAPSSPPluginIOS.OnFailLoadVideoInterstitial    -= OnFailLoad;
        NAPSSPPluginIOS.OnFailShowVideoInterstitial    -= OnFailShow;
        NAPSSPPluginIOS.OnCloseVideoInterstitial       -= OnClose;
    }

    void Start()
    {
        AdMixer.Instance.LoadVideoInterstitial();
    }

    void OnSuccessLoad()          { AdMixer.Instance.ShowVideoInterstitial(); }
    void OnFailLoad(string error) { Debug.Log($"전면 비디오 광고 로드 실패: {error}"); }
    void OnFailShow(string error) { Debug.Log($"전면 비디오 광고 노출 실패: {error}"); }
    void OnClose()                { Debug.Log("전면 비디오 광고 닫힘"); }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyVideoInterstitial();
    }
}
```

> ⚠️ **노출 실패(`OnFailShowVideoInterstitial`)도 반드시 처리하세요.** 성공·닫힘 이벤트만으로 흐름을 짜면 노출에 실패했을 때 앱이 계속 대기할 수 있습니다.
