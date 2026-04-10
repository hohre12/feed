# Feed — SNS 콘텐츠 생성 & 발행 플러그인

> 피드에 올릴 글을 만들어주는 Claude Code 플러그인.
> 플랫폼별 맞춤 콘텐츠를 인터뷰 기반으로 생성하고, 저장하고, 발행한다.

---

## 1. 프로젝트 개요

### 문제

- SNS 플랫폼마다 톤, 포맷, 잘 먹히는 스타일이 다름
- AI가 만든 글은 "AI 티"가 나서 반응이 안 됨
- 매번 플랫폼별로 따로 글을 쓰는 건 비효율적
- 어떤 주제/스타일이 반응이 좋았는지 추적이 안 됨

### 해결

- 플랫폼별 프로필 + 스타일 템플릿으로 맞춤 콘텐츠 생성
- 사람이 쓴 것 같은 입말/구어체 규칙 강제
- 인터뷰 기반 UX로 빠르게 생성 → 저장 → 발행
- 포스팅 히스토리 + 반응 데이터 축적

### 핵심 원칙

1. **사람 냄새** — AI 티 나면 실패. 구어체, 불완전 문장, 변주 필수.
2. **인프라 제로** — DB 없음, 서버 없음. Claude Code + 로컬 파일만.
3. **3번 선택이면 끝** — 인터뷰는 최대 3~4단계. 길어지면 안 씀.
4. **데이터가 쌓인다** — 모든 포스팅과 반응이 저장되어 추후 분석/추천에 활용.

---

## 2. 아키텍처

### 설계 원칙

- **플러그인 코드와 사용자 데이터 분리** — 플러그인은 read-only 템플릿, 사용자 데이터는 `~/.config/feed/`
- **계층적 분산 + 필요 시 로드** — SKILL.md는 라우터만, 실제 지침은 분산된 파일에서 필요한 것만 로드
- **파일 추가만으로 확장** — 새 플랫폼/스타일/규칙/플로우 추가 시 기존 파일 수정 불필요

### 분리 기준

| 기준 | 분리 | 합침 |
|------|------|------|
| 요청마다 다른 조합으로 로드 | O (platforms/, styles/) | |
| 항상 같이 로드되지만 독립적으로 성장 | O (rules/ 내 개별 파일) | |
| 항상 같이 로드되고 양이 적음 | | O (하나의 파일) |

---

## 3. 프로젝트 구조

### 플러그인 (GitHub에 올라감 — read-only 템플릿)

```
feed/
├── .claude-plugin/
│   └── plugin.json                        # 플러그인 매니페스트
│
├── settings.json                          # 기본 권한 설정
├── CLAUDE.md                              # 플러그인 개발 가이드
├── README.md                              # 사용자 설치/사용 가이드
├── LICENSE
├── CHANGELOG.md
│
├── skills/                                # 스킬 (공식 플러그인 구조)
│   ├── feed/                              #   /feed — 메인 라우터
│   │   ├── SKILL.md                       #     라우터 + 인자 파싱
│   │   ├── interview.md                   #     인터뷰 플로우 상세
│   │   └── reference.md                   #     라우팅 가이드 (어떤 상황에 어떤 파일을 읽을지)
│   │
│   ├── feed-publish/                      #   /feed-publish — 미발행 포스트 발행
│   │   └── SKILL.md
│   │
│   ├── feed-report/                       #   /feed-report — 리포트
│   │   └── SKILL.md
│   │
│   └── feed-recommend/                    #   /feed-recommend — 주제 추천
│       └── SKILL.md
│
├── rules/                                 # 작성 규칙
│   ├── common.md                          #   AI 티 방지, 공통 작성 규칙
│   ├── scoring.md                         #   채점 기준 + 통과 조건
│   ├── ko.md                              #   한글 작성 규칙
│   └── en.md                              #   영어 작성 규칙
│
├── platforms/                             # 플랫폼 프로필
│   ├── threads.md
│   ├── x.md
│   ├── reddit.md
│   └── devto.md
│
├── styles/                                # 스타일 템플릿
│   ├── twist-poem.md                      #   반전시
│   ├── debate.md                          #   논쟁
│   ├── relatable.md                       #   공감
│   ├── one-liner.md                       #   한줄반전
│   ├── quote-parody.md                    #   명언패러디
│   └── self-deprecation.md                #   자조
│
├── flows/                                 # 작업 플로우
│   ├── save.md                            #   저장 형식 + frontmatter 스펙
│   └── publish.md                         #   발행 분기 로직 (API/클립보드)
│
└── docs/
    └── design.md                          # 이 문서
```

