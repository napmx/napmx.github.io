---
name: ios-guide-check
description: iOS Native 가이드 문서를 Confluence 공식 페이지와 정합성 점검한다. 5개 페이지 URL이 미리 박혀 있어 추가 입력 없이 `/ios-guide-check`로 호출 가능. 워크플로우와 원칙은 부모 스킬 `guide-check`를 그대로 따른다. 사용자가 슬래시 커맨드로 명시적으로 호출했을 때만 발동.
---

# ios-guide-check

`guide-check` 스킬의 iOS Native 전용 프리셋. 다음 5개 Confluence 페이지와 저장소 `ios/native/*.md` 매핑이 이미 박혀 있다. 사용자는 URL을 매번 입력하지 않아도 된다.

워크플로우, 핵심 원칙, 오타 정정 패턴, 절대 하지 말 것 등 **모든 동작 규칙은 부모 스킬 `guide-check`를 그대로 따른다**. 이 파일은 매핑과 호출 인자만 정의한다.

---

## URL ↔ 저장소 파일 매핑

| 키워드 | Confluence 페이지 ID | Confluence URL | 저장소 파일 |
|---|---|---|---|
| `getting-started` (별칭: `gs`, `setup`) | 743473203 | https://nasmob.atlassian.net/wiki/spaces/1/pages/743473203/iOS+SDK | `ios/native/getting-started.md` |
| `banner` | 754089985 | https://nasmob.atlassian.net/wiki/spaces/1/pages/754089985/iOS+Banner | `ios/native/banner.md` |
| `native` (별칭: `native-ad`) | 754090002 | https://nasmob.atlassian.net/wiki/spaces/1/pages/754090002/iOS+Native | `ios/native/native-ad.md` |
| `reward` (별칭: `rewarded`, `rewarded-video`) | 754188289 | https://nasmob.atlassian.net/wiki/spaces/1/pages/754188289/iOS+Reward+Interstitial+Video | `ios/native/rewarded-video.md` |
| `video` | 788463642 | https://nasmob.atlassian.net/wiki/spaces/1/pages/788463642/iOS+Video | `ios/native/video.md` |

Confluence cloudId: `nasmob.atlassian.net`

---

## 호출 형식

### 전체 5개 점검 (인자 없음)
```
/ios-guide-check
```
- 기본 순서: `banner → native → reward → video → getting-started`
- 각 파일 작업 종료 시 사용자 확인 후 다음 파일로 진행
- guide-check의 "단계적 진행" 원칙 적용

### 특정 파일만 점검
```
/ios-guide-check banner
/ios-guide-check reward
/ios-guide-check getting-started
```
- 위 매핑 키워드 또는 별칭 사용
- 단일 파일 점검 후 종료

### 여러 파일 선택
```
/ios-guide-check banner,native,video
/ios-guide-check banner native video    (공백 구분도 허용)
```
- 표기된 순서대로 단계적 진행
- 각 파일 작업 종료 시 사용자 확인

### 순서 지정
인자에 표기한 순서대로 진행한다. 예: `/ios-guide-check getting-started banner` → getting-started 먼저, banner 다음.

---

## 실행 절차

1. 인자를 파싱해 (Confluence URL, 저장소 파일 경로) 쌍의 리스트로 변환
2. 부모 스킬 `guide-check`의 워크플로우 2단계(병렬 수집)부터 그대로 실행
3. 각 파일 작업 종료 후 사용자 확인 → 다음 파일

부모 스킬의 1단계(인자 파싱)는 이 파일에서 매핑으로 대체된다. 그 외 모든 단계(병렬 수집, 차이 표, 사전 질문, 핀포인트 Edit, git diff, 단계 확인)는 guide-check를 따른다.

---

## 기본 원칙 (guide-check 상속)

- **코드만 수정**, 안내·설명 문구 유지
- **명백한 Confluence 오타는 정정**해서 적용 (예: `AMMInterstital` → `AMMInterstitial`)
- **import 통일**: 저장소가 일관되게 사용 중인 모듈명을 우선 (현재 `AdMixerMediation`)
- **단계적 진행**: 매 파일 종료 시 사용자 확인
- **저장소 단독 설명 표·인용구 유지**, 장식성 더미 코드 최소화
- **본문-코드 충돌, 큰 섹션 추가 여부 등 의문점은 사전 질문**

상세는 `.claude/skills/guide-check/SKILL.md` 참고.

---

## Confluence 빠른 호출

```javascript
mcp__atlassian__getConfluencePage({
  cloudId: "nasmob.atlassian.net",
  pageId: "<위 표의 페이지 ID>",
  contentFormat: "markdown"
})
```

인증이 안 됐다면 `mcp__atlassian__authenticate`로 사용자 인증 안내 후 대기.

---

## 신규 가이드 페이지가 생겼다면

- 매핑 표에 새 키워드/페이지 ID/URL/파일 경로를 추가
- 별칭이 필요하면 키워드 옆에 괄호로 표기
- iOS Unity 가이드는 별도 스킬 `ios-unity-guide-check`로 분리 권장 (대응 Confluence 페이지가 있을 경우)

---

## 참고

- 부모 스킬: `.claude/skills/guide-check/SKILL.md`
- 본 매핑의 기준 작업: 커밋 `4e28319` (2026-05-15)
