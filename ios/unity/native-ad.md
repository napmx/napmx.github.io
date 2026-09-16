# 네이티브 - Unity (iOS)

네이티브 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

---

## 1. 구성

`Assets/Plugins/iOS/NativeTemplates/NapSSPNativeTemplateView.xib` 파일을 열어 네이티브 뷰를 커스텀해주세요.

네이티브 광고는 6가지 asset으로 구성되어 있으며, 각 asset을 사용하여 자유롭게 UI를 구성할 수 있습니다.

| Asset | 설명 |
|-------|------|
| `icon` | 아이콘 이미지 |
| `headline` | 광고 제목 |
| `advertiser` | 광고주 이름 |
| `description` | 광고 설명 |
| `media` | 미디어 (이미지 또는 동영상) |
| `cta` | 행동 유도 버튼 |

> `headline`(제목), `icon`(아이콘), `media`(미디어) 중 **최소 1개**는 반드시 사용해야 합니다.

---

## 2. 네이티브 인스턴스 생성

`AdMixer` Inspector에 `iOS Native AdUnitId`를 입력하면 `Awake()` 시 400×200 네이티브 인스턴스가 자동 생성됩니다. 위치는 `iOS Native Top Position`으로 선택합니다 (`true`: 상단, `false`: 하단).

직접 인스턴스를 만들려면 `NAPSSPPluginIOS`의 정적 메서드를 호출합니다.

```csharp
// 위치 기반 생성 — adUnitId 는 int
NAPSSPPluginIOS.NativeViewInit(adUnitId, NAPSSPPluginIOS.SSPPositionTop, 300, 250);

// 좌표 기반 생성 — 원점은 화면 왼쪽 상단
NAPSSPPluginIOS.NativeViewInit(adUnitId, x, y, 300, 250);
```

| 매개변수 | 타입 | 설명 |
|---------|------|------|
| `adUnitId` | int | 네이티브 Adunit ID |
| `position` | int | `NAPSSPPluginIOS.SSPPositionTop`(0) 또는 `SSPPositionBottom`(1) |
| `x`, `y` | float | 좌표 기반 생성 시 왼쪽 상단 위치 |
| `width`, `height` | float | 네이티브 뷰 크기 |

---

## 3. 네이티브 광고 요청

로드 성공 시 화면에 자동으로 부착·노출됩니다.

```csharp
AdMixer.Instance.LoadNativeAd();
```

---

## 4. 네이티브 제거

```csharp
AdMixer.Instance.DestroyNativeAd();
```

---

## 5. 이벤트

| 이벤트 | 시그니처 | 설명 |
|-----------|------|------|
| `OnSuccessNative` | `Action` | 네이티브 광고 로드 성공 (자동 표시) |
| `OnFailNative` | `Action` | 네이티브 광고 로드 실패 |
| `OnTapNative` | `Action` | 네이티브 광고 클릭 |

```csharp
using UnityEngine;

public class NativeAd : MonoBehaviour
{
    void OnEnable()
    {
        NAPSSPPluginIOS.OnSuccessNative += OnSuccessNative;
        NAPSSPPluginIOS.OnFailNative    += OnFailNative;
        NAPSSPPluginIOS.OnTapNative     += OnTapNative;
    }

    void OnDisable()
    {
        NAPSSPPluginIOS.OnSuccessNative -= OnSuccessNative;
        NAPSSPPluginIOS.OnFailNative    -= OnFailNative;
        NAPSSPPluginIOS.OnTapNative     -= OnTapNative;
    }

    void Start()
    {
        AdMixer.Instance.LoadNativeAd();
    }

    void OnSuccessNative() { Debug.Log("네이티브 광고 로드 성공"); }
    void OnFailNative()    { Debug.Log("네이티브 광고 로드 실패"); }
    void OnTapNative()     { Debug.Log("네이티브 광고 클릭"); }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyNativeAd();
    }
}
```