### 사용자 데이터 (`~/.config/feed/` — 플러그인 밖, git 밖)

```
~/.config/feed/
├── config/
│   ├── accounts.json                      # 실제 API 키/토큰
│   └── preferences.json                   # 사용자 설정
│
├── posts/                                 # 생성된 포스팅
│   └── {platform}/
│       └── {YYYY}/
│           └── {MM}/
│               └── {DD}/
│                   └── {HHMMSS}-{style}-{NNN}.md
│
└── analytics/
    ├── engagement.json                    # 플랫폼별 반응 데이터
    └── topics.json                        # 주제별 성과 추적
```

### 초기화 플로우

```
/feed 최초 실행
  │
  ├── ~/.config/feed/ 존재 확인
  │
  ├── 없으면:
  │   ├── ~/.config/feed/config/ 생성
  │   ├── accounts.json 빈 템플릿 생성
  │   ├── preferences.json 기본값 생성
  │   ├── ~/.config/feed/posts/ 생성
  │   ├── ~/.config/feed/analytics/ 생성 (빈 JSON)
  │   └── "Feed 초기 설정 완료" 메시지
  │
  └── 있으면:
      └── 기존 설정 로드 → 정상 진행
```

---

## 4. 스킬 설계

### 4.1 공통 — SKILL.md frontmatter (Claude Code 공식 스펙)

```yaml
---
name: feed
description: SNS 플랫폼별 맞춤 콘텐츠를 인터뷰 기반으로 생성하고 발행합니다
disable-model-invocation: true
argument-hint: <platform> <style> <topic>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion
---
```

| 필드 | 용도 |
|------|------|
| `name` | 스킬 이름. `/name`으로 호출 |
| `description` | 스킬 설명. 250자 이내 |
| `disable-model-invocation` | `true` → 사용자만 호출 가능 (자동 실행 방지) |
| `argument-hint` | 자동완성 시 인자 힌트 |
| `allowed-tools` | 허가 없이 사용 가능한 도구 |

### 4.2 스킬 목록

| 스킬 | 경로 | 용도 |
|------|------|------|
| `/feed` | `skills/feed/SKILL.md` | 메인. 인터뷰 → 생성 → 저장 → 발행 |
| `/feed-publish` | `skills/feed-publish/SKILL.md` | 미발행 포스트 발행 |
| `/feed-report` | `skills/feed-report/SKILL.md` | 기간별 통계 리포트 |
| `/feed-recommend` | `skills/feed-recommend/SKILL.md` | 주제 추천 |

### 4.3 `/feed` — 메인 스킬

#### 구조

```
skills/feed/
├── SKILL.md          # 라우터: 인자 파싱 → 어떤 파일 읽을지 분기
├── interview.md      # 인터뷰 플로우 상세 (Step 1~6)
└── reference.md      # 라우팅 가이드 (상황별 로드할 파일 매핑)
```

#### SKILL.md 역할 (라우터만)

```
1. $ARGUMENTS 파싱
2. ~/.config/feed/ 존재 확인 → 없으면 초기화
3. 인자 부족 시 → interview.md 읽고 인터뷰 진행
4. reference.md 읽고 → 상황에 맞는 파일 로드
5. 생성 → 저장 → 발행 여부 인터뷰
```

#### reference.md 라우팅 테이블

