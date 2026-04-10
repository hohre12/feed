# Reddit

Platform profile for Reddit. Loaded when the user selects Reddit as the target platform.

---

## Spec

- **Character limit**: no hard limit on text posts (practical limit ~40,000 chars)
- **Format**: markdown supported — headers, bold, italic, lists, code blocks, blockquotes
- **Subreddit-specific rules**: every subreddit has its own posting rules, flairs, and culture
- **Titles**: required, critically important — the title often determines if the post gets clicked
- **Links**: link posts and text posts are separate post types

## Algorithm

- **Upvote/downvote system**: community-driven ranking, not algorithmic recommendation
- **Time decay**: new posts rise fast but decay quickly — posting time matters
- **Subreddit-specific**: each subreddit is its own ecosystem with unique norms
- **"Hot" vs "New" vs "Top"**: different sort modes surface different content
- **Comment quality**: top comments matter as much as the post itself
- **Karma thresholds**: some subreddits require minimum karma to post
- **Anti-spam filters**: new accounts and self-promotional content get auto-flagged

## What Works

- **Genuine value posts** — teach something, share a real experience, provide useful data
- **Specific subreddit targeting** — tailor tone and content to the subreddit's culture
- **Detailed content** — Reddit rewards depth; long-form posts with structure perform well
- **Debate/discussion starters** — "What's your take on X?" or unpopular opinion posts
- **Personal stories with lessons** — "I did X, here's what happened and what I learned"
- **Tool/resource lists** — curated recommendations backed by personal experience
- **Show HN-style posts** — "I built this" with genuine context about the problem and solution
- **AMA-style engagement** — being responsive in comments boosts the post significantly

## What Doesn't Work

- **Self-promotion** — Reddit users sniff out marketing instantly; community value must come first
- **Low-effort posts** — one-liners without substance get downvoted to oblivion
- **Cross-posting spam** — posting the same thing to 10 subreddits = ban risk
- **Clickbait titles** — Reddit users punish misleading titles with downvotes
- **Corporate/polished tone** — anything that sounds like a press release or ad
- **Ignoring subreddit rules** — auto-removed, and repeated violations lead to bans
- **AI-generated content** — many subreddits actively ban AI content; detection is high
- **Hashtags** — hashtags have zero function on Reddit; using them signals outsider

## Recommended Styles

Styles that perform well on Reddit:

| File | Style | Reddit Strength |
|---|---|---|
| `insight.md` | insight | high upvote + save rate in technical subreddits |
| `debate.md` | debate | drives massive comment threads |
| `curated-list.md` | curated list | high save rate, "saving this" comments |
| `self-deprecation.md` | self-deprecation | works in casual/humor subreddits |

## API Publishing

- **Available**: Yes (Reddit API via PRAW — Python Reddit API Wrapper)
- **Auth**: OAuth2 (script app or web app flow)
- **Key fields in accounts.json**: `reddit.client_id`, `reddit.client_secret`, `reddit.username`, `reddit.password`, `reddit.user_agent`
- **Subreddit selection**: required — must specify target subreddit(s) before publishing
- **Rate limits**: ~60 requests per minute; new accounts have stricter limits
- **Note**: Reddit API now requires paid access for higher-volume usage. Always respect subreddit rules and Reddit's content policy. PRAW handles most auth complexity.
