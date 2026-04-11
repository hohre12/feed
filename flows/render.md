# Render Flow

When text content generation is complete (Step 6 done), convert it to visual card-news images (PNG) according to the rules in this file.
All rendered images are saved under `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/`.

---

## Prerequisites

### Playwright MCP Check

Before starting the render pipeline, verify Playwright MCP is available.

```
If Playwright MCP is NOT connected:
  -> Display message:
     "Playwright MCP가 필요합니다.
      설치 명령: claude mcp add playwright npx @playwright/mcp@latest
      재시작 후 다시 시도해주세요.
      HTML 파일만 저장하고 종료합니다."
  -> Save completed HTML files to ~/.config/feed/tmp/
  -> Stop (do not proceed to screenshot step)
```

### Temp Directory

Ensure the tmp directory exists before writing any HTML files.

```bash
mkdir -p ~/.config/feed/tmp/
```

---

## Section 1: Content-to-Slide Mapping

Split generated text content into individual slides based on the selected style.

### Style-Based Split Rules

#### `curated-list`

| Slide | Content |
|-------|---------|
| 1 | Cover — title + subtitle |
| 2 ~ N | 2–3 list items per slide |
| Last | CTA — comment prompt + follow |

#### `insight`

| Slide | Content |
|-------|---------|
| 1 | Cover — core claim or hook question |
| 2 | Background / problem setup |
| 3 ~ N | Key points — one point per slide |
| Last | CTA |

#### `debate`

| Slide | Content |
|-------|---------|
| 1 | Cover — debate question |
| 2 | Position A |
| 3 | Position B |
| Last | CTA — vote via comment |

#### `self-deprecation`

| Slide | Content |
|-------|---------|
| 1 | Cover — title |
| 2 ~ N | Number + self-deprecation point, one per slide |
| Last | CTA |

#### Single-Image Styles

The following styles render as a **single slide** (not a carousel):

| Style | Notes |
|-------|-------|
| `one-liner` | Single card, centered text |
| `quote-parody` | Single card, quote layout |
| `relatable` | Single card, short body |
| `twist-poem` | Single card, poem layout |

### Per-Slide Text Limits

Exceeding these limits requires splitting the slide.

| Element | Limit |
|---------|-------|
| Title | Max 20 characters (Korean) |
| Body per line | Max 30 characters (Korean) |
| Body lines per slide | 5–7 lines |

---

## Section 2: Image Sourcing (MANDATORY for Instagram)

Image sourcing is **mandatory** for Instagram carousel posts. Every carousel must include sourced images — screenshots, logos, or product visuals. Text-only slides without any visual element are not acceptable for Instagram.

### What to Source (Required)

- **Cover slide**: Background image related to the topic (screenshot, hero image, or product visual)
- **Content slides**: Tool/product screenshots, logos, or UI captures for each recommended item
- **CTA slide**: No image needed (uses decorative CSS background)

### Sourcing Steps

```
1. WebSearch: "{tool/product name} screenshot" or "{tool name} UI" for each item
2. WebSearch: "{topic} {related keyword}" for cover background
3. Collect image URLs from search results
4. Store each image URL with its source domain
5. Append attribution to slide: "*출처:{domain}" at bottom-right
6. For curated-list style: source at least 1 image per slide (tool screenshot or logo)
```

### Image Placement

| Slide Type | Image Placement | Required |
|------------|----------------|----------|
| Cover | Full background with gradient overlay | **Yes** |
| Content (list-item) | Top portion of slide or beside item | **Yes** — at least 1 per slide |
| Content (text/insight) | Background or inline | **Yes** |
| CTA | None (CSS decorative background) | No |

### Fallback on Sourcing Failure

If a specific image URL fails to load at render time:

```
Primary fallback   -> Try alternative search query for the same item
Secondary fallback -> Use tool/brand logo instead of screenshot
Last resort        -> CSS gradient background with tool name in large text
```

**Note**: The fallback is for individual image failures only. You must still attempt to source images for every content slide. Skipping image sourcing entirely is not allowed.

### Image Quality Requirements

| Criterion | Requirement |
|-----------|-------------|
| Minimum width | 800px |
| Accepted formats | JPG, PNG, WebP |
| Watermarks | Exclude watermarked images |

### Image Validation (MANDATORY — Do Not Skip)

Every sourced image MUST be validated before use. An invalid image in the final output is worse than no image at all.

#### Step 1: File Integrity Check