```
생성 시 로드:
  - rules/common.md (항상)
  - rules/scoring.md (항상)
  - rules/{language}.md (선택된 언어)
  - platforms/{platform}.md (선택된 플랫폼)
  - styles/{style}.md (선택된 스타일)

저장 시 로드:
  - flows/save.md

발행 시 로드:
  - flows/publish.md
```

#### 컨텍스트 로드 흐름

`/feed threads twist-poem 알람` 실행 시:

```
SKILL.md 로드 (자동)
  │
  ├── Read reference.md              ← 라우팅 가이드
  ├── Read rules/common.md           ← 공통 규칙
  ├── Read rules/scoring.md          ← 채점 기준
  ├── Read rules/ko.md               ← 한글 규칙
  ├── Read platforms/threads.md      ← Threads 프로필
  ├── Read styles/twist-poem.md      ← 반전시 스타일
  │
  ▼ 생성 + 채점
  │
  ├── Read flows/save.md             ← 저장 형식
  ├── Write ~/.config/feed/posts/...        ← 포스트 저장
  │
  ▼ 발행 선택 시
  │
  └── Read flows/publish.md          ← 발행 로직
```

#### 인자 파싱 규칙

```
/feed                              → 풀 인터뷰
/feed threads                      → 플랫폼 지정, 나머지 인터뷰
/feed threads twist-poem           → 플랫폼+스타일 지정, 주제만 인터뷰
/feed threads twist-poem 알람      → 전부 지정, 바로 생성
```

#### 인터뷰 플로우 (interview.md)

```
Step 1. 언어
  → [1] 한글  [2] English
  (preferences.json에 기본값 있으면 스킵)

Step 2. 플랫폼
  → Glob platforms/*.md로 사용 가능한 플랫폼 목록 조회
  → [1] Threads  [2] X  [3] Reddit  [4] Dev.to  [5] 전체
  (전체 선택 시 모든 플랫폼용으로 각각 생성)

Step 3. 스타일
  → 선택한 플랫폼 프로필에서 추천 스타일 확인
  → Glob styles/*.md로 사용 가능한 스타일 목록 조회
  → 해당 플랫폼 호환 스타일만 필터링해서 표시

Step 4. 주제
  → [1] 직접 입력  [2] 추천받기
  → 추천받기: ~/.config/feed/analytics/topics.json 기반 3개 제시 (데이터 없으면 트렌드 기반)
  → 직접 입력: 자유 텍스트

Step 5. 생성
  → 콘텐츠 생성 + 자체 평가
  → 점수 미달 시 자동 재생성 (최대 2회)
  → 결과 표시

Step 6. 발행
  → [1] 발행  [2] 나중에  [3] 수정 요청
  → 발행/나중에 → ~/.config/feed/posts/에 저장
  → 발행 선택 시 → flows/publish.md 로직 실행
  → 수정 요청 → 수정 지시 입력 → Step 5로
```

### 4.4 `/feed-publish` — 발행 스킬

```
/feed-publish                      → 미발행 목록 표시 → 선택 → 발행
/feed-publish today                → 오늘 미발행분만
/feed-publish all                  → 전체 미발행분
```

- `~/.config/feed/posts/`에서 `published: false`인 파일 검색
- 목록 표시 → 사용자 선택
- `flows/publish.md` 로직 실행

### 4.5 `/feed-report` — 리포트 스킬

```
/feed-report                       → 이번 주 요약
/feed-report month                 → 이번 달 요약
/feed-report threads               → 특정 플랫폼만
```

리포트 내용:
- 기간 내 생성/발행 수
- 플랫폼별 분포
- 스타일별 분포
- 반응 데이터 (`~/.config/feed/analytics/engagement.json` 기반)
- 가장 반응 좋았던 포스트 Top 3
- API 지원 플랫폼: 자동으로 반응 데이터 갱신
- API 미지원: 인터뷰로 수동 입력 가능

### 4.6 `/feed-recommend` — 추천 스킬

```
/feed-recommend                    → 인터뷰로 추천
/feed-recommend threads            → 특정 플랫폼용 추천
```

