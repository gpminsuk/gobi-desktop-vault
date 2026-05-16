---
name: writeup
description: Draft an artifact from recent conversation history (DTA dialogue, brainstorming, casual chat). Presents 2-4 draft variations via AskUserQuestion, then asks whether to save to Articles/, post to gobi space, or both. Default mode requires approval; pass `bypass` to auto-pick the highest-confidence draft and apply Recommended destinations.
disable-model-invocation: true
---

# Writeup

이전 대화를 재료로 글 초안을 만들고, 사용자가 고른 안을 `Articles/`에 저장하거나 gobi space에 게재한다. 사용자 발화를 1인칭으로 살려 쓴다 — 에이전트가 자기 말투로 바꿔쓰지 않는다.

## Args

- (default) `approval` — interactive: 초안 선택 → 게재 선택 → 실행
- `bypass` — 자동: 가장 자신감 높은 초안 선택 + 권장 게재처(로컬 저장 + 스페이스 게시) 자동 실행

## Flow

### 1. CONTEXT GATHERING

직전 대화에서 (최근 20–50턴 또는 현재 세션) 다음을 추출:

- **주제**: 대화에서 다뤄진 핵심 토픽
- **사용자의 핵심 발화**: 직접 인용 가능한 입장/주장/표현
- **결론/통찰**: 대화가 도달한 지점
- **열린 질문**: 미해결로 남은 부분

도구 호출, 파일 작업, 스케줄링 같은 잡무 턴은 건너뛴다. DTA 대화에서 넘어왔다면 REFLECT 단계의 핵심 통찰 한 줄을 시작점으로 잡는다.

### 2. DRAFT GENERATION

**2–4개 초안 변형**을 만든다. 각 초안은 다른 앵글:

- **A. Reflection (긴 글)** — 에세이 스타일, 사고 흐름 포함, 1인칭 산문. ~400-600 words. Articles에 적합.
- **B. Insight (짧은 글)** — 핵심 인사이트 + 보조 포인트 2-3개 + 질문/캡션 한 줄. ~150-250 words.
- **C. Story (스토리)** — 일화 → 깨달음 구조. ~200-300 words. 경험담이 있을 때만.
- **D. Hot take (도발적 한 줄)** — 도발적 한 줄 + 정당화 2-3문장. ~50-100 words.

대화에 맞지 않는 변형은 건너뛴다 (서사가 없으면 Story 빼기, 짧은 주제면 Reflection 빼기 등). 최소 2개는 만들어 비교 가능하게 한다.

### 3. DRAFT SELECTION (approval 모드만)

`AskUserQuestion`으로 각 초안을 `preview` 필드에 전체 내용 담아 제시:

```
question: "어떤 톤으로 정리할까요?"
header: "Draft pick"
options:
  - label: "A. Reflection (긴 글)"
    description: "에세이 스타일, 사고 과정 포함, ~500 words"
    preview: <full draft A>
  - label: "B. Insight (짧은 글)"
    description: "핵심 인사이트 + 보조, ~200 words"
    preview: <full draft B>
  - ...
```

음성 환경이면 번호로 호명하고 각 초안의 첫 1-2문장만 읽어준다 (전체 read-out은 길어서 부적합).

**bypass 모드**: 가장 자신감 높은 초안 자동 선택 — 대화의 dominant register(서사/통찰/짧은 한 줄)와 가장 잘 맞고 본문이 비어있지 않은 변형을 고른다.

### 4. DESTINATION SELECTION (approval 모드만)

초안 선택 후 게재처 묻기:

```
question: "어디에 게재할까요?"
header: "Destination"
options:
  - label: "로컬 저장만 (Articles/)"
    description: "Articles/YYYY-MM-DD <제목>.md 로 저장. 게재 없음."
  - label: "gobi space에만 게재"
    description: "스페이스에 게시. 로컬 저장 안 함."
  - label: "둘 다 (Recommended)"
    description: "로컬 저장 + 스페이스 게시"
```

**bypass 모드**: "둘 다"를 기본으로.

### 5. EXECUTE

선택된 게재처에 따라:

- **로컬 저장**:
  - MATCH-OR-CREATE: `grep -ri "<주제 키워드>" Articles/` 로 후보 검색
  - 후보 있음: AskUserQuestion으로 "기존 글 [[파일]]에 이어 쓸까요, 새 글로 만들까요?" (approval 모드)
  - 후보 없음 또는 bypass: `Articles/YYYY-MM-DD <한 줄 제목>.md` 새로 생성
  - frontmatter: `title`, `created` (YYYY-MM-DD HH:MM:SS), `tags`, `source_dialogue: writeup` (DTA에서 넘어왔으면 `source_dialogue: DTA→writeup`)
- **스페이스 게재**:
  - `gobi space create-post` 로 게시

### 6. REPORT

실행 결과를 명확히 알린다:

- 로컬: "저장했어요: [[Articles/...]]"
- Post: "스페이스 게시 완료: <URL>"
- 부분 실패: 성공 항목과 실패 항목 분리해서 보고 + 실패 항목 재시도 방법 안내

## Caveats

### 사용자 목소리 보존 — CRITICAL
- 대화에서 사용자가 한 말은 그대로 인용하거나 최소한의 다듬기만 (추임새 정리 수준).
- 에이전트의 표현으로 바꿔쓰지 않는다. 단어 선택과 비유는 사용자 것을 유지.
- 직접 인용은 blockquote (`> "..."`)으로 마킹.

### Source 링크
- 대화에서 참조한 클리핑/Article이 있으면 글 끝에 `## 관련` 섹션으로 위키링크.
- DTA에서 넘어왔다면 DTA 세션에서 다뤄진 seed 파일도 링크.

### 언어 / 톤
- `Context/preferences.md`의 "언어 / 톤"과 "콘텐츠 형식" 따른다.
- 미설정 시 `.gobi/settings.yaml`의 `primaryLanguage` (현재: ko).

### Bypass 안전장치
- bypass 모드여도 다음 경우 자동 진행 금지하고 사용자에게 확인:
  - 초안에 식별 가능한 제3자 정보 (이메일, 주소, 미공개 인물 이름)
  - 미공개 회사/프로젝트 정보로 의심되는 내용
  - 대화에서 사용자가 명시적으로 "이건 게재하지 마" 라고 한 부분
- 위 경우 발견 시 approval 모드로 강등 + "이 부분 때문에 자동 게재 안 했어요" 안내

### MATCH-OR-CREATE 정확도
- 잘못된 파일에 섹션을 추가하면 사용자의 기존 글을 오염시킨다.
- 매칭이 모호하면 무조건 사용자에게 확인 후 진행 (bypass 모드여도).

### 게재 실패 시
- 로컬 저장은 항상 우선 시도. 게재가 실패해도 글은 남는다.
- 게재 실패 시 사용자에게 "글은 로컬에 잘 저장됐어요. 나중에 '다시 게재해줘' 라고 하시면 다시 시도할게요." 안내.
