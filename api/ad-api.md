# 광고 연동 API

nap SSP 광고 연동 API는 **애드믹서 광고(NAP 네트워크)만 송출**하는 API 방식입니다.  
Google 등 외부 네트워크를 함께 사용하려면 SDK 또는 Script 연동을 권장합니다.

문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 요청 방식

- **Method**: `HTTP POST`
- **Content-Type**: `application/json`
- **인증**: HTTP Header에 `apiKey` 포함

---

## 광고 요청 엔드포인트

```
POST https://amssp.admixer.co.kr/api/v1/ad
```

### 요청 헤더

| 헤더 | 설명 |
|------|------|
| `Content-Type` | `application/json` |
| `apiKey` | 파트너 사이트에서 발급받은 API Key |

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `mediaKey` | String | ✅ | 미디어 고유 키 |
| `adUnitId` | String | ✅ | 광고 단위 ID |
| `platform` | String | ✅ | `android` / `ios` / `mweb` / `pcweb` |
| `adFormat` | String | ✅ | `banner` / `native` / `video` / `interstitial` |
| `width` | Int | — | 배너 너비 |
| `height` | Int | — | 배너 높이 |
| `deviceId` | String | — | 광고 ID (GAID / IDFA) |
| `ip` | String | — | 클라이언트 IP |
| `userAgent` | String | — | 클라이언트 User-Agent |

### 요청 예시

```json
{
  "mediaKey": "발급받은_MEDIA_KEY",
  "adUnitId": "발급받은_ADUNIT_ID",
  "platform": "android",
  "adFormat": "banner",
  "width": 320,
  "height": 50,
  "deviceId": "aabbccdd-1234-...",
  "ip": "1.2.3.4",
  "userAgent": "Mozilla/5.0 ..."
}
```

---

## 응답 형식

```json
{
  "status": "ok",
  "code": 200,
  "message": "success",
  "data": {
    "adType": "banner",
    "creative": {
      "title": "광고 제목",
      "description": "광고 설명",
      "imageUrl": "https://cdn.example.com/image.jpg",
      "landingUrl": "https://advertiser.example.com",
      "html": "<div>...</div>"
    },
    "trackingUrls": {
      "impression": ["https://track.admixer.co.kr/imp?..."],
      "click": ["https://track.admixer.co.kr/clk?..."]
    }
  }
}
```

---

## 응답 코드

| code | 설명 |
|------|------|
| `200` | 성공 |
| `204` | 광고 없음 (No Fill) |
| `400` | 잘못된 요청 파라미터 |
| `401` | 인증 실패 (apiKey 오류) |
| `404` | 미디어/Adunit 없음 |
| `500` | 서버 오류 |

---

## 노출/클릭 트래킹

광고 수신 후 반드시 노출 및 클릭 트래킹 URL을 호출해야 합니다.

```
// 노출 시 호출
GET {data.trackingUrls.impression[0]}

// 클릭 시 호출
GET {data.trackingUrls.click[0]}
```
