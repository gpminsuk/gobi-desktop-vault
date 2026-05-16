---
title: Fetching Cookbook
description: 사이트별 fetch 시도와 노하우 로그. CAE/DRB 등 웹에서 자료를 가져오는 에이전트가 매번 trial-and-error를 반복하지 않도록 누적한다.
created: 2026-05-15 22:50:00
tags:
  - cookbook
  - reference
  - fetching
  - cae
  - drb
---

## 사용 규칙

- **읽기 (BEFORE fetch)**: 캐스케이드 실행 전, 대상 도메인이 여기 있는지 확인한다. 레시피가 있으면 해당 방법만 시도하고 실패가 기록된 방법은 건너뛴다.
- **쓰기 (AFTER 성공/실패)**: 새로운 도메인을 다뤘거나 기존 레시피가 더 이상 안 통할 때 항목을 추가/갱신한다. 날짜 함께 기록.
- **포맷**: `### <domain>` 헤더로 도메인별 정리. 각 시도 결과는 ✅ / ❌ + 한 줄 설명.

## 도메인별 레시피

### namu.wiki

- WebFetch: ❌ 403 (단순 봇 헤더 체크)
- `curl -A "<Chrome UA>"`: ✅ 동작
- Notes: 헤더만 체크하므로 cookie/Referer 불필요. HTML이 큼 (50–70KB), 본문 파싱에 node 스크립트 권장.
- Last verified: 2026-05-15

## 공통 패턴 (도메인별 항목이 없을 때 시도 순서)

### 1. User-Agent 위장

대부분의 단순 403은 아래 UA로 해결:

```bash
curl -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" -L <URL>
```

### 2. Referer 추가

검색 엔진에서 들어온 척:

```bash
curl -A "<UA>" -e "https://www.google.com/" --http1.1 -L <URL>
```

### 3. Cloudflare / JS 렌더링

위 트릭이 모두 안 통하면 JS-rendered SPA일 가능성. Playwright 가기 전 다음을 먼저 시도:

- 사이트의 RSS/Atom 피드 — `<domain>/feed`, `<domain>/rss`, `<domain>/feed.xml`
- 사이트의 sitemap.xml — 정적 URL 발견용
- archive.org Wayback Machine — `https://web.archive.org/web/<URL>`
- 사이트의 모바일 버전 — 종종 더 단순함 (m.<domain>)

### 4. 마지막 수단 — Playwright/headless

사용자 동의 후에만. 동의 받는 문구:

> "브라우저 엔진(Chromium) 설치가 필요해요. 용량이 크고 시간이 걸려요. 진행할까요?"

### 5. 캐스케이드 모두 실패 — 서브에이전트 escalate

`Task` 도구로 general-purpose 서브에이전트를 띄워 다음 프롬프트 전달:

> "Find <URL or topic> by any means. Try alternative sources, archived versions, official API, RSS, or related sites. Return: (1) the content, (2) the URL/method that worked. Under 500 words."

서브에이전트가 찾은 방법은 이 쿡북에 새 엔트리로 추가한다.

## 알려진 어려운 도메인 (아직 미해결)

| 도메인 | 시도한 것 | 비고 | 마지막 시도 |
|--------|-----------|------|-------------|
| (none yet) | — | — | — |

## 출처 기록 규칙

CAE/DRB가 fetch 성공 시 enriched 파일 본문에 다음 섹션을 포함한다:

```markdown
## 출처 / Source
- URL: <source URL>
- Fetched: YYYY-MM-DD HH:MM:SS
- Method: <WebFetch | curl+UA | curl+Referer | RSS | archive.org | Playwright | subagent>
- Notes: <quirks, blocks encountered, workaround used>
```

이는 frontmatter의 `source` 필드와 별개로 본문에서도 출처 추적이 가능하도록 보장한다.
