# 광고 연동 API

NAP MX 광고 API로, 자체 광고 및 제휴 DSP(외부) 광고를 제공합니다.
**애드믹서(NAP) 광고만 송출**되는 방식이며, Google 수익화를 포함하여 진행하는 경우 Script 연동으로 진행해주세요.

연동 및 이용 방법 문의: nap_mx@nasmedia.co.kr

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에 가입 후 미디어 등록 및 애드유닛 생성을 완료하면 연동에 필요한 `media_key`와 `adunit_id`를 확인할 수 있습니다.

---

## 1. 개요

HTTP GET 방식으로 정의된 객체를 작성해 NAP MX 인터페이스를 호출합니다.

> - 원활한 광고 응답 및 물량 확보를 위해 최대한 많은 객체를 포함하여 Request URL을 구성해주세요.
> - 구글 클라우드, AWS 등 해외 서버에서 호출하는 경우, IP가 해외로 인식될 수 있으니 HTTP 헤더에 `x-forwarded-for`를 설정해주세요.
> - UA(User-agent) 값은 사용자 디바이스의 브라우저 정보를 최대한 많이 포함해야 더 많은 광고를 응답받을 수 있습니다.
> - 모든 문자열 파라미터는 서버에서 UTF-8 URL 디코딩 후 양쪽 공백이 제거됩니다. 값에 특수문자가 포함될 경우 반드시 URL 인코딩하여 전달해주세요.

---

## 2. Request

### 2-1. 도메인 정보

| 구분 | URL |
|------|-----|
| 상용 | `https://amssp.admixer.co.kr/api/v1/mad/dads` |

### 2-2. 기본 객체

| 필드 | 유형 | 필수 | 설명 | 값 / 예시 |
|------|------|:---:|------|----------|
| `media_key` | string | ✔️ | 미디어 키 | 파트너 사이트에서 발급 |
| `adunit_id` | integer | ✔️ | 애드유닛 아이디 | 파트너 사이트에서 발급 |
| `platform` | string | — | 매체 플랫폼 | `android` / `ios` / `m.web` / `pc.web` / `etc` |
| `bundle` | string | — | 앱 패키지명 또는 웹사이트 URL | `com.domain.service`, `admixer.co.kr` |
| `support_ctm` | integer | — | 클릭 측정 URL 사용 여부 (미기입 시 `0`) | `0`: no / `1`: yes |
| `coppa` | integer | — | 어린이 온라인 사생활 보호법 준수 여부 (미기입 시 `0`) | `0`: no / `1`: yes |
| `callback` | string | — | JSONP 콜백 함수명. 지정 시 응답이 `callback({...});` 형태로 래핑되어 반환됩니다 | `myCallback` |
| `ntvad_sv` | string | — | OpenRTB Native 광고 스펙 버전 (Native 요청에만 유효, 기본값 `1.2`) | `1.2` |

### 2-3. 디바이스 객체

| 필드 | 유형 | 설명 | 예시 |
|------|------|------|------|
| `os` | string | 디바이스 OS명 | `android`, `ios`, `windows`, `mac`, `etc` |
| `osv` | string | 디바이스 OS 버전 | `v1.0.0` |
| `sdkv` | string | SDK 버전 | `v1.0.0` |
| `lang` | string | 디바이스 언어 (ISO-639-1-alpha-2) | `ko`, `en` |
| `model` | string | 디바이스 모델 | `SM-S921`, `iPhone` |
| `carrier` | string | 디바이스 통신사 | `KT`, `SK`, `LG UPLUS`, `Vodafone`, `AT&T`, `Verizon` |
| `network` | string | 디바이스 네트워크 | `WIFI`, `4G`, `5G` |
| `ifa` | string | Android: GAID / iOS: IDFA. **광고 타겟팅 핵심 값 — 미전달 시 물량에 큰 영향** | `550e8400-e29b-41d4-a716-446655440000` |
| `ifa_use` | integer | ADID 표준 추적 플래그 (미기입 시 `0`) | `0`: no / `1`: yes |

### 2-4. 요청 예시

```
GET /api/v1/mad/dads
    ?media_key=MEDIA_KEY
    &adunit_id=1,2,3
    &platform=android
    &os=android&osv=14
    &sdkv=v1.0.0
    &lang=ko
    &model=SM-S921
    &carrier=KT
    &network=WIFI
    &ifa=550e8400-e29b-41d4-a716-446655440000
    &ifa_use=1
    &bundle=com.domain.service
    &support_ctm=1
    &coppa=0
    &ntvad_sv=1.2
HTTP/1.1
Host: amssp.admixer.co.kr
User-Agent: Mozilla/5.0 ...
X-Forwarded-For: 203.0.113.10
```