After downloading/sourcing each image, verify it is a valid image file:

```
Bash: file {image_path}
```

- Must contain "image data" (PNG, JPEG, WebP) — NOT "HTML document", "XML", "ASCII text"
- If the file is HTML/text → the download was blocked or redirected. **Re-source immediately** with a different URL or search query.

#### Step 2: Visual Content Check

After confirming the file is a valid image, **read the image using the Read tool** to visually inspect it:

```
Read: {image_path}
```

Check for these failure conditions:

| Condition | Action |
|-----------|--------|
| Image is mostly white/blank/empty | **Re-source** — this is likely a logo on transparent background or a failed capture |
| Image shows a login page, CAPTCHA, or security check | **Re-source** — the website blocked the request |
| Image shows a cookie consent popup covering the content | **Re-source** — the actual content is hidden |
| Image is a generic placeholder or icon (< 100x100 effective content) | **Re-source** — too small to be meaningful |
| Image content is unrelated to the tool/topic | **Re-source** — wrong image |

#### Step 3: Resolution Check

```
Bash: file {image_path} | grep -oE '[0-9]+ x [0-9]+'
```

- Width must be ≥ 800px for background images
- Width must be ≥ 400px for inline/thumbnail images
- If below threshold → re-source with "{tool name} screenshot HD" or "{tool name} hero image"

#### Step 4: Re-source Protocol

When an image fails validation:

```
1. Try: WebSearch "{tool name} screenshot 2026"
2. Try: WebSearch "{tool name} UI interface preview"
3. Try: WebFetch on the tool's official website → extract og:image URL
4. Try: WebSearch "{tool name} review" → find blog posts with embedded screenshots
5. Last resort: Use CSS gradient background with tool name + emoji as visual placeholder
```

**Maximum 3 re-source attempts per image.** After 3 failures, use the last-resort CSS fallback and log a warning.

#### Validation Summary

All image validation must complete BEFORE the render pipeline starts. Do not render slides with unvalidated images. The validation checklist:

- [ ] File is a valid image format (not HTML/XML/text)
- [ ] Visual content is meaningful (not blank/login/CAPTCHA)
- [ ] Resolution meets minimum threshold
- [ ] Content is relevant to the tool/topic

---

## Section 3: Template Variable Injection

### Variable Syntax

All template variables use double curly braces: `{{variable_name}}`

### Common Variables (All Slides)

| Variable | Description | Example |
|----------|-------------|---------|
| `{{brand_name}}` | Channel name | `짐코딩` |
| `{{brand_tagline}}` | Slogan / hashtag | `#AI세컨드브레인` |
| `{{accent_color}}` | Accent color hex | `#58a6ff` |
| `{{theme}}` | Theme class | `dark` or `light` |
| `{{slide_number}}` | Current slide number (zero-padded) | `02` |
| `{{total_slides}}` | Total slide count (zero-padded) | `07` |
| `{{source_credit}}` | Image attribution | `*출처:vibeindex.ai` |

### Cover Slide Variables

| Variable | Description |
|----------|-------------|
| `{{cover_title}}` | Main headline |
| `{{cover_subtitle}}` | Subtitle / description |
| `{{cover_label}}` | Top label badge (e.g., `AI NEWS`) |
| `{{cover_image_url}}` | Background image URL |
| `{{author_line}}` | Author attribution line (empty string if not needed) |

### Content Slide Variables

| Variable | Description |
|----------|-------------|
| `{{content_title}}` | Slide headline |
| `{{content_body}}` | Body HTML (use `<br>` for line breaks) |
| `{{content_image_url}}` | Slide image URL (empty string if none) |
| `{{layout_variant}}` | Layout type: `list-item`, `text`, `number`, `quote` |

### CTA Slide Variables

| Variable | Description |
|----------|-------------|
| `{{cta_title}}` | CTA main message |
| `{{cta_subtitle}}` | CTA supporting line |
| `{{cta_button}}` | Button label text |
| `{{cta_emoji}}` | Emoji icon above CTA text (empty string hides the element) |

### Injection Steps

```
1. Read template HTML file (cover / content / cta)
2. Replace all {{variable}} tokens with actual values (string substitution)
3. Write completed HTML to ~/.config/feed/tmp/slide-{n}.html
4. Proceed to render
5. Delete tmp HTML files after all screenshots are captured
```

---

## Section 4: Playwright Render Pipeline

### Output Path Convention

