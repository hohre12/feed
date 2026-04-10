---
name: feed-recommend
description: 과거 반응 데이터와 플랫폼 특성을 기반으로 주제를 추천합니다
disable-model-invocation: true
argument-hint: <platform>
allowed-tools: Read Write Glob Grep AskUserQuestion
---

# /feed-recommend — 주제 추천

과거 반응 데이터와 플랫폼 특성을 기반으로 SNS 포스팅 주제를 추천한다.

---

## 1. 인자 파싱

`$ARGUMENTS`를 분석한다.

| 입력 | 동작 |
|------|------|
| `/feed-recommend threads` | `platform = threads` → 해당 플랫폼용 추천 |
| `/feed-recommend` | 플랫폼 미지정 → Step 2에서 질문 |

### 파싱 규칙

- 첫 번째 인자가 있으면 `platform`으로 취급한다.
- `platform`이 `platforms/` 디렉토리에 존재하는지 확인한다.
  - **존재하면**: 해당 플랫폼으로 진행
  - **존재하지 않으면**: 사용 가능한 플랫폼 목록을 보여주고 다시 질문한다

---

## 2. 플랫폼 선택 (인자 없을 때)

`platform`이 비어 있으면 AskUserQuestion으로 질문한다.

먼저 `Glob platforms/*.md`로 사용 가능한 플랫폼 목록을 조회한다.

```
어떤 플랫폼용 주제를 추천할까요?

[1] Threads
[2] X
[3] Reddit
...
```

사용자가 선택하면 해당 `platform` 값을 설정한다.

---

## 3. 전제 조건 확인

### ~/.config/feed/ 디렉토리 존재 확인

`Glob ~/.config/feed/` 로 확인한다.

- **없으면**: 아래 메시지를 출력하고 종료한다.

```
아직 생성된 포스트가 없습니다.
먼저 /feed로 포스트를 생성해주세요.
```

- **있으면**: 다음 단계로 진행한다.

### 플랫폼 유효성 확인

`platforms/{platform}.md` 파일이 존재하는지 확인한다.

- **없으면**: 사용 가능한 플랫폼 목록을 보여주고 AskUserQuestion으로 다시 질문한다.

```
'{platform}' 플랫폼을 찾을 수 없습니다.

사용 가능한 플랫폼:
[1] Threads
[2] X
...

어떤 플랫폼을 선택할까요?
```

---

## 4. 데이터 로드

아래 파일들을 순서대로 읽는다.

### 4.1 과거 주제 성과

```
Read ~/.config/feed/analytics/topics.json
```

`topics.json` 구조:
```json
{
  "topics": [
    {
      "name": "alarm",
      "display_name": "알람",
      "category": "daily-habits",
      "used_count": 3,
      "avg_engagement": { "likes": 150, "comments": 30, "reposts": 8 },
      "platforms": ["threads", "x"],
      "styles": ["twist-poem", "one-liner"]
    }
  ]
}
```

### 4.2 반응 데이터

```
Read ~/.config/feed/analytics/engagement.json
```

`engagement.json` 구조:
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

### 4.3 플랫폼 프로필

```
Read platforms/{platform}.md
```

여기서 확인할 것:
- **What Works**: 이 플랫폼에서 잘 먹히는 톤/구조
- **Recommended Styles**: 호환되는 스타일 목록
- **Algorithm**: engagement 가중치 (댓글 > 리포스트 > 좋아요 등)

### 4.4 기존 포스트 목록

```
Glob ~/.config/feed/posts/{platform}/**/*.md
```

이미 다룬 주제 목록을 추출한다. 각 포스트의 frontmatter에서 `topic` 필드를 읽는다.

---

## 5. 추천 로직

### 5.1 데이터가 있는 경우 (topics.json에 entries가 있음)

**분석 단계:**

