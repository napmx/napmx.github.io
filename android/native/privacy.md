# 개인정보 / 테스트 설정

이 페이지에서는 **아동 대상(child-directed) 설정**과 **테스트 모드/테스트 디바이스** 설정을 안내합니다.

> ℹ️ 두 설정 모두 `AdMixer`에 **전역으로 1회** 설정하면 워터폴에서 각 광고가 로드될 때 SDK가 각 네트워크로 전파합니다.

---

## 아동 대상 앱 설정 (Child-Directed)

> 🚨 **아동 대상 앱이라면 반드시 설정하세요.** Google Play의 **Families 정책**은 아동 대상 앱이 광고 SDK에 child-directed 플래그를 전달하도록 요구하며, 이는 **국가와 무관하게** 적용됩니다. 누락 시 정책 위반 및 Google Ad Manager 계정 조치 대상이 될 수 있습니다.

`Application.onCreate()`에서 설정하세요.

#### Java
```java
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE);  // 아동 대상
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_FALSE); // 일반 대상
```

#### Kotlin
```kotlin
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE)
```

> ℹ️ `TRUE`(아동 대상) / `FALSE`(일반 대상) / **미설정**은 서로 다른 상태입니다. 처음부터 값을 설정하지 않으면
> SDK는 새 아동 대상 신호를 강제하지 않고 각 네트워크의 기존 설정·기본 정책을 따릅니다. AdMixer가 Google
> 전역값을 덮어쓴 뒤 미설정으로 전환하면 덮어쓰기 직전 값을 복원하며, 매체가 그 사이 직접 변경한 값은 보존합니다.
> **Unity MetaData에는 clear API가 없으므로 `TRUE`를 설정한 뒤 일반 대상으로 전환할 때는 반드시 `FALSE`를
> 명시하세요.** 미지정 트래픽을 아동 트래픽으로 취급할 수도 있습니다.
> `TRUE`/`FALSE` 이외의 값을 넣으면 미설정으로 처리됩니다.

### 네트워크별 전파

SDK에 설정한 값은 각 네트워크 SDK가 제공하는 API로 전달됩니다. **네트워크마다 받아들이는 항목이 다릅니다.**

| 네트워크 | 아동 대상(COPPA) | GDPR 동의 | CCPA(판매 거부) |
|---|---|---|---|
| **Google AdManager** · **GMA NextGen** | ✅ `tagForChildDirectedTreatment` | — | — |
| **Naver Ad Manager** | ✅ `childDirectedTreatment` | — | — |
| **Kakao Adfit** | ✅ `setRestrictedAge` | ✅ | — |
| **Unity Ads** | ✅ `user.nonbehavioral` (＋대시보드 설정 필요) | ✅ | ✅ |
| **AppLovin** | 🚫 아동 대상 시 **이 네트워크를 사용하지 않음** | ✅ | ✅ |
| **Pangle** | ⚠️ 비개인화 광고로 강등(COPPA 전용 API 없음) | — | ✅ |
| **Teads** | — | — | ✅ (US Privacy 문자열) |
| **MobWith** | — | — | — |

- **✅** 는 해당 네트워크 SDK의 아동 대상 전용 API로 값이 그대로 전달됩니다.
- **🚫 AppLovin** — AppLovin 공식 정책은 아동 사용자에게 *"SDK를 초기화하거나 어떤 형태로도 사용하지 말 것"*을
  요구하며, 아동 대상 플래그를 받는 API 자체가 없습니다. 따라서 아동 대상으로 설정하면 SDK가 **AppLovin을
  워터폴에서 건너뛰고** 다음 네트워크로 넘어갑니다(해당 지면의 AppLovin 수익은 발생하지 않습니다).
  이 보장은 **첫 광고 요청 전에 COPPA를 설정한 경우**에 한합니다. 성인 트래픽으로 AppLovin을 이미 초기화한 뒤
  같은 프로세스에서 COPPA를 켜면 요청은 스킵되지만 SDK 자체를 되돌려 초기화 해제할 API는 없습니다.
- **⚠️ Pangle** — Pangle SDK는 COPPA 설정 API를 제거했습니다(v7.1.0.4). 아동 대상으로 설정하면 SDK가
  **개인화 광고 비동의**로 전달해 행동기반 타겟팅을 끄지만, 이는 완화책이며 **COPPA 준수를 보증하지 않습니다.**
  아동 대상 지면에서 Pangle 사용 여부는 담당자와 협의해 주세요.
- **⚠️ Unity Ads** — SDK 전달 외에 **Unity Monetization 대시보드**에서 앱 단위 연령 설정
  (*"This app is directed to children"* / *Mixed Audience*)이 함께 되어 있어야 합니다.
- **—** 는 해당 네트워크 SDK에 대응 설정 API가 없어 값을 설정해도 반영되지 않는 항목입니다.
- 설정하지 않은 항목은 새 값을 강제하지 않습니다. 단, 앞선 AdMixer 명시값으로 덮어쓴 Google 전역값은
  미설정 전환 시 안전하게 복원합니다. Unity의 `TRUE → 미설정`은 clear할 수 없으므로 위 안내대로 `FALSE`를 사용하세요.

### 커스텀 어댑터 마이그레이션

