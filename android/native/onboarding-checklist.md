# 온보딩 체크리스트 (네트워크별 필수 설정)

연동 시작 전, 네트워크별 **필수 키 / 매니페스트 메타데이터 / 운영 Ad Unit 발급 경로**를 1페이지로 정리했습니다. `[SKIP] ... (Missing Keys)`/`no-fill` 원인 파악에 활용하세요.

---

## 1. 공통 (필수)

- `AdMixer.getInstance().initialize(context, MEDIA_KEY, adUnits)` — `Application.onCreate`에서 1회.
- 미디어 키 / 광고 단위 ID는 nap mx 운영팀(**nap_mx@nasmedia.co.kr**) 발급.
- 네트워크 SDK는 **어댑터가 워터폴에서 자동(lazy) 초기화** — `Application`에서 `PAGSdk.init`/`MobileAds.initialize` 등 직접 호출 불필요.

---

## 2. 네트워크별 필수 항목

| 네트워크 | 필수 키(서버 media-conf) | 매니페스트 meta-data | 비고 |
|---|---|---|---|
| **Google AdManager** | `adunit_code` | **`com.google.android.gms.ads.APPLICATION_ID`** (호스트 앱 필수) | 미설정 시 **런치 크래시** |
| **Naver NAM** | `adunit_id` | `com.naver.gfpsdk.PUBLISHER_CD` — **SDK(admixer-naveradmanager)가 제공·고정. 호스트 설정 금지** | 운영 Ad Unit ID 발급 필요 |
| **Pangle** | `placement_id`(필수), `app_id` | 없음 | 키는 media-conf 또는 `setAdapterConfig` |
| **AppLovin** | `zone_id`(필수) | 없음 | SDK key는 `setAdapterConfig("AppLovin",{"sdkKey":...})` |
| **Unity Ads** | `game_id`, `placement_id` | 없음 | |
| **Teads / Adfit / Mobwith** | placement_id / data-ad-unit / zone_id | 없음 | 각 광고단위 식별자 |

---

## 3. 키 미제공(`[SKIP] Missing Keys`) 대응

서버(media-conf)에 네트워크 필수 키(예: Pangle `placement_id`, AppLovin `zone_id`)가 없으면 워터폴에서 조용히 SKIP됩니다.

1. (권장) 운영팀에 **운영 유닛 media-conf 키 프로비저닝** 요청.
2. (보완) 매체가 직접 주입 — 서버 우선 병합(서버에 없는 키만 채움):
   ```java
   Map<String,String> pangle = new HashMap<>();
   pangle.put("placement_id", "..."); // 필수
   pangle.put("app_id", "...");
   AdInfo adInfo = new AdInfo.Builder(adUnitId)
       .setAdapterConfig(AdMixer.ADAPTER_PANGLE, pangle)
       .build();
   ```

---

## 4. Naver no-fill (`GFP_SERVER_ERROR`) 구분

- `PUBLISHER_CD`는 **SDK가 공급**하므로 호스트가 신경쓸 필요 없습니다(매니페스트 설정 금지).
- `test_aos_*` 같은 **테스트 Ad Unit**은 `GFP_SERVER_ERROR/NO_FILL`이 정상입니다 → **운영 Ad Unit ID 발급/전환**이 필요합니다(운영팀).

---

## 5. 테스트 광고 / 개인정보

- 테스트: `AdMixer.setTestMode(true)` + `AdMixer.setTestDeviceIds([...])`.
- 개인정보: `AdMixer.setGdprConsent/setCcpaDoNotSell/setTagForChildDirectedTreatment`. → [개인정보 동의 및 테스트 설정](privacy.md)

---

## 문의

**nap_mx@nasmedia.co.kr**
