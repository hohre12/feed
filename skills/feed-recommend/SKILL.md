---
name: feed-recommend
description: Recommends topics based on past engagement data and platform characteristics
disable-model-invocation: true
argument-hint: <platform>
allowed-tools: Read Write Glob Grep AskUserQuestion
---

# /feed-recommend — Topic Recommendation

Recommends SNS posting topics based on past engagement data and platform characteristics.

---

## 1. Argument Parsing

Analyze `$ARGUMENTS`.

| Input | Behavior |
|-------|----------|
| `/feed-recommend threads` | `platform = threads` → recommend for that platform |
| `/feed-recommend` | No platform specified → ask in Step 2 |

### Parsing Rules

- If a first argument exists, treat it as `platform`.
- Check whether `platform` exists in the `platforms/` directory.
  - **Exists**: proceed with that platform
  - **Does not exist**: show the list of available platforms and ask again

---

## 2. Platform Selection (When No Argument)

If `platform` is empty, ask via AskUserQuestion.

First run `Glob platforms/*.md` to get the list of available platforms.

```
어떤 플랫폼용 주제를 추천할까요?

[1] Threads
[2] X
[3] Reddit
...
```

Once the user selects, set that `platform` value.

---

## 3. Precondition Check

### Check ~/.config/feed/ Directory Existence

Verify with `Glob ~/.config/feed/`.

- **Does not exist**: output the message below and terminate.

```
아직 생성된 포스트가 없습니다.
먼저 /feed로 포스트를 생성해주세요.
```

- **Exists**: proceed to the next step.

### Platform Validity Check

Check whether `platforms/{platform}.md` exists.

- **Does not exist**: show the list of available platforms and ask again via AskUserQuestion.

```
'{platform}' 플랫폼을 찾을 수 없습니다.

사용 가능한 플랫폼:
[1] Threads
[2] X
...

어떤 플랫폼을 선택할까요?
```

---

## 4. Data Loading

Read the following files in order.

### 4.1 Past Topic Performance

```
Read ~/.config/feed/analytics/topics.json
```

`topics.json` structure:
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

### 4.2 Engagement Data

```
Read ~/.config/feed/analytics/engagement.json
```

`engagement.json` structure:
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

### 4.3 Platform Profile

```
Read platforms/{platform}.md
```

Key sections to check:
- **What Works**: tone/structure that performs well on this platform
- **Recommended Styles**: list of compatible styles
- **Algorithm**: engagement weights (comments > reposts > likes, etc.)

### 4.4 Existing Post List

```
Glob ~/.config/feed/posts/{platform}/**/*.md
```

Extract the list of already-covered topics. Read the `topic` field from each post's frontmatter.

---

## 5. Recommendation Logic

### 5.1 When Data Exists (topics.json has entries)

**Analysis phase:**

1. **Identify high-performing categories**: Analyze which `category` received strong engagement based on `avg_engagement`.
   - Apply the platform's algorithm weights. Example: Threads weights `comments` the highest.
   - Weighted score = `(comments * 3) + (reposts * 2) + (likes * 1)` (adjusted per platform)
2. **Identify style-topic patterns**: Determine which `style` + `category` combinations showed high engagement.
3. **Exclude already-covered topics**: Use the list of existing post `topic` values collected in Step 4.4 as an exclusion list.

**Recommendation criteria:**

- Select new topics from categories that are **the same as or adjacent to** high-performing categories
- Only recommend topics that **do not overlap** with already-covered ones
- Prioritize topics that **fit the characteristics** of the platform
- Suggest the **best-matching style** alongside each recommendation

**Category adjacency map:**

| Category | Adjacent Categories |
|----------|-------------------|
| `daily-habits` (daily life/habits) | `relationships`, `self-improvement`, `work-life` |
| `relationships` (relationships) | `daily-habits`, `emotions`, `communication` |
| `self-improvement` (self-improvement) | `work-life`, `daily-habits`, `motivation` |
| `work-life` (work) | `self-improvement`, `daily-habits`, `money` |
| `seasonal` (seasonal/timely) | adjacent to all categories |
| `emotions` (emotions) | `relationships`, `daily-habits`, `seasonal` |
| `communication` (communication) | `relationships`, `work-life`, `emotions` |
| `motivation` (motivation) | `self-improvement`, `emotions`, `seasonal` |
| `money` (money) | `work-life`, `daily-habits`, `self-improvement` |

### 5.2 When No Data Exists (topics.json is empty or missing)

Recommend based on the platform profile's What Works section.

**Default category pool:**

