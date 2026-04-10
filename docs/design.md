# Feed -- SNS Content Generator & Publisher Plugin

> A Claude Code plugin that creates posts for your feed.
> Generates platform-optimized content via interview-driven workflow, saves it, and publishes it.

---

## 1. Project Overview

### Problem

- Each SNS platform has different tones, formats, and styles that perform well
- AI-generated text has an obvious "AI feel" that kills engagement
- Writing separate posts for each platform is inefficient
- No way to track which topics/styles get the best reactions

### Solution

- Platform profiles + style templates for tailored content generation
- Enforced colloquial/spoken-language rules to sound human-written
- Interview-based UX for rapid generate -> save -> publish workflow
- Post history + engagement data accumulation

### Core Principles

1. **Human feel** -- If it reads like AI, it's a failure. Colloquial tone, incomplete sentences, and variation are mandatory.
2. **Zero infrastructure** -- No database, no server. Claude Code + local files only.
3. **Done in 3 choices** -- The interview is 7 steps max. If it feels long, people won't use it.
4. **Data accumulates** -- Every post and its engagement is saved for future analysis and recommendations.

---

## 2. Architecture

### Design Principles

- **Separate plugin code from user data** -- Plugin is read-only templates, user data lives in `~/.config/feed/`
- **Hierarchical distribution + load on demand** -- SKILL.md is a router only; actual instructions are loaded from distributed files as needed
- **Extend by adding files** -- New platforms/styles/rules/flows require no modifications to existing files

### Separation Criteria

| Criterion | Separate | Merge |
|-----------|----------|-------|
| Loaded in different combinations per request | O (platforms/, styles/, audiences/) | |
| Always loaded together but grow independently | O (individual files in rules/) | |
| Always loaded together and small in volume | | O (single file) |

---

## 3. Project Structure

### Plugin (committed to GitHub -- read-only templates)

```
feed/
+-- .claude-plugin/
|   +-- plugin.json                        # Plugin manifest
|
+-- settings.json                          # Default permission settings
+-- CLAUDE.md                              # Plugin development guide
+-- README.md                              # User installation/usage guide
+-- LICENSE
+-- CHANGELOG.md
|
+-- skills/                                # Skills (official plugin structure)
|   +-- feed/                              #   /feed -- Main router
|   |   +-- SKILL.md                       #     Router + argument parsing
|   |   +-- interview.md                   #     Interview flow details
|   |   +-- reference.md                   #     Routing guide (which files to read in which situation)
|   |
|   +-- feed-publish/                      #   /feed-publish -- Publish unpublished posts
|   |   +-- SKILL.md
|   |
|   +-- feed-report/                       #   /feed-report -- Report
|   |   +-- SKILL.md
|   |
|   +-- feed-recommend/                    #   /feed-recommend -- Topic recommendation
|       +-- SKILL.md
|
+-- rules/                                 # Writing rules
|   +-- common.md                          #   AI prevention, common writing rules
|   +-- scoring.md                         #   Scoring criteria + pass conditions
|   +-- ko.md                              #   Korean writing rules
|   +-- en.md                              #   English writing rules
|
+-- platforms/                             # Platform profiles
|   +-- threads.md
|   +-- x.md
|   +-- reddit.md
|   +-- devto.md
|
+-- styles/                                # Style templates
|   +-- twist-poem.md                      #   Twist poem
|   +-- debate.md                          #   Debate
|   +-- relatable.md                       #   Relatable
|   +-- one-liner.md                       #   One-liner
|   +-- quote-parody.md                    #   Quote parody
|   +-- self-deprecation.md                #   Self-deprecation
|   +-- insight.md                         #   Insight
|   +-- curated-list.md                    #   Curated list
|
+-- audiences/                             # Target audience profiles
|   +-- developer.md                       #   Developers
|   +-- designer.md                        #   Designers
|   +-- marketer.md                        #   Marketers
|   +-- indie-hacker.md                    #   Indie hackers / founders
|   +-- worker.md                          #   Office workers
|   +-- self-improvement.md                #   Self-improvement enthusiasts
|   +-- investor.md                        #   Investors / personal finance
|   +-- general.md                         #   General audience
|
+-- flows/                                 # Workflow definitions
|   +-- save.md                            #   Save format + frontmatter spec
|   +-- publish.md                         #   Publish branching logic (API/clipboard)
|
+-- docs/
    +-- design.md                          # This document
```

