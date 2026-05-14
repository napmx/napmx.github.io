# 리포트 API

nap mx 퍼블리셔를 위한 수익 리포트 API입니다.

> 버전: v1.0.0 | 릴리즈: 2024-12-18

문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 요청 방식

- **Method**: `HTTP GET`
- **인증**: HTTP Header에 `apiKey` 포함
- **파라미터**: 쿼리스트링

---

## 엔드포인트

```
GET https://publisher.admixer.co.kr/api/v1/report/daily
```

---

## 요청 헤더

| 헤더 | 설명 |
|------|------|
| `apiKey` | 파트너 사이트에서 발급받은 API Key |

---

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `type` | String | ✅ | `total` / `adunit` |
| `startDate` | String | ✅ | 조회 시작일 (`YYYY-MM-DD`) |
| `endDate` | String | ✅ | 조회 종료일 (`YYYY-MM-DD`) |

> - 조회 기간: 최근 **6개월 이내**, 최대 **30일** 범위  
> - `type=total`: 전체 합산 리포트  
> - `type=adunit`: Adunit 별 리포트

### 요청 예시

```
GET https://publisher.admixer.co.kr/api/v1/report/daily
    ?type=total&startDate=2024-11-01&endDate=2024-11-30
```

---

## 응답 형식

### type=total

```json
{
  "status": "ok",
  "code": 200,
  "message": "success",
  "data": {
    "type": "total",
    "list": [
      {
        "date": "2024-11-01",
        "impression": 12345,
        "click": 234,
        "revenue": 15000.00,
        "cpm": 1215.43,
        "ctr": 1.89
      }
    ]
  }
}
```

### type=adunit

```json
{
  "status": "ok",
  "code": 200,
  "message": "success",
  "data": {
    "type": "adunit",
    "list": [
      {
        "date": "2024-11-01",
        "adUnitId": "ADUNIT_001",
        "adUnitName": "메인_배너",
        "impression": 5000,
        "click": 100,
        "revenue": 6000.00,
        "cpm": 1200.00,
        "ctr": 2.00
      }
    ]
  }
}
```

---

## 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `date` | String | 날짜 (`YYYY-MM-DD`) |
| `impression` | Int | 광고 노출 수 |
| `click` | Int | 클릭 수 |
| `revenue` | Float | 수익금 (원) |
| `cpm` | Float | 1000회 노출당 수익 |
| `ctr` | Float | 클릭률 (%) |

---

## 응답 코드

| code | 설명 |
|------|------|
| `200` | 성공 |
| `400` | 잘못된 파라미터 (날짜 형식, 범위 초과 등) |
| `401` | 인증 실패 |
| `500` | 서버 오류 |

---

## API Key 발급

파트너 사이트 → 계정 → API Key 발급 메뉴에서 발급받을 수 있습니다.  
[파트너 사이트 바로가기](https://publisher.admixer.co.kr/)
