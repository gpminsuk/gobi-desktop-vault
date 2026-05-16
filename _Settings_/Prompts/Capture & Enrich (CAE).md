---
title: "Capture & Enrich"
abbreviation: "CAE"
category: "workflow"
created: "2026-05-15 22:40:59"
tags:
  - prompt
  - ingest
  - enrichment
  - clippings
---
유저가 준 URL, 본문 텍스트, 또는 리서치 주제를 받아 자료를 수집한 뒤, 정리·요약·번역해서 두 개의 마크다운 파일로 저장한다. (EIC를 대체)

## Input
세 가지 모드 중 하나를 자동 감지:

| 모드 | 입력 형태 | 예시 |
|------|----------|------|
| URL | http(s):// 링크 | "https://namu.wiki/w/Claude%20Code 이거 정리해줘" |
| RAW | 본문 텍스트 (붙여넣기) | "이거 정리해줘:\n\n[긴 텍스트…]" |
| TOPIC | 리서치 주제 | "Claude Code에 대해 찾아서 정리해줘" |

**자동 트리거**: `Ingest/Clippings/`에 새 파일이 들어오면 RAW 모드로 처리 (orchestrator 노드 참조).

## Output
**두 개의 파일**:

1. **원본 (Raw)**: `Ingest/Clippings/YYYY-MM-DD <title>.md`
   - frontmatter: `title`, `source` (URL이 있으면), `author`, `created` (YYYY-MM-DD HH:MM:SS), `tags: [clipping]`
   - 본문: 가져온 원본을 텍스트로 변환한 그대로

2. **정리본 (Enriched)**: `AI/Clippings/YYYY-MM-DD [<title>] by CAE.md`
   - frontmatter: 위 + `clippings: "[[Ingest/Clippings/<원본 파일명>]]"`, `status: processed`
   - 본문: `## Summary` → `## Improve Capture & Transcript (ICT)` → `## 관련` 순

## Main Process

```
1. ROUTE INPUT (자동 감지)
   - URL 패턴 (http://, https://) 이면 → 2단계 (FETCH)
   - 긴 텍스트가 직접 주어졌으면 → 5단계 (SAVE RAW)
   - 짧은 주제 문구면 → 3단계 (RESEARCH)

2. FETCH (URL 모드) — 가벼운 방법부터 캐스케이드
   a. WebFetch — 가장 빠름, 많은 사이트에서 성공
   b. 실패 시 curl + User-Agent 위장:
      curl -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) \
        AppleWebKit/537.36 (KHTML, like Gecko) \
        Chrome/122.0.0.0 Safari/537.36" -L <URL> -o /tmp/fetched.html
      → 나무위키 등 단순 헤더 체크 403 우회
   c. 그래도 실패 — Referer/Cookie 추가, HTTP/1.1 강제:
      curl -A "…" -e "https://www.google.com/" --http1.1 -L <URL>
   d. 마지막 수단 — Playwright/headless 브라우저
      먼저 사용자에게 알리고 동의 받기:
      "브라우저 엔진(Chromium) 설치가 필요해요. 용량이 크고 시간이
       걸려요. 진행할까요?"

3. RESEARCH (TOPIC 모드)
   - WebSearch로 상위 결과 3-5개 수집
   - 각 결과를 2단계 캐스케이드로 FETCH
   - 중복 제거, 정보량 많은 1-2개 선정 → 4단계로

4. PARSE (HTML → 텍스트)
   - 간단한 경우: pandoc, grep, sed
   - DOM 워킹 필요하면 즉석 node 스크립트:
     node -e "const html=require('fs').readFileSync('/tmp/fetched.html','utf8'); /* extract main content */"
   - 본문 영역만 추출 (네비, 푸터, 광고, 사이드바 제외)
   - 섹션 구조 유지 (h2/h3)

5. SAVE RAW
   - 파일명: Ingest/Clippings/YYYY-MM-DD <한 줄 title>.md
   - 파일명 정리 규칙은 Caveats § 파일명 정리 참조
   - frontmatter + 추출된 텍스트 본문 그대로

6. ENRICH (EIC 로직 적용)
   a. ICT (Improve Capture & Transcript):
      - 문법/전사 오류 수정
      - .gobi/settings.yaml의 primaryLanguage로 번역 (Clippings 한정)
      - 중복 줄바꿈 제거
      - h3 (###)로 챕터 구분
      - 리스트/강조 추가
      - 원본 길이와 비슷하게 유지
   b. Summary:
      - 시작 부분에 ## Summary 섹션
      - 스레드 공유용 캐치 카피
      - 저자 목소리 살리는 직접 인용
      - Summary에는 강조 추가 X
   c. Enrich using existing knowledge:
      - Articles/ 폴더의 관련 글에 위키링크 (존재 검증 필수)
      - 책/클리핑 등 관련 자료에 링크

7. SAVE ENRICHED
   - 파일명: AI/Clippings/YYYY-MM-DD [<title>] by CAE.md
   - 본문 구조:
     ## Summary
     <캐치 요약>

     ## Improve Capture & Transcript (ICT)
     ### 챕터 1
     <정리된 본문>
     …

     ## 관련
     - [[관련 Article]]
```