### User Data (`~/.config/feed/` -- outside plugin, outside git)

```
~/.config/feed/
+-- config/
|   +-- accounts.json                      # Actual API keys/tokens
|   +-- preferences.json                   # User preferences
|
+-- posts/                                 # Generated posts
|   +-- {platform}/
|       +-- {YYYY}/
|           +-- {MM}/
|               +-- {DD}/
|                   +-- {HHMMSS}-{style}-{NNN}.md
|
+-- analytics/
    +-- engagement.json                    # Per-platform engagement data
    +-- topics.json                        # Per-topic performance tracking
```

### Initialization Flow

```
/feed first run
  |
  +-- Check if ~/.config/feed/ exists
  |
  +-- If not:
  |   +-- Create ~/.config/feed/config/
  |   +-- Create empty accounts.json template
  |   +-- Create preferences.json with defaults
  |   +-- Create ~/.config/feed/posts/
  |   +-- Create ~/.config/feed/analytics/ (empty JSON files)
  |   +-- "Feed 초기 설정 완료" message
  |
  +-- If exists:
      +-- Load existing config -> proceed normally
```

---

## 4. Skill Design

### 4.1 Common -- SKILL.md Frontmatter (Claude Code Official Spec)

```yaml
---
name: feed
description: SNS 플랫폼별 맞춤 콘텐츠를 인터뷰 기반으로 생성하고 발행합니다
disable-model-invocation: true
argument-hint: <platform> <style> <topic>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion
---
```

| Field | Purpose |
|-------|---------|
| `name` | Skill name. Invoked via `/name` |
| `description` | Skill description. Max 250 characters |
| `disable-model-invocation` | `true` -> Only user can invoke (prevents auto-execution) |
| `argument-hint` | Argument hint for autocomplete |
| `allowed-tools` | Tools usable without additional permission |

### 4.2 Skill List

| Skill | Path | Purpose |
|-------|------|---------|
| `/feed` | `skills/feed/SKILL.md` | Main. Interview -> generate -> save -> publish |
| `/feed-publish` | `skills/feed-publish/SKILL.md` | Publish unpublished posts |
| `/feed-report` | `skills/feed-report/SKILL.md` | Period-based analytics report |
| `/feed-recommend` | `skills/feed-recommend/SKILL.md` | Topic recommendation |

### 4.3 `/feed` -- Main Skill

#### Structure

```
skills/feed/
+-- SKILL.md          # Router: argument parsing -> branch to appropriate files
+-- interview.md      # Interview flow details (Steps 1-7)
+-- reference.md      # Routing guide (file mapping per situation)
```

#### SKILL.md Role (Router Only)

```
1. Parse $ARGUMENTS
2. Check if ~/.config/feed/ exists -> initialize if not
3. If arguments incomplete -> read interview.md and run interview
4. Read reference.md -> load situation-appropriate files
5. Generate -> save -> interview for publish decision
```

#### reference.md Routing Table

```
On generation, load:
  - rules/common.md (always)
  - rules/scoring.md (always)
  - rules/{language}.md (selected language)
  - platforms/{platform}.md (selected platform)
  - styles/{style}.md (selected style)
  - audiences/{audience}.md (selected target audience)

On save, load:
  - flows/save.md

On publish, load:
  - flows/publish.md
```

#### Context Loading Flow

When `/feed threads twist-poem 알람` is executed:

```
SKILL.md loaded (automatic)
  |
  +-- Read reference.md              <- Routing guide
  +-- Read rules/common.md           <- Common rules
  +-- Read rules/scoring.md          <- Scoring criteria
  +-- Read rules/ko.md               <- Korean rules
  +-- Read platforms/threads.md      <- Threads profile
  +-- Read styles/twist-poem.md      <- Twist poem style
  +-- Read audiences/{audience}.md   <- Target audience profile
  |
  v Generate + score
  |
  +-- Read flows/save.md             <- Save format
  +-- Write ~/.config/feed/posts/... <- Save post
  |
  v If publish selected
  |
  +-- Read flows/publish.md          <- Publish logic
```

#### Argument Parsing Rules

