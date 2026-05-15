---
name: guide-check
description: 외부 가이드 페이지(주로 Confluence)와 저장소 문서의 코드/네이밍 정합성을 점검하고 정정한다. 사용자가 `/guide-check` 슬래시 커맨드로 명시적으로 호출했을 때만 발동한다.
---

# guide-check

외부 공식 가이드(Confluence/Notion/GitHub 등)와 저장소 가이드 문서의 **코드 샘플과 API 네이밍**이 어긋났을 때, 외부 가이드를 출처(source of truth)로 삼아 저장소를 정렬한다. 안내·설명 문구는 건드리지 않는다.

`/guide-check` 슬래시 커맨드로만 발동한다. 키워드 자동 감지는 하지 않는다.

---

## 핵심 원칙 (이번 napmx 작업에서 합의된 기본값)

매 실행 시 이 원칙이 적용된다. 사용자가 시작 시점에 명시적으로 재조정을 요청하면 변경한다.

1. **코드만 수정한다.** 코드 블록과 본문 안의 인라인 코드 참조(`` `ClassName` ``, `` `method()` `` 등)만 수정한다. 안내 문구·설명·표는 유지한다.
2. **명백한 외부 가이드 오타는 정정해서 적용한다.** 클래스명 t 누락(`AMMInterstital` → `AMMInterstitial`), 함수명 호출-정의 불일치(`addViewToView` vs `addBannerViewToView`), Delegate 설명 텍스트와 코드의 메서드명 충돌, 변수명 중복 선언(`let config` 두 번) 등 컴파일·논리 오류는 정정한다. 의도 모호 시 사용자에게 확인.
3. **import 통일.** 외부 가이드가 명시한 모듈명을 따른다. 저장소 내 다른 파일이 이미 변경된 형태로 통일됐다면 일관성을 우선.
4. **단계적 진행.** 여러 파일을 점검할 때는 한 번에 하나씩 진행한다. 각 파일 작업 종료 시 사용자에게 다음 파일로 진행해도 될지 확인한다.
5. **저장소에만 있는 설명용 표·인용구는 유지**, **장식성 더미 샘플 코드는 최소화.** 외부 가이드에 없는 헬퍼 함수(`grantReward()` 같은 빈 더미)는 제거.
6. **외부 가이드 자체가 모호하면 사전에 묻는다.** 본문 텍스트와 코드가 충돌하는 경우(`loadAd()` 본문 vs `load()` 코드), 외부 가이드에만 있는 큰 섹션을 추가할지 등은 작업 시작 전에 정리한다.

---

## 워크플로우

### 1단계: 인자 파싱

사용자가 `/guide-check <url> [<repo-path>...]` 형태로 호출한다.
- URL은 Confluence/Notion/GitHub Wiki 등 외부 가이드 페이지 주소
- repo-path는 비교 대상 저장소 파일. 생략 시 사용자에게 묻는다.

여러 (url, path) 쌍을 처리할 때는 한 번에 모두 받아두고 1차 비교 후 단계적으로 적용한다.

### 2단계: 외부 가이드 + 저장소 파일 병렬 수집

다음을 단일 메시지에서 병렬로 호출한다.
- Confluence: `mcp__atlassian__getConfluencePage` (contentFormat=markdown)
  - 인증이 안 됐다면 `mcp__atlassian__authenticate` 안내 후 사용자 인증 완료 대기
- Notion/GitHub 등 다른 외부 가이드: `WebFetch` (인증 필요 시 사용자에게 자격 안내)
- 저장소 파일: `Read`
- 최근 커밋 컨벤션 확인이 필요하면 `git log --oneline -10`

### 3단계: 차이점 표 작성 + 의문점 식별

각 파일별로 표 형식 비교를 사용자에게 제시한다.

```
| 항목 | 현재 (repo) | 변경 후 (외부 가이드 기준) |
```

식별 대상:
- import 문, 모듈명
- 클래스명·인스턴스 변수 타입
- 초기화 시그니처(`init(frame:)` vs `init(rootViewController:)` 등)
- 프로퍼티명(`adUnitId` vs `adUnitID`, 대소문자)
- 메서드명(`load()` vs `loadAd()`, `show(from:)` vs `show(rootViewController:)`)
- Delegate 시그니처(파라미터 유무)
- 리소스 해제 패턴(`deinit` vs `viewDidDisappear` + 가드)
- 외부 가이드의 명백한 오타·논리 오류
- 외부 가이드와 저장소의 본문 vs 코드 충돌

### 4단계: 의문점 사전 질문

확인이 필요한 항목은 작업 시작 **전에** AskUserQuestion으로 일괄 정리한다. 매 파일마다 묻지 않고 첫 파일 시작 전 또는 새로운 파일 진입 시 묻는다. 묻기 좋은 질문 카테고리:

