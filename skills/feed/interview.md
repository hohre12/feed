# Interview Flow

> Detailed step-by-step instructions for the `/feed` skill interview.
> All user interactions use the AskUserQuestion tool.
> If a value has already been provided via $ARGUMENTS, skip that step.

---

## Step 1. Language Selection

### Skip Condition
- Read `~/.config/feed/config/preferences.json`.
- If `interview.skip_language` is `true` and `default_language` is set → use that language and skip this step.

### Question
```
AskUserQuestion:
  "언어를 선택해주세요.

  [1] 한글
  [2] English"
```

### Processing
- `1` or `한글` → `language = "ko"` → load target: `rules/ko.md`
- `2` or `English` → `language = "en"` → load target: `rules/en.md`

---

## Step 2. Platform Selection

### Skip Condition
- If `$ARGUMENTS[0]` is a valid platform ID → use that platform and skip this step.

### List Construction
1. Run `Glob platforms/*.md` to get the list of available platform files.
2. Remove `.md` from each filename to build the platform ID list.
3. Construct a numbered list with an "all" option.

### Question
```
AskUserQuestion:
  "플랫폼을 선택해주세요.

  [1] Threads
  [2] X
  [3] Reddit
  [4] Dev.to
  ...
  [N] 전체 (모든 플랫폼용으로 각각 생성)"
```

The actual list is dynamically constructed from the Glob results. The above is an example.

### Processing
- Number or platform name input → select that platform
- Selecting `전체` → generate content for each platform individually. Repeat Steps 3-7 per platform.
- **If a platform not in the list is entered** (e.g., "인스타그램", "링크드인") → dynamically adapt using LLM knowledge about that platform. Infer character limits, tone, and algorithm characteristics for content generation. Common rules such as `rules/common.md` and `rules/scoring.md` still apply.

### Error Handling
- If `$ARGUMENTS[0]` is provided but does not match any valid platform ID or Korean alias → show the available platform list and ask the user to select again via AskUserQuestion.

---

## Step 2.5. Author Profile Check

This step runs automatically after platform selection. **No step number is shown to the user** — it's a background check.

### Logic
1. Read `~/.config/feed/config/preferences.json`.
2. Check if `profiles.{platform}` exists and has a non-empty `persona` field.
3. **If profile exists** → load it silently and continue to Step 3.
4. **If profile does NOT exist** → run the author profile interview below.

### First-Time Interview (only runs once per platform)

```
AskUserQuestion:
  "{platform}에서 사용할 프로필을 설정합니다. (최초 1회만)

  [1] 직접 입력 (SNS 프로필 복붙 가능)
  [2] 질문으로 만들기"
```

### Option 1: Direct Input

```
AskUserQuestion:
  "프로필을 입력해주세요. SNS 프로필을 그대로 붙여넣어도 됩니다.

  (예)
  개발 10년차, AI로 일하는 방식을 바꾸는 중
  Claude Code 헤비유저
  AI × 개발 실무 이야기를 검증하고 나눕니다."
```

From the user's input, extract:
- **persona**: The core role/identity (e.g., "developer" from "개발 10년차")
- **description**: The full input text as-is (preserve everything the user typed)

### Option 2: Guided Questions

Step-by-step interview to build a profile:

```
AskUserQuestion:
  "역할/직업이 무엇인가요?
  (예: 개발자, 디자이너, 마케터, 학생, 프리랜서 등)"
```

```
AskUserQuestion:
  "경력은 어느 정도인가요?
  (예: 3년차, 10년차, 신입, 취준생 등)"
```

```
AskUserQuestion:
  "주로 어떤 주제로 포스팅하고 싶으신가요?
  (예: AI 도구, 프론트엔드, 자기계발, 재테크 등)"
```

```
AskUserQuestion:
  "본인을 한 줄로 소개한다면?
  (예: AI로 일하는 방식을 바꾸는 개발자)"
```

Combine the answers into a profile:
- **persona**: From role answer (e.g., "developer")
- **description**: Assembled from all answers (e.g., "개발 10년차. AI 도구 주제. AI로 일하는 방식을 바꾸는 개발자.")

Show the assembled profile and ask for confirmation:

```
AskUserQuestion:
  "이렇게 프로필이 만들어졌어요:

  [역할] 개발자
  [소개] 개발 10년차. AI 도구 주제. AI로 일하는 방식을 바꾸는 개발자.

  [1] 확인
  [2] 다시 입력"
```

### Save to preferences.json

Save the profile under `profiles.{platform}`:

```json
{
  "profiles": {
    "threads": {
      "persona": "developer",
      "description": "풀스택 개발자. AI 도구와 자동화에 관심 많음."
    }
  }
}
```

### How the Profile Affects Content Generation