| Category | Korean Name | Example Topics |
|----------|------------|----------------|
| `daily-habits` | 일상/습관 | 알람, 커피, 출근루틴, 잠버릇, 샤워 |
| `relationships` | 관계 | 읽씹, 연락, 썸, 친구, 가족 |
| `self-improvement` | 자기개발 | 다이어트, 운동, 독서, 새해목표, 루틴 |
| `work-life` | 직장 | 월요일, 야근, 회의, 퇴근, 점심 |
| `seasonal` | 계절/시의성 | Topics matching the current season and timing |

**Per-platform tone matching:**

- **Threads**: emotional/relatable/self-deprecating → prioritize `relationships`, `emotions`, `daily-habits`
- **X**: witty/twist/one-liner → prioritize `work-life`, `daily-habits`, `communication`
- **Reddit**: informational/debate/depth → prioritize `self-improvement`, `work-life`, `money`
- **Dev.to**: development/tech/experience → prioritize `work-life`, `self-improvement`

Reference the current date to include at least 1 **seasonal/timely** topic.

---

## 6. Output Recommendations

Output 3-5 topics as a numbered list.

### Output Format

```
주제 추천 ({Platform})

[1] {topic name} — {recommended style}
    → {one-sentence reason why this topic is good}

[2] {topic name} — {recommended style}
    → {one-sentence reason why this topic is good}

[3] {topic name} — {recommended style}
    → {one-sentence reason why this topic is good}

[4] {topic name} — {recommended style}
    → {one-sentence reason why this topic is good}

[5] {topic name} — {recommended style}
    → {one-sentence reason why this topic is good}
```

### Output Example

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

### Recommendation Quality Rules

- **Be specific**: Do not use vague topics like "일상". Use concrete subjects like "알람", "커피", "출근길".
- **One-sentence reason**: Keep it brief. If it gets long, it won't be read.
- **Evidence-based style**: Cross-reference the platform profile's Recommended Styles with the style file's Compatible Platforms.
- **No duplicates**: Never recommend a topic identical to one already used in existing posts.
- **At least 1 seasonal**: Always include at least one timely/seasonal topic.

Ask the user to select a number via AskUserQuestion.

```
추천 주제 중 하나를 선택해주세요. (번호 입력)
```

---

## 7. Post-Selection Handling

When the user selects a number, confirm the topic and style.

Ask via AskUserQuestion:

```
'{주제명}'(으)로 '{스타일}' 스타일의 포스트를 바로 생성할까요?

[1] 예
[2] 아니오
```

### [1] Yes → Generation Guidance

Output the following message:

```
아래 명령어로 생성할 수 있습니다:

/feed {platform} {style} {topic}
```

Example:
```
아래 명령어로 생성할 수 있습니다:

/feed threads twist-poem 알림
```

### [2] No → Terminate

```
알겠습니다. 나중에 /feed-recommend로 다시 추천받을 수 있습니다.
```

---

## Appendix: Edge Cases

| Situation | Handling |
|-----------|----------|
| `~/.config/feed/` directory does not exist | Output "먼저 /feed로 포스트를 생성해주세요" and terminate |
| `platforms/{platform}.md` does not exist | Show available platform list and ask again via AskUserQuestion |
| `topics.json` missing or empty | Branch to no-data logic (5.2) |
| `engagement.json` missing or empty | Analyze with `topics.json` only; if that is also empty, branch to 5.2 |
| Too many existing posts covering all default topics | Recommend already-covered topics with different style combinations. Add "(재해석)" label |
| No compatible styles for the selected platform | Should not happen, but if it does, default to the most versatile style (`relatable`) |

---

## 8. Full Flow Summary

```
/feed-recommend [platform]
        │
        ▼
   platform argument provided?
   ├── Yes → validate
   └── No  → AskUserQuestion to select
        │
        ▼
   ~/.config/feed/ exists?
   ├── No  → "먼저 /feed로 포스트를 생성해주세요" → terminate
   └── Yes → proceed
        │
        ▼
   Load data
   ├── topics.json
   ├── engagement.json
   ├── platforms/{platform}.md
   └── Glob ~/.config/feed/posts/{platform}/**/*.md
        │
        ▼
   topics.json has data?
   ├── Yes → data-driven recommendation (5.1)
   └── No  → platform-characteristics recommendation (5.2)
        │
        ▼
   Output 3-5 topic recommendations
        │
        ▼
   User selects a number
        │
        ▼
   "Generate now?"
   ├── [1] Yes → show "/feed {platform} {style} {topic}" command
   └── [2] No  → terminate
```
