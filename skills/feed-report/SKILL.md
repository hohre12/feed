---
name: feed-report
description: 기간별 포스팅 통계와 반응 리포트를 생성합니다
disable-model-invocation: true
argument-hint: <week|month|platform>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion
---

# /feed-report

기간별 포스팅 통계와 반응 데이터를 분석하여 리포트를 생성한다.

---

## Step 0. 사전 확인

`~/.config/feed/` 디렉토리가 존재하는지 확인한다.

```
Bash: test -d ~/.config/feed
```

- **존재하지 않으면** → 메시지 출력 후 종료:
  ```
  먼저 /feed로 포스트를 생성해주세요.
  ```
- **존재하면** → Step 1로 진행.

---

## Step 1. 인자 파싱

`$ARGUMENTS`를 공백으로 분리하여 **기간**과 **플랫폼 필터**를 결정한다.

### 기간 키워드

| 키워드 | 의미 |
|--------|------|
| `week` | 이번 주 (월요일 ~ 오늘) |
| `month` | 이번 달 (1일 ~ 오늘) |
| *(없음)* | 기본값 = `week` |

### 플랫폼 키워드

| 키워드 | 플랫폼 |
|--------|--------|
| `threads` | threads |
| `x` | x |
| `reddit` | reddit |
| `devto` | devto |
| *(없음)* | 전체 플랫폼 |

### 조합 예시

```
/feed-report                    → 이번 주, 전체 플랫폼
/feed-report week               → 이번 주, 전체 플랫폼
/feed-report month              → 이번 달, 전체 플랫폼
/feed-report threads            → 이번 주, Threads만
/feed-report month threads      → 이번 달, Threads만
/feed-report x week             → 이번 주, X만 (순서 무관)
```

### 파싱 로직

1. `$ARGUMENTS`를 공백으로 분리하여 토큰 배열을 만든다.
2. 각 토큰을 순회하며:
   - `week` 또는 `month` → 기간으로 설정
   - `threads`, `x`, `reddit`, `devto` → 플랫폼 필터로 설정
   - 인식 불가 → 무시
3. 기간이 설정되지 않았으면 `week`을 기본값으로 사용한다.
4. 플랫폼 필터가 설정되지 않았으면 전체 플랫폼을 대상으로 한다.

---

## Step 2. 기간 범위 계산

오늘 날짜를 기준으로 시작일과 종료일을 계산한다.

```bash
# 오늘 날짜
TODAY=$(date +%Y-%m-%d)
```

### week인 경우

이번 주 월요일부터 오늘까지.

```bash
# macOS
WEEKDAY=$(date +%u)  # 1=월 ~ 7=일
DAYS_SINCE_MONDAY=$((WEEKDAY - 1))
START_DATE=$(date -v-${DAYS_SINCE_MONDAY}d +%Y-%m-%d)
END_DATE=$TODAY
```

### month인 경우

이번 달 1일부터 오늘까지.

```bash
START_DATE=$(date +%Y-%m-01)
END_DATE=$TODAY
```

---

## Step 3. 포스트 스캔

`~/.config/feed/posts/` 아래에서 기간 내 포스트 파일들을 수집한다.

### 스캔 대상 플랫폼

- 플랫폼 필터가 있으면 → 해당 플랫폼 디렉토리만
- 없으면 → `~/.config/feed/posts/` 아래 모든 플랫폼 디렉토리

### 날짜 기반 디렉토리 탐색

포스트는 `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/` 구조로 저장된다.
기간 범위에 해당하는 YYYY/MM/DD 조합을 계산하여 해당 디렉토리만 탐색한다.

```
# 플랫폼 목록 확인
Glob ~/.config/feed/posts/*/

# 기간 내 포스트 파일 수집 (각 플랫폼별)
# 예: 2026-04-07 ~ 2026-04-09 범위라면
Glob ~/.config/feed/posts/{platform}/2026/04/07/*.md
Glob ~/.config/feed/posts/{platform}/2026/04/08/*.md
Glob ~/.config/feed/posts/{platform}/2026/04/09/*.md
```

### 각 포스트 파일에서 frontmatter 읽기

수집된 각 `.md` 파일의 frontmatter를 Read로 읽어 아래 정보를 추출한다:

| 필드 | 용도 |
|------|------|
| `platform` | 플랫폼별 집계 |
| `style` | 스타일별 집계 |
| `topic` | Top 3 표시, topics.json 업데이트 |
| `published` | 발행 여부 집계 |
| `created_at` | 기간 내 확인 (디렉토리 구조와 이중 검증) |
| `score.*` | 평균 점수 계산, Top 3 순위 |
| `engagement.*` | 반응 데이터 집계, Top 3 순위 |

