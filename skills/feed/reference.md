# Routing Guide

> Defines which files to load for each situation.
> SKILL.md and interview.md reference this file to load only the necessary files.

---

## Routing Table

| Situation | File to Load | Notes |
|-----------|-------------|-------|
| Content generation | `rules/common.md` | Always |
| | `rules/scoring.md` | Always |
| | `rules/{lang}.md` | Selected language (`ko`, `en`) |
| | `platforms/{platform}.md` | Selected platform |
| | `styles/{style}.md` | Selected style |
| | `audiences/{audience}.md` | Selected target audience |
| Save | `flows/save.md` | After generation completes |
| Publish | `flows/publish.md` | When publish is selected |

---

## File Discovery

### Available Platform List
```
Glob platforms/*.md
```
Remove `.md` from the filename to get the platform ID. (e.g., `threads.md` → `threads`)

### Available Style List
```
Glob styles/*.md
```
Remove `.md` from the filename to get the style ID. (e.g., `twist-poem.md` → `twist-poem`)

---

## Style-Platform Compatibility

### Platform → Recommended Styles
Read the **"Recommended Styles"** section of each platform file (`platforms/{platform}.md`).
This section contains the list of style filenames that work well on that platform.

### Style → Compatible Platforms
Read the **"Compatible Platforms"** section of each style file (`styles/{style}.md`).
This section contains the list of platforms where that style is effective.

### Available Target Audience List
```
Glob audiences/*.md
```
Remove `.md` from the filename to get the target ID. (e.g., `developer.md` → `developer`)

### Role of Target Audience
The target audience file determines **search sources** and **tone/terminology level**:
- When no topic is given: discover material from the target file's search sources
- When a topic is given: verify facts from the target file's search sources
- Match content tone and terminology level to the target file

### Style Selection Priority
1. Styles listed in the selected platform's Recommended Styles → **shown first**
2. Styles whose Compatible Platforms section includes the selected platform → **shown next**
3. Styles not on the compatibility list → not shown, or shown with an incompatibility warning

---

## User Data Locations

| Path | Purpose |
|------|---------|
| `~/.config/feed/config/preferences.json` | User settings (default language, default platform/style, interview skip options) |
| `~/.config/feed/config/accounts.json` | Platform API keys/tokens |
| `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/` | Saved generated posts |
| `~/.config/feed/analytics/engagement.json` | Per-platform engagement data |
| `~/.config/feed/analytics/topics.json` | Per-topic performance tracking |

---

## Structure to Create on Initialization

Create the following directories and default files when `~/.config/feed/` does not exist.

### Directories
```
~/.config/feed/config/
~/.config/feed/posts/
~/.config/feed/analytics/
```

### Default Files

**`~/.config/feed/config/preferences.json`**
```json
{
  "default_language": "ko",
  "default_platforms": {},
  "interview": {
    "skip_language": false,
    "skip_style": false
  }
}
```

**`~/.config/feed/config/accounts.json`**
```json
{
  "threads": { "access_token": "", "enabled": false },
  "x": { "api_key": "", "api_secret": "", "access_token": "", "access_secret": "", "enabled": false },
  "reddit": { "client_id": "", "client_secret": "", "username": "", "password": "", "enabled": false },
  "devto": { "api_key": "", "enabled": false }
}
```

**`~/.config/feed/analytics/engagement.json`**
```json
{ "entries": [] }
```

**`~/.config/feed/analytics/topics.json`**
```json
{ "topics": [] }
```