```
~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/slide-{n}.png
```

Where `{id}` matches the post filename without extension (e.g., `143022-curated-list-001`).

```bash
mkdir -p ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/
```

### Viewport and Resolution

Render at 2x DPR (Device Pixel Ratio) for retina-quality output.

| Setting | Value |
|---------|-------|
| Viewport width | 540px |
| Viewport height | 675px |
| Device Scale Factor | 2 |
| Output image size | 1080px × 1350px |

### Render Loop

Run the following sequence for each slide (slide index `n` starts at 1):

```
For each slide n of total_slides:

  1. Select template type
       n == 1            -> cover template
       n == total_slides -> cta template
       otherwise         -> content template

  2. Inject variables into template HTML

  3. Write to ~/.config/feed/tmp/slide-{n}.html

  4. Start local HTTP server (if not already running)
       Port: 18923 (fixed). If busy, increment by 1 until available (max 18933).
       Bash: python3 -m http.server 18923 --directory ~/.config/feed/tmp/ &

  5. Playwright: browser_navigate
       url: http://localhost:18923/slide-{n}.html

  6. Playwright: browser_evaluate — set viewport style
       function: "() => { Object.assign(document.documentElement.style, { width: '540px', height: '675px' }); }"

  7. Wait for fonts + images — max 3 seconds
       Playwright: browser_evaluate
       function: "() => document.fonts.ready.then(() => true)"
       If this does not resolve within 3 seconds, proceed anyway (system fonts will be used)

  8. Playwright: browser_take_screenshot
       path: ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/slide-{n}.png

  9. Advance to next slide

After all slides:
  Stop HTTP server (kill the python3 process)
  Delete all ~/.config/feed/tmp/slide-*.html files
```

### Font Loading

Load Korean web fonts via Google Fonts CDN in each template's `<head>`.

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
```

Always wait for `document.fonts.ready` before taking the screenshot.

---

## Section 5: Preview

After all slides are rendered, show results to the user before finalizing.

```
Render complete:

  1. Open first slide in macOS Preview:
       Bash: open ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/slide-1.png

  2. Print the full file path list to terminal:
       slide-1.png: ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/slide-1.png
       slide-2.png: ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/slide-2.png
       ...

  3. AskUserQuestion:
       "미리보기를 확인하셨나요?

       [1] 확인 (저장 진행)
       [2] 수정 요청
       [3] 취소"
```

### After Preview Decision

| Choice | Action |
|--------|--------|
| `[1]` | Proceed to save flow (update frontmatter with `render_path`) |
| `[2]` | Ask for edit instructions → re-run render pipeline from Section 1 |
| `[3]` | Delete rendered PNGs + tmp files → return to Step 7 of interview |

---

## Section 6: Error Handling

| Situation | Action |
|-----------|--------|
| Playwright MCP not installed | Save HTML only + display install instructions (see Prerequisites) |
| Image URL fails to load | Use CSS gradient fallback; continue rendering |
| Font fails to load | Fall back to system font (`Apple SD Gothic Neo, sans-serif`) |
| Screenshot fails on a slide | Retry once → if still failing, preserve HTML for that slide and continue |
| Disk space insufficient | Display error message + abort entire render |

### Retry Logic for Screenshot Failure

```
On screenshot failure for slide n:
  -> Wait 1 second
  -> Retry browser_take_screenshot once
  -> If retry fails:
       Save ~/.config/feed/tmp/slide-{n}.html as fallback artifact
       Log: "슬라이드 {n} 캡처 실패 — HTML 보존됨"
       Continue to slide n+1
```

---

## Section 7: Frontmatter Update

After successful render, update the post file's frontmatter to record image paths.

Add the following field to the frontmatter:

```yaml
render:
  rendered_at: 2026-04-11T10:22:00   # Fresh timestamp from `date` command
  slide_count: 7
  path: ~/.config/feed/posts/instagram/2026/04/11/143022-curated-list-001/
  slides:
    - slide-1.png
    - slide-2.png
    - slide-3.png
    - slide-4.png
    - slide-5.png
    - slide-6.png
    - slide-7.png
```

**CRITICAL:** `rendered_at` must be a fresh timestamp captured at render completion. Always generate using `Bash(date +%Y-%m-%dT%H:%M:%S)`. Never copy from `created_at`.

---

## Completion Message

```
렌더링 완료!
슬라이드: {total_slides}장
저장 위치: ~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/{id}/
```