### 포스트가 없는 경우

스캔 결과 파일이 0개이면 메시지 출력 후 종료:

```
해당 기간에 포스트가 없습니다. (2026-04-07 ~ 2026-04-09)
```

---

## Step 4. 데이터 집계

수집된 포스트들의 frontmatter 데이터를 집계한다.

### 4.1 전체 요약

- `total_created`: 전체 포스트 수
- `total_published`: `published: true`인 포스트 수
- `start_date`: 기간 시작일
- `end_date`: 기간 종료일

### 4.2 플랫폼별 집계

각 플랫폼에 대해:

- `count`: 생성 수
- `published`: 발행 수
- `avg_score`: score 항목들의 전체 평균 (소수점 첫째 자리)

### 4.3 스타일별 집계

각 스타일에 대해:

- `count`: 생성 수
- `avg_score`: score 항목들의 전체 평균 (소수점 첫째 자리)

### 4.4 Top 3 포스트

순위 기준:
1. engagement 데이터가 있는 포스트 → `likes + comments * 2 + reposts * 1.5` 가중 합산으로 순위
2. engagement 데이터가 없는 포스트 → score 항목들의 평균으로 순위
3. engagement 있는 포스트가 우선 (engagement 데이터가 있으면 score보다 우선)

각 포스트에서 표시할 정보:
- 플랫폼, 스타일, 주제, score 평균, engagement (있으면)

### 4.5 반응 데이터 요약

`~/.config/feed/analytics/engagement.json`을 Read로 읽는다.

- 기간 내 entries 필터링 (`checked_at`이 기간 범위 내)
- 총 likes, comments, reposts 합산
- 가장 반응 좋았던 포스트 (위 4.4와 동일한 가중 합산 기준)

engagement.json에 데이터가 없거나 기간 내 데이터가 없으면 이 섹션은 생략한다.

---

## Step 5. 리포트 출력

아래 형식으로 리포트를 출력한다. **모든 텍스트는 한국어로 작성한다.**

```markdown
# Feed 리포트

---

### 📊 기간 요약

- **기간**: {start_date} ~ {end_date} ({period_label})
- **총 생성**: {total_created}개
- **총 발행**: {total_published}개

---

### 📱 플랫폼별

| 플랫폼 | 생성 | 발행 | 평균 점수 |
|--------|:----:|:----:|:---------:|
| threads | 5 | 3 | 8.2 |
| x | 2 | 1 | 7.8 |

---

### 🎨 스타일별

| 스타일 | 생성 | 평균 점수 |
|--------|:----:|:---------:|
| twist-poem | 3 | 8.5 |
| debate | 2 | 7.6 |
| one-liner | 2 | 8.0 |

---

### 🏆 Top 3 포스트

**1. {topic}**
- 플랫폼: {platform} / 스타일: {style}
- 점수: {avg_score}
- 반응: 👍 {likes} 💬 {comments} 🔄 {reposts}

**2. {topic}**
- 플랫폼: {platform} / 스타일: {style}
- 점수: {avg_score}

**3. {topic}**
- 플랫폼: {platform} / 스타일: {style}
- 점수: {avg_score}

---

### 📈 반응 데이터

- **총 좋아요**: {total_likes}
- **총 댓글**: {total_comments}
- **총 리포스트**: {total_reposts}
- **최고 반응 포스트**: {best_post_topic} ({platform}, {style}) — 👍 {likes} 💬 {comments} 🔄 {reposts}
```

### period_label 값

| 기간 | 라벨 |
|------|------|
| week | 이번 주 |
| month | 이번 달 |

### 플랫폼 필터 적용 시

리포트 제목에 플랫폼을 명시한다:

```markdown
# Feed 리포트 — Threads
```

플랫폼별 테이블은 해당 플랫폼 1행만 표시한다.

---

## Step 6. 반응 데이터 업데이트 (Engagement Update)

리포트 출력 후, 기간 내 발행된 포스트(`published: true`)에 대해 반응 데이터 업데이트를 진행한다.

### 6.1 accounts.json 확인

```
Read ~/.config/feed/config/accounts.json
```

각 플랫폼의 `enabled` 상태를 확인한다.

### 6.2 플랫폼별 분기

#### API 지원 + enabled 플랫폼

해당 플랫폼의 발행된 포스트에 대해:

