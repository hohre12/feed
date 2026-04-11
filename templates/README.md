# Feed Templates

Instagram 카드뉴스(Carousel) 렌더링용 HTML/CSS 템플릿 모음.

---

## 파일 구성

| 파일 | 역할 |
|---|---|
| `base.css` | 테마 변수, 타이포그래피, 카드 레이아웃, 브랜드 헤더, 캐러셀 닷, 출처 표기 |
| `components.css` | 재사용 UI 컴포넌트 (label, quote-block, cta-button 등) |
| `cover.html` | 커버 슬라이드 (첫 번째 슬라이드) |
| `content.html` | 콘텐츠 슬라이드 — 4가지 layout variant |
| `cta.html` | CTA 슬라이드 (마지막 슬라이드) |

---

## 변수 문법

모든 변수는 `{{variable_name}}` 형태로 작성합니다.  
렌더 시 문자열 치환(string substitution)으로 실제 값으로 교체됩니다.

```
{{brand_name}}  →  짐코딩
{{theme}}       →  dark
```

JavaScript 없이 동작합니다.

---

## 전체 변수 목록

### 공통 변수 (모든 슬라이드)

| 변수 | 설명 | 예시 |
|---|---|---|
| `{{brand_name}}` | 채널명 | `짐코딩` |
| `{{brand_tagline}}` | 슬로건 / 해시태그 | `#AI세컨드브레인` |
| `{{accent_color}}` | 액센트 컬러 hex | `#58a6ff` |
| `{{theme}}` | 테마 클래스 | `dark` 또는 `light` |
| `{{slide_number}}` | 현재 슬라이드 번호 (2자리) | `02` |
| `{{total_slides}}` | 전체 슬라이드 수 (2자리) | `07` |
| `{{source_credit}}` | 이미지 출처 표기 | `*출처:vibeindex.ai` |
| `{{carousel_dots}}` | 캐러셀 닷 HTML 블록 | 아래 섹션 참고 |

### 커버 슬라이드 (`cover.html`)

| 변수 | 설명 |
|---|---|
| `{{cover_title}}` | 메인 헤드라인 (20자 이내 권장) |
| `{{cover_subtitle}}` | 부제목 / 설명 (비워두면 숨김) |
| `{{cover_label}}` | 상단 라벨 뱃지 (예: `AI NEWS`, 비워두면 숨김) |
| `{{cover_image_url}}` | 배경 이미지 URL (비워두면 그라데이션 fallback) |
| `{{author_line}}` | 작성자 표기 (비워두면 숨김) |

### 콘텐츠 슬라이드 (`content.html`)

| 변수 | 설명 |
|---|---|
| `{{content_title}}` | 슬라이드 제목 |
| `{{content_body}}` | 본문 HTML (layout_variant 에 따라 구조 상이, 아래 참고) |
| `{{content_image_url}}` | 슬라이드 이미지 URL (없으면 빈 문자열) |
| `{{layout_variant}}` | 레이아웃 타입 (`list-item`, `text`, `number`, `quote`) |

### CTA 슬라이드 (`cta.html`)

| 변수 | 설명 |
|---|---|
| `{{cta_emoji}}` | 이모지 (예: `💾`, `👇`) |
| `{{cta_title}}` | CTA 메인 문구 |
| `{{cta_subtitle}}` | CTA 부가 설명 |
| `{{cta_button}}` | 버튼 라벨 텍스트 |

---

## 테마 전환

`body` 클래스로 테마를 전환합니다.

```html
<!-- 다크 테마 (기본) -->
<body class="dark">

<!-- 라이트 테마 -->
<body class="light">
```

`{{theme}}` 변수로 주입됩니다.

---

## layout_variant 사용 방법

`content.html`의 `body` 클래스가 `variant-{layout_variant}` 형태로 설정됩니다.  
`{{layout_variant}}` 를 교체하면 자동 적용됩니다.

### list-item (curated-list 스타일)

번호 뱃지 + 카테고리 라벨 + 도구/아이템명 + 코멘트. 슬라이드당 2~3개 아이템.

`{{content_body}}` 에 주입할 HTML:

