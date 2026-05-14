# 배너 - Unity (iOS)

배너 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

---

## 1. 배너(Banner) 광고

### 1-1. 배너 인스턴스 생성

배너 광고 노출을 위해 `BannerViewInit` API를 호출하여 인스턴스를 생성합니다.

```csharp
// 위치 기반 생성
BannerViewInit("발급받은_ADUNIT_ID", SSPPosition.SSPPositionBottom, 320, 50);
```

| 매개변수 | 설명 |
|---------|------|
| `adUnitId` | 로드할 배너 광고의 광고 단위 ID |
| `position` | 배너 뷰 위치 (`SSPPositionTop`: 상단, `SSPPositionBottom`: 하단) |
| `width` | 배너 뷰의 가로 길이 |
| `height` | 배너 뷰의 세로 길이 |

**맞춤 위치로 배너 뷰 만들기**

`position` 대신 x, y 좌표로 세부 위치를 지정할 수 있습니다. 원점은 화면의 왼쪽 상단입니다.

```csharp
BannerViewInit("발급받은_ADUNIT_ID", x, y, 320, 50);
```

### 1-2. 배너 광고 요청

```csharp
LoadBanner();
```

### 1-3. Delegate

배너 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.

| 델리게이트 | 설명 |
|-----------|------|
| `OnSuccessBanner` | 배너 광고 로드 성공 |
| `OnFailBanner` | 배너 광고 로드 실패 |
| `OnTapBanner` | 배너 광고 클릭 |

```csharp
public class BannerAd : MonoBehaviour
{
    void Start()
    {
        BannerViewInit("발급받은_ADUNIT_ID", SSPPosition.SSPPositionBottom, 320, 50);
        LoadBanner();
    }

    void OnSuccessBanner()
    {
        Debug.Log("배너 광고 로드 성공");
    }

    void OnFailBanner(string errorMsg)
    {
        Debug.Log($"배너 광고 로드 실패: {errorMsg}");
    }

    void OnTapBanner()
    {
        Debug.Log("배너 광고 클릭");
    }
}
```

---

## 2. 전면 배너(Interstitial) 광고

### 2-1. 전면 배너 인스턴스 생성 및 설정

전면 배너 광고 노출을 위해 `InterstitialInit` API를 호출하여 인스턴스를 생성합니다.

```csharp
InterstitialInit("발급받은_ADUNIT_ID");
```

### 2-2. 전면 배너 광고 요청

```csharp
LoadInterstitial();
```

### 2-3. Delegate

| 델리게이트 | 설명 |
|-----------|------|
| `OnSuccessInterstitial` | 전면 광고 로드 성공 |
| `OnFailInterstitial` | 전면 광고 로드 실패 |
| `OnCloseInterstitial` | 전면 광고 닫기 |
| `OnTapInterstitial` | 전면 광고 클릭 |

```csharp
public class InterstitialAd : MonoBehaviour
{
    void Start()
    {
        InterstitialInit("발급받은_ADUNIT_ID");
        LoadInterstitial();
    }

    void OnSuccessInterstitial()
    {
        ShowInterstitial();
    }

    void OnFailInterstitial(string errorMsg)
    {
        Debug.Log($"전면 광고 로드 실패: {errorMsg}");
    }

    void OnCloseInterstitial()
    {
        Debug.Log("전면 광고 닫힘");
    }

    void OnTapInterstitial()
    {
        Debug.Log("전면 광고 클릭");
    }
}
```

### 2-4. 전면 배너 커스텀

전면 배너 형식으로 `basic`, `popup`, `countDown` 세 가지 형태를 제공합니다.  
일부 네트워크(AdMixer, AdFit, MobWith)에만 적용됩니다.

```csharp
// type: 0 = basic, 1 = popup, 2 = countDown
InterstitialSetType(1);
```

| 형식 | 설명 |
|------|------|
| `basic` (0) | 우측 상단에 "X" 이미지 형태의 닫기 버튼 노출 |
| `popup` (1) | 광고 소재 하단에 텍스트 형태의 닫기 버튼 노출 |
| `countDown` (2) | 설정된 시간이 지난 후 닫기 버튼 노출 |

**popup 커스텀**

`InterstitialSetPopupOption`을 통해 닫기 버튼의 텍스트, 타이틀 색상, 버튼 배경색을 설정할 수 있습니다.

```csharp
InterstitialSetPopupOption("닫기", "#000000", "#FFFFFF");
```

**countDown 커스텀**

`InterstitialSetCountDownOption`을 통해 카운트다운 시간 및 UI 타입을 설정할 수 있습니다.

```csharp
// time: 카운트다운 시간 (최소 2초 ~ 최대 5초)
// countDownType: 0 = 게이지 형태, 1 = 텍스트 형태
InterstitialSetCountDownOption(3, 0);
```
