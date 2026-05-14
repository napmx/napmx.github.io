# 광고 연동 API

nap ssp 광고 API로써, 자체 광고 및 제휴 DSP(외부) 광고를 제공합니다.  
**애드믹서(NAP) 광고만 송출**되는 방식이며, Google 수익화를 포함하여 진행하는 경우 Script 연동으로 진행해주세요.

연동 및 이용 방법 문의: nap_adx@nasmedia.co.kr

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에 가입 후 미디어 등록 및 애드유닛 생성을 완료하면 연동에 필요한 `media_key`와 `adunit_id`를 확인할 수 있습니다.

---

## 1. 개요

HTTP GET 방식으로 정의된 객체를 작성해 nap ssp 인터페이스를 호출합니다.

> - 원활한 광고 응답 및 물량 확보를 위해 최대한 많은 객체를 포함하여 Request URL을 구성해주세요.
> - 구글 클라우드, AWS 등 해외 서버를 사용하여 호출하는 경우, IP 주소가 해외로 인식될 수 있으니 HTTP 헤더에 `x-forwarded-for`를 설정해주세요.
> - UA(User-agent) 값은 사용자 디바이스의 브라우저 정보를 최대한 많이 포함해야 더 많은 광고를 응답받을 수 있습니다.

---

## 2. Request

### 2-1. 도메인 정보

| 구분 | URL |
|------|-----|
| 상용 | `https://amssp.admixer.co.kr/api/v1/mad/dads` |

### 2-2. 기본 객체

| 필드 | 유형 | 필수 | 설명 | 값 |
|------|------|------|------|-----|
| `media_key` | string | ✔️ | 미디어 키 | 파트너 사이트에서 발급 |
| `adunit_id` | integer | ✔️ | 애드유닛 아이디 | 파트너 사이트에서 발급 |
| `platform` | string | — | 매체 플랫폼 | `android` / `ios` / `m.web` / `pc.web` / `etc` |
| `bundle` | string | — | 앱 패키지명 또는 웹사이트 URL | ex) `kr.co.nasmedia.app` / `admixer.co.kr` |
| `support_ctm` | integer | — | 클릭 측정 URL 사용 여부 (미기입 시 `0`) | `0`: no / `1`: yes |
| `coppa` | integer | — | 어린이 온라인 사생활 보호법 준수 여부 (미기입 시 `0`) | `0`: no / `1`: yes |

### 2-3. 디바이스 객체

| 필드 | 유형 | 설명 | 예시 |
|------|------|------|------|
| `os` | string | 디바이스 OS명 | `android`, `ios`, `windows` |
| `osv` | string | 디바이스 OS 버전 | `v1.0.0` |
| `lang` | string | 디바이스 언어 (ISO-639-1-alpha-2) | `ko`, `en` |
| `model` | string | 디바이스 모델 | `SM-S921`, `iPhone` |
| `carrier` | string | 디바이스 통신사 | `KT`, `SK`, `LG UPLUS` |
| `network` | string | 디바이스 네트워크 | `WIFI`, `4G`, `5G` |
| `ifa` | string | Android: Google Advertise ID / iOS: IDFA (**광고 타겟팅 핵심 값 — 미전달 시 물량에 큰 영향**) | `550e8400-e29b-41d4-a716-446655440000` |
| `ifa_use` | string | ADID 표준 추적 플래그 (미기입 시 `0`) | `0`: no / `1`: yes |

---

## 3. Response

### 3-1. 기본 객체

| 필드 | 필수 | 설명 |
|------|------|------|
| `success` | ✔️ | `true`: 광고 응답 성공 / `false`: 광고 응답 실패 |
| `error_code` | ✔️ (success: false 시) | 에러 코드 |
| `error_message` | ✔️ (success: false 시) | 에러 메세지 |
| `data` | ✔️ (success: true 시) | 애드유닛 정보와 광고 응답 |

### 3-2. data 객체

| 필드 | 필수 | 설명 |
|------|------|------|
| `inventory` | ✔️ | 파트너 사이트에 등록된 애드유닛 정보 |
| `ads` | ✔️ | 광고 응답 |

#### inventory 객체

| 필드 | 필수 | 설명 |
|------|------|------|
| `media_key` | ✔️ | 미디어 키 |
| `adunit_id` | ✔️ | 애드유닛 아이디 |
| `adformat` | ✔️ | 광고 형태 (`banner` / `native` / `video`) |
| `width` | ✔️ | 가로 사이즈 |
| `height` | ✔️ | 세로 사이즈 |
| `fullscreen` | ✔️ | 전면 여부 (`0`: no, `1`: yes) |

#### ads 객체

