# 온보딩 체크리스트 (네트워크별 필수 설정)

---

## 1. 공통 (필수)

- `AdMixer.getInstance().initialize(context, MEDIA_KEY, adUnits)` — `Application.onCreate`에서 1회.
- 미디어 키 / 광고 단위 ID는 nap mx 운영팀(**[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)**) 발급.
- 네트워크 SDK는 **어댑터가 워터폴에서 자동(lazy) 초기화** — `Application`에서 `PAGSdk.init`/`MobileAds.initialize` 등 직접 호출 불필요.

---

## 2. 네트워크별 필수 항목

| 네트워크 | 필수 키(mx 운영팀에서 네트워크 코드 매핑 진행) | 매니페스트 meta-data | 비고 |
|---|---|---|---|
| **Google AdManager** | `adunit_code` | **`com.google.android.gms.ads.APPLICATION_ID`** (호스트 앱 필수) | 미설정 시 **런치 크래시** |
| **Naver NAM** | `adunit_id` | `com.naver.gfpsdk.PUBLISHER_CD` — **SDK(admixer-naveradmanager)가 제공·고정. 호스트 설정 금지** |  |
| **Pangle** | `placement_id`(필수), `app_id` | 없음 | 키는 media-conf 또는 `setAdapterConfig` |
| **AppLovin** | `zone_id`(필수) | 없음 | SDK key는 `setAdapterConfig("AppLovin",{"sdkKey":...})` |
| **Unity Ads** | `game_id`, `placement_id` | 없음 | |
| **Teads / Adfit / Mobwith** | placement_id / data-ad-unit / zone_id | 없음 |  |

---

## 3. 테스트 광고 / 개인정보

- 테스트: `AdMixer.setTestMode(true)` + `AdMixer.setTestDeviceIds([...])`.
- 개인정보: `AdMixer.setGdprConsent/setCcpaDoNotSell/setTagForChildDirectedTreatment`. → [개인정보 동의 및 테스트 설정](privacy.md)

---

## 문의

**[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)**
