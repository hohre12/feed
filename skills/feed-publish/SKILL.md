---
name: feed-publish
description: Publishes unpublished posts (via API or clipboard copy)
disable-model-invocation: true
argument-hint: <today|all>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion
---

# /feed-publish — Publish Unpublished Posts

This skill finds unpublished posts (`published: false`) saved in `~/.config/feed/posts/` and publishes them.

---

## Step 1. Environment Check

Check whether the `~/.config/feed/` directory exists.

- If it does not exist:
  ```
  먼저 /feed로 포스트를 생성해주세요.
  ```
  Output the message and **terminate immediately**.

---

## Step 2. Argument Parsing

Parse `$ARGUMENTS`.

| Argument | Behavior |
|----------|----------|
| `today` | Filter posts matching today's date (`YYYY/MM/DD`) only |
| `all` | Show all unpublished posts |
| (empty) | Show all unpublished posts (same as `all`) |

Today's date is determined from the current time (run `date +%Y/%m/%d` via Bash).

---

## Step 3. Search for Unpublished Posts

### 3-1. File Search

Use Glob to search for all post files under `~/.config/feed/posts/`.

Posts exist in two formats:
- **Text posts**: single `.md` files (e.g., `143022-twist-poem-001.md`)
- **Visual posts**: directories containing `post.md` (e.g., `143022-curated-list-001/post.md`)

Search for both:

