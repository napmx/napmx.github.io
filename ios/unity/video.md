# 동영상 - Unity (iOS)

동영상 광고를 추가하기 전 [iOS SDK 시작하기 - Unity](/ios/unity/getting-started) 설정을 완료해주세요.

> 전면(Interstitial) 동영상 광고 연동을 원하시는 경우, 아래 **전면 비디오** 섹션을 참고해주세요.

---

## 1. 동영상(Video) 광고

### 1-1. 비디오 인스턴스 생성

비디오 광고 노출을 위해 `VideoViewInit` API를 호출하여 인스턴스를 생성합니다.

비디오 뷰의 왼쪽 상단 모서리는 생성자에 전달된 x, y값에 배치됩니다. 원점은 화면의 왼쪽 상단입니다.

```csharp
VideoViewInit("발급받은_ADUNIT_ID", x, y);
```

### 1-2. 비디오 광고 요청

```csharp
LoadVideoAd();
```

### 1-3. Delegate

비디오 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.

| 델리게이트 | 설명 |
|-----------|------|
| `OnSuccessVideo` | 비디오 광고 로드 성공 |
| `OnFailVideo` | 비디오 광고 로드 실패 |
| `OnSkipVideo` | 비디오 광고 내 skip 버튼 클릭 |
| `OnTapVideoViewMore` | 비디오 광고 내 더보기 버튼 클릭 |
| `OnCompleteVideo` | 비디오 광고 재생 완료 |

```csharp
public class VideoAd : MonoBehaviour
{
    void Start()
    {
        VideoViewInit("발급받은_ADUNIT_ID", 0, 0);
        LoadVideoAd();
    }

    void OnSuccessVideo()
    {
        Debug.Log("비디오 광고 로드 성공");
    }

    void OnFailVideo(string errorMsg)
    {
        Debug.Log($"비디오 광고 로드 실패: {errorMsg}");
    }

    void OnSkipVideo()
    {
        Debug.Log("비디오 광고 skip");
    }

    void OnTapVideoViewMore()
    {
        Debug.Log("더보기 버튼 클릭");
    }

    void OnCompleteVideo()
    {
        Debug.Log("비디오 광고 재생 완료");
    }
}
```

---

## 2. 전면 비디오(Interstitial Video) 광고

### 2-1. 전면 비디오 인스턴스 생성

전면 비디오 광고 노출을 위해 `VideoInterstitialInit` API를 호출하여 인스턴스를 생성합니다.

```csharp
VideoInterstitialInit("발급받은_ADUNIT_ID");
```

### 2-2. 전면 비디오 광고 요청

```csharp
LoadVideoInterstitial();
```

### 2-3. Delegate

전면 비디오 광고에서 발생하는 이벤트에 대한 델리게이트를 제공합니다.

| 델리게이트 | 설명 |
|-----------|------|
| `OnSuccessVideoInterstitial` | 전면 비디오 광고 로드 성공 |
| `OnFailVideoInterstitial` | 전면 비디오 광고 로드 실패 |
| `OnCloseVideoInterstitial` | 전면 비디오 광고 닫기 |
| `OnTapVideoInterstitialViewMore` | 전면 비디오 광고 내 더보기 버튼 클릭 |
| `OnCompleteVideoInterstitial` | 전면 비디오 광고 재생 완료 |

```csharp
public class VideoInterstitialAd : MonoBehaviour
{
    void Start()
    {
        VideoInterstitialInit("발급받은_ADUNIT_ID");
        LoadVideoInterstitial();
    }

    void OnSuccessVideoInterstitial()
    {
        ShowVideoInterstitial();
    }

    void OnFailVideoInterstitial(string errorMsg)
    {
        Debug.Log($"전면 비디오 광고 로드 실패: {errorMsg}");
    }

    void OnCloseVideoInterstitial()
    {
        Debug.Log("전면 비디오 광고 닫힘");
    }

    void OnTapVideoInterstitialViewMore()
    {
        Debug.Log("더보기 버튼 클릭");
    }

    void OnCompleteVideoInterstitial()
    {
        Debug.Log("전면 비디오 광고 재생 완료");
    }
}
```