| 필드 | 필수 | 설명 |
|------|------|------|
| `adm` | ✔️ | 광고 응답 전문 (Banner: HTML, Native: native 1.2 규격, Video: VAST 3.0 규격) |

---

## 4. Macro 지원

| Macro | 지원 | 설명 |
|-------|------|------|
| `${CLICK_TRACK_URL}` | ✔️ | 클릭 측정 URL |

### ${CLICK_TRACK_URL} 사용 방법

1. Request 기본 객체 내 `support_ctm` 값을 `1`로 설정
2. Response의 `data.ads.adm` 내부에서 `${CLICK_TRACK_URL}` 매크로 검색
3. 매크로를 매체사의 클릭 측정 URL로 변경 (인코딩)
4. 교체된 클릭 측정 URL은 이미지 1×1 픽셀로 응답

---

## 5. 코드 정의

### 5-1. 에러 코드

| error_code | error_msg | 설명 |
|-----------|-----------|------|
| `E0001` | Bad request | 유효하지 않은 요청 |
| `E0002` | Unauthorized | 등록되지 않음 |
| `E0003` | Internal server error | 내부 서버 에러 |
| `E0004` | No Ads | 광고 없음 |
| `E1001` | Invalid params. | 파라미터 값이 유효하지 않음 |
| `E1002` | Invalid media key or missing! | media key가 유효하지 않거나 없음 |
| `E1004` | Invalid adunit id or missing! | adunit id가 유효하지 않거나 없음 |
| `E2101` | Error during internal processing | 내부 처리 과정의 에러 |
| `E2202` | Unregistered media. | 등록되지 않은 미디어 |
| `E2203` | Media is not active. | 미디어가 활성화되어 있지 않음 |
| `E2301` | Unregistered adunit. | 등록되지 않은 애드유닛 |
| `E2302` | No adunits are available. | 사용 가능한 애드유닛이 없음 |
| `E2401` | Platform is null or not supported. | 플랫폼 값이 없거나 지원되지 않음 |
| `E9001` | Unknown error. | 알 수 없음 |

### 5-2. 코드 정의

**adformat**

| 값 | 설명 |
|----|------|
| `banner` | 배너 |
| `native` | 네이티브 |
| `video` | 비디오 |

**platform**

| 값 | 설명 |
|----|------|
| `android` | Android 플랫폼 |
| `ios` | iOS 플랫폼 |
| `m.web` | 모바일 웹 플랫폼 |
| `pc.web` | PC 웹 플랫폼 |
| `etc` | 기타 플랫폼 |

### 5-3. Native asset id

| asset id | 설명 |
|----------|------|
| `100` | title (제목) |
| `101` | advertiser (광고주명) |
| `102` | cta |
| `103` | description (설명) |
| `200` | img (메인이미지) |
| `201` | icon (아이콘이미지) |
| `202` | video |

### 5-4. Native tracker

| 값 | 설명 |
|----|------|
| `clicktracker` | 광고 클릭 집계를 위한 트래킹 URL. 다수의 URL이 포함될 수 있으며, 광고 클릭 시 해당 URL을 호출해야 합니다. |
| `eventtracker` | 광고 노출 집계를 위한 트래킹 URL. 다수의 URL이 포함될 수 있으며, 광고 이미지 노출(show) 시 해당 URL을 호출해야 합니다. |

---

## 6. Reward Callback (선택사항)

유저가 리워드 동영상 광고 시청을 완료(complete 이벤트 발생)하면, 매체사가 정의한 외부 서버로 시청 완료 여부를 전달하는 기능입니다.  
광고 시청 완료 후 바로 콜백을 수행하지만, 몇 분 정도 지연될 수 있습니다.

파트너 사이트의 리워드 광고 애드유닛 상세 설정에서 사용할 수 있습니다.

### 6-1. 콜백 서버 URL 입력

**파트너 사이트 → 미디어 관리 → 애드유닛 광고 설정**에서 콜백 서버 URL을 입력합니다.

기본 파라미터 (자동 포함):

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| `media_key` | 미디어 키 | `12345678` |
| `adunit_id` | 애드유닛 아이디 | `87654321` |
| `adid` | Android: Google Advertise ID / iOS: IDFA | `860635ea-65bc-eaed-d355-1b5283b30b94` |
| `complete` | 리워드 동영상 광고 시청 완료 이벤트 | — |
| `timestamp` | complete 이벤트가 발생한 시간 | `1546300800` |

### 6-2. CustomData 추가 (선택사항)

CustomData를 통해 콜백에서 추가 데이터를 수집할 수 있습니다. String Map 형태로 추가해야 합니다.

```c
params = new HashMap<>();
params.put("cpl_uid", user_id);
params.put("cpl_nmv", value);
```