- For `today` argument:
  ```
  ~/.config/feed/posts/*/{YYYY}/{MM}/{DD}/*.md
  ~/.config/feed/posts/*/{YYYY}/{MM}/{DD}/*/post.md
  ```
  (substitute with today's date)

- For `all` or empty value:
  ```
  ~/.config/feed/posts/**/*.md
  ```
  This catches both `{name}.md` files and `{name}/post.md` files.

**Post identifier extraction:**
- Text posts: identifier = filename without `.md` (e.g., `143022-twist-poem-001`)
- Visual posts: identifier = parent directory name (e.g., `143022-curated-list-001`)

### 3-2. Filter Unpublished

Read the frontmatter of each found file and collect only those matching these conditions:

- `published: false` **or**
- `publish_method: failed` (retry targets for failed publishes)

Extract the following frontmatter fields from each file:
- `platform`
- `style`
- `topic`
- `created_at`
- `publish_method` (to check for failed status)

### 3-3. No Results

If there are no unpublished posts:
```
발행할 포스트가 없습니다.
```
Output the message and **terminate immediately**.

---

## Step 4. Display Unpublished List

Display the collected unpublished posts as a numbered list.
Sort in ascending order by `created_at` (oldest first).

### Display Format

```
미발행 포스트 목록:

[1] threads / twist-poem / 알람 / 2026-04-09 14:30
[2] x / one-liner / 커피 / 2026-04-09 20:15
[3] threads / debate / 재택근무 / 2026-04-08 09:30  ⚠️ 발행 실패 (재시도)
```

- Format: `[number] {platform} / {style} / {topic} / {date+time from created_at}`
- Posts with `publish_method: failed` get ` ⚠️ 발행 실패 (재시도)` appended.

---

## Step 5. User Selection

Use AskUserQuestion to let the user select which posts to publish.

```
발행할 포스트를 선택하세요 (번호를 쉼표로 구분, 또는 "all"):
```

### Input Handling

| Input | Behavior |
|-------|----------|
| `1` | Publish post #1 only |
| `1,3` or `1, 3` | Publish posts #1 and #3 |
| `all` | Publish all |

If invalid numbers are included, process only valid numbers and ignore invalid ones.
If no valid numbers remain, output "유효한 선택이 없습니다." and terminate.

---

## Step 6. Execute Publish

For each selected post, execute the following logic in order.

### 6-0. Load Publish Flow

Read `flows/publish.md` from the plugin directory using Read to reference the publish logic.
(Relative path from plugin root: `flows/publish.md`)

### 6-1. Check Account Settings

Read `~/.config/feed/config/accounts.json` using Read.

- If the file does not exist → treat all platforms as `enabled: false`.
- Check the `enabled` value for the post's `platform`.

### 6-2. Publish Branching

#### Case A: `enabled = true` (API Publish Available)

Ask via AskUserQuestion:
```
[{platform}] 자동 발행할까요?
[1] 예 (API 발행)
[2] 아니오 (클립보드 복사)
```

- **[1] Yes** → Attempt API publish.
  - Execute according to the platform-specific API publish section in `flows/publish.md`.
  - Use `curl` etc. via Bash for API calls.
  - **On success:**
    - Update frontmatter:
      ```yaml
      published: true
      published_at: {current ISO 8601 timestamp}
      publish_method: api
      publish_url: {URL returned by the platform}
      error: null
      ```
  - **On failure:**
    - Display the error message.
    - Automatically fall back to clipboard copy (handle same as Case B).
    - Update frontmatter:
      ```yaml
      published: false
      published_at: null
      publish_method: failed
      publish_url: null
      error: "{error message}"
      ```
    - Message:
      ```
      발행에 실패했습니다: {에러 메시지}
      대신 클립보드에 복사했습니다. 직접 붙여넣기 해주세요.
      ```

- **[2] No** → Proceed to Case B.

#### Case B: `enabled = false` or User Selected Clipboard

Copy to clipboard.

**For text posts:**

1. Extract **body only** from the post file, excluding frontmatter.
   - The body is everything after the second `---`.

**For visual posts (Instagram):**

1. Extract the **`## Caption`** section from `post.md` (not the full body).
2. Copy caption text to clipboard via `pbcopy`.
3. Open the post directory in Finder: `open {post_directory_path}`
4. Display slide file paths and manual upload instructions.

**For text posts (continued):**

2. Use `pbcopy` via Bash to copy to clipboard:
   ```bash
   # Copy body content using pbcopy
   ```

3. Update frontmatter:
   ```yaml
   published: true
   published_at: {current ISO 8601 timestamp}
   publish_method: clipboard
   publish_url: null
   error: null
   ```

4. Display a preview of the first 2-3 lines of the copied content:
   ```
   클립보드에 복사했습니다. 직접 붙여넣기 해주세요.

   복사된 내용:
   ---
   안 보면 될 걸
   자꾸 확인하고
   ...
   ---
   ```

### 6-3. Frontmatter Update

Use the Write tool to rewrite the entire post file to update the frontmatter.
Do not modify the body. Only update publish-related fields in the frontmatter.

---

## Step 7. Publish Summary

After all selected posts have been published, display a summary.

```
발행 완료 요약:

✅ [1] threads / twist-poem / 알람 → API 발행 (https://threads.net/...)
✅ [2] x / one-liner / 커피 → 클립보드 복사
❌ [3] threads / debate / 재택근무 → 발행 실패 (401 Unauthorized)

발행: 2건 | 실패: 1건
```

### Summary Format

- Success (API): `✅ [{number}] {platform} / {style} / {topic} → API 발행 ({URL})`
- Success (clipboard): `✅ [{number}] {platform} / {style} / {topic} → 클립보드 복사`
- Failure: `❌ [{number}] {platform} / {style} / {topic} → 발행 실패 ({error})`

The final line shows total publish count and failure count.

---

## Notes

- When copying multiple posts to clipboard, only the last post remains in the clipboard. Notify the user at each copy:
  ```
  ⚠️ 여러 포스트를 클립보드 복사하면 마지막 포스트만 클립보드에 남습니다.
  각 포스트를 복사한 뒤 바로 붙여넣기 해주세요.
  ```
- `pbcopy` is macOS only. Currently only macOS is supported.
- If API tokens/keys in `accounts.json` are empty, treat as failure even if `enabled: true`.
