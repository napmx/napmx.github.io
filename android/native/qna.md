# Q&A (실전 문의 모음)

매체 연동 과정에서 실제로 접수된 질문과 답변을 정리했습니다. 일반적인 질문은 [FAQ](faq.md)를, 에러 코드는 [에러 코드](error-codes.md)를 먼저 확인하세요.

---

## 초기화 · 키 / 네트워크 설정

**Q. 각 네트워크 SDK를 `Application`에서 직접 초기화해야 하나요? (`PAGSdk.init`, `MobileAds.initialize` 등)**

대부분 **불필요**합니다. 워터폴에서 어댑터가 해당 네트워크 SDK를 **요청 시점에 자동(lazy) 초기화**합니다.

- `Application`에서 **제거 가능**: Pangle `PAGSdk.init(...)` 블록, Google `MobileAds.initialize()`.
- **유지 필수**: `AdMixer.getInstance().initialize(...)`(코어), Google AdManager의 `com.google.android.gms.ads.APPLICATION_ID` 매니페스트 meta-data.

> 자동 초기화에는 해당 네트워크의 초기화 키가 필요합니다. 예를 들어 Pangle은 `app_id`가 media-conf 또는 `setAdapterConfig`로 전달되어야 어댑터가 SDK를 초기화할 수 있습니다. 초기화 요건은 네트워크마다 다르므로, 특정 네트워크만 계속 실패한다면 [에러 코드 — 특정 네트워크 광고만 나오지 않는 경우](error-codes.md#특정-네트워크-광고만-나오지-않는-경우)를 확인하세요.

---

**Q. 서버(media-conf)에 네트워크 키가 안 내려와서 워터폴에서 `[SKIP] ... (Missing Keys)`로 빠집니다.**

해당 유닛에 네트워크 **필수 키**(예: Pangle `placement_id`, AppLovin `zone_id`)가 서버에 프로비저닝되지 않은 경우입니다. 두 가지 방법이 있습니다.

1. (권장) nap mx 운영팀에 요청하여 **media-conf에 키 프로비저닝**.
2. (보완) 매체가 `AdInfo.Builder.setAdapterConfig(adapterName, Map)`로 키를 직접 주입. 서버 값이 우선이며 **서버에 없는 키만** 채워집니다.

```java
Map<String, String> pangle = new HashMap<>();
pangle.put("placement_id", "982611225");   // 필수 — 이 키가 있어야 SKIP 안 됨
pangle.put("app_id", "8642312");           // Pangle SDK 초기화용 — 아래 설명 참고
AdInfo adInfo = new AdInfo.Builder(adUnitId)
    .setAdapterConfig(AdMixer.ADAPTER_PANGLE, pangle)
    .build();
```

- `placement_id`: **필수**. 이 키가 없으면 워터폴에서 `(Missing Keys)`로 SKIP됩니다.
- `app_id`: Pangle SDK가 아직 초기화되지 않은 경우 어댑터가 이 값으로 SDK를 초기화합니다. 앱이나 다른 경로에서 이미 Pangle SDK가 초기화되어 있지 않다면 함께 전달해야 합니다.

> ⚠️ **적용 범위 주의** — 클라이언트 키 주입은 해당 AdUnit의 **서버 워터폴 목록에 이미 포함된 네트워크**에만 적용됩니다. 네트워크 자체가 media-conf의 유닛 구성에 없으면 `setAdapterConfig`를 넣어도 워터폴에 추가되지 않으므로, 이 경우에는 1번(운영팀 프로비저닝)이 유일한 해결책입니다.

> 참고: `(Missing Keys)`의 실제 누락 키는 보통 `app_id`가 아니라 **per-placement 식별자**(Pangle `placement_id` / AppLovin `zone_id`)입니다. 필수 키 이름은 네트워크마다 다릅니다.

---

**Q. AppLovin SDK Key는 어디에 넣나요?**

번들 AppLovin 13.x는 코드 기반 초기화이므로, `AdInfo.Builder.setAdapterConfig(AdMixer.ADAPTER_APPLOVIN, {"sdkKey": "..."})`로 전달합니다(미지정 시 어댑터 기본 키 사용). 매니페스트 `applovin.sdk.key`는 사용하지 않습니다.

---

**Q. Naver Ad Manager의 `PUBLISHER_CD`는 우리가 매니페스트에 넣어야 하나요?**

**아니요.** `com.naver.gfpsdk.PUBLISHER_CD`는 nap mx(네트워크 사업자)가 **SDK(`admixer-naveradmanager` aar)에서 제공·관리**합니다. 매체 앱은 별도로 설정하지 마세요(이미 추가했다면 제거 권장). 최신 SDK 동기화 시 자동 포함됩니다.

> 운영 환경에서 광고가 노필(GFP_NO_FILL)이라면, 이는 PUBLISHER_CD가 아니라 **운영 Ad Unit ID 프로비저닝** 이슈입니다. 운영 Ad Unit으로 전환이 필요한지 운영팀에 확인하세요.

---

## 빌드 · 의존성

**Q. `Duplicate class com.google.android.gms.ads...` 같은 클래스 중복 빌드 오류가 납니다.**

이미 같은 네트워크 SDK를 직접(또는 타 솔루션으로) 사용 중인 경우입니다. 어댑터 의존성에서 중복 모듈을 `exclude` 하세요.

```gradle
implementation("io.github.nasmedia-tech:admixer-admanager:2.0.4") {
    exclude group: "com.google.android.gms", module: "play-services-ads"
}
```
exclude 후 의존성 트리에서 해당 SDK가 1개만 남는지 확인하세요.

---

**Q. Teads/Adfit/Pangle를 추가했더니 의존성을 못 찾습니다(`Failed to resolve`).**

해당 네트워크 전용 Maven 저장소를 `settings.gradle`에 추가해야 합니다(Adfit=`devrepo.kakao.com`, Pangle=`artifact.bytedance.com`, Teads=`sdk.teads.tv`+`teads.jfrog.io`). 필요한 저장소는 네트워크·SDK 버전에 따라 달라질 수 있으므로, 전체 목록은 [SDK 시작하기 — 네트워크별 추가 Maven 저장소](getting-started.md#1-3-네트워크별-추가-maven-저장소)를 기준으로 확인하세요. [FAQ — 빌드/Gradle](faq.md#빌드--gradle)도 참고할 수 있습니다.

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
| 닫힘/실패 콜백 후 · `Activity#onDestroy` | `stop()` |

---

**Q. `media-conf` 재동기화 직후 표시 중이던 풀스크린 광고가 다시 로드되거나 중복 노출됩니다.**

v2.0.0에서 수정되었습니다. 표시 중(SHOWING)/이미 로드된 유닛은 config 재동기화로 **자동 재로드되지 않으며**, 미디에이션 컨트롤러 중복 생성도 차단됩니다. 최신 SDK로 업데이트하고, 광고 닫힘/실패/`onDestroy` 시 `stop()` teardown을 호출하면 해당 유닛이 재동기화 대상에서 제외됩니다.

---

## 배너 / 네이티브 (생명주기 · 리스너)

**Q. 배너/네이티브의 `onResume`/`onPause`/`stop`을 꼭 연결해야 하나요?**

네, **필수**입니다. Activity 생명주기에 맞춰 `adView.onResume()`/`adView.onPause()`/`adView.stop()`을 연결하지 않으면 자동 갱신 타이머가 어긋나거나 메모리 누수가 발생할 수 있습니다.

---

**Q. 리스너 콜백(`onReceivedAd` 등)이 가끔 호출되지 않습니다.**

`AMMBannerView`는 `AdListener`를 내부적으로 `WeakReference`로 보유합니다. **익명 클래스로 바로 넘기면 GC에 수집**될 수 있으니, 리스너를 **멤버 변수**로 선언해 참조를 유지하세요(다른 포맷도 동일하게 멤버로 유지하는 것을 권장합니다).

```java
private final AdListener adListener = new AdListener() { /* ... */ };
// ...
adView.setAdViewListener(adListener);
```

---

**Q. 한 화면에 여러 개의 광고(배너+네이티브 등)를 띄워도 되나요?**

됩니다. 단 **AdUnit ID 1개당 광고 객체 1개** 원칙을 지키고, 각 객체의 생명주기 메서드(`onResume`/`onPause`/`stop`)를 모두 개별 연결하세요.

---

## 동의 전파 · 네트워크별 동작

**Q. GDPR/CCPA 동의값을 SSP가 하위 네트워크로 전파하나요?**

국내(한국) 이용자만 대상으로 하는 서비스라면 일반적으로 GDPR(EU)·CCPA(캘리포니아) 적용 대상이 아니어서 별도 설정이 필요하지 않습니다. **해외 서비스로 확장하거나 EU·미국 트래픽이 있다면** 네트워크마다 동의 전달 방식과 지원 범위가 달라 지면 구성에 따라 안내가 달라집니다 — [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의해 주세요. (적용 여부에 대한 최종 판단은 각 사의 법무 검토를 따르십시오.)

참고로 **Google·Pangle·Teads·Naver**는 매체가 IAB **TCF 규격 CMP**를 연동하면 각 네트워크 SDK가 동의 문자열을 자동으로 읽어갑니다(SDK 설정 불요). 그 외 네트워크는 동의 전달 방식이 다르거나 **자동 전파를 지원하지 않을 수 있으므로**, 지원 여부는 해당 네트워크 공식 문서를 참고하십시오. EU 트래픽이 있는데 동의가 반영되지 않는 것 같다면 앱의 CMP(TCF v2) 연동부터 확인하세요.

---

**Q. 아동 대상 앱은 무엇을 설정하나요?**

`AdMixer.setTagForChildDirectedTreatment(...)`를 설정하세요. Google Play **Families 정책**은 국가와 무관하게 적용되므로 국내 서비스도 필수입니다. 이 값은 SDK 자체 광고 요청에 반영되며, 하위 네트워크로는 AdManager·GMA NextGen(`setTagForChildDirectedTreatment`), Unity(`user.nonbehavioral`)로 전파됩니다.

> 아동 대상 설정을 노출하는 API는 네트워크마다 다르며, **일부 네트워크는 아동 대상 설정을 지원하지 않을 수 있습니다**(해당 네트워크 SDK가 관련 API를 제공하지 않는 경우). 지원 여부는 각 네트워크 공식 문서를 참고하십시오. 자세한 내용은 [개인정보 / 테스트 설정](privacy.md)을 참고하세요.

---

**Q. AdManager 배너 사이즈가 기기마다 다르게 보입니다.**

v2.0.0부터 AdManager **표준 배너는 디바이스 너비 기반 anchored adaptive**로 요청되어 높이가 기기마다 달라집니다(호스트 API 변경 없음, 렌더 사이즈만 변동). 높이 250(MREC 300×250)과 높이 480(320×480) 슬롯은 고정 크기로 종전대로 유지됩니다.

---

## Compose

**Q. Jetpack Compose에서 사용할 수 있나요?**

네. `admixer-compose` 모듈을 추가하면 `@Composable AdMixerBanner(...)`, `AdMixerNativeAd(...)`, `rememberInterstitialAd(...)` 등을 사용할 수 있습니다. 코어에 Compose 의존성을 강제하지 않는 선택 모듈입니다. 사용법은 [Compose 가이드](compose.md)를 참고하세요.

---

## 개인정보 / 테스트

**Q. QA용 테스트 광고를 실기기에서 띄우려면?**

`AdMixer.setTestMode(true)` + `AdMixer.setTestDeviceIds([GAID])`를 초기화 시 설정하세요. 기기 GAID는 안드로이드 설정 → Google → 광고에서 확인할 수 있습니다.

두 API는 반영되는 네트워크가 서로 다릅니다.

| API | 전파되는 네트워크 |
|---|---|
| `setTestDeviceIds(...)` | AdManager, GMA NextGen, AppLovin (각 SDK의 테스트 디바이스 설정) |
| `setTestMode(true)` | Unity(테스트 모드), Pangle(디버그 로그) |

> 테스트 광고 제공 방식은 네트워크 정책에 따라 달라질 수 있으며, **일부 네트워크는 테스트 디바이스 지정을 지원하지 않을 수 있습니다.** 지원 여부와 테스트 지면 발급 방법은 해당 네트워크 공식 문서를 참고하십시오. 자세한 내용은 [개인정보 / 테스트 설정](privacy.md)을 참고하세요.

---

## 문의

여기에 없는 문의는 **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)** 또는 GitHub 이슈로 등록해 주세요. `AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE)` 설정 후 `adb logcat | grep AdMixerSDK`로 수집한 로그를 함께 전달하면 진단이 빠릅니다.