```
/feed                              -> Full interview
/feed threads                      -> Platform specified, interview for the rest
/feed threads twist-poem           -> Platform+style specified, interview for topic only
/feed threads twist-poem 알람      -> All specified, generate immediately
```

#### Interview Flow (interview.md)

```
Step 1. Language
  -> [1] 한글  [2] English
  (Skip if preferences.json has a default)

Step 2. Platform
  -> Glob platforms/*.md to list available platforms
  -> [1] Threads  [2] X  [3] Reddit  [4] Dev.to  [5] 전체
  (Selecting 전체 generates for all platforms individually)

Step 3. Style
  -> Check recommended styles from selected platform profile
  -> Glob styles/*.md to list available styles
  -> Filter for styles compatible with the selected platform

Step 4. Target Audience
  -> Glob audiences/*.md to list available audiences
  -> [1] 개발자  [2] 디자이너  [3] 마케터  [4] 창업가/인디해커
     [5] 직장인  [6] 자기계발  [7] 투자/재테크  [8] 일반인
  (Dynamically constructed from Glob results)

Step 5. Topic
  -> [1] 직접 입력  [2] 추천받기
  -> Recommendations: based on ~/.config/feed/analytics/topics.json (or trends if no data)
  -> Direct input: free text

Step 6. Generation
  -> Generate content + self-evaluate
  -> Auto-regenerate on score failure (max 2 retries)
  -> Display result

Step 7. Publish Decision
  -> [1] 발행  [2] 나중에  [3] 수정 요청
  -> Publish/Save later -> save to ~/.config/feed/posts/
  -> Publish -> execute flows/publish.md logic
  -> Request edits -> receive instructions -> back to Step 6
```

### 4.4 `/feed-publish` -- Publish Skill

```
/feed-publish                      -> Show unpublished list -> select -> publish
/feed-publish today                -> Today's unpublished only
/feed-publish all                  -> All unpublished
```

- Search `~/.config/feed/posts/` for files with `published: false`
- Display list -> user selects
- Execute `flows/publish.md` logic

### 4.5 `/feed-report` -- Report Skill

```
/feed-report                       -> This week's summary
/feed-report month                 -> This month's summary
/feed-report threads               -> Specific platform only
```

Report contents:
- Posts generated/published in the period
- Distribution by platform
- Distribution by style
- Engagement data (based on `~/.config/feed/analytics/engagement.json`)
- Top 3 highest-engagement posts
- API-supported platforms: automatically refresh engagement data
- Non-API platforms: manual input via interview

### 4.6 `/feed-recommend` -- Recommendation Skill

```
/feed-recommend                    -> Interview-based recommendation
/feed-recommend threads            -> Recommendation for a specific platform
```

Recommendation logic:
1. Check past engagement data from `~/.config/feed/analytics/topics.json`
2. Analyze which topic categories performed well
3. Suggest 3-5 similar but uncovered topics
4. If no data: recommend based on platform characteristics + current trends

---

## 5. Platform Profiles

### File Location

`platforms/{platform}.md`

### Contents

```markdown
# {Platform Name}

## Spec
- Character limits
- Supported formats (text, image, link, etc.)
- Hashtag usage

## Algorithm
- Engagement weights (comments > reposts > likes, etc.)
- Exposure method (follower-based, recommendation-based, etc.)

## What Works
- Tone (emotional, informational, humorous, etc.)
- Length
- Structure

## What Doesn't Work
- Patterns to avoid

## Recommended Styles
- List of style filenames effective on this platform

## API Publishing
- Supported: true/false
- Auth method
- Required key fields (see accounts.json)
```

### Per-Platform API Publishing Support

| Platform | Filename | API Publish | Notes |
|----------|----------|:-----------:|-------|
| Threads | `threads.md` | Partial | Meta Threads API -- limited |
| X (Twitter) | `x.md` | Yes | Twitter API v2 |
| Reddit | `reddit.md` | Yes | Reddit API (PRAW) |
| Dev.to | `devto.md` | Yes | Dev.to REST API |

---

## 6. Style Templates

### File Location

`styles/{style}.md`

### Contents

