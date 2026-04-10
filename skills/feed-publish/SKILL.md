---
name: feed-publish
description: 미발행 상태의 포스트를 발행합니다 (API 자동 발행 또는 클립보드 복사)
disable-model-invocation: true
argument-hint: <today|all>
allowed-tools: Read Write Glob Grep Bash AskUserQuestion
---

# /feed-publish — 미발행 포스트 발행

이 스킬은 `~/.config/feed/posts/`에 저장된 미발행 포스트(`published: false`)를 찾아 발행한다.

---

## Step 1. 환경 확인

`~/.config/feed/` 디렉토리가 존재하는지 확인한다.

- 존재하지 않으면:
  ```
  먼저 /feed로 포스트를 생성해주세요.
  ```
  출력 후 **즉시 종료**한다.

---

## Step 2. 인자 파싱

`$ARGUMENTS`를 파싱한다.

| 인자 | 동작 |
|------|------|
| `today` | 오늘 날짜(`YYYY/MM/DD`)에 해당하는 포스트만 필터링 |
| `all` | 전체 미발행 포스트 표시 |
| (빈 값) | 전체 미발행 포스트 표시 (`all`과 동일) |

오늘 날짜는 현재 시각 기준으로 결정한다 (Bash로 `date +%Y/%m/%d` 실행).

---

## Step 3. 미발행 포스트 검색

### 3-1. 파일 검색

Glob으로 `~/.config/feed/posts/` 하위의 모든 `.md` 파일을 검색한다.

- `today` 인자인 경우:
  ```
  ~/.config/feed/posts/*/{YYYY}/{MM}/{DD}/*.md
  ```
  (오늘 날짜로 치환)

- `all` 또는 빈 값인 경우:
  ```
  ~/.config/feed/posts/**/*.md
  ```

### 3-2. 미발행 필터링

검색된 각 파일의 frontmatter를 읽어서 다음 조건에 해당하는 것만 수집한다:

- `published: false` **또는**
- `publish_method: failed` (발행 실패 재시도 대상)

각 파일에서 다음 frontmatter 필드를 추출한다:
- `platform`
- `style`
- `topic`
- `created_at`
- `publish_method` (failed 여부 확인용)

### 3-3. 결과 없음 처리

미발행 포스트가 하나도 없으면:
```
발행할 포스트가 없습니다.
```
출력 후 **즉시 종료**한다.

---

## Step 4. 미발행 목록 표시

수집된 미발행 포스트를 번호 목록으로 표시한다.
`created_at` 기준 오름차순(오래된 것 먼저)으로 정렬한다.

### 표시 형식

```
미발행 포스트 목록:

[1] threads / twist-poem / 알람 / 2026-04-09 14:30
[2] x / one-liner / 커피 / 2026-04-09 20:15
[3] threads / debate / 재택근무 / 2026-04-08 09:30  ⚠️ 발행 실패 (재시도)
```

- 형식: `[번호] {platform} / {style} / {topic} / {created_at에서 날짜+시분만}`
- `publish_method: failed`인 포스트는 끝에 ` ⚠️ 발행 실패 (재시도)` 를 붙인다.

---

## Step 5. 사용자 선택

AskUserQuestion으로 발행할 포스트를 선택받는다.

```
발행할 포스트를 선택하세요 (번호를 쉼표로 구분, 또는 "all"):
```

### 입력 처리

| 입력 | 동작 |
|------|------|
| `1` | 1번 포스트만 발행 |
| `1,3` 또는 `1, 3` | 1번과 3번 발행 |
| `all` | 전체 발행 |

유효하지 않은 번호가 포함된 경우, 유효한 번호만 처리하고 무효한 번호는 무시한다.
유효한 번호가 하나도 없으면 "유효한 선택이 없습니다." 출력 후 종료.

---

## Step 6. 발행 실행

선택된 각 포스트에 대해 아래 로직을 순서대로 실행한다.

### 6-0. 발행 플로우 로드

플러그인 디렉토리에서 `flows/publish.md`를 Read로 읽어 발행 로직을 참조한다.
(이 파일은 플러그인 루트 기준 상대 경로: `flows/publish.md`)

### 6-1. 계정 설정 확인

