# 자주 묻는 질문 (FAQ)

---

## 초기화 및 설정

**Q. 하나의 앱에 여러 개의 Media Key를 적용할 수 있나요?**

한 개의 앱에는 한 개의 Media Key만 적용 가능합니다.

---

**Q. 동일한 AdUnit ID로 여러 광고 객체를 생성해도 되나요?**

안 됩니다. 한 개의 AdUnit ID는 한 개의 광고 객체에서만 사용할 수 있습니다. 동일한 AdUnit ID를 여러 객체에서 사용하면 광고가 정상적으로 동작하지 않습니다.

---

**Q. SDK 초기화는 언제 어디서 해야 하나요?**

`Application.onCreate()`에서 광고 로드 전에 1회만 호출하세요.

```java
@Override
public void onCreate() {
    super.onCreate();
    AdMixer.getInstance().initialize(this, MEDIA_KEY, adUnits);
}
```

---

**Q. 상세 로그를 확인하려면 어떻게 해야 하나요?**

초기화 전에 로그 레벨을 `VERBOSE`로 설정하세요.

```java
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE);
```

LogCat에서 `AdMixer` 태그로 필터링하면 SDK 내부 동작을 상세히 확인할 수 있습니다.

---

## 배너 광고

**Q. `AdView`를 레이아웃에 추가하지 않았는데 광고가 표시되지 않습니다.**

광고가 표시되려면 반드시 `container.addView(adView)`로 레이아웃에 추가해야 합니다.

- **콜백 기반 노출**: `loadAd()` 후 `onReceivedAd()` 콜백에서 `container.addView(adView)`와 `adView.showAd()`를 호출하세요.
- **완전 지연 노출** (콜백 이후 원하는 시점에 표시): `AdInfo.Builder.isLoadOnly(true)`를 설정한 후 `loadAd()`를 호출하세요. 광고가 수신되어도 자동 노출되지 않으며, 원하는 시점에 `container.addView(adView)` + `adView.showAd()`를 호출합니다.

---

**Q. AdUnit 설정 사이즈와 다른 광고 사이즈가 노출됩니다.**

AdUnit 설정 사이즈는 보장되지 않을 수 있습니다. 광고 네트워크 및 소재 유형에 따라 실제 노출 사이즈가 달라질 수 있습니다. `WRAP_CONTENT`로 레이아웃을 설정하면 소재 크기에 맞게 자동 조정됩니다.

---

## 네이티브 광고

**Q. 네이티브 광고 레이아웃에서 RelativeLayout을 반드시 사용해야 하나요?**

RelativeLayout 사용을 강력히 권장합니다. 다른 레이아웃을 사용해야 하는 경우, `RelativeLayout`을 부모 뷰로 감싸고 원하는 레이아웃을 내부에 넣는 방식을 사용할 수 있습니다.

---

**Q. `NativeAdView`에 `setViewBinder()`를 설정하지 않으면 어떻게 되나요?**

`setViewBinder()` 없이는 네이티브 광고가 렌더링되지 않습니다. `loadNativeAd()` 호출 전에 반드시 `setViewBinder()`를 설정하세요.

---

## 미디에이션

**Q. Adfit 상용 광고는 언제부터 응답되나요?**

Adfit은 매체 심사 과정을 통해 상용 광고가 응답됩니다.

```
연동 테스트 → 매체 라이브 → Adfit 매체 심사 신청 → 심사 완료 → 상용 광고 송출
```

심사 신청은 nap_mx@nasmedia.co.kr로 문의하세요.

---

**Q. Google AdManager와 다른 네트워크를 동시에 운영 중인 앱에 추가해도 되나요?**

Google AdManager는 기존 운영 중인 지면과 **다른 지면**에 한해 중복 사용이 가능합니다. 동일 지면에서 사용하는 경우 `exclude`로 중복을 방지하세요. 자세한 내용은 [SDK 시작하기 — 네트워크 SDK 중복 예외 처리](getting-started.md#네트워크-sdk-중복-예외-처리)를 참고하세요.

---

## 리워드 동영상

**Q. EARNEDREWARD 이벤트와 COMPLETION 이벤트의 차이는 무엇인가요?**

- `EARNEDREWARD`: 리워드 지급 조건 충족 (동영상 시청 완료). **리워드 지급은 이 이벤트에서 처리하세요.**
- `COMPLETION`: 동영상 재생이 끝까지 완료됨. 네트워크에 따라 `EARNEDREWARD`와 동시 또는 별도로 발생할 수 있습니다.

---

**Q. 사용자가 Skip을 누르면 리워드가 지급되나요?**

아니요. `SKIPPED` 이벤트는 리워드 미지급 상황입니다. 리워드는 반드시 `EARNEDREWARD` 이벤트에서만 지급하세요.

---

## 전면 광고

**Q. 전면 광고가 뒤로가기(BACK)로 닫히지 않습니다.**

v2.0.0부터 전면 광고는 **BACK 키를 기본 차단**합니다(닫기는 'X' 버튼으로만). 뒤로가기 닫기를 허용하려면 `PopupInterstitialAdOption.setDisableBackKey(false)`를 명시하세요. 자세한 내용은 [전면 배너 광고 — 뒤로가기 키 정책](interstitial.md#뒤로가기back-키-정책) 참고.

---

**Q. 표시 중인 광고는 유지하면서 진행 중 로드만 취소하려면?**

`cancelLoad()`를 호출하세요. 로딩 중일 때만 취소하고 표시 중(SHOWING)이면 아무 동작도 하지 않습니다. 전체 정리는 `stopXxx()`/`onDestroy()`입니다.

---

**Q. `onReceivedAd`에서 `startInterstitial()`을 불렀더니 광고가 계속 재로드됩니다.**

`startInterstitial()`은 "로드 + 자동 노출"이라 수신 콜백 안에서 호출하면 매번 **재로드**됩니다. 수신 후 표시는 **`showInterstitial()`** 을 사용하세요(자동 노출을 원하면 `startInterstitial()`을 **1회만** 호출). v2.0.0 SDK는 이미 로드된(READY) 광고를 재로드 요청으로 파기하지 않고, 빠른 연속 재로드는 백오프(지연) 처리하여 무한 루프를 차단합니다.

---

**Q. `maxRetryCountInSlot`은 무엇이고 권장값은?**

광고 수신 실패 시 슬롯 내 자동 재시도 횟수입니다. `-1`(기본)/`0`은 무제한, 양수 N은 최대 N회입니다(주로 배너에 적용, 재시도 간격 최소 5초). 무제한보다 **유한값(예 3~5)** 을 권장합니다. 전면/리워드는 SDK 루프 가드가 무한 재로드를 별도로 차단합니다.

---

## 개인정보 및 테스트

**Q. GDPR/CCPA 동의를 각 네트워크에 일일이 설정해야 하나요?**

아니요. `AdMixer.setGdprConsent(...)`, `setCcpaDoNotSell(...)`, `setTagForChildDirectedTreatment(...)`를 전역으로 설정하면 워터폴에서 각 네트워크로 자동 전파됩니다. (일부 네트워크는 공식 API 부재로 미전파 — [개인정보 동의 및 테스트 설정](privacy.md) 매핑표 참고)

---

**Q. 테스트 광고는 어떻게 받나요?**

`AdMixer.setTestMode(true)`와 `AdMixer.setTestDeviceIds([GAID])`를 초기화 시 설정하세요. AppLovin·Unity·AdManager·Pangle에 반영됩니다.

---

## 문의

추가 문의사항은 **nap_mx@nasmedia.co.kr**로 연락해 주세요.
