---
title: "Brain Bootstrap (BBT)"
abbreviation: BBT
category: publish
created: "2026-03-14"
---
사용자의 이력서, LinkedIn, 소셜 미디어, 개인 웹사이트 등에서 프로필을 추출하여 BRAIN.md를 풍성하게 채우고, 관련 커뮤니티 브레인을 발견한다. 온보딩 2단계 확장 또는 독립 실행 가능.

## Input
- **최소 1개 이상**: 이력서 파일 경로, LinkedIn URL, 소셜 미디어 URL, 개인 웹사이트 URL, 또는 사용자가 직접 말한 관심사
- **Optional**: 대상 Gobi space slug (커뮤니티 브레인 탐색용)
- **Context**: 기존 `BRAIN.md`

## Output
1. **BRAIN.md** — 업데이트 (in place)
2. **BRAIN_PROFILE.md** — 상세 구조화 프로필 (로컬 전용, 브레인 에이전트용)
3. **Optional**: 1-3개 브레인 업데이트 드래프트 (`_Outbox_/BrainUpdates/`)
4. **추천 커뮤니티 브레인 연결** (화면 출력)

## Main Process

```
PHASE 1: SOURCE COLLECTION
   - 사용자에게 질문: "이력서나 LinkedIn, 개인 홈페이지 같은 온라인 프로필이 있으세요?"
   - 1-3개 URL 또는 파일 경로 수집
   - 파일 (이력서 PDF/DOCX): EDM 로직으로 마크다운 추출
   - URL: 소스별 전략으로 fetch (아래 Source Fetching Strategies 참조)
   - 모든 fetch를 병렬 실행

PHASE 2: PROFILE EXTRACTION
   각 소스에서 canonical schema로 추출:
   - name, title/role, organization, location
   - education, experience, skills (카테고리별)
   - interests (전문적 + 개인적)
   - publications/projects, social links
   - bio summary

PHASE 3: MERGE & DEDUPLICATE
   - 우선순위: Resume > LinkedIn > Personal site > Social media > User-stated
   - Skills/interests: 전체 소스 합집합, 중복 제거
   - 충돌 사항은 사용자에게 음성으로 확인

PHASE 4: BRAIN.md + BRAIN_PROFILE.md GENERATION
   BRAIN.md (공개용, 간결):
   - frontmatter에 profile: "[[BRAIN_PROFILE.md]]" 추가
   - ## Welcome to {Name}'s Second Brain
   - ### About — 합성된 바이오 (2-3 문단, 1인칭)
   - ### Expertise — 도메인, 기술 스킬, 산업
   - ### Interests — 탐구/학습 중인 주제
   - ### Current Focus — 최근 역할/프로젝트
   - ### Connect — 승인된 소셜 링크

   BRAIN_PROFILE.md (로컬, 상세):
   - 전체 교육/경력 타임라인
   - 상세 스킬 분류
   - 모든 소셜 링크
   - 각 데이터 포인트의 소스 출처

   Privacy: 급여, 전화번호, 주소, 추천인 제외. 공개 전 확인.

PHASE 5: BRAIN NETWORK DISCOVERY
   - 사용자의 interests/skills에서 5-10개 검색 쿼리 생성
   - 각 쿼리로 gobi brain search --query 실행 (병렬)
   - 관련성 순 정렬, 중복 제거
   - 상위 3-5개: gobi brain ask --vault-slug <slug> --question "What are your main areas of expertise?"
   - 매칭 결과를 대화형으로 제시

PHASE 6: INITIAL BRAIN UPDATES (Optional)
   - PBU 형식으로 1-3개 업데이트 드래프트 작성:
     1. 자기소개: 누구인지, 무엇을 하는지
     2. 전문 분야 심화: 주요 도메인
     3. 현재 프로젝트: 지금 하고 있는 일
   - _Outbox_/BrainUpdates/에 저장, 사용자에게 게시 여부 확인
```

## Source Fetching Strategies

각 소스에 대해 **WebFetch를 먼저** 시도 (단순, 빠름). 실패하거나 불완전한 데이터(인증 벽, JS 렌더링)일 경우 **Playwright로 폴백**.

### Fallback Chain (모든 소스 공통)
```
WebFetch → Playwright (headless) → 사용자에게 텍스트 붙여넣기 요청 → 우아하게 건너뛰기
```