1. **고성과 카테고리 파악**: `avg_engagement`를 기준으로 어떤 `category`가 좋은 반응을 얻었는지 분석한다.
   - 플랫폼 알고리즘 가중치를 적용한다. 예: Threads는 `comments` 가중치가 가장 높다.
   - 가중 점수 = `(comments * 3) + (reposts * 2) + (likes * 1)` (플랫폼별 조정)
2. **스타일-주제 패턴 파악**: 어떤 `style` + `category` 조합이 높은 engagement를 보였는지 확인한다.
3. **이미 다룬 주제 제외**: Step 4.4에서 수집한 기존 포스트의 `topic` 값을 제외 목록으로 사용한다.

**추천 기준:**

- 고성과 카테고리와 **같거나 인접한** 카테고리에서 새 주제를 선택한다
- 이미 다룬 주제와 **중복되지 않는** 것만 추천한다
- 해당 플랫폼의 **특성에 맞는** 주제를 우선한다
- 각 추천에 **가장 잘 어울리는 스타일**을 함께 제시한다

**카테고리 인접 맵:**

| 카테고리 | 인접 카테고리 |
|---------|-------------|
| `daily-habits` (일상/습관) | `relationships`, `self-improvement`, `work-life` |
| `relationships` (관계) | `daily-habits`, `emotions`, `communication` |
| `self-improvement` (자기개발) | `work-life`, `daily-habits`, `motivation` |
| `work-life` (직장) | `self-improvement`, `daily-habits`, `money` |
| `seasonal` (계절/시의성) | 모든 카테고리와 인접 |
| `emotions` (감정) | `relationships`, `daily-habits`, `seasonal` |
| `communication` (소통) | `relationships`, `work-life`, `emotions` |
| `motivation` (동기부여) | `self-improvement`, `emotions`, `seasonal` |
| `money` (돈) | `work-life`, `daily-habits`, `self-improvement` |

### 5.2 데이터가 없는 경우 (topics.json이 비어 있거나 없음)

플랫폼 프로필의 What Works 섹션을 기반으로 추천한다.

**기본 카테고리 풀:**

| 카테고리 | 한글명 | 예시 주제 |
|---------|-------|----------|
| `daily-habits` | 일상/습관 | 알람, 커피, 출근루틴, 잠버릇, 샤워 |
| `relationships` | 관계 | 읽씹, 연락, 썸, 친구, 가족 |
| `self-improvement` | 자기개발 | 다이어트, 운동, 독서, 새해목표, 루틴 |
| `work-life` | 직장 | 월요일, 야근, 회의, 퇴근, 점심 |
| `seasonal` | 계절/시의성 | 현재 계절, 시기에 맞는 주제 |

**플랫폼별 톤 매칭:**

- **Threads**: 감성/공감/자조 → `relationships`, `emotions`, `daily-habits` 우선
- **X**: 위트/반전/한줄 → `work-life`, `daily-habits`, `communication` 우선
- **Reddit**: 정보/논쟁/깊이 → `self-improvement`, `work-life`, `money` 우선
- **Dev.to**: 개발/기술/경험 → `work-life`, `self-improvement` 우선

현재 날짜를 참고하여 **계절/시의성** 주제를 최소 1개 포함한다.

---

## 6. 추천 결과 출력

3~5개의 주제를 번호 목록으로 출력한다.

### 출력 형식

```
주제 추천 ({Platform})

[1] {주제명} — {추천 스타일}
    → {이 주제가 좋은 이유 1문장}

[2] {주제명} — {추천 스타일}
    → {이 주제가 좋은 이유 1문장}

[3] {주제명} — {추천 스타일}
    → {이 주제가 좋은 이유 1문장}

[4] {주제명} — {추천 스타일}
    → {이 주제가 좋은 이유 1문장}

[5] {주제명} — {추천 스타일}
    → {이 주제가 좋은 이유 1문장}
```

### 출력 예시

