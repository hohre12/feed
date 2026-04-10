# Dev.to

Platform profile for Dev.to. Loaded when the user selects Dev.to as the target platform.

---

## Spec

- **Character limit**: no character limit (articles can be any length)
- **Format**: full markdown support — headers, code blocks with syntax highlighting, embeds, images
- **Title**: required field; shown prominently in feed cards
- **Tags**: up to 4 tags per article; tags drive discoverability
- **Cover image**: optional but strongly recommended for feed visibility
- **Series**: articles can be grouped into a series for multi-part content

## Algorithm

- **Community-curated**: reactions (heart, unicorn, bookmark) + comment count determine ranking
- **Tag-based discovery**: users follow tags, not just authors — tag selection is critical
- **"Top" ranking**: weekly/monthly/yearly top posts surface in tag pages and homepage
- **Reading time matters**: articles with 3-7 min reading time perform best
- **Fresh content boost**: new articles get brief homepage visibility; early engagement extends it
- **No downvote system**: unlike Reddit, bad content just gets ignored rather than buried

## What Works

- **Tutorials** — step-by-step guides with code examples; the bread and butter of Dev.to
- **TIL (Today I Learned) posts** — short, focused learnings from actual dev work
- **Tool reviews / comparisons** — "I tried X for 30 days, here's my honest take"
- **Career advice** — salary negotiation, interview tips, remote work insights
- **Technical deep-dives** — explaining a concept or architecture decision in detail
- **"How I built X"** — project walkthroughs with real code and honest tradeoffs
- **Lists with substance** — "5 VS Code extensions that actually changed my workflow" (with explanation, not just names)
- **Opinionated takes on tech** — "Why I stopped using X" or "X is overrated, here's why"

## What Doesn't Work

- **Short posts without substance** — a 2-sentence post with no code or depth gets zero traction
- **Clickbait titles** — community is savvy; misleading titles get called out in comments
- **Non-tech content** — Dev.to is developer-focused; off-topic content gets ignored
- **Rehashed beginner content** — "What is JavaScript?" has been written 10,000 times
- **Posts without code** — dev audience expects examples, snippets, or at least technical specifics
- **Overly promotional** — "Check out my SaaS" without genuine technical value
- **AI-generated filler** — the Dev.to community is technical enough to spot AI content easily

## Recommended Styles

Styles that perform well on Dev.to:

| File | Style | Dev.to Strength |
|---|---|---|
| `insight.md` | insight | high bookmark + reaction rate; "this is useful" content |
| `curated-list.md` | curated list | high bookmark rate; tool/resource roundups perform well |

## API Publishing

- **Available**: Yes (Dev.to API / Forem API)
- **Auth**: API key (generated from user settings page)
- **Key fields in accounts.json**: `devto.api_key`
- **Required fields**: `title` (string), `body_markdown` (string)
- **Optional fields**: `tags` (array, max 4), `published` (boolean, default false = draft), `series` (string), `canonical_url` (string), `cover_image` (URL string)
- **Endpoint**: `POST /api/articles`
- **Rate limits**: 30 requests per 30 seconds
- **Note**: articles are created as drafts by default. Set `published: true` to publish immediately, or publish manually from the Dev.to dashboard after review.
