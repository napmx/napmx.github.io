# Q&A (실전 문의 모음)

매체 연동 과정에서 실제로 접수된 질문과 답변을 정리했습니다. 일반적인 질문은 [FAQ](faq.md)를 먼저 확인하세요.

---

## 초기화 · 키 / 네트워크 설정

**Q. 각 네트워크 SDK를 `Application`에서 직접 초기화해야 하나요? (PAGSdk.init, MobileAds.initialize 등)**

대부분 **불필요**합니다. nap ssp는 워터폴에서 각 네트워크 SDK를 어댑터가 **요청 시 자동(lazy) 초기화**합니다.

- `Application`에서 **제거 가능**: Pangle `PAGSdk.init(...)` 블록, Google `MobileAds.initialize()`.
- **유지 필수**: `AdMixer.getInstance().initialize(...)`(코어), Google AdManager의 `com.google.android.gms.ads.APPLICATION_ID` 매니페스트 meta-data.

---

**Q. 서버(media-conf)에 네트워크 키가 안 내려와서 워터폴에서 `[SKIP] ... (Missing Keys)`로 빠집니다.**

해당 유닛에 네트워크 **필수 키**(예: Pangle `placement_id`, AppLovin `zone_id`)가 서버에 프로비저닝되지 않은 경우입니다. 두 가지 방법이 있습니다.

1. (권장) nap ssp 운영팀에 요청하여 **media-conf에 키 프로비저닝**.
2. (보완) 매체가 `AdInfo.Builder.setAdapterConfig(adapterName, Map)`로 키를 직접 주입. 서버 값이 우선이며 **서버에 없는 키만** 채워집니다.

```java
Map<String, String> pangle = new HashMap<>();
pangle.put("placement_id", "982611225");   // 필수 — 이 키가 있어야 SKIP 안 됨
pangle.put("app_id", "8642312");            // 선택
AdInfo adInfo = new AdInfo.Builder(adUnitId)
    .setAdapterConfig(AdMixer.ADAPTER_PANGLE, pangle)
    .build();
```

> 참고: `(Missing Keys)`의 실제 누락 키는 보통 `app_id`가 아니라 **per-placement 식별자**(Pangle `placement_id` / AppLovin `zone_id`)입니다.

---

**Q. AppLovin SDK Key는 어디에 넣나요?**

번들 AppLovin 13.x는 코드 기반 초기화이므로, `AdInfo.Builder.setAdapterConfig(AdMixer.ADAPTER_APPLOVIN, {"sdkKey": "..."})`로 전달합니다(미지정 시 어댑터 기본 키 사용). 매니페스트 `applovin.sdk.key`는 사용하지 않습니다.

---

**Q. Naver Ad Manager의 `PUBLISHER_CD`는 우리가 매니페스트에 넣어야 하나요?**

**아니요.** `com.naver.gfpsdk.PUBLISHER_CD`는 nap ssp(네트워크 사업자)가 **SDK(`admixer-naveradmanager` aar)에서 제공·관리**합니다. 매체 앱은 별도로 설정하지 마세요(이미 추가했다면 제거 권장). 최신 SDK 동기화 시 자동 포함됩니다.

> 운영 환경에서 광고가 노필(GFP_NO_FILL)이라면, 이는 PUBLISHER_CD가 아니라 **운영 Ad Unit ID 프로비저닝** 이슈입니다. 운영 Ad Unit으로 전환이 필요한지 운영팀에 확인하세요.

---

## 전면 / 리워드 (생명주기)

**Q. 화면 전환/백그라운드 진입 시 표시 중인 광고는 끊지 않고 진행 중 로드만 취소하고 싶습니다.**

`cancelLoad()`를 사용하세요. 내부 상태가 로딩 중일 때만 취소하고, **표시 중(SHOWING)이면 아무 동작도 하지 않습니다**. 전체 정리는 `stop()`입니다.

```java
interstitialAd.cancelLoad();  // 진행 중 로드만 취소 (표시 중이면 no-op)
// 화면 종료 시:
interstitialAd.stop();        // 전체 리소스 해제 (필수)
```

| 상황 | 호출 |
|---|---|
| 화면 전환·백그라운드 (표시 광고 유지) | `cancelLoad()` |
| 닫힘/실패 콜백 후 · `Activity#onDestroy` | `stopXxx()` → `onDestroy()` |

---

**Q. `media-conf` 재동기화 직후 표시 중이던 풀스크린 광고가 다시 로드되거나 중복 노출됩니다.**

v2.0.0에서 수정되었습니다. 표시 중(SHOWING)/이미 로드된 유닛은 config 재동기화로 **자동 재로드되지 않으며**, 미디에이션 컨트롤러 중복 생성도 차단됩니다. 최신 SDK로 업데이트하세요. 추가로, 광고 닫힘/실패/`onDestroy` 시 `stopXxx()` teardown을 호출하면 해당 유닛이 재동기화 대상에서 제외됩니다.

---

## 개인정보 / 테스트

**Q. GDPR/CCPA 동의값을 SSP가 하위 네트워크로 전파하나요?**

네. `AdMixer.setGdprConsent / setCcpaDoNotSell / setTagForChildDirectedTreatment`로 설정하면 워터폴에서 각 어댑터가 자기 네트워크 privacy API로 전파합니다. 자세한 매핑은 [개인정보 동의 및 테스트 설정](privacy.md)을 참고하세요. (일부 네트워크는 공식 API 부재로 미전파)

---

**Q. CCPA는 US Privacy 문자열과 DoNotSell 플래그 중 무엇을 쓰나요?**

국내(KR) 서비스로 CCPA 대상이 아니면 설정이 불필요합니다. 필요 시 단순한 `setCcpaDoNotSell(boolean)`을 권장하며, IAB CMP를 연동한 경우에만 `setUsPrivacy("1YNN")` 문자열을 사용하세요.

---

**Q. QA용 테스트 광고를 실기기에서 띄우려면?**

`AdMixer.setTestMode(true)` + `AdMixer.setTestDeviceIds([GAID])`를 초기화 시 설정하세요. AppLovin·Unity·AdManager·Pangle(debugLog)에 반영됩니다. [개인정보 동의 및 테스트 설정](privacy.md) 참고.

---

## 문의

여기에 없는 문의는 **nap_mx@nasmedia.co.kr** 또는 GitHub 이슈로 등록해 주세요.