추천 로직:
1. `~/.config/feed/analytics/topics.json`에서 과거 반응 데이터 확인
2. 잘 먹힌 주제 카테고리 분석
3. 비슷하지만 아직 안 다룬 주제 3~5개 제시
4. 데이터 없으면: 플랫폼 특성 + 현재 트렌드 기반 추천

---

## 5. 플랫폼 프로필

### 파일 위치

`platforms/{platform}.md`

### 포함 내용

```markdown
# {Platform Name}

## Spec
- 글자 수 제한
- 지원 포맷 (텍스트, 이미지, 링크 등)
- 해시태그 사용 여부

## Algorithm
- 반응 가중치 (댓글 > 리포스트 > 좋아요 등)
- 노출 방식 (팔로워 기반, 추천 기반 등)

## What Works
- 톤 (감성, 정보, 유머 등)
- 길이
- 구조

## What Doesn't Work
- 피해야 할 패턴

## Recommended Styles
- 이 플랫폼에서 효과적인 스타일 파일명 목록

## API Publishing
- 가능 여부: true/false
- 인증 방식
- 필요한 키 필드 (accounts.json 참조)
```

### 플랫폼별 API 발행 가능 여부

| 플랫폼 | 파일명 | API 발행 | 비고 |
|--------|--------|:---:|------|
| Threads | `threads.md` | △ | Meta Threads API — 제한적 |
| X (Twitter) | `x.md` | O | Twitter API v2 |
| Reddit | `reddit.md` | O | Reddit API (PRAW) |
| Dev.to | `devto.md` | O | Dev.to REST API |

---

## 6. 스타일 템플릿

### 파일 위치

`styles/{style}.md`

### 포함 내용

```markdown
# {Style Name}

## Structure
- 줄 수, 호흡, 반전 위치 등

## Rules
- 스타일별 고유 규칙

## Tone
- 감정, 말투

## Good Examples
- 레퍼런스 (이 수준을 목표로)

## Bad Examples
- AI 티 나는 예시 + 왜 나쁜지

## Scoring
- 스타일별 채점 항목과 가중치 (rules/scoring.md의 기본 기준 위에 추가)

## Compatible Platforms
- 이 스타일이 잘 먹히는 플랫폼 파일명 목록
```

### 스타일 매핑 (v1)

| 한글 이름 | 파일명 | 설명 | 호환 플랫폼 |
|----------|--------|------|------------|
| 반전시 | `twist-poem.md` | 감성 독백 → 마지막 줄 화자 반전 | threads, x |
| 논쟁 | `debate.md` | 양자택일 도발. 댓글 유도 | threads, x, reddit |
| 공감 | `relatable.md` | "나만 그런 줄 알았는데" 형 | threads, x |
| 한줄반전 | `one-liner.md` | 2줄. 기대 → 반전 | x, threads |
| 명언패러디 | `quote-parody.md` | 유명 격언 비틀기 | threads, x, reddit |
| 자조 | `self-deprecation.md` | 숫자 + 자기비하 유머 | threads, x |

---

## 7. 규칙 (rules/)

### 파일 구조

| 파일 | 내용 | 로드 시점 |
|------|------|----------|
| `common.md` | AI 티 방지, 구조 변주, 금지 패턴, 입말 강제 | 생성 시 항상 |
| `scoring.md` | 채점 항목, 통과 기준, 재생성 조건 | 생성 시 항상 |
| `ko.md` | 한글 작성 규칙 (반말, 축약, 구어체) | 언어=ko 시 |
| `en.md` | 영어 작성 규칙 (casual, contractions) | 언어=en 시 |

### 확장 시

새 언어 추가: `rules/ja.md` 추가 → 기존 파일 수정 불필요
새 규칙 카테고리: `rules/brand-voice.md` 추가 → reference.md에 로드 조건 추가

---

## 8. 플로우 (flows/)

### 파일 구조

| 파일 | 내용 | 로드 시점 |
|------|------|----------|
| `save.md` | frontmatter 스펙, 파일 경로 규칙, 저장 프로세스 | 생성 완료 후 |
| `publish.md` | API/클립보드 분기, 실패 처리, 상태 업데이트 | 발행 선택 시 |

