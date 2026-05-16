---
name: writeup
description: Draft a writeup from recent conversation, present 2-3 options, then save to Articles/ and/or post to gobi space. Default mode requires approval; pass `bypass` to act on Recommended without confirmation.
argument-hint: "[approval|bypass]"
disable-model-invocation: true
---

대화 내용에서 글감을 뽑아 2-3개의 writeup 초안을 만들고, 사용자가 고른 걸 `Articles/`에 저장하거나 gobi space에 게재한다 (또는 둘 다).

## Trigger

- `/writeup` — approval mode (기본). 모든 AskUserQuestion 단계 거침.
- `/writeup bypass` — Recommended 옵션 자동 선택. preference.md 기본값 따름.

## Input

- **현재 대화** (conversation context — 자동 확보)
- (선택) 시드 참조: 사용자가 대화 중 언급한 클리핑/Article/세션 파일
- `Context/preference.md` — 콘텐츠 형식·언어·톤·기본 게시 동작
- `Context/interest.md` — 글의 태그 추론에 활용

## Output

사용자 선택에 따라 조합:
- **Local 저장**: `Articles/YYYY-MM-DD <한 줄 제목>.md`
- **BU 게시**: `gobi brain post-update --title "<제목>" --content "<요약>"`
- **스레드 게시**: `gobi space create-thread`

## Main Process

```
1. GATHER CONTEXT
   - 현재 conversation에서 핵심 주제·통찰·결론·인용 추출
   - 사용자의 1인칭 발화는 그대로 보존 (paraphrase 금지)
   - 대화에서 언급된 Articles/Ingest 파일 있으면 가볍게 참조
   - DTA 대화 직후 호출된 경우: REFLECT 단계의 "핵심 통찰" 문구를 시드로 사용

2. DRAFT VARIATIONS (2-3개)
   서로 다른 형식/길이/톤으로 초안 생성:
   - A: 긴 형식 (Article) — 제목 + ## 핵심 통찰 + ## 생각의 흐름 +
        ## 열려있는 질문 + ## 관련
   - B: 짧은 형식 (BU/스레드용) — 1-2단락, 280-600자
   - C: 미디엄 (블로그 포스트) — ## Summary + 3-4개 ## 섹션
   - preference.md "콘텐츠 형식 → 응답 길이/스타일" 따라 후보 우선순위
     조정. 사용자가 평소 medium-mixed면 C를 Recommended로.

3. PRESENT DRAFTS — AskUserQuestion (single-select)
   각 초안의 제목·길이·스타일을 옵션으로:
   - 옵션 A: 긴 형식 Article (~XXX단어)
   - 옵션 B: 짧은 BU/스레드 (~XXX자)
   - 옵션 C: 미디엄 블로그 (~XXX단어)
   - 옵션 D: 다 마음에 안 듦 — 다른 톤으로 재생성
   - 사용자 평소 스타일 매칭 옵션에 (Recommended) 마크

   bypass 모드: Recommended 자동 선택, 4단계로 직진.

4. PRESENT ACTIONS — AskUserQuestion (single-select)
   고른 초안으로 무엇을 할지:
   - 옵션 A (Recommended): Articles/에만 저장
   - 옵션 B: gobi space BU로 게재 (저장 X)
   - 옵션 C: gobi space 스레드로 게재 (저장 X)
   - 옵션 D: 저장 + BU
   - 옵션 E: 저장 + 스레드
   - 옵션 F: 저장 + BU + 스레드

   bypass 모드: preference.md "작업 흐름 → 기본 게시 동작" 따름.
   미설정 시 옵션 A.

5. ADJUST (선택)
   사용자가 "이 부분 수정해줘" 요청하면 반영 후 3단계로 돌아감.
   만족하면 6단계로.

6. EXECUTE
   - 저장: Articles/YYYY-MM-DD <제목>.md 생성
     - frontmatter: title, created (YYYY-MM-DD HH:MM:SS), tags,
       source_dialogue: writeup, seed: "[[<시드 파일>]]" (있으면)
   - BU 게시: gobi brain post-update --title "<제목>" --content "<본문>"
   - 스레드 게시: gobi space create-thread (현재 active space 사용)
   - 게재 본문 끝에 "📝 원문: <vault URL or wikilink>" 자동 추가

7. CONFIRM
   - 성공: "[[Articles/...]]에 저장했어요. BU로도 게재됨: <URL>"
   - 부분 실패: "저장은 됐는데 게재가 실패했어요. 다시 시도할까요?"
   - 전체 실패: "문제가 생겼어요. <에러>. 어떻게 할까요?"
```

## Caveats

### 사용자 목소리 보존 — CRITICAL
- 대화에서 사용자가 한 말은 그대로 인용. paraphrase 금지.
- 추임새/말더듬 정도만 정리.
- 에이전트의 의견이나 외부 지식이 글에 들어가지 않는다.
- 의심스러우면 인용 부호로 감싸 명시.

### 게재 콘텐츠 길이
- **BU**: 본문 600자 권장
- **스레드**: 280자 권장
- 초과 시 자동 축약: 핵심 문장 + 처음 2-3문장 발췌 + "📝 전체 글: [[Articles/...]]" 링크

### 중복 방지
- 같은 주제 글이 이미 `Articles/`에 있으면 5단계 진입 전 경고:
  "[[기존글]]과 비슷한 주제예요. 새로 만들까요, 기존에 추가할까요?"
  (AskUserQuestion으로 처리)
- 게재 후 5분 내 같은 글을 다시 게재 요청하면 중복 확인.

### bypass 모드 안전장치
- bypass는 신뢰할 만한 짧은 대화에서만 사용
- 3분 이상 대화 또는 5+개 outbound 메시지가 있는 경우, bypass 요청도
  4단계 PRESENT ACTIONS만은 무조건 묻기 (게재는 되돌릴 수 없음)

### preference.md 연동
- "콘텐츠 형식 → 응답 길이/스타일" → 초안 변형 우선순위
- "작업 흐름 → 기본 게시 동작" (확장 키) → bypass 액션 기본값
- "언어 / 톤 → 격식/존댓말" → 초안 작성 시 적용
- "콘텐츠 형식 → 이모지" → 게재 본문 이모지 사용 여부

### DTA와의 관계
- DTA는 질문·대화만 담당하고 글로 옮기는 건 writeup이 한다 (관심사
  분리). DTA 세션 끝에 "writeup으로 정리할까요?"가 자연스러운 흐름.
- writeup은 DTA 없이도 단독 호출 가능 (예: RVA 대화 도중에도).

### Articles/ 외 출력 금지
- 저장은 항상 `Articles/` 하위. AI/, Ingest/ 등 다른 폴더에 쓰지 않는다.
- 게재된 콘텐츠는 외부(gobi space)에 있지만, 로컬 사본은 사용자 선택에
  따라 Articles/에만 둔다.
