# X (formerly Twitter)

Platform profile for X. Loaded when the user selects X as the target platform.

---

## Spec

- **Character limit**: 280 characters (English), ~140 characters (Korean due to wider chars)
- **Format**: text-first, supports images, polls, and link cards
- **Hashtags**: 1-2 relevant hashtags max; more than that looks spammy
- **Links**: link cards show preview; algorithm doesn't heavily penalize links (unlike Threads)
- **Threads**: multi-post threads supported natively for long-form content

## Algorithm

- **Engagement weighting**: retweets/reposts > quote tweets > replies > likes
- **Trending topics**: riding a trending wave = massive reach; timing matters
- **Quote tweet culture**: strong — adding your own spin on someone's post is a core mechanic
- **Recommendation-based feed**: "For You" tab pushes content beyond followers
- **Early velocity matters**: first 30 min of engagement determines reach
- **Dwell time**: posts people stop scrolling to read get boosted

## What Works

- **Hot takes** — bold, slightly controversial opinions that trigger replies and quote tweets
- **Threads (multi-post)** — numbered 1/n threads for deep dives, tutorials, lists
- **News commentary** — first-mover advantage on breaking news/trends
- **One-liners** — punchy, screenshot-worthy single sentences
- **Tech insights** — "TIL" moments, tool discoveries, dev opinions
- **Ratio bait (controlled)** — saying something slightly provocative to spark debate
- **Self-deprecation** — relatable failures, "it me" content
- **Curated lists as threads** — "10 tools I actually use daily" format

## What Doesn't Work

- **Long paragraphs in a single tweet** — wall of text in 280 chars looks bad
- **Excessive hashtags** — more than 2 feels like a bot or a marketer
- **Generic motivational quotes** — "hustle harder" content is dead
- **Thread cop-outs** — "thread (1/n)" that could have been one tweet
- **Formal corporate tone** — reads like a press release
- **Engagement bait without substance** — "Like if you agree" with nothing else
- **AI-sounding content** — "In this thread, I will explore..." instant unfollow energy

## Recommended Styles

Styles that perform well on X:

| File | Style | X Strength |
|---|---|---|
| `one-liner.md` | one-liner | highest screenshot + retweet potential |
| `debate.md` | debate/hot take | drives quote tweets and replies |
| `insight.md` | insight | high bookmark/save rate |
| `twist-poem.md` | twist-poem | surprise factor, high shareability |
| `self-deprecation.md` | self-deprecation | "it me" relatability, high retweet |
| `curated-list.md` | curated list (as thread) | high bookmark + follow conversion |

## API Publishing

- **Available**: Yes (Twitter API v2)
- **Auth**: OAuth 1.0a (user context) or OAuth 2.0 (app context)
- **Key fields in accounts.json**: `x.api_key`, `x.api_secret`, `x.access_token`, `x.access_token_secret`
- **Endpoints**: `POST /2/tweets` for single tweets, thread creation via reply chains
- **Rate limits**: 300 tweets per 3 hours (user), 200 per 15 min (app). Check current limits before publishing.
- **Note**: Free tier API access is heavily restricted. Paid tiers (Basic/Pro/Enterprise) required for meaningful publishing.
