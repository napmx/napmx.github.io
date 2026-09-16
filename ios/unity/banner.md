# 배너 - Unity (iOS)

배너 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

---

## 1. 배너(Banner) 광고

### 1-1. 배너 인스턴스 생성

`AdMixer` Inspector에 `iOS Banner AdUnitId`를 입력하면 `Awake()` 시 320×50 배너 인스턴스가 자동 생성됩니다. 위치는 `iOS Banner Top Position`으로 선택합니다 (`true`: 상단, `false`: 하단, Safe Area 기준).

직접 인스턴스를 만들려면 `NAPSSPPluginIOS`의 정적 메서드를 호출합니다.

```csharp
// 위치 기반 생성 — adUnitId 는 int
NAPSSPPluginIOS.BannerViewInit(adUnitId, NAPSSPPluginIOS.SSPPositionBottom, 320, 50);

// 좌표 기반 생성 — 원점은 화면 왼쪽 상단
NAPSSPPluginIOS.BannerViewInit(adUnitId, x, y, 320, 50);
```

| 매개변수 | 타입 | 설명 |
|---------|------|------|
| `adUnitId` | int | 배너 Adunit ID |
| `position` | int | `NAPSSPPluginIOS.SSPPositionTop`(0) 또는 `SSPPositionBottom`(1) |
| `x`, `y` | float | 좌표 기반 생성 시 왼쪽 상단 위치 |
| `width`, `height` | float | 배너 뷰 크기 |

### 1-2. 배너 광고 요청

`LoadBanner()`는 로드만 수행합니다.

```csharp
AdMixer.Instance.LoadBanner();
```

### 1-3. 배너 광고 노출

`OnSuccessBanner` 수신 후 호출하면 화면에 부착되어 노출됩니다. 로드가 끝나기 전에 호출하면 무시됩니다.

```csharp
AdMixer.Instance.ShowBanner();
```

### 1-4. 배너 제거

```csharp
AdMixer.Instance.DestroyBanner();
```

### 1-5. 이벤트

| 이벤트 | 시그니처 | 설명 |
|-----------|------|------|
| `OnSuccessBanner` | `Action` | 배너 광고 로드 성공 |
| `OnFailBanner` | `Action` | 배너 광고 로드 실패 |
| `OnTapBanner` | `Action` | 배너 광고 클릭 |

```csharp
using UnityEngine;

public class BannerAd : MonoBehaviour
{
    void OnEnable()
    {
        NAPSSPPluginIOS.OnSuccessBanner += OnSuccessBanner;
        NAPSSPPluginIOS.OnFailBanner   += OnFailBanner;
        NAPSSPPluginIOS.OnTapBanner    += OnTapBanner;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessBanner -= OnSuccessBanner;
        NAPSSPPluginIOS.OnFailBanner   -= OnFailBanner;
        NAPSSPPluginIOS.OnTapBanner    -= OnTapBanner;
    }

    void Start()
    {
        AdMixer.Instance.LoadBanner();
    }

    void OnSuccessBanner() { AdMixer.Instance.ShowBanner(); }   // 로드 성공 후 노출
    void OnFailBanner()    { Debug.Log("배너 광고 로드 실패"); }
    void OnTapBanner()     { Debug.Log("배너 광고 클릭"); }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyBanner();
    }
}
```

---

## 2. 전면 배너(Interstitial) 광고

### 2-1. 전면 배너 광고 요청

전면 광고는 별도 인스턴스 생성 없이 `LoadInterstitial()`로 바로 요청합니다. Adunit ID는 `AdMixer` Inspector의 `iOS Interstitial AdUnitId`를 사용합니다.

```csharp
AdMixer.Instance.LoadInterstitial();
```

### 2-2. 전면 배너 광고 노출

`OnSuccessLoadInterstitial` 수신 후 원하는 시점에 호출합니다.

```csharp
AdMixer.Instance.ShowInterstitial();
```

### 2-3. 전면 배너 제거

```csharp
AdMixer.Instance.DestroyInterstitial();
```

### 2-4. 이벤트

| 이벤트 | 시그니처 | 설명 |
|-----------|------|------|
| `OnSuccessLoadInterstitial` | `Action` | 전면 광고 로드 성공 |
| `OnFailLoadInterstitial` | `Action<string>` | 전면 광고 로드 실패 (에러 메시지) |
| `OnSuccessShowInterstitial` | `Action` | 전면 광고 노출 성공 |
| `OnFailShowInterstitial` | `Action<string>` | 전면 광고 노출 실패 (에러 메시지) |
| `OnCloseInterstitial` | `Action` | 전면 광고 닫기 |
| `OnTapInterstitial` | `Action` | 전면 광고 클릭 |

```csharp
using UnityEngine;

public class InterstitialAd : MonoBehaviour
{
    void OnEnable()
    {
        NAPSSPPluginIOS.OnSuccessLoadInterstitial += OnSuccessLoad;
        NAPSSPPluginIOS.OnFailLoadInterstitial    += OnFailLoad;
        NAPSSPPluginIOS.OnFailShowInterstitial    += OnFailShow;
        NAPSSPPluginIOS.OnCloseInterstitial       += OnClose;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessLoadInterstitial -= OnSuccessLoad;
        NAPSSPPluginIOS.OnFailLoadInterstitial    -= OnFailLoad;
        NAPSSPPluginIOS.OnFailShowInterstitial    -= OnFailShow;
        NAPSSPPluginIOS.OnCloseInterstitial       -= OnClose;
    }

    void Start()
    {
        AdMixer.Instance.LoadInterstitial();
    }

    void OnSuccessLoad()            { AdMixer.Instance.ShowInterstitial(); }
    void OnFailLoad(string error)   { Debug.Log($"전면 광고 로드 실패: {error}"); }
    void OnFailShow(string error)   { Debug.Log($"전면 광고 노출 실패: {error}"); }
    void OnClose()                  { Debug.Log("전면 광고 닫힘"); }
}
```

> ⚠️ **노출 실패(`OnFailShowInterstitial`) 처리를 반드시 준비하세요.** 성공·닫힘 이벤트만으로 흐름을 구성하면 앱이 대기 상태에 빠질 수 있습니다.
