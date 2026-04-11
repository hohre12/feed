# Feed

SNS content generator & publisher plugin for Claude Code.  
Interview-driven workflow that creates platform-optimized posts.

## Supported Platforms

- **Threads** / **X (Twitter)** / **Reddit** / **Dev.to** / **Instagram** (carousel card news)

## Supported Styles

| Style | Description |
|-------|-------------|
| twist-poem (반전시) | Reverse-twist poetry |
| debate (논쟁) | Provocative discussion starter |
| relatable (공감) | Relatable everyday moments |
| one-liner (한줄반전) | One-liner with a twist |
| quote-parody (명언패러디) | Famous quote parody |
| self-deprecation (자조) | Self-deprecating humor |
| insight (인사이트) | Sharp insight |
| curated-list (족보) | Curated list format |

## Installation

### Via Marketplace (Recommended)

```
/plugin marketplace add hohre12/jwbae-plugins
/plugin install feed
```

### Standalone

```
/plugin marketplace add hohre12/feed
/plugin install feed@hohre12/feed
```

### Local Development

```bash
claude --plugin-dir ./feed
```

## Usage

```
/feed                                    # Full interview mode
/feed threads                            # Platform selected, interview the rest
/feed threads twist-poem                 # Platform + style, interview topic
/feed threads twist-poem alarm clocks    # All provided, skip interview
```

Korean aliases are supported:

```
/feed 스레드 반전시 알람
/feed 인스타 족보 AI 디자인 도구
```

### Other Commands

| Command | Description |
|---------|-------------|
| `/feed-publish` | Publish saved unpublished posts |
| `/feed-report` | Generate analytics report |
| `/feed-recommend` | Get topic recommendations |

## Prerequisites (Instagram only)

Instagram card news generation requires Playwright MCP server:

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

Text-only platforms (Threads, X, Reddit, Dev.to) work without Playwright.

## How It Works

1. **Interview** -- Asks about platform, style, audience, language, and topic
2. **Generate** -- Creates platform-optimized content following style rules
3. **Render** -- (Instagram) Converts text to visual card news via HTML/CSS + Playwright
4. **Save** -- Stores post to `~/.config/feed/posts/`
5. **Publish** -- Publishes to the selected platform via API or clipboard

## Data Storage

All user data is stored locally at `~/.config/feed/`:

```
~/.config/feed/
  config/
    preferences.json  # User preferences + brand settings
    accounts.json     # Platform API keys
  posts/              # Generated posts (text + card news images)
  analytics/          # Engagement & topic analytics
  tmp/                # Temporary render files
```

## License

MIT