- **외부 가이드 오타 처리**: 정정 vs 1:1 미러링
- **import 모듈명**: 저장소 통일 / 외부 가이드 따라 변경 / 제거
- **저장소 단독 내용**: 설명용 표·인용구는 유지, 장식성 코드는 최소화 (기본값)
- **본문 vs 코드 충돌**: 어느 쥐을 기준으로
- **외부 가이드에만 있는 큰 섹션**: 추가 여부
- **클래스/변수명**: 가이드 그대로 vs 저장소 명명 컨벤션 유지

### 5단계: 핀포인트 Edit

코드 블록과 본문 인라인 참조를 `Edit` 도구로 하나씩 정확히 교체한다. `Write` 전체 재작성은 본문/표까지 손대게 되므로 금지.

순서:
- 코드 블록 우선 → 본문 내 인라인 코드 참조 마지막
- 같은 파일 안에서 같은 패턴이 반복되면 `replace_all`이 아닌 unique context로 개별 교체 (의도하지 않은 곳까지 바뀌면 위험)

### 6단계: git diff 검증

각 파일 작업 후 `git diff <file>`로 변경 범위를 사용자에게 보여준다. 코드 블록만 바뀌고 설명 문구가 보존됐는지 1차 확인.

### 7단계: 단계 종료 + 사용자 확인

각 파일 작업 종료 시 변경 요약 표를 보여주고, 다음 파일 진행 가능 여부를 사용자에게 확인한다.

```
이 단계 검토 후 Step N (<다음 파일>)으로 진행해도 될지 알려주세요.
```

전체 작업 종료 후에는 전체 통계(`git diff --stat`)와 함께 마무리 보고. 커밋 여부는 별도 지시 시.

---

## 외부 가이드 명백한 오타·논리 오류 정정 패턴

자주 마주치는 패턴 예시. 의도가 명백하면 정정해서 적용한다.

| 외부 가이드 원본 | 정정 후 | 이유 |
|---|---|---|
| `AMMInterstital` | `AMMInterstitial` | t 누락 타이핑 미스 |
| `interstital?.stop()` | `interstitial?.stop()` | 동일 |
| `AMMVideoView(rootViewController:)` (실제 타입은 `AMMVideoAdView`) | `AMMVideoAdView(rootViewController:)` | 타입 불일치 |
| `addViewToView` 호출, `addBannerViewToView` 정의 | 호출을 정의명에 맞춤 | 컴파일 오류 |
| 본문 "loadAd()를 호출", 코드 `load()` | 사용자 확인 필요 — 보통 코드를 진실로 채택 | 본문 vs 코드 충돌 |
| Delegate 설명 `onSuccessBanner`, 코드 `onSuccessVideo` | 코드 기준 | 본문 vs 코드 충돌 |
| 같은 함수 안 `let config` 두 번 선언 | 변수명 분리 (예: `pangleConfig`, `config`) | Swift 컴파일 오류 |

---

## 절대 하지 말 것

- **Write로 전체 재작성하지 않는다.** 본문·표까지 함께 바뀌어 합의 원칙을 위반한다. 항상 `Edit`로 코드 블록만 핀포인트 교체.
- **외부 가이드에만 있는 큰 섹션을 자동 추가하지 않는다.** 사용자에게 먼저 묻는다.
- **저장소에만 있는 설명용 표·인용구를 자동 제거하지 않는다.** 사용자에게 먼저 묻는다.
- **여러 파일을 한 번에 일괄 수정하지 않는다.** 단계적으로 진행하고 매 단계 사용자 확인.
- **확신이 안 서면 추측으로 적용하지 않는다.** AskUserQuestion으로 확인.

---

## 예시 호출

```
/guide-check https://nasmob.atlassian.net/wiki/spaces/1/pages/754089985 ios/native/banner.md
```

여러 파일:
```
/guide-check banner=https://.../754089985:ios/native/banner.md native=https://.../754090002:ios/native/native-ad.md
```

또는 URL과 파일을 따로 받아 단계적으로 매칭:
```
/guide-check
→ Confluence URL과 비교 대상 저장소 파일을 받아 처리합니다. 어떤 가이드와 어떤 파일을 비교할까요?
```

---

## 참고 작업 이력

- `4e28319` (2026-05-15) — Confluence 5개 페이지(754089985 banner, 754090002 native, 754188289 reward, 788463642 video, 743473203 getting-started) 기준 iOS Native 5개 문서 정합성 정정. 245+ / 146- lines. 본 스킬의 표준 워크플로우는 이 작업에서 도출됨.
