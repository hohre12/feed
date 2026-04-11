# Publish Flow

When the user selects publish, process it according to the logic in this file.
The target post must already be saved in `~/.config/feed/posts/` with `published: false`.

---

## Publish Branching Logic

```
User: selects [Publish]
  |
  +-- Post is saved in ~/.config/feed/posts/ (published: false)
  |
  +-- Read ~/.config/feed/config/accounts.json
  |   Check the enabled value for the target platform
  |
  +-- enabled = true:
  |   Ask: "자동 발행할까요? [1] 예 [2] 아니오 (클립보드 복사)"
  |   +-- [1] Yes -> API publish -> update frontmatter
  |   |         (published: true, publish_method: api, publish_url: ...)
  |   +-- [2] No -> copy to clipboard -> update frontmatter
  |                 (published: true, publish_method: clipboard)
  |
  +-- enabled = false:
      -> automatically copy to clipboard -> update frontmatter
        (published: true, publish_method: clipboard)
      -> Message: "클립보드에 복사했습니다. 직접 붙여넣기 해주세요."
```

---

## Clipboard Copy

### Method

Use the macOS `pbcopy` command to copy only the content body to the clipboard.
**Exclude the frontmatter** and copy only the pure content body.

```bash
# Extract only the body (after the second ---) from the post file and copy
# (content after the second ---)
```

### Post-Copy Message

```
클립보드에 복사했습니다. 직접 붙여넣기 해주세요.

복사된 내용:
---
안 보면 될 걸
자꾸 확인하고
...
---
```

Show a preview of the first 2-3 lines of the copied content.

---

## API Publishing (Per Platform)

### Threads (Meta Threads API)

- **Auth**: `access_token` (`threads.access_token` from accounts.json)
- **Method**: POST request to create a text container, then publish
- **Endpoint**: Meta Graph API (`https://graph.threads.net/v1.0/`)
- **Note**: Subject to rate limits and content policy restrictions. Verify the 500-character limit.
- **Rate Limit**: 250 API calls per user per hour. If rate-limited (HTTP 429), wait and retry after the period indicated in the `Retry-After` header. Do NOT retry immediately.

### X (Twitter API v2)

- **Auth**: OAuth 1.0a (`x.api_key`, `x.api_secret`, `x.access_token`, `x.access_secret` from accounts.json)
- **Method**: POST tweet request
- **Endpoint**: `https://api.twitter.com/2/tweets`
- **Note**: Verify character limits (280 chars English / ~140 chars Korean).
- **Rate Limit**: 200 tweets per 15-minute window per user (app-level limits also apply). On HTTP 429, read the `x-rate-limit-reset` header for the UNIX timestamp when the limit resets. Fall back to clipboard copy if the reset time is more than 5 minutes away.

### Reddit (Reddit API -- PRAW pattern)

- **Auth**: OAuth2 (`reddit.client_id`, `reddit.client_secret`, `reddit.username`, `reddit.password` from accounts.json)
- **Method**: POST request to submit to a subreddit
- **Endpoint**: `https://oauth.reddit.com/api/submit`
- **Note**: Requires subreddit selection. Confirm the target subreddit via interview at publish time.
- **Rate Limit**: 100 requests per minute per OAuth client. Reddit returns HTTP 429 with a `Retry-After` header in seconds. Respect the delay before retrying. Some subreddits also enforce posting cooldowns (typically 10 minutes between posts).

### Dev.to (Dev.to REST API)

- **Auth**: API Key (`devto.api_key` from accounts.json)
- **Method**: POST article request
- **Endpoint**: `https://dev.to/api/articles`
- **Note**: The `title` field is required. Confirm the title via interview at publish time.
- **Rate Limit**: 30 requests per 30 seconds per API key. On HTTP 429, wait at least 30 seconds before retrying. Dev.to also enforces a limit of approximately 30 articles per day.

### Instagram (V1 — Manual Upload Only)

Instagram API publishing requires a Business/Creator account, Facebook Page link, and publicly hosted images. These barriers are too high for V1.

**V1 workflow:**

1. Extract the `## Caption` section from `post.md` and copy to clipboard via `pbcopy`.
2. Open the post directory in Finder:
   ```bash
   open ~/.config/feed/posts/instagram/{YYYY}/{MM}/{DD}/{HHMMSS}-{style}-{NNN}/
   ```
3. Display instructions:
   ```
   캡션이 클립보드에 복사되었습니다.

   1. 인스타그램 앱에서 새 게시물 → 캐러셀 선택
   2. 아래 폴더에서 slide-1.png ~ slide-N.png를 순서대로 선택
   3. 캡션에 붙여넣기 (Cmd+V)
   4. 공유

   폴더: ~/.config/feed/posts/instagram/...
   ```
4. Update frontmatter: `published: true`, `publish_method: clipboard`

---

## Failure Handling

### On API Publish Failure

```
On API publish failure:
  |
  +-- Display error message
  |   "발행에 실패했습니다: {error message}"
  |
  +-- Automatically copy to clipboard (fallback)
  |   "대신 클립보드에 복사했습니다. 직접 붙여넣기 해주세요."
  |
  +-- Update frontmatter
      publish_method: failed
      error: {error message}
      published: false (remains false since publish failed)
```

### Retry

- Failed posts can be retried later with `/feed-publish`.
- Posts with `publish_method: failed` are included in the unpublished list.

---

## Frontmatter Update

Update the post file's frontmatter based on the publish result.

**CRITICAL:** `published_at` must always be a fresh timestamp captured at the moment of publishing. NEVER copy the value from `created_at`. Always generate a new timestamp using `Bash(date +%Y-%m-%dT%H:%M:%S)`.

### API Publish Success

```yaml
published: true
published_at: 2026-04-09T14:31:05    # Timestamp at publish completion (from `date` command)
publish_method: api
publish_url: https://threads.net/...  # URL returned by the platform
error: null
```

### Clipboard Copy (Manual Publish)

```yaml
published: true
published_at: 2026-04-09T15:31:05    # Timestamp at clipboard copy (fresh from `date` command, NOT from created_at!)
publish_method: clipboard
publish_url: null                     # No URL for manual publish
error: null
```

### API Publish Failure

```yaml
published: false                      # Remains false since publish failed
published_at: null
publish_method: failed
publish_url: null
error: "401 Unauthorized: Invalid access token"  # Actual error message
```

---

## Completion Messages

### On API Publish Success

```
발행 완료!
URL: https://threads.net/...
파일: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```

### On Clipboard Copy

```
클립보드에 복사했습니다. 직접 붙여넣기 해주세요.
파일: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```

### On Publish Failure

```
발행에 실패했습니다: 401 Unauthorized: Invalid access token
대신 클립보드에 복사했습니다. 직접 붙여넣기 해주세요.

나중에 /feed-publish로 재시도할 수 있습니다.
파일: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```