```markdown
# {Style Name}

## Structure
- Line count, pacing, twist placement, etc.

## Rules
- Style-specific rules

## Tone
- Emotion, voice

## Good Examples
- Reference material (target this quality level)

## Bad Examples
- AI-sounding examples + why they are bad

## Scoring
- Style-specific scoring items and weights (on top of the base criteria in rules/scoring.md)

## Compatible Platforms
- List of platform filenames where this style works well
```

### Style Mapping

| Name | Filename | Description | Compatible Platforms |
|------|----------|-------------|---------------------|
| Twist Poem | `twist-poem.md` | Emotional monologue -> last line reveals a twist in the speaker/subject | threads, x |
| Debate | `debate.md` | Binary-choice provocation. Drives comments | threads, x, reddit |
| Relatable | `relatable.md` | "I thought it was just me" type | threads, x |
| One-liner | `one-liner.md` | 2 lines. Expectation -> twist | x, threads |
| Quote Parody | `quote-parody.md` | Twisting famous quotes/sayings | threads, x, reddit |
| Self-deprecation | `self-deprecation.md` | Numbers + self-deprecating humor | threads, x |
| Insight | `insight.md` | Sharp observation or non-obvious insight | threads, x, devto |
| Curated List | `curated-list.md` | Curated list of resources/tips/tools | threads, devto, reddit |

---

## 7. Target Audiences

### File Location

`audiences/{audience}.md`

### Contents

Each audience profile defines the tone, terminology level, and search sources used during content generation. The audience influences both what topics are recommended and how the content is written.

### Audience Mapping

| Name | Filename | Description |
|------|----------|-------------|
| Developer | `developer.md` | Software developers -- technical terms OK, references to GitHub/Stack Overflow |
| Designer | `designer.md` | Designers -- visual/UX terminology, design tool references |
| Marketer | `marketer.md` | Marketers -- growth/funnel terminology, campaign references |
| Indie Hacker | `indie-hacker.md` | Founders/indie hackers -- startup terminology, bootstrapping context |
| Worker | `worker.md` | Office workers -- workplace humor, corporate life references |
| Self-improvement | `self-improvement.md` | Self-improvement enthusiasts -- productivity, habit-building context |
| Investor | `investor.md` | Investors / personal finance -- financial terminology, market references |
| General | `general.md` | General audience -- no jargon, universally relatable topics |

### Flexible Handling

Unlike platforms/styles, target audience handling is flexible. If the user enters a value not in the list, load the closest matching profile as a base and apply additional context:
- "Frontend developer" -> `developer.md` base + frontend context
- "College student" -> `general.md` base + student context
- "PM" -> `worker.md` base + product management context

---

## 8. Rules (rules/)

### File Structure

| File | Contents | Loaded When |
|------|----------|-------------|
| `common.md` | AI prevention, structure variation, forbidden patterns, colloquial tone enforcement | Always during generation |
| `scoring.md` | Scoring items, pass criteria, regeneration conditions | Always during generation |
| `ko.md` | Korean writing rules (casual speech, abbreviations, spoken style) | When language=ko |
| `en.md` | English writing rules (casual tone, contractions) | When language=en |

### Extending

New language: Add `rules/ja.md` -> no existing file modifications needed
New rule category: Add `rules/brand-voice.md` -> add load condition in reference.md

---

## 9. Flows (flows/)

### File Structure

| File | Contents | Loaded When |
|------|----------|-------------|
| `save.md` | Frontmatter spec, file path convention, save process | After generation completes |
| `publish.md` | API/clipboard branching, failure handling, status update | When publish is selected |

### Extending

Scheduled publishing: Add `flows/schedule.md`
Multi-platform simultaneous publishing: Add `flows/multi-publish.md`

---

## 10. Publish Flow

```
User: selects [Publish]
         |
         v
    Save to ~/.config/feed/posts/ (published: false)
         |
         v
    Check enabled in
    ~/.config/feed/config/accounts.json
    for the target platform
         |
    +----+----+
    |         |
  enabled    disabled
  = true     = false
    |         |
    v         v
 Interview:  "클립보드에 복사했습니다"
 "자동       published: true
  발행       publish_method: clipboard
  할까요?"
    |
  +-+-+
  |   |
 Yes  No
  |   |
  v   v
 API  Clipboard
 pub  copy
  |   |
  v   v
 published: true
 publish_method: api | clipboard
```

### Failure Handling