```
{platform}의 반응 데이터를 API로 가져올까요? [1] 예 [2] 건너뛰기
```

- **[1] 예** → API 호출하여 engagement 데이터 수집 (현재는 placeholder — 실제 API 연동은 Phase 3)
- **[2] 건너뛰기** → 다음 플랫폼으로

#### API 미지원 또는 disabled 플랫폼

해당 플랫폼의 발행된 포스트 각각에 대해 수동 입력을 제안한다.

포스트 식별자는 파일명에서 추출한다:
`{HHMMSS}-{style}-{NNN}.md` → `{platform} {style}-{NNN}`

```
threads twist-poem-001의 반응을 입력하시겠습니까? [1] 예 [2] 건너뛰기
```

- **[2] 건너뛰기** → 다음 포스트로
- **[1] 예** → 순서대로 질문:

```
좋아요 수:
```
→ 사용자 입력 (숫자)

```
댓글 수:
```
→ 사용자 입력 (숫자)

```
리포스트 수:
```
→ 사용자 입력 (숫자)

### 6.3 데이터 저장

반응 데이터를 입력받았으면 두 곳을 업데이트한다.

#### A. 포스트 frontmatter 업데이트

해당 포스트 파일의 frontmatter에서 engagement 필드를 업데이트한다:

```yaml
engagement:
  likes: {입력값}
  comments: {입력값}
  reposts: {입력값}
  last_checked: {현재시각 ISO 8601}
```

Read로 파일 전체를 읽고, engagement 블록만 교체하여 Write로 저장한다.

#### B. engagement.json 업데이트

```
Read ~/.config/feed/analytics/engagement.json
```

`entries` 배열에 새 항목을 추가하거나, 동일 `post_path`의 기존 항목을 갱신한다:

```json
{
  "post_path": "posts/{platform}/{YYYY}/{MM}/{DD}/{filename}",
  "platform": "{platform}",
  "checked_at": "{현재시각 ISO 8601}",
  "likes": {입력값},
  "comments": {입력값},
  "reposts": {입력값}
}
```

- `post_path`는 `~/.config/feed/` 이후의 상대 경로를 사용한다.
- 동일 `post_path`가 이미 있으면 해당 항목을 덮어쓴다 (최신 데이터 유지).
- Write로 engagement.json을 저장한다.

---

## Step 7. topics.json 업데이트

모든 데이터 수집이 끝난 후, `~/.config/feed/analytics/topics.json`을 업데이트한다.

```
Read ~/.config/feed/analytics/topics.json
```

### 업데이트 로직

기간 내 수집된 모든 포스트의 topic을 순회하며:

1. **기존 topic이 있으면** → 해당 항목 업데이트:
   - `used_count`: 전체 포스트에서 해당 topic이 사용된 총 횟수로 재계산
   - `avg_engagement`: engagement 데이터가 있는 포스트들의 평균으로 재계산
   - `platforms`: 사용된 플랫폼 목록 (중복 제거)
   - `styles`: 사용된 스타일 목록 (중복 제거)

2. **기존 topic이 없으면** → 새 항목 추가:
   ```json
   {
     "name": "{topic}",
     "display_name": "{topic}",
     "category": "uncategorized",
     "used_count": 1,
     "avg_engagement": {
       "likes": 0,
       "comments": 0,
       "reposts": 0
     },
     "platforms": ["{platform}"],
     "styles": ["{style}"]
   }
   ```

3. Write로 topics.json을 저장한다.

### 업데이트 완료 메시지

```
topics.json 업데이트 완료 — {업데이트된 토픽 수}개 토픽 반영
```

---

## 엣지 케이스 정리

| 상황 | 처리 |
|------|------|
| `~/.config/feed/` 미존재 | "먼저 /feed로 포스트를 생성해주세요" 출력 후 종료 |
| 기간 내 포스트 0개 | "해당 기간에 포스트가 없습니다. ({start} ~ {end})" 출력 후 종료 |
| engagement.json 미존재 또는 빈 entries | 반응 데이터 섹션 생략 |
| topics.json 미존재 | 빈 `{ "topics": [] }`로 새로 생성 후 업데이트 |
| score 필드가 비어있는 포스트 | 평균 계산에서 제외 |
| engagement 필드가 모두 null인 포스트 | engagement 없는 포스트로 분류 (score 기반 순위) |
| 사용자가 숫자가 아닌 값 입력 | "숫자를 입력해주세요"로 재요청 |
| accounts.json 미존재 | 모든 플랫폼을 disabled로 간주 → 수동 입력만 제안 |
