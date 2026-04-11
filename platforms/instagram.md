# Instagram (Carousel / Card News)

Platform profile for Instagram. Loaded when the user selects Instagram as the target platform.

---

## Spec

- **Format**: Carousel (card news) — minimum 2 slides, maximum 20 slides
- **Resolution**: 1080x1350px (4:5 portrait) recommended. All slides must use the same aspect ratio.
- **File format**: PNG preferred for text-heavy card news
- **File size**: Max 8MB per image
- **Caption limit**: Max 2,200 characters. Only the first 125 characters are visible before "more"
- **Hashtags**: Max 30 per post. Optimal: 3-5. Filling all 30 triggers spam detection penalty.
- **Alt text**: Up to 100 characters per image (accessibility + SEO)
- **Profile grid crop**: Square center-crop (1:1) — place key visual elements in the center of each slide

## Algorithm

- **Carousel reach boost**: Carousels reach ~1.4x more accounts than single images
- **Engagement weight**: saves > shares > comments > likes. Saves are the strongest signal.
- **Dwell time**: Carousels naturally increase time-on-post → strong ranking signal
- **Reels vs. Carousel**: Reels win on raw reach; Carousels win on follower conversion
- **Caption hook zone**: Only the first 125 characters appear before "more" — the hook must live here
- **Re-exposure mechanic**: If a post gets low engagement on first impression, Instagram shows the next slide as a re-entry point to pull the user back in

## What Works

- **Educational / informational carousels** — "Swipe to learn" format. Highest save + follower conversion.
- **List-format content** — numbered items split across slides. Save + share + comment trifecta.
- **Slide numbering** — "3 / 7" style progress indicators explicitly encourage swiping
- **Last-slide CTA** — "Follow for more", "Save this", "Drop your answer in the comments"
- **First slide as thumbnail** — the cover must work as a standalone image in the profile grid
- **Bold typography + dark backgrounds** — readability + premium aesthetic
- **Real screenshots / logos / product images on every slide** — visual proof builds credibility. Text-only slides feel empty on Instagram. Sourced images with attribution are mandatory.

## What Doesn't Work

- **Text-only slides without images** — Instagram is a visual platform. Slides with only text and a dark background look like a presentation, not a social media post. Always include screenshots, logos, or product images.
- **Text-dense slides** — ~30 characters per line, 5–7 lines per slide is the ceiling. More = drop-off.
- **Hashtag spam** — using all 30 slots signals bot behavior and triggers reach penalty
- **Inconsistent design across slides** — visual language must be uniform throughout the carousel
- **Over-polished AI aesthetic** — too clean, too symmetrical, too structured → users scroll past
- **First slide overloaded with information** — the cover is a hook only. Save content for slide 2+.

## Caption Rules

Instagram captions follow a three-part structure:

1. **Hook (first 125 chars)**: This is what users see before tapping "more". Must be a standalone statement or question that compels the tap. No preamble.
2. **Body**: Core summary of the carousel content + value proposition. Expands on the hook.
3. **CTA**: Direct, low-friction action. Examples: "저장해두고 필요할 때 꺼내 써", "빠진 거 있으면 댓글로"
4. **Hashtags**: Always at the very end, after a line break. Never mid-caption.

Tone: 반말 + 독백체. Same register as Threads — conversational, not editorial.

## Hashtag Strategy

Instagram hashtags should be selective and relevant, not exhaustive:

| Category | Count | Example |
|---|---|---|
| Category hashtags | 2–3 | `#디자인` `#디자인툴` |
| Niche hashtags | 1–2 | `#비전공자디자인` |
| Brand hashtag | 0–1 | Channel-specific tag if established |
| Trending hashtags | 0 | Off-topic trend tags incur penalty |

Total: 3–5 hashtags. Never use hashtags unrelated to the post's content.

## Recommended Styles

### Carousel (Multi-Slide)

Styles built for swipe-through card format:

| File | Style | Instagram Strength |
|---|---|---|
| `curated-list.md` | Curated List | Native carousel format. Highest save rate + follower conversion. |
| `insight.md` | Insight | Educational carousel. High save rate. |
| `debate.md` | Debate | Interactive carousel. High comment rate. |
| `self-deprecation.md` | Self-deprecation | Number-driven content maps naturally to per-slide structure. |

### Single Image

Styles that work as a single bold card:

| File | Style | Instagram Strength |
|---|---|---|
| `one-liner.md` | One-liner | Bold typography single card. |
| `quote-parody.md` | Quote Parody | Quote card format. High share rate. |
| `relatable.md` | Relatable | Single-panel empathy card. |
| `twist-poem.md` | Twist Poem | Twist must land in one screen to be effective. |

## API Publishing

- **Account requirement**: Business or Creator account only. Personal accounts cannot use the API.
- **Facebook Page link**: Required. Instagram account must be connected to a Facebook Page.
- **Image hosting**: Images must be publicly accessible URLs. Local file paths are not supported.
- **App review**: Meta app review required before production use.
- **V1 status**: API publishing is not supported in V1 of this plugin. Workflow: copy caption to clipboard + open image folder for manual upload.
- **Key fields in accounts.json**: `ig_user_id`, `access_token`, `facebook_page_id`, `enabled`
- **Endpoints**: `POST /<IG_ID>/media` (per-item container), `POST /<IG_ID>/media` with `media_type=CAROUSEL` (carousel container), `POST /<IG_ID>/media_publish` (publish)
- **API call cost**: 1 carousel post = N+2 API calls (N slides + 1 carousel container + 1 publish)
- **Rate limits**: 100 API-published posts per 24-hour window. Carousel counts as 1 post. On HTTP 429, respect the `Retry-After` header.