`~/.config/feed/config/accounts.json`을 Read로 읽는다.

- 파일이 없으면 → 모든 플랫폼을 `enabled: false`로 취급한다.
- 해당 포스트의 `platform`에 대한 `enabled` 값을 확인한다.

### 6-2. 발행 분기

#### Case A: `enabled = true` (API 발행 가능)

AskUserQuestion으로 질문한다:
```
[{platform}] 자동 발행할까요?
[1] 예 (API 발행)
[2] 아니오 (클립보드 복사)
```

- **[1] 예** → API 발행을 시도한다.
  - `flows/publish.md`의 플랫폼별 API 발행 섹션을 참조하여 실행.
  - Bash로 `curl` 등을 사용하여 API 호출.
  - **성공 시:**
    - frontmatter 업데이트:
      ```yaml
      published: true
      published_at: {현재 ISO 8601 시각}
      publish_method: api
      publish_url: {플랫폼이 반환한 URL}
      error: null
      ```
  - **실패 시:**
    - 에러 메시지를 표시한다.
    - 자동으로 클립보드 복사로 폴백한다 (Case B와 동일하게 처리).
    - frontmatter 업데이트:
      ```yaml
      published: false
      published_at: null
      publish_method: failed
      publish_url: null
      error: "{에러 메시지}"
      ```
    - 메시지:
      ```
      발행에 실패했습니다: {에러 메시지}
      대신 클립보드에 복사했습니다. 직접 붙여넣기 해주세요.
      ```

- **[2] 아니오** → Case B로 진행.

#### Case B: `enabled = false` 또는 사용자가 클립보드 선택

클립보드에 복사한다.

1. 포스트 파일에서 frontmatter를 제외한 **본문만** 추출한다.
   - 두 번째 `---` 이후의 내용이 본문이다.

2. Bash로 `pbcopy`를 사용하여 클립보드에 복사한다:
   ```bash
   # 본문 내용을 pbcopy로 복사
   ```

3. frontmatter 업데이트:
   ```yaml
   published: true
   published_at: {현재 ISO 8601 시각}
   publish_method: clipboard
   publish_url: null
   error: null
   ```

4. 복사된 내용의 첫 2~3줄을 미리보기로 표시한다:
   ```
   클립보드에 복사했습니다. 직접 붙여넣기 해주세요.

   복사된 내용:
   ---
   안 보면 될 걸
   자꾸 확인하고
   ...
   ---
   ```

### 6-3. Frontmatter 업데이트

Write 도구로 포스트 파일 전체를 다시 작성하여 frontmatter를 업데이트한다.
본문은 변경하지 않는다. frontmatter의 발행 관련 필드만 업데이트한다.

---

## Step 7. 발행 요약

모든 선택된 포스트의 발행이 완료되면 요약을 표시한다.

```
발행 완료 요약:

✅ [1] threads / twist-poem / 알람 → API 발행 (https://threads.net/...)
✅ [2] x / one-liner / 커피 → 클립보드 복사
❌ [3] threads / debate / 재택근무 → 발행 실패 (401 Unauthorized)

발행: 2건 | 실패: 1건
```

### 요약 형식

- 성공 (API): `✅ [{번호}] {platform} / {style} / {topic} → API 발행 ({URL})`
- 성공 (클립보드): `✅ [{번호}] {platform} / {style} / {topic} → 클립보드 복사`
- 실패: `❌ [{번호}] {platform} / {style} / {topic} → 발행 실패 ({에러})`

마지막 줄에 총 발행 건수와 실패 건수를 표시한다.

---

## 주의사항

- 여러 포스트를 클립보드에 복사할 경우, 마지막 포스트만 클립보드에 남는다. 각 복사 시점에 사용자에게 이를 알린다:
  ```
  ⚠️ 여러 포스트를 클립보드 복사하면 마지막 포스트만 클립보드에 남습니다.
  각 포스트를 복사한 뒤 바로 붙여넣기 해주세요.
  ```
- `pbcopy`는 macOS 전용이다. 현재 macOS만 지원한다.
- API 발행 시 `accounts.json`의 토큰/키가 비어있으면 `enabled: true`여도 실패로 처리한다.