- API publish failure -> display error -> automatically fall back to clipboard copy
- Record `publish_method: failed`, `error: {message}` in the file

---

## 11. Post Save Format

### File Path

```
~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{HHMMSS}-{style}-{NNN}.md
```

Examples:

```
~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
~/.config/feed/posts/x/2026/04/09/201500-one-liner-001.md
```

### File Contents (frontmatter + body)

```markdown
---
platform: threads
style: twist-poem
topic: alarm
audience: developer
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

## 12. User Data

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

### Engagement Data Collection

- API-supported platforms: Automatically call API on `/feed-report` -> update engagement.json
- Non-API platforms: Manual input via interview on `/feed-report`

---

## 13. Plugin Configuration Files

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

## 14. Scheduling

Leverages Claude Code's `/schedule` feature.

| Schedule | Command | Description |
|----------|---------|-------------|
| Daily 9 AM | `/feed-recommend` | Today's topic recommendation |
| Every Monday | `/feed-report week` | Weekly report |
| 1st of each month | `/feed-report month` | Monthly report |

---

## 15. Extensibility Matrix

| Scenario | File to Add | Modify Existing Files |
|----------|-------------|:---------------------:|
| New platform (LinkedIn) | `platforms/linkedin.md` | No |
| New style (informative) | `styles/informative.md` | No |
| New audience (students) | `audiences/student.md` | No |
| New language (Japanese) | `rules/ja.md` | No |
| New flow (scheduled publish) | `flows/schedule.md` | No |
| New skill (A/B test) | `skills/feed-ab-test/SKILL.md` | No |
| Change scoring criteria | `rules/scoring.md` edit | Yes (that file only) |
| User custom style | `~/.config/feed/styles/custom.md` | No |

---

## 16. Implementation Order

### Phase 1: Core (Threads-centric)

- [ ] `.claude-plugin/plugin.json` manifest
- [ ] `settings.json` default permissions
- [ ] `CLAUDE.md` plugin development guide
- [ ] `rules/common.md` AI prevention rules
- [ ] `rules/scoring.md` scoring criteria
- [ ] `rules/ko.md` Korean rules
- [ ] `platforms/threads.md` Threads profile
- [ ] `styles/twist-poem.md` twist poem
- [ ] `styles/debate.md` debate
- [ ] `styles/relatable.md` relatable
- [ ] `styles/one-liner.md` one-liner
- [ ] `styles/quote-parody.md` quote parody
- [ ] `styles/self-deprecation.md` self-deprecation
- [ ] `styles/insight.md` insight
- [ ] `styles/curated-list.md` curated list
- [ ] `audiences/developer.md` developer profile
- [ ] `audiences/designer.md` designer profile
- [ ] `audiences/marketer.md` marketer profile
- [ ] `audiences/indie-hacker.md` indie hacker profile
- [ ] `audiences/worker.md` worker profile
- [ ] `audiences/self-improvement.md` self-improvement profile
- [ ] `audiences/investor.md` investor profile
- [ ] `audiences/general.md` general profile
- [ ] `flows/save.md` save format
- [ ] `flows/publish.md` publish logic
- [ ] `skills/feed/SKILL.md` main router
- [ ] `skills/feed/interview.md` interview flow
- [ ] `skills/feed/reference.md` routing guide
- [ ] `~/.config/feed/` initialization logic

### Phase 2: Publishing & Analytics

- [ ] `skills/feed-publish/SKILL.md` publish skill
- [ ] `skills/feed-report/SKILL.md` report skill
- [ ] `skills/feed-recommend/SKILL.md` recommendation skill
- [ ] Manual engagement input interview
- [ ] `README.md` user guide

### Phase 3: Platform Expansion

- [ ] `platforms/x.md` X profile
- [ ] `platforms/reddit.md` Reddit profile
- [ ] `platforms/devto.md` Dev.to profile
- [ ] `rules/en.md` English rules
- [ ] API publish integration (X, Reddit, Dev.to)

### Phase 4: Advanced

- [ ] Cron schedule setup guide
- [ ] Engagement-based recommendation improvements
- [ ] User custom style support (`~/.config/feed/styles/`)
- [ ] `LICENSE`, `CHANGELOG.md`
- [ ] Plugin distribution (GitHub)
