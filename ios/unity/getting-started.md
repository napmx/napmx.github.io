# iOS SDK 시작하기 - Unity

Unity 프로젝트에서 nap mx iOS SDK를 연동하는 방법입니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. 권한 설정 (Info.plist)

```xml
<key>NSUserTrackingUsageDescription</key>
<string>맞춤형 광고 제공을 위해 광고 추적 권한이 필요합니다.</string>
```

Google AdManager 사용 시:

```xml
<key>GADApplicationIdentifier</key>
<string>발급받은_GAD_APP_ID</string>
```

AppLovin 사용 시:

```xml
<key>AppLovinSdkKey</key>
<string>발급받은 키</string>
```

---

## 2. SDK 설정

Unity Package Manager 또는 `.unitypackage`를 통해 nap mx Unity Plugin을 설치합니다.

```
Assets > Import Package > Custom Package > NapSSP_Unity.unitypackage
```

---

## 3. SDK 초기화 (C#)

```csharp
using UnityEngine;

public class AdManager : MonoBehaviour
{
    void Awake()
    {
        NapSspIOS.InitSdk("발급받은_MEDIA_KEY");
    }
}
```

---

## 4. 앱 추적 권한 요청 (iOS 14+)

```csharp
IEnumerator RequestTrackingAuthorization()
{
#if UNITY_IOS && !UNITY_EDITOR
    yield return Application.RequestUserAuthorization(UserAuthorization.Microphone);
    NapSspIOS.RequestTrackingAuthorization((status) => {
        // status: authorized, denied, restricted, notDetermined
        NapSspIOS.InitSdk("발급받은_MEDIA_KEY");
    });
#else
    NapSspIOS.InitSdk("발급받은_MEDIA_KEY");
    yield return null;
#endif
}
```

---

## 5. 광고 타입별 흐름

| 광고 타입 | 로드 | 표시 | 제거 |
|-----------|------|------|------|
| 배너 | `LoadBanner()` | `ShowBanner()` | `DestroyBanner()` |
| 전면 | `LoadInterstitial()` | `ShowInterstitial()` | `DestroyInterstitial()` |
| 네이티브 | `LoadNativeAd()` | — | `DestroyNativeAd()` |
| 리워드 동영상 | `LoadRewardVideo()` | `ShowRewardVideo()` | — |
| 동영상 | `LoadVideoAd()` | — | `DestroyVideoAd()` |

---

## 다음 단계

- [배너 - Unity](/ios/unity/banner)
- [리워드 동영상 - Unity](/ios/unity/rewarded-video)
