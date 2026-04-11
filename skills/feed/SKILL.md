---
name: feed
description: Generates interview-driven, platform-optimized SNS content and publishes it
disable-model-invocation: true
argument-hint: <스레드|트위터|레딧|데브투|인스타> <반전시|논쟁|공감|한줄반전|명언패러디|자조|인사이트|족보> <주제>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion WebSearch WebFetch mcp__playwright__browser_navigate mcp__playwright__browser_take_screenshot mcp__playwright__browser_run_code mcp__playwright__browser_close mcp__playwright__browser_evaluate mcp__playwright__browser_snapshot
---

# /feed

## 1. Argument Parsing

Parse `$ARGUMENTS` by fixed position:
- `$ARGUMENTS[0]` → platform (English ID or Korean alias)

### Platform Korean Alias Mapping

| 한글 | 영어 ID |
|------|---------|
| 스레드 | threads |
| 트위터 | x |
| 레딧 | reddit |
| 데브투 | devto |
| 인스타 | instagram |
| 인스타그램 | instagram |

Example: `/feed 스레드 반전시 알람` → platform = `threads`
- `$ARGUMENTS[1]` → style (English ID or Korean alias)

### Style Korean Alias Mapping

Korean input is converted to the corresponding English ID:

| 한글 | 영어 ID |
|------|---------|
| 반전시 | twist-poem |
| 논쟁 | debate |
| 공감 | relatable |
| 한줄반전 | one-liner |
| 명언패러디 | quote-parody |
| 자조 | self-deprecation |
| 인사이트 | insight |
| 족보 | curated-list |

Example: `/feed threads 반전시 알람` → style = `twist-poem`
- `$ARGUMENTS[2:]` → topic — **Everything from position 2 onward is joined together as a single topic.**

### Examples

| Input | platform | style | topic |
|-------|----------|-------|-------|
| `/feed` | interview | interview | interview |
| `/feed threads` | threads | interview | interview |
| `/feed threads twist-poem` | threads | twist-poem | interview |
| `/feed threads twist-poem 알람` | threads | twist-poem | 알람 |
| `/feed threads twist-poem 오늘 진짜 화나는 일이 있었는데` | threads | twist-poem | 오늘 진짜 화나는 일이 있었는데 |

If the topic is a long sentence, treat it as **source material/raw text** and process it according to the selected style.

Arguments that are already provided skip the corresponding interview step.

## 2. Initialization

Check whether the `~/.config/feed/` directory exists.

- **Does not exist**: Create the directory structure and default files as described in the "Structure to Create on Initialization" section of `reference.md`, then display a "Feed initial setup complete" message.
- **Exists**: Load the existing configuration and proceed normally.

## 3. Load Routing Guide

Read `reference.md` from the same directory as this file. All subsequent file loading follows this routing guide.

## 4. Conduct Interview

Read `interview.md` from the same directory as this file and proceed through Step 1 to Step 7 in order. Skip any step whose value has already been provided via `$ARGUMENTS`.
