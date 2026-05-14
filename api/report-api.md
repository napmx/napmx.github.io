# 리포트 API

nap ssp 퍼블리셔를 위한 리포트 데이터 API입니다.  
연동 및 이용 방법 문의: nap_adx@nasmedia.co.kr

---

## 사전 준비

API Key는 **파트너 사이트 → 계정 → 계정관리 → '퍼블리셔 리포트 API KEY'** 에서 확인할 수 있습니다.

---

## 1. 개요

HTTP GET 방식으로 요청하며, `apiKey` 값은 **header**에 넣어야 하고 나머지는 **queryString**으로 요청합니다.

**리포트 기간 조회 조건:**
- 최근 6개월 이내
- 한 번에 최대 30일

---

## 2. 도메인 정보

| 구분 | URL |
|------|-----|
| 상용 | `https://publisher.admixer.co.kr/api/v1/report/daily` |

---

## 3. 요청 파라미터

| 항목 | 구분 | 필수 | 설명 | 값 |
|------|------|------|------|-----|
| `apiKey` | String (header) | ✔️ | 파트너 사이트에서 발급받은 API Key | — |
| `type` | String (query) | ✔️ | 리포트 유형 | `total`: 일자별 / `adunit`: 애드유닛별 |
| `startDate` | String (query) | ✔️ | 조회 시작 날짜 | `YYYY-MM-DD` |
| `endDate` | String (query) | ✔️ | 조회 마지막 날짜 (startDate와 같거나 이후) | `YYYY-MM-DD` |

### 요청 예시

```
GET https://publisher.admixer.co.kr/api/v1/report/daily
    ?type=total&startDate=2024-11-01&endDate=2024-11-30

Header: apiKey: {발급받은_API_KEY}
```

---

## 4. 응답

### 4-1. 기본 객체

| 항목 | 설명 |
|------|------|
| `status` | 결과 코드 |
| `code` | 코드 |
| `message` | 결과 메세지 |
| `data` | 리포트 응답 데이터 |

### 4-2. data 객체

| 항목 | 설명 | 비고 |
|------|------|------|
| `ymd` | 날짜 | `type=total`인 경우만 노출 |
| `adunitId` | 애드유닛 ID | `type=adunit`인 경우만 노출 |
| `adunitName` | 애드유닛 이름 | `type=adunit`인 경우만 노출 |
| `req` | 요청수 | — |
| `calcImp` | 노출수 | — |
| `calcClick` | 클릭수 | — |
| `ctr` | CTR | — |
| `ecpmNet` | eCPM (원화) | — |
| `ecpmNetUsd` | eCPM (달러) | — |
| `fillrate` | Fillrate | — |
| `cost` | 수익금 (원화) | — |
| `costUsd` | 수익금 (달러) | — |

---

## 5. 결과 코드

### 5-1. 기본 코드

| status | 설명 |
|--------|------|
| `200` | 성공 |
| `400` | 잘못된 요청 |
| `403` | 권한 없음 |
| `404` | 리포트를 찾을 수 없음 |
| `500` | 서버 내부 오류 |

### 5-2. 400 응답 상세 메세지

| 응답 메세지 | 설명 |
|-----------|------|
| `유효하지 않은 API KEY 입니다.` | header의 apiKey 검증 실패 |
| `날짜 형식이 잘못되었습니다. yyyy-MM-dd 형식에 맞게 다시 요청해주세요.` | startDate, endDate 날짜 형식 오류 (예: `2024-10-10`(o), `20241010`(x)) |
| `endDate는 startDate와 같거나, 그 이후여야 합니다. 다시 요청해주세요.` | endDate가 startDate보다 이전인 경우 |
| `최근 180일 이내의 데이터만 조회할 수 있습니다. 다시 요청해주세요.` | 조회 일자가 최근 6개월보다 이전인 경우 |
| `한 번에 최대 30일치 데이터만 조회 가능합니다. 다시 요청해주세요.` | 조회 기간이 30일보다 긴 경우 |
| `잘못된 요청입니다. 다시 확인해주세요.` | type이 total 또는 adunit이 아닌 경우 |