The author profile determines the **voice and credibility angle** of the content:
- The post is written FROM the author's perspective, not the target audience's perspective
- Example: author=developer, target=designer, topic=vibe coding
  - Correct: "개발자인데 디자이너 친구들한테 바이브코딩 도구 추천해봄"
  - Wrong: "코드 1도 모르는데 앱 만든 도구편" (this sounds like the author IS a designer)
- The author's persona adds authenticity. A developer recommending tools to designers is more credible than pretending to be a designer.

### Profile Persistence
- Once saved, the profile is permanent for that platform until the user manually changes it
- The profile is loaded automatically on every subsequent `/feed` execution for that platform
- To change: user edits `~/.config/feed/config/preferences.json` directly

---

## Step 3. Style Selection

### Skip Condition
- If `$ARGUMENTS[1]` is a valid style ID → use that style and skip this step.

### List Construction
1. Read the selected platform file (`platforms/{platform}.md`) and extract the recommended style list from the **"Recommended Styles"** section.
2. Run `Glob styles/*.md` to get the full list of style files.
3. Check the **"Compatible Platforms"** section of each style file and filter for styles compatible with the selected platform.
4. **All compatible styles must be listed in a single question with numbered options.** Do not show only some and separate the rest as "additional options".
5. Display order: Mark recommended styles with a star, but include non-recommended styles in the same numbered list.

### Question
**Important: List all compatible styles in a single AskUserQuestion with numbered options. Do not split into multiple messages even if there are 4 or more.**

```
AskUserQuestion:
  "{platform}에서 사용할 스타일을 선택해주세요.

  [1] 반전시 (twist-poem) ⭐
  [2] 공감 (relatable) ⭐
  [3] 한줄반전 (one-liner) ⭐
  [4] 명언패러디 (quote-parody) ⭐
  [5] 자조 (self-deprecation) ⭐
  [6] 논쟁 (debate) ⭐"
```

The actual list is dynamically constructed from Glob results and platform recommendations. The above is an example for Threads.

### Processing
- Number or style name/ID input → select that style
- **If a style not in the list is entered** (e.g., "스토리텔링", "뉴스") → dynamically adapt using LLM knowledge about that style. Infer the structure and tone described by the user and reflect it in generation. Common rules such as `rules/common.md` and `rules/scoring.md` still apply.

### Error Handling
- If `$ARGUMENTS[1]` is provided but does not match any valid style ID or Korean alias → show the available style list and ask the user to select again via AskUserQuestion.

---

## Step 4. Target Audience Selection

### List Construction
1. Run `Glob audiences/*.md` to get the list of available targets.
2. Remove `.md` from each filename to build the target ID list.
3. Display all targets in a single numbered list with Korean display names.

### Question
**List all targets in a single AskUserQuestion with numbered options.**

```
AskUserQuestion:
  "타겟 독자를 선택해주세요.

  [1] 개발자 (developer)
  [2] 디자이너 (designer)
  [3] 마케터 (marketer)
  [4] 창업가/인디해커 (indie-hacker)
  [5] 직장인 (worker)
  [6] 자기계발 (self-improvement)
  [7] 투자/재테크 (investor)
  [8] 일반인 (general)"
```

The actual list is dynamically constructed from Glob results. The above is an example.

### Processing
- Number or target name/ID input → select that target
- Load the selected target's `audiences/{audience}.md` file
- The **search sources** in this file determine the search direction for Step 5 (topic input)
- The **tone/terminology level** in this file determines the writing style for Step 6 (generation)

### Handling Targets Not in the List
Unlike platforms/styles, target audience handling is **flexible**:
- "프론트엔드 개발자" → based on `developer.md` + frontend context applied (weight toward React/Vue/CSS in searches)
- "대학생" → based on `general.md` + college student context applied (weight toward campus communities, student discounts in searches)
- "PM" → based on `worker.md` + PM context applied (weight toward product management, agile in searches)
- Load the closest matching profile file, but additionally reflect the specific context the user entered in tone/search
- Inform the user which profile was used as the base: "개발자(developer) 프로필 기반으로 프론트엔드에 맞춰 조정합니다."

---

## Step 5. Topic Input

### Skip Condition
- If topic was determined from smart parsing → use that value as the topic and skip this step.
- If the topic is a long sentence (10+ characters) → treat it as "source material text". Since it is original text written by the user (not a keyword), process and reinterpret this content to create content fitting the selected style.

### Question (Method Selection)
```
AskUserQuestion:
  "주제를 어떻게 정할까요?

  [1] 직접 입력
  [2] 추천받기"
```

### [1] Direct Input
```
AskUserQuestion:
  "주제를 입력해주세요.
  키워드(예: 알람, 다이어트)도 되고, 하고 싶은 말을 길게 써도 됩니다."
```

If long text is entered → treat as **source material text**. Extract the core message from the user's original text and process it to fit the selected style. Preserve the original emotion/intent but transform according to style rules.

