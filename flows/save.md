# 포스트 저장 플로우

콘텐츠 생성이 완료되면 이 파일의 규칙에 따라 저장한다.
모든 포스트는 `~/.config/feed/posts/` 아래에 frontmatter + 본문 형식으로 저장된다.

---

## 파일 경로 규칙

```
~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{HHMMSS}-{style}-{NNN}.md
```

### 각 요소

| 요소 | 설명 | 예시 |
|------|------|------|
| `platform` | 대상 플랫폼 이름. 소문자. | `threads`, `x`, `reddit`, `devto` |
| `YYYY` | 생성 연도 (4자리) | `2026` |
| `MM` | 생성 월 (2자리, 제로패딩) | `04` |
| `DD` | 생성 일 (2자리, 제로패딩) | `09` |
| `HHMMSS` | 생성 시각 (24시간제, 구분자 없음) | `143022` |
| `style` | 스타일 파일명에서 `.md`를 뺀 것 | `twist-poem`, `debate`, `one-liner` |
| `NNN` | 같은 날짜 + 같은 스타일의 순번 (3자리, 제로패딩) | `001`, `002`, `003` |

### 예시

```
~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
~/.config/feed/posts/x/2026/04/09/201500-one-liner-001.md
~/.config/feed/posts/reddit/2026/04/10/093000-debate-002.md
```

---

## Frontmatter 스펙

모든 포스트 파일은 YAML frontmatter로 시작한다.

```yaml
---
platform: threads
style: twist-poem
topic: alarm
audience: developer
language: ko
created_at: 2026-04-09T14:30:22
published: false
published_at: null
publish_method: null
publish_url: null
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
```

### 필드 설명

#### 기본 정보

| 필드 | 타입 | 설명 |
|------|------|------|
| `platform` | string | 대상 플랫폼. `threads`, `x`, `reddit`, `devto` 중 하나. |
| `style` | string | 사용된 스타일. 스타일 파일명에서 `.md`를 뺀 값. (예: `twist-poem`, `debate`) |
| `topic` | string | 콘텐츠 주제. 인터뷰에서 입력받은 주제 키워드. |
| `language` | string | 작성 언어. `ko` 또는 `en`. |
| `created_at` | datetime | 생성 시각. ISO 8601 형식 (`YYYY-MM-DDTHH:MM:SS`). 초 단위까지. |

#### 발행 상태

| 필드 | 타입 | 설명 |
|------|------|------|
| `published` | boolean | 발행 여부. 저장 시 `false`, 발행 완료 시 `true`. |
| `published_at` | datetime \| null | 발행 시각. 미발행이면 `null`. 발행 시 ISO 8601 형식으로 기록. |
| `publish_method` | string \| null | 발행 방식. `null`(미발행), `api`, `clipboard`, `failed` 중 하나. |
| `publish_url` | string \| null | 발행된 포스트 URL. API 발행 시 플랫폼이 반환한 URL. 클립보드 복사 시 `null`. |
| `error` | string \| null | 발행 실패 시 에러 메시지. 정상이면 `null`. |

#### 자체 평가 점수

| 필드 | 타입 | 설명 |
|------|------|------|
| `score.twist` | integer (1-10) | 반전력. 마지막 줄이 앞 맥락을 뒤집는 정도. |
| `score.relatability` | integer (1-10) | 공감도. 읽는 사람이 "나도 그래"라고 느끼는 정도. |
| `score.humor` | integer (1-10) | 웃음. 실제로 웃게 만드는 정도. |
| `score.brevity` | integer (1-10) | 간결함. 군더더기 없이 타이트한 정도. |
| `score.human-feel` | integer (1-10) | 사람 느낌. AI가 쓴 티가 안 나는 정도. 가장 중요한 항목. |

#### 반응 데이터

| 필드 | 타입 | 설명 |
|------|------|------|
| `engagement.likes` | integer \| null | 좋아요 수. 수집 전이면 `null`. |
| `engagement.comments` | integer \| null | 댓글 수. 수집 전이면 `null`. |
| `engagement.reposts` | integer \| null | 리포스트/공유 수. 수집 전이면 `null`. |
| `engagement.last_checked` | datetime \| null | 반응 데이터 마지막 수집 시각. ISO 8601 형식. |

---

## 저장 프로세스

### Step 1. 디렉토리 생성

저장 경로의 디렉토리가 존재하지 않으면 생성한다.

```bash
mkdir -p ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/
```

예시:
```bash
mkdir -p ~/.config/feed/posts/threads/2026/04/09/
```

### Step 2. 순번(NNN) 결정

같은 날짜 디렉토리(`~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/`) 안에서
같은 스타일의 기존 파일 수를 확인하여 다음 순번을 부여한다.

```
기존 파일:
  143022-twist-poem-001.md
  150000-twist-poem-002.md

→ 다음 순번: 003
```

- Glob 패턴: `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/*-{style}-*.md`
- 매칭된 파일이 없으면 `001`부터 시작
- 매칭된 파일 중 가장 큰 NNN + 1

### Step 3. 파일 작성

frontmatter + 빈 줄 + 콘텐츠 본문을 하나의 파일로 작성한다.

```markdown
---
platform: threads
style: twist-poem
topic: alarm
language: ko
created_at: 2026-04-09T14:30:22
published: false
published_at: null
publish_method: null
publish_url: null
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

**주의:** frontmatter 닫는 `---` 바로 다음에 빈 줄 하나를 넣고, 그 아래에 콘텐츠 본문을 작성한다.

### Step 4. 저장 확인

저장 완료 후 사용자에게 파일 경로를 표시한다.

```
저장 완료: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```