```html
<div class="list-item">
  <div class="number-badge">01</div>
  <div class="list-item__body">
    <span class="label">카테고리</span>
    <p class="list-item__name">도구명</p>
    <p class="list-item__comment">한 줄 코멘트</p>
  </div>
</div>
<div class="item-divider"></div>
<div class="list-item">
  <div class="number-badge">02</div>
  <div class="list-item__body">
    <span class="label">카테고리</span>
    <p class="list-item__name">도구명</p>
    <p class="list-item__comment">한 줄 코멘트</p>
  </div>
</div>
```

### text (insight / debate 스타일)

센터 텍스트 블록 + 선택적 이미지.

`{{content_body}}` 에 주입할 HTML:

```html
<p class="text-slide-body">
  본문 텍스트.<br>
  <span class="highlight">핵심 키워드</span>는 이렇게 강조합니다.
</p>
<!-- 이미지가 있을 때만 포함 -->
<div class="content-image">
  <img src="https://..." alt="">
</div>
```

### number (self-deprecation 스타일)

큰 숫자 + 설명 텍스트. 숫자 한 개 당 슬라이드 한 장.

`{{content_body}}` 에 주입할 HTML:

```html
<div class="number-display">
  <span class="number-big">03</span>
  <span class="number-unit">번째</span>
</div>
<p class="number-desc">자기비하 포인트 제목</p>
<p class="number-sub">부연 설명 텍스트</p>
```

### quote (quote-parody 스타일)

인용 부호 + 큰 텍스트 + 출처.

`{{content_body}}` 에 주입할 HTML:

```html
<div class="quote-deco">"</div>
<div class="quote-block">
  <p class="quote-block__text">인용 텍스트 내용</p>
  <p class="quote-block__source">— 출처 또는 화자</p>
</div>
```

---

## carousel_dots 생성 방법

전체 슬라이드 수만큼 `.carousel-dot` 을 생성하고,  
현재 슬라이드에 해당하는 닷에 `active` 클래스를 추가합니다.

예시 — 7장 중 2번째 슬라이드:

```html
<div class="carousel-dot"></div>
<div class="carousel-dot active"></div>
<div class="carousel-dot"></div>
<div class="carousel-dot"></div>
<div class="carousel-dot"></div>
<div class="carousel-dot"></div>
<div class="carousel-dot"></div>
```

활성 닷은 pill 형태(`width: 22px`) + 액센트 컬러로 표시됩니다.

---

## accent_color 교체

`base.css`의 `:root { --accent }` 기본값은 `#58a6ff` (fallback)입니다.  
렌더 시 각 HTML 파일의 `:root { --accent: #58a6ff }` 를 실제 액센트 컬러로 교체합니다.

```bash
# 예: 오렌지 액센트로 교체
sed 's/#58a6ff/#ff6b35/g' cover.html > cover_rendered.html
```

또는 `{{accent_color}}` 토큰을 직접 `:root` 블록에 넣어도 됩니다:

```html
<style>
  :root { --accent: {{accent_color}}; }
</style>
```

---

## 로컬 테스트 방법

1. 브라우저에서 직접 열기

```bash
open /Users/baejaewon/Victor-workspace/projects/feed/templates/cover.html
```

2. 로컬 HTTP 서버 (Google Fonts 로딩 필요 시)

```bash
cd /Users/baejaewon/Victor-workspace/projects/feed/templates
python3 -m http.server 8080
# → http://localhost:8080/cover.html
```

3. 실제 렌더 크기 확인

브라우저 개발자도구 → 디바이스 모드 → 540 × 675 로 설정.  
DPR 2x 캡처 시 실제 출력은 1080 × 1350 px.

---

## 설계 원칙

- **다크 테마 기본**: `body.dark` — `#0d1117` 계열 배경
- **볼드 타이포**: Noto Sans KR 900 weight, 제목 68~72px
- **비대칭 레이아웃**: AI 느낌 상쇄를 위해 의도적 비대칭 요소 포함
- **한글 최적화**: `word-break: keep-all`, 넉넉한 line-height (1.55~1.7)
- **그라데이션 fallback**: 이미지 없어도 CSS 배경으로 완성도 유지
- **JS 불필요**: 순수 HTML/CSS 문자열 치환만으로 동작