### 확장 시

예약 발행: `flows/schedule.md` 추가
멀티 플랫폼 동시 발행: `flows/multi-publish.md` 추가

---

## 9. 발행 플로우

```
사용자: [발행] 선택
         │
         ▼
    ~/.config/feed/posts/에 저장 (published: false)
         │
         ▼
    ~/.config/feed/config/accounts.json에서
    해당 플랫폼 enabled 확인
         │
    ┌────┴────┐
    │         │
  enabled    disabled
  = true     = false
    │         │
    ▼         ▼
 인터뷰:    "클립보드에 복사했습니다"
 "자동      published: true
  발행       publish_method: clipboard
  할까요?"
    │
  ┌─┴─┐
  │   │
 예  아니오
  │   │
  ▼   ▼
 API  클립보드
 발행  복사
  │   │
  ▼   ▼
 published: true
 publish_method: api | clipboard
```

### 발행 실패 처리

- API 발행 실패 → 에러 표시 → 자동으로 클립보드 복사 폴백
- 파일에 `publish_method: failed`, `error: {메시지}` 기록

---

## 10. 포스트 저장 형식

### 파일 경로

```
~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{HHMMSS}-{style}-{NNN}.md
```

예시:

```
~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
~/.config/feed/posts/x/2026/04/09/201500-one-liner-001.md
```

### 파일 내용 (frontmatter + 본문)

```markdown
---
platform: threads
style: twist-poem
topic: alarm
language: ko
created_at: 2026-04-09T14:30:22
published: true
published_at: 2026-04-09T14:31:05
publish_method: api
publish_url: https://threads.net/...
error: null
score:
  twist: 9
  relatability: 8
  humor: 8
  brevity: 9
  human-feel: 8
engagement:
  likes: null
  comments: null
  reposts: null
  last_checked: null
---

안 보면 될 걸
자꾸 확인하고
달라진 거 없으면서
또 확인하고

관심 없는 척하다가
결국 먼저 연락하는 건
항상 나였다

- 조회수
```

---

## 11. 사용자 데이터

### ~/.config/feed/config/preferences.json

```json
{
  "default_language": "ko",
  "default_platforms": {
    "threads": {
      "default_style": "twist-poem",
      "auto_publish": false
    },
    "x": {
      "default_style": "one-liner",
      "auto_publish": false
    }
  },
  "interview": {
    "skip_language": true,
    "skip_style": false
  }
}
```

### ~/.config/feed/config/accounts.json

```json
{
  "x": {
    "api_key": "",
    "api_secret": "",
    "access_token": "",
    "access_secret": "",
    "enabled": false
  },
  "reddit": {
    "client_id": "",
    "client_secret": "",
    "username": "",
    "password": "",
    "enabled": false
  },
  "devto": {
    "api_key": "",
    "enabled": false
  },
  "threads": {
    "access_token": "",
    "enabled": false
  }
}
```

### ~/.config/feed/analytics/engagement.json

```json
{
  "entries": [
    {
      "post_path": "posts/threads/2026/04/09/143022-twist-poem-001.md",
      "platform": "threads",
      "checked_at": "2026-04-10T09:00:00",
      "likes": 234,
      "comments": 45,
      "reposts": 12
    }
  ]
}
```

### ~/.config/feed/analytics/topics.json

```json
{
  "topics": [
    {
      "name": "alarm",
      "display_name": "알람",
      "category": "daily-habits",
      "used_count": 3,
      "avg_engagement": {
        "likes": 150,
        "comments": 30,
        "reposts": 8
      },
      "platforms": ["threads", "x"],
      "styles": ["twist-poem", "one-liner"]
    }
  ]
}
```

### 반응 데이터 수집

- API 지원 플랫폼: `/feed-report` 실행 시 자동으로 API 호출 → engagement.json 업데이트
- API 미지원: `/feed-report` 실행 시 인터뷰로 수동 입력 가능

---

## 12. 플러그인 설정 파일

### .claude-plugin/plugin.json

