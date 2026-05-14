# 네이티브 - Unity (iOS)

네이티브 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

---

## 1. 구성

`Plugin/iOS/NativeTemplates` 내부의 `NapSSPNativeTemplateView.xib` 파일을 열어 네이티브 뷰를 커스텀해주세요.

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

## 2. 네이티브 인스턴스 생성 및 설정

네이티브 광고 노출을 위해 `NativeViewInit` API를 호출하여 인스턴스를 생성합니다.

```csharp
// 위치 기반 생성
NativeViewInit("발급받은_ADUNIT_ID", 0, 300, 250);
```

| 매개변수 | 설명 |
|---------|------|
| `adUnitId` | 로드할 네이티브 광고의 광고 단위 ID |
| `position` | 네이티브 뷰 위치 (`0`: 화면 상단, `1`: 화면 하단) |
| `width` | 네이티브 뷰의 가로 길이 |
| `height` | 네이티브 뷰의 세로 길이 |

**맞춤 위치로 네이티브 뷰 만들기**

`position` 대신 x, y 좌표로 세부 위치를 지정할 수 있습니다. 원점은 화면의 왼쪽 상단입니다.

```csharp
NativeViewInit("발급받은_ADUNIT_ID", x, y, 300, 250);
```

---

## 3. 네이티브 광고 요청

```csharp
LoadNativeAd();
```

---

## 4. Delegate

네이티브 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.

| 델리게이트 | 설명 |
|-----------|------|
| `OnSuccessNative` | 네이티브 광고 로드 성공 |
| `OnFailNative` | 네이티브 광고 로드 실패 |
| `OnTapNative` | 네이티브 광고 클릭 |

```csharp
public class NativeAd : MonoBehaviour
{
    void Start()
    {
        NativeViewInit("발급받은_ADUNIT_ID", 0, 300, 250);
        LoadNativeAd();
    }

    void OnSuccessNative()
    {
        Debug.Log("네이티브 광고 로드 성공");
    }

    void OnFailNative(string errorMsg)
    {
        Debug.Log($"네이티브 광고 로드 실패: {errorMsg}");
    }

    void OnTapNative()
    {
        Debug.Log("네이티브 광고 클릭");
    }

    void OnDestroy()
    {
        DestroyNativeAd();
    }
}
```
