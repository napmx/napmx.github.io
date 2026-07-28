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

### 네트워크별 전파

SDK에 설정한 값은 각 네트워크 SDK가 제공하는 API로 전달됩니다. **네트워크마다 받아들이는 항목이 다릅니다.**

| 네트워크 | 아동 대상(COPPA) | GDPR 동의 | CCPA(판매 거부) |
|---|---|---|---|
| **Google AdManager** · **GMA NextGen** | ✅ | — | — |
| **AppLovin** | — | ✅ | ✅ |
| **Unity Ads** | ✅ | ✅ | ✅ |
| **Pangle** | — | — | ✅ |
| **Teads** | — | — | ✅ (US Privacy 문자열) |
| **Naver Ad Manager** · **Kakao Adfit** | — | — | — |

- **—** 는 해당 네트워크 SDK에 대응 설정 API가 없거나 SDK가 전달하지 않는 항목입니다. 값을 설정해도 그 네트워크에는 반영되지 않습니다.
- 설정하지 않은 항목은 **어떤 네트워크에도 적용하지 않습니다**(미설정 상태 유지).

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
