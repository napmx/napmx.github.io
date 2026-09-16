# 네이티브 - Unity (Android)

광고를 추가하기 전 [Android SDK 시작하기 - Unity](/android/unity/getting-started) 설정을 완료해주세요.

> ⚠️ **에셋 텍스트만 꺼내 Unity UI에 그리면 노출·클릭이 집계되지 않습니다.** 네이티브 광고는 SDK에 **뷰를 등록**해야 임프레션과 클릭이 잡히고, 그래야 수익으로 인정됩니다. 플러그인은 네이티브 레이아웃(`nativeadlayout-release.aar`)으로 SDK가 직접 렌더링하는 방식을 사용합니다. (네이티브 기준 동작은 [네이티브 광고](/android/native/native-ad) 참고)

---

## 1. 구성

`nativeadlayout-release.aar`에 포함된 레이아웃 중 하나를 `AdMixer` Inspector의 `Android Native Use Large Layout`으로 선택합니다.

| 값 | 레이아웃 | 크기 |
|---|---|---|
| `true` (기본) | `item_320x480` | 대형 |
| `false` | `item_320x100` | 소형 |

레이아웃은 아이콘 · 제목 · 광고주 · 설명 · 메인 이미지 · CTA 버튼으로 구성됩니다. 다른 레이아웃(`item_300x250`, `item_320x50`)을 쓰거나 View ID를 바꾸려면 [시작하기 - 네이티브 레이아웃](/android/unity/getting-started?id=_2-5-네이티브-레이아웃)을 참고하세요.

---

## 2. 네이티브 인스턴스 설정

Adunit ID는 `AdMixer` Inspector의 `Android Native AdUnitId`를 사용합니다. 로드 성공 시 화면 **하단**에 자동으로 부착됩니다.

---

## 3. 네이티브 광고 요청

```csharp
AdMixer.Instance.LoadNativeAd();   // 로드 성공 시 자동 표시
```

---

## 4. 네이티브 제거

```csharp
AdMixer.Instance.DestroyNativeAd();
```

---

## 5. 이벤트

`AdMixerAdListener`로 수신합니다.

| 이벤트 | 설명 |
|---|---|
| `OnReceivedAd` | 네이티브 광고 로드 성공 (자동 표시) |
| `OnFailedToReceiveAd` | 네이티브 광고 로드 · 표시 실패 |
| `OnEventAd("DISPLAYED")` | 네이티브 광고 노출 |
| `OnEventAd("CLICK")` | 네이티브 광고 클릭 |
| `OnEventAd("NATIVE_DESTROYED")` | 제거 완료 |

```csharp
using UnityEngine;

public class NativeAd : MonoBehaviour
{
    void Start()
    {
        AdMixer.Instance.LoadNativeAd();
    }

    void OnDestroy()
    {
        AdMixer.Instance.DestroyNativeAd();
    }
}
```
