# Threads (Meta)

Threads platform profile. Loaded when the user selects Threads as the target platform.

---

## Spec

- **Character limit**: 500 characters
- **Format**: Text-focused (images optional, but this plugin focuses on text)
- **Hashtags**: Not commonly used (unlike X)
- **Links**: Posts containing links are deprioritized by the algorithm

## Algorithm

- **Engagement weight**: comments > reposts > likes (comments are most important)
- **Discovery**: Recommendation-based, not follower-based
- **Instagram crossover**: Emotional/relatable content performs well due to Instagram user base overlap
- **Short content wins**: 3-7 lines is the golden zone
- **Early engagement matters**: High engagement immediately after posting triggers broader distribution

## What Works

- **Emotional/relatable tone** — Content that triggers "me too" reactions
- **Self-deprecating humor** — Self-mockery + wit combination
- **Twist endings** — Structure that makes people screenshot and share
- **Comment-bait structure** — "Am I the only one?", either-or debates, open questions
- **Mobile-first vertical scroll** — One line ≤ 15 characters (Korean)
- **Late-night posting** — Late night / early morning time slots (peak emotional browsing)
- **Curated lists** — Save + share + comment trifecta. Highest follower conversion rate

## Follower Growth Formula (3 Factors)

These three conditions, when hit simultaneously, drive **follower growth** (not just likes):

1. **Overwhelming informational value** — Specific names/numbers. Not "good product recommendations" but "유니클로 수피마 코튼T" level specificity
2. **Save/share trigger** — Content worth bookmarking even if not needed right now. "I'll need this later" impulse
3. **Comment participation** — Open questions like "what else should I add?", "what do you use?" that invite responses

When all 3 align, follower growth explodes. A single post can drive thousands of new followers.

> "Writing roughly is closest to spoken language. The more you polish, the more it reads like written prose."
> — Insight validated in real Threads testing. The more casually you write, the higher the views/follows

## What Doesn't Work

- **Long posts** — If it takes 3+ scrolls, drop-off rate spikes
- **Posts with links** — Algorithm penalty on link-containing posts
- **Hashtag spam** — This isn't X
- **Formal/stiff tone** — Fails to generate empathy
- **Information-heavy posts** — Save those for Dev.to/Reddit
- **AI-sounding content** — Users are increasingly good at detecting AI-generated text

## Recommended Styles

Styles that perform well on Threads:

| File | Style | Threads Strength |
|------|-------|-----------------|
| `twist-poem.md` | Twist Poem | High shareability |
| `relatable.md` | Relatable | High comment rate |
| `one-liner.md` | One-liner | Easy to consume |
| `quote-parody.md` | Quote Parody | High repost rate |
| `self-deprecation.md` | Self-deprecation | High relatability |
| `debate.md` | Debate | Highest comment rate |
| `insight.md` | Insight | High save rate |
| `curated-list.md` | Curated List | Highest follower conversion |

## API Publishing

- **Available**: Limited (Meta Threads API)
- **Auth**: `access_token`
- **Key field in accounts.json**: `threads.access_token`
- **Note**: Subject to rate limits and content policy restrictions. Verify compliance before posting.
- **Rate Limit**: 250 API calls per user per hour. On HTTP 429, respect the `Retry-After` header.