`applyPrivacy(Context, PrivacyConsent)`는 네트워크 SDK의 첫 요청부터 신호를 적용하기 위해
`initAdapter(...)` **이전에** 호출됩니다. 외부 커스텀 어댑터는 `initAdapter`가 채우는 필드에 의존하지 말고,
필요한 Context는 `applyPrivacy` 인자로 받은 값을 사용하세요. 적용 중 예외는 SDK error 로그에 기록되지만,
해당 네트워크의 개인정보 신호가 적용되지 않을 수 있으므로 어댑터 업데이트 시 반드시 전파 테스트를 수행하세요.

> ⚠️ 위 매핑은 **각 네트워크 SDK 버전에 따라 변경될 수 있습니다.** 규제 준수가 필요한 서비스라면 각 네트워크 공식 문서에서 최신 요구사항을 확인하고, 아래 안내대로 문의해 주세요.

> ℹ️ **IAB TCF 규격 CMP를 연동한 경우** — 일부 네트워크 SDK는 CMP가 저장한 동의 문자열을 직접 읽어갑니다. 이 경로는 nap mx 설정과 무관하게 동작하며, 지원 여부는 네트워크마다 다를 수 있습니다.

---

## GDPR / CCPA 동의 전파

국내(한국) 서비스는 GDPR(EU)·CCPA(미국 캘리포니아) 적용 대상이 아니므로 별도 설정이 필요하지 않습니다.

**해외 서비스로 확장하거나 EU·미국 트래픽이 있는 경우**, 네트워크마다 동의 전달 방식과 지원 범위가 달라 지면 구성에 따라 안내가 달라집니다. **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의**해 주시면 사용 중인 네트워크 조합에 맞춰 안내해 드립니다.

> ℹ️ Google·Pangle·Teads·Naver는 매체가 **IAB TCF 규격 CMP**를 연동하면 각 네트워크 SDK가 동의 문자열을 자동으로 읽어갑니다. 이 경로는 SDK 설정 없이 동작합니다.

---

## 테스트 모드 / 테스트 디바이스

QA·심사 단계에서 테스트 광고를 받으려면 테스트 모드와 테스트 디바이스 광고 ID를 설정하세요.

#### Java
```java
// 전역 테스트 모드
AdMixer.setTestMode(true);

// 테스트 디바이스 광고 ID(GAID) 목록
AdMixer.setTestDeviceIds(Arrays.asList("AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE"));
```

#### Kotlin
```kotlin
AdMixer.setTestMode(true)
AdMixer.setTestDeviceIds(listOf("AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE"))
```

### 네트워크별 테스트 적용

| 네트워크 | 테스트 적용 |
|---|---|
| **Google AdManager** · **GMA NextGen** | `RequestConfiguration.setTestDeviceIds(testDeviceIds)` |
| **AppLovin** | 초기화 시 `setTestDeviceAdvertisingIds(testDeviceIds)` |
| **Unity Ads** | `UnityAds.initialize(..., testMode)` 인자에 반영 |
| **Pangle** | `PAGConfig.debugLog(testMode)` (테스트 디바이스는 Pangle 대시보드에서 GAID 등록) |
| **Naver Ad Manager** · **Kakao Adfit** · **Teads** | SDK 차원의 테스트 모드 연동이 없습니다. 각 네트워크 대시보드에서 테스트 지면을 설정하세요 |

> ⚠️ 테스트 모드 지원 방식은 네트워크 SDK 버전에 따라 변경될 수 있습니다. 테스트 디바이스 등록 절차는 각 네트워크 공식 문서를 참고하세요.

> ℹ️ 테스트 디바이스 광고 ID(GAID)는 LogCat에서 각 네트워크 SDK가 출력하는 안내 메시지로 확인하거나, 기기 설정 > Google > 광고에서 확인할 수 있습니다.

---

## 네트워크별 키 주입 (setAdapterConfig)

서버 설정(media-conf)에 특정 네트워크 키가 내려오지 않는 경우, 매체가 `AdInfo.Builder.setAdapterConfig()`로 직접 주입할 수 있습니다. 서버 값이 우선이며, **서버에 없는 키만** 클라이언트 값으로 채워집니다(Server-Precedence).

```java
// 예: AppLovin SDK Key를 직접 주입
Map<String, String> applovinConfig = new HashMap<>();
applovinConfig.put("sdkKey", "YOUR_APPLOVIN_SDK_KEY");

AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
    .setAdapterConfig(AdMixer.ADAPTER_APPLOVIN, applovinConfig)
    .build();
```

| 네트워크 | adapterName 상수 | 키 예시 |
|---|---|---|
| Pangle | `AdMixer.ADAPTER_PANGLE` (`"Pangle"`) | `app_id`, `placement_id` |
| AppLovin | `AdMixer.ADAPTER_APPLOVIN` (`"AppLovin"`) | `zone_id`, `sdkKey` |

> ℹ️ 일반적으로 네트워크 키는 nap mx 서버(media-conf)가 공급합니다. `setAdapterConfig`는 서버 미제공 상황의 보완 수단입니다.

---

## 문의

개인정보/테스트 설정 관련 문의는 **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)**로 연락해 주세요.