### LinkedIn
- **WebFetch first**: `WebFetch(url)` — 공개 프로필이면 작동
- **Auth-wall 대응**: LinkedIn은 미인증 접근을 자주 차단. 차단 시: 사용자에게 "About" 섹션 붙여넣기 또는 PDF 내보내기(LinkedIn Settings > Data Privacy > Get a copy) 요청
- **추출 대상**: 이름, 헤드라인, 요약/소개, 경력 목록, 학력, 스킬, 추천 게시물

### X (Twitter)
- **WebFetch first**: `WebFetch(url)` — 바이오 추출에 충분한 경우 많음
- **Auth-wall 대응**: 바이오는 보통 접근 가능. 차단 시: 사용자에게 바이오 붙여넣기 요청
- **추출 대상**: 표시 이름, 바이오, 위치, 웹사이트 링크, 고정 트윗, 최근 트윗 주제 (관심사 마이닝)

### Threads (Meta)
- **WebFetch first**: JS 기반이라 불완전할 수 있음
- **차단 시**: 사용자에게 Threads 활동 주제 설명 요청
- **추출 대상**: 바이오, 최근 게시물 주제/테마

### GitHub
- **WebFetch first**: `WebFetch(https://github.com/username)` — 공개 프로필이므로 잘 작동
- **추출 대상**: 바이오, 주요 언어, 고정 리포지토리 (이름 + 설명 + 언어), 프로필 README (있으면: `https://github.com/username/username`), 소속 조직

### Personal Website / Blog
- **WebFetch first**: 홈페이지 + `/about` 페이지 병렬 fetch
- **Playwright fallback**: JS 기반 SPA 사이트용
- **추출 대상**: About/바이오, 프로젝트 목록, 블로그 주제 (관심사 마이닝), 연락처/소셜 링크

### Resume / CV (파일)
- **PDF/DOCX**: EDM (Extract Document to Markdown) 프롬프트 로직으로 마크다운 변환
- **구조화 추출**: 학력, 경력, 스킬, 프로젝트, 출판물
- **비영어**: 추출 후 주요 필드 번역

### Chat Input (카카오톡 등)
- **입력**: 카카오톡 대화명 + 메시지 텍스트 (붙여넣기 또는 파일)
- **추출 대상**: 표시 이름, 관심사/주제 (대화 내용 마이닝), 직업/역할 힌트, 성격/커뮤니케이션 스타일
- **처리**: 대화 내용에서 자기소개, 관심 분야, 전문성 관련 발언을 추출하여 프로필 보강
- **우선순위**: 다른 공식 소스보다 낮음 (보조 데이터로 활용)

## Caveats

### BRAIN.md 구조
```yaml
---
title: "{Name}'s Second Brain"
description: "합성된 프로필 요약"
thumbnail: "[[BRAIN.jpg]]"
prompt: "[[BRAIN_PROMPT.md]]"
profile: "[[BRAIN_PROFILE.md]]"
tags:
  - profile
  - second-brain
created: YYYY-MM-DD HH:MM:SS
---
```

### BRAIN_PROFILE.md 구조
```yaml
---
title: "{Name} - Detailed Profile"
created: YYYY-MM-DD HH:MM:SS
sources:
  - type: resume/linkedin/website/social/chat
    url_or_path: "..."
    fetched: YYYY-MM-DD
tags:
  - profile
  - brain-bootstrap
---
```
본문에 아래 섹션 포함:
- `## Personal Info` — 이름, 위치, 연락처 (공개 가능한 것만)
- `## Education` — 학력 타임라인
- `## Experience` — 경력 타임라인
- `## Skills` — 카테고리별 스킬 분류
- `## Interests` — 전문적 + 개인적
- `## Projects & Publications` — 주요 프로젝트/출판물
- `## Social Links` — 모든 소셜 링크
- `## Source Attribution` — 각 데이터 포인트의 출처

### Privacy 규칙
- 급여, 전화번호, 주소, 추천인은 절대 포함하지 않음
- BRAIN.md (공개용)에는 사용자가 승인한 정보만 포함
- BRAIN_PROFILE.md는 로컬 전용

### 독립 실행 vs 온보딩 통합
- **독립 실행**: 사용자가 "BBT" 또는 "브레인 부트스트랩"이라고 말하면 Phase 1-6 전체 실행
- **온보딩 통합**: Step 2-2에서 소스 제공 시 Phase 1-4 실행, Step 5-0에서 Phase 5 실행
- **소스 없음**: 기존 웹 검색 로직으로 폴백
