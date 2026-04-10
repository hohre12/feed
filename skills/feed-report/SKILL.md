---
name: feed-report
description: Generates posting statistics and engagement reports for a given period
disable-model-invocation: true
argument-hint: <week|month|platform>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion
---

# /feed-report

Analyzes posting statistics and engagement data for a given period and generates a report.

---

## Step 1. Precondition Check

Check whether the `~/.config/feed/` directory exists.

```
Bash: test -d ~/.config/feed
```

- **Does not exist** → display message and terminate:
  ```
  먼저 /feed로 포스트를 생성해주세요.
  ```
- **Exists** → proceed to Step 2.

---

## Step 2. Argument Parsing

Split `$ARGUMENTS` by whitespace to determine the **period** and **platform filter**.

### Period Keywords

| Keyword | Meaning |
|---------|---------|
| `week` | This week (Monday through today) |
| `month` | This month (1st through today) |
| *(none)* | Default = `week` |

### Platform Keywords

| Keyword | Platform |
|---------|----------|
| `threads` | threads |
| `x` | x |
| `reddit` | reddit |
| `devto` | devto |
| *(none)* | All platforms |

### Combination Examples

```
/feed-report                    → this week, all platforms
/feed-report week               → this week, all platforms
/feed-report month              → this month, all platforms
/feed-report threads            → this week, Threads only
/feed-report month threads      → this month, Threads only
/feed-report x week             → this week, X only (order does not matter)
```

### Parsing Logic

1. Split `$ARGUMENTS` by whitespace into a token array.
2. Iterate through each token:
   - `week` or `month` → set as period
   - `threads`, `x`, `reddit`, `devto` → set as platform filter
   - Unrecognized → ignore
3. If no period is set, use `week` as the default.
4. If no platform filter is set, target all platforms.

---

## Step 3. Calculate Date Range

Calculate the start and end dates based on today's date.

```bash
# Today's date
TODAY=$(date +%Y-%m-%d)
```

### For `week`

From this week's Monday through today.

```bash
# macOS
WEEKDAY=$(date +%u)  # 1=Mon ~ 7=Sun
DAYS_SINCE_MONDAY=$((WEEKDAY - 1))
START_DATE=$(date -v-${DAYS_SINCE_MONDAY}d +%Y-%m-%d)
END_DATE=$TODAY
```

### For `month`

From the 1st of this month through today.

```bash
START_DATE=$(date +%Y-%m-01)
END_DATE=$TODAY
```

---

## Step 4. Scan Posts

Collect post files within the date range from `~/.config/feed/posts/`.

### Target Platforms

- If a platform filter is set → only that platform's directory
- If not → all platform directories under `~/.config/feed/posts/`

### Date-Based Directory Traversal

Posts are stored in the `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/` structure.
Calculate the YYYY/MM/DD combinations that fall within the date range and traverse only those directories.

```
# Check platform list
Glob ~/.config/feed/posts/*/

# Collect post files within the date range (per platform)
# Example: for the range 2026-04-07 ~ 2026-04-09
Glob ~/.config/feed/posts/{platform}/2026/04/07/*.md
Glob ~/.config/feed/posts/{platform}/2026/04/08/*.md
Glob ~/.config/feed/posts/{platform}/2026/04/09/*.md
```

### Read Frontmatter from Each Post File

Read the frontmatter of each collected `.md` file using Read and extract the following information:

| Field | Purpose |
|-------|---------|
| `platform` | Per-platform aggregation |
| `style` | Per-style aggregation |
| `topic` | Top 3 display, topics.json update |
| `published` | Publish status aggregation |
| `created_at` | Date range verification (double-check against directory structure) |
| `score.*` | Average score calculation, Top 3 ranking |
| `engagement.*` | Engagement data aggregation, Top 3 ranking |

### No Posts Found

If the scan returns 0 files, display message and terminate:

```
해당 기간에 포스트가 없습니다. (2026-04-07 ~ 2026-04-09)
```

---

## Step 5. Data Aggregation

Aggregate the frontmatter data from collected posts.

### 5.1 Overall Summary

- `total_created`: Total number of posts
- `total_published`: Number of posts with `published: true`
- `start_date`: Period start date
- `end_date`: Period end date

### 5.2 Per-Platform Aggregation

For each platform:

- `count`: Number created
- `published`: Number published
- `avg_score`: Overall average of score items (one decimal place)

### 5.3 Per-Style Aggregation

For each style:

- `count`: Number created
- `avg_score`: Overall average of score items (one decimal place)

### 5.4 Top 3 Posts

