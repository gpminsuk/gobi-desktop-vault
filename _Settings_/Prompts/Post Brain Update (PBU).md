---
title: "Post Brain Update (PBU)"
abbreviation: PBU
category: publish
created: "2026-03-08"
---
Gobi 커뮤니티에 공유할 Brain Update 드래프트를 소스 파일 기반으로 작성한다.

## Input
- **Source file(s)**: 볼트 내 아무 콘텐츠 (Clippings, Summary, Roundup, Lifelog, Events, Analysis 등)
- **Optional**: 포커스할 토픽/앵글 (소스에 여러 주제가 있을 때)
- **Optional**: 대상 독자 컨텍스트

## Output
- 드래프트 파일: `_Outbox_/BrainUpdates/YYYY-MM-DD [제목] - Claude Code.md`
- Frontmatter에 `approve_for_publish: false` 포함
- 작성 후 Obsidian에서 파일 열기

## Main Process
```
1. SOURCE ANALYSIS
   - 소스 파일 전체 읽기
   - 핵심 인사이트, 스토리, 인용구 식별
   - 토픽/앵글이 지정되지 않은 경우 가장 임팩트 있는 주제 선택

2. CONTENT CRAFTING
   - 400-800 단어의 에세이 스타일로 작성
   - H2 제목 없이 바로 본문 시작 (frontmatter title이 제목 역할)
   - 오프닝 훅 (1-2 패러그래프): 독자의 관심을 끄는 도입
   - 본론 (2-3 패러그래프): 분석, 맥락, 의미 설명
   - 원문 인용은 블록쿼트(>) 형식으로 포함
   - 마무리에 심층 분석 링크: → **관련 분석**: [[path|표시 텍스트]]
   - 링크는 항상 처리된 노트(AI/Summary, AI/Analysis 등)로 연결 (원본 Ingest/Clippings가 아님)

3. DRAFT CREATION
   - _Outbox_/BrainUpdates/에 파일 생성
   - Frontmatter 작성 (아래 형식 참조)
   - 소스에서 참조하는 파일/이미지를 .gobi/syncfiles에 추가

4. POST-CREATION
   - Obsidian에서 파일 열기
   - 유저 확인 후 approve_for_publish: true로 변경 시 gobi brain post-update로 발행
```

## Caveats
### Frontmatter 형식
```yaml
---
title: "업데이트 제목"
date: YYYY-MM-DD
approve_for_publish: false
tags:
  - tag1
  - tag2
  - tag3
---
```
- 태그는 3-5개, plain text

### 글쓰기 원칙
- **에세이 스타일**: 불릿 포인트 나열이 아닌 산문체로 작성
- **패러그래프 응집력**: 각 패러그래프 최소 2-3문장, 한 문장짜리 패러그래프 금지
- **언어**: 한국어 기본, 영어 원문 인용은 그대로 보존
- **원문 인용**: 블록쿼트(>) 형식 필수

### 이미지 링크 형식
- **Vault root 기준 상대 경로 사용**: `![](_Outbox_/BrainUpdates/_files_/image.png)`
- ❌ `![](_files_/image.png)` — 이미지가 표시되지 않음
- ❌ `![[image.png]]` — wiki link 형식은 브레인에서 지원되지 않음

### 참조 파일 동기화
- 드래프트에서 참조하는 파일 경로를 `.gobi/syncfiles`에 추가
- 이미지(`_files_/` 등)도 포함
- 관련 도서 요약 등 링크된 콘텐츠 파일도 syncfiles에 추가

### 발행 워크플로우
- 생성 시 항상 `approve_for_publish: false`
- 유저가 Obsidian에서 확인 후 `true`로 변경
- 발행: `gobi brain post-update --title "제목" --content "본문 전체"`
- 발행 시 드래프트 본문 전체를 그대로 사용 (누락 금지)

### 발행 후 수정
- `gobi brain edit-update <updateId> --content "수정된 본문"` 사용
- 삭제: `gobi brain delete-update <updateId>`
- 이미지/링크 문제 발생 시 edit-update로 즉시 수정 가능
