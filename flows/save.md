# Post Save Flow

When content generation is complete, save the post according to the rules in this file.
All posts are saved under `~/.config/feed/posts/` in frontmatter + body format.
All files MUST be written in UTF-8 encoding.

---

## File Path Convention

```
~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{HHMMSS}-{style}-{NNN}.md
```

### Path Components

| Component | Description | Example |
|-----------|-------------|---------|
| `platform` | Target platform name. Lowercase. | `threads`, `x`, `reddit`, `devto` |
| `YYYY` | Creation year (4 digits) | `2026` |
| `MM` | Creation month (2 digits, zero-padded) | `04` |
| `DD` | Creation day (2 digits, zero-padded) | `09` |
| `HHMMSS` | Creation time (24-hour, no separators) | `143022` |
| `style` | Style filename without `.md` extension | `twist-poem`, `debate`, `one-liner` |
| `NNN` | Sequence number for the same date + same style (3 digits, zero-padded) | `001`, `002`, `003` |

### Examples

```
~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
~/.config/feed/posts/x/2026/04/09/201500-one-liner-001.md
~/.config/feed/posts/reddit/2026/04/10/093000-debate-002.md
```

---

## Frontmatter Spec

Every post file begins with YAML frontmatter.

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

### Field Descriptions

#### Basic Information

| Field | Type | Description |
|-------|------|-------------|
| `platform` | string | Target platform. One of `threads`, `x`, `reddit`, `devto`. |
| `style` | string | Style used. Style filename without `.md`. (e.g., `twist-poem`, `debate`) |
| `topic` | string | Content topic. The topic keyword entered during the interview. |
| `language` | string | Writing language. `ko` or `en`. |
| `created_at` | datetime | Creation time. ISO 8601 format (`YYYY-MM-DDTHH:MM:SS`). Second precision. |

#### Publish Status

| Field | Type | Description |
|-------|------|-------------|
| `published` | boolean | Whether published. `false` on save, `true` on successful publish. |
| `published_at` | datetime \| null | Publish time. `null` if unpublished. ISO 8601 format when published. |
| `publish_method` | string \| null | Publish method. One of `null` (unpublished), `api`, `clipboard`, `failed`. |
| `publish_url` | string \| null | Published post URL. The URL returned by the platform on API publish. `null` for clipboard copy. |
| `error` | string \| null | Error message on publish failure. `null` if no error. |

#### Self-Evaluation Scores

| Field | Type | Description |
|-------|------|-------------|
| `score.twist` | integer (1-10) | Twist power. How much the last line subverts the preceding context. |
| `score.relatability` | integer (1-10) | Relatability. How much readers feel "that's so me". |
| `score.humor` | integer (1-10) | Humor. How much it actually makes people laugh. |
| `score.brevity` | integer (1-10) | Brevity. How tight and free of filler. |
| `score.human-feel` | integer (1-10) | Human feel. How much it does NOT read like AI-generated text. Most important criterion. |

#### Engagement Data

| Field | Type | Description |
|-------|------|-------------|
| `engagement.likes` | integer \| null | Like count. `null` before collection. |
| `engagement.comments` | integer \| null | Comment count. `null` before collection. |
| `engagement.reposts` | integer \| null | Repost/share count. `null` before collection. |
| `engagement.last_checked` | datetime \| null | Last engagement data collection time. ISO 8601 format. |

---

## Save Process

### Step 1. Create Directory

If the directory for the save path does not exist, create it.

```bash
mkdir -p ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/
```

Example:
```bash
mkdir -p ~/.config/feed/posts/threads/2026/04/09/
```

### Step 2. Determine Sequence Number (NNN)

Check the number of existing files with the same style in the same date directory
(`~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/`) and assign the next sequence number.

```
Existing files:
  143022-twist-poem-001.md
  150000-twist-poem-002.md

-> Next sequence number: 003
```

- Glob pattern: `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/*-{style}-*.md`
- If no matching files exist, start from `001`
- Otherwise, take the highest NNN among matched files + 1

#### Filename Collision Handling

If a file with the exact same `HHMMSS-style-NNN` already exists (e.g., two posts generated in the same second), increment NNN until a unique filename is found.

```
Target filename: 143022-twist-poem-003.md
  -> Already exists? -> Try 143022-twist-poem-004.md
  -> Already exists? -> Try 143022-twist-poem-005.md
  -> Does not exist  -> Use 143022-twist-poem-005.md
```

### Step 3. Write File

Write the frontmatter + blank line + content body as a single file.
The file MUST be encoded in UTF-8.

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

**Note:** Insert exactly one blank line after the closing `---` of the frontmatter, then write the content body.

### Step 4. Confirm Save

After saving, display the file path to the user.

```
저장 완료: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```