Ranking criteria:
1. Posts with engagement data → rank by weighted sum: `likes + comments * 2 + reposts * 1.5`
2. Posts without engagement data → rank by average of score items
3. Posts with engagement data take priority (engagement data outranks score)

Information to display for each post:
- Platform, style, topic, average score, engagement (if available)

### 5.5 Engagement Data Summary

Read `~/.config/feed/analytics/engagement.json` using Read.

- Filter entries within the date range (`checked_at` falls within the range)
- Sum total likes, comments, reposts
- Identify the best-performing post (same weighted sum criteria as 5.4)

If engagement.json has no data or no data within the date range, omit this section.

---

## Step 6. Report Output

Output the report in the format below. **All user-facing text is written in Korean.**

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

### period_label Values

| Period | Label |
|--------|-------|
| week | 이번 주 |
| month | 이번 달 |

### When Platform Filter Is Applied

Add the platform name to the report title:

```markdown
# Feed 리포트 — Threads
```

The per-platform table shows only the single filtered platform row.

---

## Step 7. Engagement Data Update

After outputting the report, proceed with engagement data updates for published posts (`published: true`) within the period.

### 7.1 Check accounts.json

```
Read ~/.config/feed/config/accounts.json
```

Check each platform's `enabled` status.

### 7.2 Per-Platform Branching

#### API-Supported + Enabled Platforms

For published posts on that platform:

```
{platform}의 반응 데이터를 API로 가져올까요? [1] 예 [2] 건너뛰기
```

- **[1] Yes** → fetch engagement data via API call (currently placeholder — actual API integration is Phase 3)
- **[2] Skip** → move to the next platform

#### API-Unsupported or Disabled Platforms

Offer manual input for each published post on that platform.

Post identifier is extracted from the filename:
`{HHMMSS}-{style}-{NNN}.md` → `{platform} {style}-{NNN}`

```
threads twist-poem-001의 반응을 입력하시겠습니까? [1] 예 [2] 건너뛰기
```

- **[2] Skip** → move to the next post
- **[1] Yes** → ask in sequence:

```
좋아요 수:
```
→ User input (number)

```
댓글 수:
```
→ User input (number)

```
리포스트 수:
```
→ User input (number)

### 7.3 Save Data

If engagement data was received, update two locations.

#### A. Update Post Frontmatter

Update the engagement field in the post file's frontmatter:

```yaml
engagement:
  likes: {input value}
  comments: {input value}
  reposts: {input value}
  last_checked: {current ISO 8601 timestamp}
```

Read the entire file using Read, replace only the engagement block, and save with Write.

#### B. Update engagement.json

```
Read ~/.config/feed/analytics/engagement.json
```

Add a new entry to the `entries` array, or update an existing entry with the same `post_path`:

```json
{
  "post_path": "posts/{platform}/{YYYY}/{MM}/{DD}/{filename}",
  "platform": "{platform}",
  "checked_at": "{current ISO 8601 timestamp}",
  "likes": {input value},
  "comments": {input value},
  "reposts": {input value}
}
```

- `post_path` uses the relative path after `~/.config/feed/`.
- If the same `post_path` already exists, overwrite that entry (keep latest data).
- Save engagement.json with Write.

---

## Step 8. Update topics.json

After all data collection is complete, update `~/.config/feed/analytics/topics.json`.

```
Read ~/.config/feed/analytics/topics.json
```

### Update Logic

Iterate through all post topics collected within the period:

1. **If the topic already exists** → update that entry:
   - `used_count`: recalculate as the total number of times this topic was used across all posts
   - `avg_engagement`: recalculate as the average across posts that have engagement data
   - `platforms`: list of platforms used (deduplicated)
   - `styles`: list of styles used (deduplicated)

2. **If the topic does not exist** → add a new entry:
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

3. Save topics.json with Write.

### Update Complete Message

```
topics.json 업데이트 완료 — {updated topic count}개 토픽 반영
```

---

## Appendix: Edge Cases

| Situation | Handling |
|-----------|----------|
| `~/.config/feed/` does not exist | Output "먼저 /feed로 포스트를 생성해주세요" and terminate |
| 0 posts within the period | Output "해당 기간에 포스트가 없습니다. ({start} ~ {end})" and terminate |
| engagement.json missing or empty entries | Omit the engagement data section |
| topics.json missing | Create a new file with empty `{ "topics": [] }` then update |
| Post with empty score fields | Exclude from average calculation |
| Post with all null engagement fields | Classify as no-engagement post (rank by score) |
| User enters non-numeric value | Re-prompt with "숫자를 입력해주세요" |
| accounts.json missing | Treat all platforms as disabled → offer manual input only |
