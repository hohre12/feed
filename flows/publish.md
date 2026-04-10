# 발행 플로우

사용자가 발행을 선택하면 이 파일의 로직에 따라 처리한다.
발행 대상 포스트는 이미 `~/.config/feed/posts/`에 `published: false` 상태로 저장되어 있어야 한다.

---

## 발행 분기 로직

```
사용자: [발행] 선택
  │
  ├── 포스트가 ~/.config/feed/posts/에 저장됨 (published: false)
  │
  ├── ~/.config/feed/config/accounts.json 읽기
  │   해당 플랫폼의 enabled 값 확인
  │
  ├── enabled = true:
  │   질문: "자동 발행할까요? [1] 예 [2] 아니오 (클립보드 복사)"
  │   ├── [1] 예 → API 발행 → frontmatter 업데이트
  │   │         (published: true, publish_method: api, publish_url: ...)
  │   └── [2] 아니오 → 클립보드 복사 → frontmatter 업데이트
  │                   (published: true, publish_method: clipboard)
  │
  └── enabled = false:
      → 클립보드에 자동 복사 → frontmatter 업데이트
        (published: true, publish_method: clipboard)
      → 메시지: "클립보드에 복사했습니다. 직접 붙여넣기 해주세요."
```

---

## 클립보드 복사

### 방법

macOS의 `pbcopy` 명령어로 콘텐츠 본문만 클립보드에 복사한다.
**frontmatter는 제외**하고 순수 콘텐츠 본문만 복사한다.

```bash
# 포스트 파일에서 frontmatter를 제외한 본문만 추출하여 복사
# (두 번째 --- 이후의 내용)
```

### 복사 후 안내

```
클립보드에 복사했습니다. 직접 붙여넣기 해주세요.

복사된 내용:
---
안 보면 될 걸
자꾸 확인하고
...
---
```

복사된 내용의 첫 2~3줄을 미리보기로 보여준다.

---

## API 발행 (플랫폼별)

### Threads (Meta Threads API)

- **인증**: `access_token` (accounts.json의 `threads.access_token`)
- **방식**: POST 요청으로 텍스트 컨테이너 생성 → 게시
- **엔드포인트**: Meta Graph API (`https://graph.threads.net/v1.0/`)
- **주의**: rate limit과 콘텐츠 정책 제한이 있음. 500자 제한 확인 필요.

### X (Twitter API v2)

- **인증**: OAuth 1.0a (accounts.json의 `x.api_key`, `x.api_secret`, `x.access_token`, `x.access_secret`)
- **방식**: POST tweet 요청
- **엔드포인트**: `https://api.twitter.com/2/tweets`
- **주의**: 글자 수 제한 (280자 영문 / 140자 한글 기준) 확인 필요.

### Reddit (Reddit API — PRAW 패턴)

- **인증**: OAuth2 (accounts.json의 `reddit.client_id`, `reddit.client_secret`, `reddit.username`, `reddit.password`)
- **방식**: subreddit에 POST 요청으로 게시
- **엔드포인트**: `https://oauth.reddit.com/api/submit`
- **주의**: subreddit 선택이 필요함. 발행 시 대상 subreddit을 인터뷰로 확인.

### Dev.to (Dev.to REST API)

- **인증**: API Key (accounts.json의 `devto.api_key`)
- **방식**: POST article 요청
- **엔드포인트**: `https://dev.to/api/articles`
- **주의**: title 필드가 필수. 발행 시 제목을 인터뷰로 확인.

---

## 실패 처리

### API 발행 실패 시

```
API 발행 실패 시:
  │
  ├── 에러 메시지 표시
  │   "발행에 실패했습니다: {에러 메시지}"
  │
  ├── 자동으로 클립보드 복사 (폴백)
  │   "대신 클립보드에 복사했습니다. 직접 붙여넣기 해주세요."
  │
  └── frontmatter 업데이트
      publish_method: failed
      error: {에러 메시지}
      published: false (발행 실패이므로 false 유지)
```

### 재시도

- 발행에 실패한 포스트는 나중에 `/feed-publish`로 재시도할 수 있다.
- `publish_method: failed`인 포스트도 미발행 목록에 포함된다.

---

## Frontmatter 업데이트

발행 결과에 따라 포스트 파일의 frontmatter를 업데이트한다.

### API 발행 성공

```yaml
published: true
published_at: 2026-04-09T14:31:05    # 발행 완료 시각
publish_method: api
publish_url: https://threads.net/...  # 플랫폼이 반환한 URL
error: null
```

### 클립보드 복사 (수동 발행)

```yaml
published: true
published_at: 2026-04-09T15:31:05    # 클립보드 복사 시각 (현재 시각을 새로 찍을 것. created_at을 복사하지 말 것!)
publish_method: clipboard
publish_url: null                     # 수동이므로 URL 없음
error: null
```

**주의:** `published_at`은 반드시 발행 시점의 현재 시각을 새로 가져와야 한다. `created_at` 값을 그대로 복사하면 안 된다. `Bash(date +%Y-%m-%dT%H:%M:%S)`로 현재 시각을 구한다.

### API 발행 실패

```yaml
published: false                      # 실패이므로 false 유지
published_at: null
publish_method: failed
publish_url: null
error: "401 Unauthorized: Invalid access token"  # 실제 에러 메시지
```

---

## 발행 완료 메시지

### API 발행 성공 시

```
발행 완료!
URL: https://threads.net/...
파일: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```

### 클립보드 복사 시

```
클립보드에 복사했습니다. 직접 붙여넣기 해주세요.
파일: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```

### 발행 실패 시

```
발행에 실패했습니다: 401 Unauthorized: Invalid access token
대신 클립보드에 복사했습니다. 직접 붙여넣기 해주세요.

나중에 /feed-publish로 재시도할 수 있습니다.
파일: ~/.config/feed/posts/threads/2026/04/09/143022-twist-poem-001.md
```
