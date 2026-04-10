---
name: feed
description: SNS 플랫폼별 맞춤 콘텐츠를 인터뷰 기반으로 생성하고 발행합니다
disable-model-invocation: true
argument-hint: <스레드|트위터|레딧|데브투> <반전시|논쟁|공감|한줄반전|명언패러디|자조|인사이트|족보> <주제>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion WebSearch WebFetch
---

# /feed

## 1. 인자 파싱

`$ARGUMENTS`를 고정 위치로 파싱한다:
- `$ARGUMENTS[0]` → platform (영어 ID 또는 한글 별칭)

### 플랫폼 한글 별칭 매핑

| 한글 | 영어 ID |
|------|---------|
| 스레드 | threads |
| 트위터 | x |
| 레딧 | reddit |
| 데브투 | devto |

예: `/feed 스레드 반전시 알람` → platform = `threads`
- `$ARGUMENTS[1]` → style (영어 ID 또는 한글 별칭)

### 스타일 한글 별칭 매핑

한글로 입력해도 영어 ID로 변환하여 사용한다:

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

예: `/feed threads 반전시 알람` → style = `twist-poem`
- `$ARGUMENTS[2:]` → topic — **2번 위치부터 끝까지 전부 합쳐서 하나의 topic으로 취급한다.**

### 예시

| 입력 | platform | style | topic |
|------|----------|-------|-------|
| `/feed` | 인터뷰 | 인터뷰 | 인터뷰 |
| `/feed threads` | threads | 인터뷰 | 인터뷰 |
| `/feed threads twist-poem` | threads | twist-poem | 인터뷰 |
| `/feed threads twist-poem 알람` | threads | twist-poem | 알람 |
| `/feed threads twist-poem 오늘 진짜 화나는 일이 있었는데` | threads | twist-poem | 오늘 진짜 화나는 일이 있었는데 |

topic이 긴 문장일 경우, 해당 내용을 **소재/원본 텍스트**로 취급하여 선택된 스타일에 맞게 가공한다.

제공된 인자는 인터뷰에서 해당 단계를 건너뛴다.

## 2. 초기화

`~/.config/feed/` 디렉토리 존재 여부를 확인한다.

- **없으면**: `reference.md`의 "초기화 시 생성할 구조" 섹션에 따라 디렉토리와 기본 파일을 생성한 뒤, "Feed 초기 설정 완료" 메시지를 표시한다.
- **있으면**: 기존 설정을 로드하고 정상 진행한다.

## 3. 라우팅 가이드 로드

이 파일과 같은 디렉토리의 `reference.md`를 읽는다. 이후 모든 파일 로드는 이 라우팅 가이드를 따른다.

## 4. 인터뷰 진행

이 파일과 같은 디렉토리의 `interview.md`를 읽고, Step 1부터 Step 7까지 순서대로 진행한다. `$ARGUMENTS`로 이미 제공된 값이 있는 단계는 건너뛴다.
