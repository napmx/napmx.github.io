# 동영상 - Unity (iOS)

동영상 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

> 전면(Interstitial) 동영상 광고 연동을 원하시는 경우, 아래 **전면 비디오** 섹션을 참고해주세요.

---

## 1. 동영상(Video) 광고

### 1-1. 비디오 인스턴스 생성

`AdMixer` Inspector에 `iOS Video View AdUnitId`를 입력하면 `Awake()` 시 좌표 (0, 400), 크기 400×200 비디오 인스턴스가 자동 생성됩니다. 위치·크기를 바꾸려면 `AdMixer.cs`의 `VideoViewInit` 호출을 수정하거나 직접 호출합니다.

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

로드 성공 시 화면에 자동으로 부착·노출됩니다.

```csharp
AdMixer.Instance.LoadVideoAd();
```

### 1-3. 비디오 제거

```csharp
AdMixer.Instance.DestroyVideoAd();
```

### 1-4. 이벤트

| 이벤트 | 시그니처 | 설명 |
|-----------|------|------|
| `OnSuccessVideo` | `Action` | 비디오 광고 로드 성공 (자동 표시) |
| `OnFailVideo` | `Action` | 비디오 광고 로드 실패 |
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
        NAPSSPPluginIOS.OnSkipVideo        += OnSkipVideo;
        NAPSSPPluginIOS.OnTapVideoViewMore += OnTapVideoViewMore;
        NAPSSPPluginIOS.OnCompleteVideo    += OnCompleteVideo;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessVideo     -= OnSuccessVideo;
        NAPSSPPluginIOS.OnFailVideo        -= OnFailVideo;
        NAPSSPPluginIOS.OnSkipVideo        -= OnSkipVideo;
        NAPSSPPluginIOS.OnTapVideoViewMore -= OnTapVideoViewMore;
        NAPSSPPluginIOS.OnCompleteVideo    -= OnCompleteVideo;
    }

    void Start()
    {
        AdMixer.Instance.LoadVideoAd();
    }

    void OnSuccessVideo()     { Debug.Log("비디오 광고 로드 성공"); }
    void OnFailVideo()        { Debug.Log("비디오 광고 로드 실패"); }
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

전면 비디오는 `AdMixer`에 래퍼가 없으며 `NAPSSPPluginIOS`의 정적 메서드를 직접 호출합니다. Adunit ID는 별도 인스턴스 생성 없이 요청 시 인자로 전달합니다.

### 2-1. 전면 비디오 광고 요청

```csharp
NAPSSPPluginIOS.VideoInterstitialLoadAd(adUnitId);   // adUnitId 는 int
```

### 2-2. 전면 비디오 광고 노출

`OnSuccessLoadVideoInterstitial` 수신 후 원하는 시점에 호출합니다.

```csharp
NAPSSPPluginIOS.VideoInterstitialShow();
```

### 2-3. 이벤트

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
    [SerializeField] private int adUnitId;

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
        NAPSSPPluginIOS.VideoInterstitialLoadAd(adUnitId);
    }

    void OnSuccessLoad()          { NAPSSPPluginIOS.VideoInterstitialShow(); }
    void OnFailLoad(string error) { Debug.Log($"전면 비디오 광고 로드 실패: {error}"); }
    void OnFailShow(string error) { Debug.Log($"전면 비디오 광고 노출 실패: {error}"); }
    void OnClose()                { Debug.Log("전면 비디오 광고 닫힘"); }
}
```

> ⚠️ 전면 비디오 Adunit ID는 `AdMixer` Inspector에 없으므로 SDK 초기화 시 Adunit 목록에 포함되지 않습니다. 사용하려면 `AdMixer.cs`의 초기화 Adunit 목록에 추가하거나 운영팀에 문의하세요.
