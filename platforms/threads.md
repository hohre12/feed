# Threads (Meta)

Threads 플랫폼 프로필. 사용자가 Threads를 타겟 플랫폼으로 선택하면 이 파일이 로드됩니다.

---

## Spec

- **Character limit**: 500자
- **Format**: 텍스트 중심 (이미지 첨부 가능하나, 이 플러그인은 텍스트에 집중)
- **Hashtags**: 일반적으로 사용하지 않음 (X와 다름)
- **Links**: 링크가 포함된 게시물은 알고리즘에서 노출이 줄어듦

## Algorithm

- **Engagement 가중치**: comments > reposts > likes (댓글이 가장 중요)
- **Discovery**: 팔로워 기반이 아닌 추천 기반 노출 (recommendation-based)
- **Instagram 유저 기반 교차**: 감성적이고 공감 가는 콘텐츠가 잘 먹힘
- **짧은 콘텐츠가 유리**: 3~7줄이 황금 구간
- **초기 반응이 핵심**: 게시 직후 engagement가 높으면 더 많은 유저에게 노출됨

## What Works

- **감성/공감 톤** — "나도 그래" 반응을 이끌어내는 콘텐츠
- **자조적 유머** — 자기비하 + 위트 조합
- **반전 엔딩** — 스크린샷 찍어서 공유하게 만드는 구조
- **댓글 유도 구조** — "나만 그런가?", 양자택일 논쟁 등
- **모바일 퍼스트 세로 스크롤** — 한 줄 ≤ 15자 (한국어 기준)
- **새벽 감성 포스팅** — 늦은 밤 / 이른 아침 시간대
- **Curated list** — Save + share + comment trifecta. Highest follower conversion rate

## Follower Growth Formula (3 Factors)

These three conditions, when hit simultaneously, drive **follower growth** (not just likes):

1. **Overwhelming informational value** — Specific names/numbers. Not "good product recommendations" but "유니클로 수피마 코튼T" level specificity
2. **Save/share trigger** — Content worth bookmarking even if not needed right now. "I'll need this later" impulse
3. **Comment participation** — Open questions like "what else should I add?", "what do you use?" that invite responses

When all 3 align, follower growth explodes. A single post can drive thousands of new followers.

> "Writing roughly is closest to spoken language. The more you polish, the more it reads like written prose."
> — Insight validated in real Threads testing. The more casually you write, the higher the views/follows

## What Doesn't Work

- **긴 글** — 3번 이상 스크롤해야 하면 이탈률 급증
- **링크 포함 게시물** — 알고리즘 패널티
- **해시태그 남용** — 여긴 X가 아님
- **딱딱하고 격식체인 톤** — 공감이 안 됨
- **정보 과다 게시물** — 그건 Dev.to/Reddit에 올릴 것
- **AI 냄새나는 콘텐츠** — 유저들의 AI 감지력이 점점 높아지는 중

## Recommended Styles

Threads에서 잘 먹히는 스타일 파일 목록:

| 파일명 | 스타일 | Threads 강점 |
|---|---|---|
| `twist-poem.md` | 반전시 | 공유율 높음 (high shareability) |
| `relatable.md` | 공감 | 댓글율 높음 (high comment rate) |
| `one-liner.md` | 한줄반전 | 소비 부담 낮음 (easy to consume) |
| `quote-parody.md` | 명언패러디 | 리포스트율 높음 (high repost rate) |
| `self-deprecation.md` | 자조 | 공감율 높음 (high relatability) |
| `debate.md` | 논쟁 | 댓글율 최고 (highest comment rate) |
| `insight.md` | 인사이트 | 저장율 높음 (high save rate) |
| `curated-list.md` | 족보/큐레이션 | 팔로우 전환율 최고 (highest follower conversion) |

## API Publishing

- **Available**: 제한적 (Meta Threads API)
- **Auth**: `access_token`
- **Key field in accounts.json**: `threads.access_token`
- **Note**: API에 rate limit과 콘텐츠 정책 제한이 있음. 게시 전 정책 확인 필요.