---

## 3. Response

응답 본문은 JSON 형식이며, `success` 값에 따라 노출되는 필드가 달라집니다.

### 3-1. 기본 객체

| 필드 | 노출 조건 | 설명 |
|------|----------|------|
| `success` | 항상 | `true`: 광고 응답 성공 / `false`: 광고 응답 실패 |
| `error_code` | `success: false` | 에러 코드 ([5-1 에러 코드](#_5-1-에러-코드) 참조) |
| `error_message` | `success: false` 이며 메시지가 있을 때 | 에러 메시지 |
| `data` | `success: true` 이며 데이터가 있을 때 | 애드유닛 정보와 광고 응답 |

### 3-2. `data` 객체

| 필드 | 노출 조건 | 설명 |
|------|----------|------|
| `inventory` | 값이 있을 때 | 파트너 사이트에 등록된 애드유닛 정보 |
| `ads` | 값이 있을 때 | 광고 응답 |

#### `inventory` 객체

| 필드 | 유형 | 설명 |
|------|------|------|
| `media_key` | string | 미디어 키 |
| `adunit_id` | integer | 응답 시 선택된 애드유닛 아이디 |
| `adformat` | string | 광고 형태 (`banner` / `native` / `video`) |
| `width` | integer | 가로 사이즈 (banner / native) |
| `height` | integer | 세로 사이즈 (banner / native) |
| `fullscreen` | integer | 전면 여부 (`0`: no, `1`: yes) |

#### `ads` 객체

각 필드는 값이 있을 때에만 응답에 포함됩니다.

| 필드 | 유형 | 설명 |
|------|------|------|
| `dsp_id` | integer | 응답을 제공한 DSP 식별자 |
| `dsp_name` | string | DSP 이름 |
| `adm` | string | 광고 응답 전문 (Banner: HTML / Native: OpenRTB Native 1.2 규격 / Video: VAST 3.0 규격) |
| `banner_track_url` | string | 리워드 배너 트래킹 URL (리워드 광고 응답 시) |

### 3-3. 응답 예시

**성공 (배너)**

```json
{
  "success": true,
  "data": {
    "inventory": {
      "media_key": "MEDIA_KEY",
      "adunit_id": 1,
      "adformat": "banner",
      "width": 320,
      "height": 50,
      "fullscreen": 0
    },
    "ads": {
      "dsp_id": 10,
      "dsp_name": "NAP",
      "adm": "<html>...광고 HTML...</html>"
    }
  }
}
```

**실패**

```json
{
  "success": false,
  "error_code": "E0004",
  "error_message": "No Ads"
}
```

**JSONP (`callback=myCallback`)**

```
myCallback({"success":true,"data":{...}});
```

---

## 4. Macro 지원

| Macro | 지원 | 설명 |
|-------|:----:|------|
| `${CLICK_TRACK_URL}` | ✔️ | 클릭 측정 URL |

### `${CLICK_TRACK_URL}` 사용 방법

1. Request 기본 객체 내 `support_ctm` 값을 `1`로 설정
2. Response의 `data.ads.adm` 내부에서 `${CLICK_TRACK_URL}` 매크로를 검색
3. 매크로를 매체사의 클릭 측정 URL로 교체 (URL 인코딩 필요)
4. 교체된 클릭 측정 URL은 1×1 투명 PNG 픽셀로 응답됩니다.

---

## 5. 코드 정의

### 5-1. 에러 코드

모든 코드는 HTTP `200 OK` 와 함께 `error_code` / `error_message` 본문으로 전달됩니다.

| error_code | error_msg | 설명 |
|-----------|-----------|------|
| `E0001` | Bad Request | 유효하지 않은 요청 |
| `E0002` | Unauthorized | 인증 실패 |
| `E0003` | Internal Server Error | 내부 서버 에러 |
| `E0004` | No Ads | 광고 없음 |
| `E1001` | Invalid params. | 파라미터 유효성 실패 |
| `E1002` | Invalid media key or missing! | `media_key` 누락/오류 |
| `E1003` | Invalid media id or missing! | 미디어 식별 실패 |
| `E1004` | Invalid adunit id or missing! | `adunit_id` 누락/오류 |
| `E1013` | Invalid ifa or missing! | IFA 누락/오류 |
| `E2101` | Error during internal processing. | 내부 처리 에러 |
| `E2202` | Unregistered media. | 미등록 미디어 |
| `E2203` | Media is not active. | 미디어 비활성 |
| `E2301` | Unregistered adunit. | 미등록 애드유닛 |
| `E2302` | No adunits are available. | 사용 가능한 애드유닛 없음 |
| `E2303` | No network information is set up. | 애드유닛 네트워크 미설정 |
| `E2304` | Invalid network settings. | 잘못된 네트워크 설정 |
| `E2305` | No suitable ads are found. | 적합 광고 없음 |
| `E2401` | Platform is null or not supported. | 플랫폼 누락/비지원 |
| `E2501` | No adunit network settings are available. | 애드유닛 네트워크 설정 없음 |
| `E2502` | Invalid network. | 잘못된 네트워크 |
| `E2503` | No adunit network group settings are available. | 네트워크 그룹 설정 없음 |
| `E2504` | No adunit dsp settings are available. | DSP 설정 없음 |
| `E2505` | Invalid dsp. | 잘못된 DSP |
| `E2506` | No adunit dsp group settings are available. | DSP 그룹 설정 없음 |
| `E2601` | Unregistered network. | 미등록 네트워크 |
| `E2701` | Exchange rate information not found. / Invalid exchange rate. | 환율 정보 오류 |
| `E2801` | Blocked IP. | 차단 IP |
| `E2802` | Blocked IFA. | 차단 IFA |
| `E2901` | Unregistered dsp. | 미등록 DSP |
| `E9001` | Unknown error. | 알 수 없음 |

### 5-2. 코드 정의

**`adformat`**

| 값 | 설명 |
|----|------|
| `banner` | 배너 |
| `native` | 네이티브 |
| `video` | 비디오 |

**`platform`**

| 값 | 설명 |
|----|------|
| `android` | Android 플랫폼 |
| `ios` | iOS 플랫폼 |
| `m.web` | 모바일 웹 플랫폼 |
| `pc.web` | PC 웹 플랫폼 |
| `etc` | 기타 플랫폼 |

### 5-3. Native asset id

Native 광고 응답(`adm`)은 OpenRTB Native 1.2 규격을 따릅니다.

| asset id | 설명 |
|----------|------|
| `100` | title (제목) |
| `101` | advertiser / sponsored (광고주명, OpenRTB `data.type=1`) |
| `102` | cta (Call to Action, OpenRTB `data.type=12`) |
| `103` | description (설명) |
| `200` | img (메인 이미지) |
| `201` | icon (아이콘 이미지) |
| `202` | video (VAST tag) |

### 5-4. Native tracker

| 값 | 설명 |
|----|------|
| `clicktracker` | 광고 클릭 집계용 트래킹 URL. 다수의 URL이 포함될 수 있으며, 광고 클릭 시 호출해야 합니다. |
| `eventtracker` | 광고 노출 집계용 트래킹 URL. 다수의 URL이 포함될 수 있으며, 광고 이미지 노출(show) 시 호출해야 합니다. |

---

## 6. Reward Callback (선택사항)

유저가 리워드 동영상 광고 시청을 완료(complete 이벤트 발생)하면, 매체사가 정의한 외부 서버로 시청 완료 여부를 전달하는 기능입니다. 광고 시청 완료 직후 콜백을 수행하지만 몇 분 정도 지연될 수 있습니다.

파트너 사이트의 리워드 광고 애드유닛 상세 설정에서 사용할 수 있습니다.

### 6-1. 콜백 서버 URL 입력

**파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정**에서 콜백 서버 URL을 입력합니다.

기본 파라미터 (자동 포함):

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| `media_key` | 미디어 키 | `12345678` |
| `adunit_id` | 애드유닛 아이디 | `87654321` |
| `adid` | Android: GAID / iOS: IDFA | `860635ea-65bc-eaed-d355-1b5283b30b94` |
| `complete` | 리워드 동영상 광고 시청 완료 이벤트 | — |
| `timestamp` | complete 이벤트가 발생한 시간 (epoch seconds) | `1546300800` |

### 6-2. CustomData 추가 (선택사항)

CustomData를 통해 콜백에서 추가 데이터를 수집할 수 있습니다. String Map 형태로 추가해야 합니다.

```java
Map<String, String> params = new HashMap<>();
params.put("cpl_uid", user_id);
params.put("cpl_nmv", value);
```