### [2] Get Recommendations
1. Read `~/.config/feed/analytics/topics.json`.
2. Check the **search sources** in the selected target audience's `audiences/{audience}.md`.
3. **If data exists**: Analyze past engagement data to recommend 3 topics from categories that performed well but haven't been covered yet (or performed well previously).
4. **If no data exists**: Search for actual trending/popular topics from the target audience's search sources (starting from priority 1) using **WebSearch**. (e.g., target=developer → search GitHub Trending → recommend actual trending projects)

```
AskUserQuestion:
  "이런 주제는 어때요?

  [1] 알람 — 매일 아침 찾아오는 귀찮은 존재
  [2] 장바구니 — 담기만 하고 안 사는 습관
  [3] 읽씹 — 읽고도 답 안 하는 그 사람

  번호를 선택하거나, 다른 주제를 직접 입력해주세요."
```

### Processing
- Number selection → use that recommended topic
- Free text input → use the entered text as the topic

---

## Step 6. Content Generation

This step involves no user interaction. Content is generated based on the collected information.

### File Loading
Load all required files according to the routing table in `reference.md`:
- `rules/common.md` (always)
- `rules/scoring.md` (always)
- `rules/{language}.md` (selected language)
- `platforms/{platform}.md` (selected platform)
- `styles/{style}.md` (selected style)
- `audiences/{audience}.md` (selected target audience)
- `preferences.json → profiles.{platform}` (author profile — persona and description)

### Search (For Informational Styles or When Facts Are Needed)
- Check the **search sources** section of the target audience file.
- Use **WebSearch** to find the latest information from search sources relevant to the topic.
- Follow search source priority order (starting from priority 1).
- Collect verified facts/numbers/examples and incorporate them into the content.

### Generation
- Generate content adhering to all loaded rules, platform characteristics, and style templates.
- Reference the style file's structure, rules, tone, and good examples.
- Comply with the platform file's character limits, What Works/Doesn't Work guidelines.
- **Write from the author's perspective** using the loaded author profile. The post voice must match the author's persona, NOT the target audience. (e.g., author=developer, target=designer → write as a developer recommending to designers, not as a designer)
- **Reflect the target audience file's tone/terminology level.** (e.g., developer → use technical terms as-is; general → no jargon)

### Self-Evaluation
- Score the generated content strictly according to the criteria in `rules/scoring.md`.
- If the style file has scoring weight overrides, apply those priorities.
- Write the score table in the output format specified by `rules/scoring.md`.

### Pass/Regenerate Decision
- **All items 7+ points** → minimum PASS
- **Any item below 7** → FAIL → automatic regeneration
- `human-feel` below 7 → unconditional regeneration (regardless of other scores)
- `humor` + `twist` both 6 or below → regeneration
- **Average below 7.5** → PASS but **auto-improve once** before showing to user (content is acceptable but not good enough for a generation tool)
- **Average 8.0+** → EXCELLENT — target quality reached
- **Maximum 3 total attempts** (1 initial + 2 regenerations). Explicitly improve the weak areas from the previous version.
- If still below target after 3 attempts → adopt the version with the highest score and show to user.

### Result Display
Show the generated content + score table to the user:

```
---
[Generated Content]
---

| 항목 | 점수 | 근거 |
|------|------|------|
| 반전력 (twist) | 8 | ... |
| 공감도 (relatability) | 7 | ... |
| 웃음 (humor) | 7 | ... |
| 간결함 (brevity) | 8 | ... |
| 사람 느낌 (human-feel) | 8 | ... |
| **평균** | **7.6** | |
| **판정** | **PASS** | |
```

---

## Step 7. Publish Decision

### Question
```
AskUserQuestion:
  "어떻게 할까요?

  [1] 발행
  [2] 나중에 (저장만)
  [3] 수정 요청"
```

### [1] Publish
1. Read `flows/save.md` and save the post to `~/.config/feed/posts/`. (Save initially with `published: false`)
2. Read `flows/publish.md` and execute the publish logic.
   - Check `enabled` for the platform in `accounts.json`
   - `enabled = true` → interview for API publish → publish or copy to clipboard
   - `enabled = false` → automatically copy to clipboard
3. Update the post file's frontmatter based on the publish result.

### [2] Save for Later
1. Read `flows/save.md` and save the post to `~/.config/feed/posts/`. (`published: false`)
2. Display the save confirmation message and file path.
3. Inform the user that they can publish later with `/feed-publish`.

```
저장 완료: ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{HHMMSS}-{style}-{NNN}.md
나중에 /feed-publish로 발행할 수 있습니다.
```

### [3] Request Edits
1. Receive editing instructions.

```
AskUserQuestion:
  "어떤 부분을 수정할까요? 수정 방향을 알려주세요.
  (예: 반전을 더 세게, 톤을 더 가볍게, 주제를 바꿔서)"
```

2. Apply the editing instructions and return to Step 6 (content generation).
3. Repeat self-evaluation + result display for the regenerated content, then repeat Step 7.