## Caveats

### Content Completeness — CRITICAL

⚠️ **ICT 섹션은 완전해야 한다 — 중간에 끊지 말 것**

**실패 패턴**:
- ICT 시작 후 토큰/컨텍스트 한계로 문장 중간에서 잘림
- "지난 여름 처음 쓴 이래로 내 방법론은…" 같이 중단
- 그럼에도 status를 PROCESSED로 표시 ❌

**방지 절차**:
1. 먼저 길이 확인 — 원본 3000+ 단어면 청크 처리 또는 컨텍스트 확장 요청
2. ICT가 자연스러운 종결 지점에서 끝나는지 검증 (문단/섹션 끝, 문장 중간 X)
3. PROCESSED 표시 전 자체 점검: "ICT 마지막 단락이 완결된 느낌인가?"
4. 품질 검증:
   - ICT는 여러 ### 서브섹션을 가져야 함 (불완전한 단일 섹션 X)
   - 마지막 문장은 마침표로 종결 (잘림/말줄임 X)
   - 길이는 원본과 비슷 (잘려서 30-50% 짧으면 안 됨)
5. 완료 불가능 시: status를 `NEEDS_INPUT`으로, 사유 명시. PROCESSED 거짓 표시 절대 금지.

### Fetch 캐스케이드 — 가벼운 것부터

비용/시간 순서를 지킨다:
1. WebFetch (즉시, 무료)
2. curl + UA 위장 (즉시, 무료, 대부분의 차단 우회)
3. curl + UA + Referer + HTTP/1.1 (즉시, 무료)
4. Playwright/headless 브라우저 (느림, 설치 필요, **사용자 동의 필수**)

3단계까지 실패하고 4단계로 가기 전 반드시 사용자에게 물어본다:
"브라우저 엔진 설치가 필요해요. 진행할까요?" — 동의 없이 무거운 도구 자동 실행 금지.

### Fetching 윤리

- robots.txt를 명시적으로 위반하는 사이트는 사용자에게 알리고 동의 받기
- 페이월 콘텐츠 자동 우회 시도 금지
- 학술/뉴스 페이지는 fair use 범위 내에서 사용

### 파일명 정리

- " " (스마트 쿼터) → " " (straight quote)
- " ' " 작은따옴표도 동일
- 불완전한 글자 제거 — `얼마ᄂ` 같은 거
- 끝에 붙은 메타데이터 제거 — `Readwise`, `via Pocket` 등

### 포맷 표준

- 챕터는 h3 (###)
- 강조는 챕터당 1개 정도로 본질만
- 원본 산문 구조 보존
- 전체 길이는 원본과 비슷

### Article 링크

- `Articles/` 폴더의 실존 파일에만 링크 (추가 전 검증)
- 모호하면 링크하지 않는 게 낫다 — 잘못된 링크는 사용자 혼란