```json
{
  "name": "feed",
  "version": "0.1.0",
  "description": "SNS content generator & publisher — create platform-optimized posts via interview-driven workflow",
  "author": {
    "name": "",
    "url": ""
  },
  "repository": "",
  "license": "MIT",
  "keywords": ["sns", "social-media", "content", "threads", "twitter"],
  "skills": "./skills/"
}
```

### settings.json

```json
{
  "permissions": {
    "allow": [
      "Read(*)",
      "Write(~/.config/feed/*)",
      "Glob(*)",
      "Bash(pbcopy)"
    ]
  }
}
```

---

## 13. 크론 / 스케줄

Claude Code의 `/schedule` 기능 활용.

| 스케줄 | 커맨드 | 설명 |
|--------|--------|------|
| 매일 오전 9시 | `/feed-recommend` | 오늘의 주제 추천 |
| 매주 월요일 | `/feed-report week` | 주간 리포트 |
| 매월 1일 | `/feed-report month` | 월간 리포트 |

---

## 14. 확장성 매트릭스

| 확장 시나리오 | 추가할 파일 | 기존 파일 수정 |
|-------------|-----------|:---:|
| 새 플랫폼 (LinkedIn) | `platforms/linkedin.md` | X |
| 새 스타일 (정보공유) | `styles/informative.md` | X |
| 새 언어 (일본어) | `rules/ja.md` | X |
| 새 플로우 (예약발행) | `flows/schedule.md` | X |
| 새 스킬 (A/B 테스트) | `skills/feed-ab-test/SKILL.md` | X |
| 채점 기준 변경 | `rules/scoring.md` 수정 | O (해당 파일만) |
| 사용자 커스텀 스타일 | `~/.config/feed/styles/custom.md` | X |

---

## 15. 구현 순서

### Phase 1: 핵심 (Threads 중심)

- [ ] `.claude-plugin/plugin.json` 매니페스트
- [ ] `settings.json` 기본 권한
- [ ] `CLAUDE.md` 플러그인 개발 가이드
- [ ] `rules/common.md` AI 티 방지 규칙
- [ ] `rules/scoring.md` 채점 기준
- [ ] `rules/ko.md` 한글 규칙
- [ ] `platforms/threads.md` Threads 프로필
- [ ] `styles/twist-poem.md` 반전시
- [ ] `styles/debate.md` 논쟁
- [ ] `styles/relatable.md` 공감
- [ ] `styles/one-liner.md` 한줄반전
- [ ] `styles/quote-parody.md` 명언패러디
- [ ] `styles/self-deprecation.md` 자조
- [ ] `flows/save.md` 저장 형식
- [ ] `flows/publish.md` 발행 로직
- [ ] `skills/feed/SKILL.md` 메인 라우터
- [ ] `skills/feed/interview.md` 인터뷰 플로우
- [ ] `skills/feed/reference.md` 라우팅 가이드
- [ ] `~/.config/feed/` 초기화 로직

### Phase 2: 발행 & 분석

- [ ] `skills/feed-publish/SKILL.md` 발행 스킬
- [ ] `skills/feed-report/SKILL.md` 리포트 스킬
- [ ] `skills/feed-recommend/SKILL.md` 추천 스킬
- [ ] 반응 수동 입력 인터뷰
- [ ] `README.md` 사용자 가이드

### Phase 3: 플랫폼 확장

- [ ] `platforms/x.md` X 프로필
- [ ] `platforms/reddit.md` Reddit 프로필
- [ ] `platforms/devto.md` Dev.to 프로필
- [ ] `rules/en.md` 영어 규칙
- [ ] API 발행 연동 (X, Reddit, Dev.to)

### Phase 4: 고도화

- [ ] 크론 스케줄 설정 가이드
- [ ] 반응 데이터 기반 추천 고도화
- [ ] 사용자 커스텀 스타일 지원 (`~/.config/feed/styles/`)
- [ ] `LICENSE`, `CHANGELOG.md`
- [ ] 플러그인 배포 (GitHub)
