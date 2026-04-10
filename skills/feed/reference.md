# 라우팅 가이드

> 상황별로 어떤 파일을 로드해야 하는지 정의한다.
> SKILL.md와 interview.md는 이 파일을 참조하여 필요한 파일만 읽는다.

---

## 라우팅 테이블

| 상황 | 로드할 파일 | 비고 |
|------|-------------|------|
| 콘텐츠 생성 | `rules/common.md` | 항상 |
| | `rules/scoring.md` | 항상 |
| | `rules/{lang}.md` | 선택된 언어 (`ko`, `en`) |
| | `platforms/{platform}.md` | 선택된 플랫폼 |
| | `styles/{style}.md` | 선택된 스타일 |
| | `audiences/{audience}.md` | 선택된 타겟 독자 |
| 저장 | `flows/save.md` | 생성 완료 후 |
| 발행 | `flows/publish.md` | 발행 선택 시 |

---

## 파일 탐색

### 사용 가능한 플랫폼 목록
```
Glob platforms/*.md
```
파일명에서 `.md`를 빼면 플랫폼 ID가 된다. (예: `threads.md` → `threads`)

### 사용 가능한 스타일 목록
```
Glob styles/*.md
```
파일명에서 `.md`를 빼면 스타일 ID가 된다. (예: `twist-poem.md` → `twist-poem`)

---

## 스타일-플랫폼 호환성 확인

### 플랫폼 → 추천 스타일
각 플랫폼 파일(`platforms/{platform}.md`)의 **"Recommended Styles"** 섹션을 읽는다.
이 섹션에 해당 플랫폼에서 잘 먹히는 스타일 파일명 목록이 있다.

### 스타일 → 호환 플랫폼
각 스타일 파일(`styles/{style}.md`)의 **"호환 플랫폼"** (또는 "Compatible Platforms") 섹션을 읽는다.
이 섹션에 해당 스타일이 효과적인 플랫폼 목록이 있다.

### 사용 가능한 타겟 독자 목록
```
Glob audiences/*.md
```
파일명에서 `.md`를 빼면 타겟 ID가 된다. (예: `developer.md` → `developer`)

### 타겟 독자의 역할
타겟 독자 파일은 **검색 소스**와 **톤/용어 수준**을 결정한다:
- 주제 없을 때: 타겟 파일의 검색 소스에서 소재를 발굴한다
- 주제 있을 때: 타겟 파일의 검색 소스에서 팩트를 검증한다
- 콘텐츠 톤과 용어 수준을 타겟 파일에 맞춘다

### 스타일 선택 시 우선순위
1. 선택된 플랫폼의 Recommended Styles에 있는 스타일 → **우선 표시**
2. 스타일 파일의 호환 플랫폼에 선택된 플랫폼이 포함된 스타일 → **그 다음 표시**
3. 호환 목록에 없는 스타일 → 표시하지 않거나, 비호환 경고와 함께 표시

---

## 사용자 데이터 위치

| 경로 | 용도 |
|------|------|
| `~/.config/feed/config/preferences.json` | 사용자 설정 (기본 언어, 기본 플랫폼/스타일, 인터뷰 스킵 옵션) |
| `~/.config/feed/config/accounts.json` | 플랫폼별 API 키/토큰 |
| `~/.config/feed/posts/{platform}/{YYYY}/{MM}/{DD}/` | 생성된 포스트 저장 |
| `~/.config/feed/analytics/engagement.json` | 플랫폼별 반응 데이터 |
| `~/.config/feed/analytics/topics.json` | 주제별 성과 추적 |

---

## 초기화 시 생성할 구조

`~/.config/feed/`가 없을 때 아래 디렉토리와 기본 파일을 생성한다.

### 디렉토리
```
~/.config/feed/config/
~/.config/feed/posts/
~/.config/feed/analytics/
```

### 기본 파일

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
