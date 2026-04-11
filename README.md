# Feed

SNS content generator & publisher plugin for Claude Code.  
Interview-driven workflow that creates platform-optimized posts.

## Supported Platforms

- **Threads** / **X (Twitter)** / **Reddit** / **Dev.to**

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
```

### Other Commands

| Command | Description |
|---------|-------------|
| `/feed-publish` | Publish saved unpublished posts |
| `/feed-report` | Generate analytics report |
| `/feed-recommend` | Get topic recommendations |

## How It Works

1. **Interview** -- Asks about platform, style, audience, language, and topic
2. **Generate** -- Creates platform-optimized content following style rules
3. **Save** -- Stores post to `~/.config/feed/posts/`
4. **Publish** -- Publishes to the selected platform via API

## Data Storage

All user data is stored locally at `~/.config/feed/`:

```
~/.config/feed/
  config.json       # User preferences
  posts/            # Generated posts
  analytics/        # Publishing analytics
```

## License

MIT