```
주제 추천 (Threads)

[1] 알림 — twist-poem
    → 읽씹/알림 관련 공감대가 높음. 반전시와 궁합 좋음.

[2] 월요일 — self-deprecation
    → 월요일 출근 자조는 시기 불문 공감. 숫자 활용 가능.

[3] 다이어트 — one-liner
    → 만년 다이어트 소재는 계절 무관. 한줄반전에 딱.

[4] 벚꽃 — relatable
    → 봄 시즌 공감 소재. 계절 콘텐츠는 타이밍이 생명.

[5] 읽씹 — debate
    → 읽씹 논쟁은 댓글률 최고. 양자택일 구조에 딱.
```

### 추천 품질 규칙

- **구체적으로**: "일상" 같은 모호한 주제 금지. "알람", "커피", "출근길" 같은 구체적 소재로.
- **이유는 1문장**: 길어지면 안 읽음. 핵심만.
- **스타일은 근거 있게**: 플랫폼 프로필의 Recommended Styles와 스타일 파일의 호환 플랫폼을 교차 확인한다.
- **중복 방지**: 기존 포스트에서 이미 사용한 topic과 동일한 주제는 절대 추천하지 않는다.
- **계절 최소 1개**: 시의성 있는 주제를 반드시 1개 이상 포함한다.

AskUserQuestion으로 번호를 선택하게 한다.

```
추천 주제 중 하나를 선택해주세요. (번호 입력)
```

---

## 7. 선택 후 처리

사용자가 번호를 선택하면 해당 주제와 스타일을 확인한다.

AskUserQuestion으로 다음을 묻는다:

```
'{주제명}'(으)로 '{스타일}' 스타일의 포스트를 바로 생성할까요?

[1] 예
[2] 아니오
```

### [1] 예 → 생성 안내

아래 메시지를 출력한다:

```
아래 명령어로 생성할 수 있습니다:

/feed {platform} {style} {topic}
```

예시:
```
아래 명령어로 생성할 수 있습니다:

/feed threads twist-poem 알림
```

### [2] 아니오 → 종료

```
알겠습니다. 나중에 /feed-recommend로 다시 추천받을 수 있습니다.
```

---

## 8. 에지 케이스 처리

| 상황 | 처리 |
|------|------|
| `~/.config/feed/` 디렉토리 없음 | "먼저 /feed로 포스트를 생성해주세요" 출력 후 종료 |
| `platforms/{platform}.md` 없음 | 사용 가능한 플랫폼 목록 표시 후 AskUserQuestion으로 재선택 |
| `topics.json` 없거나 비어 있음 | 데이터 없는 경우 로직 (5.2)으로 분기 |
| `engagement.json` 없거나 비어 있음 | `topics.json`만으로 분석, 그마저 없으면 5.2로 분기 |
| 기존 포스트가 너무 많아 모든 기본 주제를 이미 다룸 | 이미 다룬 주제라도 다른 스타일 조합으로 추천. "(재해석)" 표시 추가 |
| 선택한 플랫폼에 호환 스타일이 없음 | 발생할 수 없지만 만약 그렇다면 가장 범용적인 스타일(`relatable`)을 기본 추천 |

---

## 9. 전체 플로우 요약

```
/feed-recommend [platform]
        │
        ▼
   platform 인자 있음?
   ├── 있음 → 유효성 확인
   └── 없음 → AskUserQuestion으로 선택
        │
        ▼
   ~/.config/feed/ 존재 확인
   ├── 없음 → "먼저 /feed로 포스트를 생성해주세요" → 종료
   └── 있음 → 진행
        │
        ▼
   데이터 로드
   ├── topics.json
   ├── engagement.json
   ├── platforms/{platform}.md
   └── Glob ~/.config/feed/posts/{platform}/**/*.md
        │
        ▼
   topics.json에 데이터 있음?
   ├── 있음 → 데이터 기반 추천 (5.1)
   └── 없음 → 플랫폼 특성 기반 추천 (5.2)
        │
        ▼
   3~5개 주제 추천 출력
        │
        ▼
   사용자 번호 선택
        │
        ▼
   "바로 생성할까요?"
   ├── [1] 예 → "/feed {platform} {style} {topic}" 안내
   └── [2] 아니오 → 종료
```
