# Common Writing Rules

> These rules apply to all styles and all platforms. Always loaded during content generation.

---

## Core Principle

If it looks like AI wrote it, it's a failure.

It should feel like something typed on a phone under the covers at 2 AM. Not a polished piece — just a thought that popped into your head, dumped out as-is. It doesn't need to be grammatically perfect. In fact, if it's perfect, people get suspicious.

---

## WebSearch Usage Guideline

Use WebSearch when the content involves:
- **Current events, trending topics, or recent releases** (anything that may have changed after training cutoff)
- **Specific facts**: prices, dates, statistics, product names, rankings
- **Viral memes or slang** that may be too new for LLM knowledge

Do NOT use WebSearch for:
- General human emotions, universal experiences (e.g., alarm snooze, procrastination)
- Well-known cultural references that are unlikely to change
- Creative writing that doesn't require factual accuracy

**Rule of thumb:** If getting a fact wrong would make the post embarrassing or misleading, search. If the post is about feelings or universal habits, just write.

---

## Structural Variation Rules

When generating 3 or more posts, the following must be observed:

### Line Count
- No two consecutive posts may have the same number of lines
- Good: 4 lines -> 6 lines -> 3 lines / Bad: 4 lines -> 4 lines -> 5 lines

### Opening Word
- Same-pattern openers (매일/매번/항상 type) -> at most 1 out of 3
- Good: "매일 ~" / "그냥 ~" / "아 ~" / Bad: "매일 ~" / "매번 ~" / "항상 ~"

### Twist Timing
- The twist must NOT always land on the last line
- Sometimes it comes mid-post, sometimes there is no twist at all

### Closing Pattern
- Same closing endings (결국/그래도/어쩌겠어) -> at most 1 out of 3

### Parallel Structure (A하면서 / 또 B하고)
- Use in at most 1 out of 3 posts

### Line Breaks (Stanza Splitting)
- Do not split lines the same way every time
- Some posts are one block, some split into 2 stanzas, some have a break after every line

---

## Colloquial Language Rules (Most Important)

The language must match what people actually use on SNS. Not literary expression — KakaoTalk / Twitter voice.

### Required Contractions
Use the natural spoken forms:
- ~걸 ("알고 있었던 걸")
- ~잖아 ("원래 그런 거잖아")
- ~거면서 ("안 좋아한다 거면서")
- ~인데 ("그게 문제인데")

### Emphasis Expressions (0-1 per post, do not overuse)
- 1도 ("관심 1도 없음")
- 진짜 ("진짜 모르겠다")
- 걍 ("걍 그런 거임")
- ㅇㅇ ("맞음 ㅇㅇ")

### Incomplete Sentences
Intentionally breaking grammar:
- Subject omission: "알면서도 또 봄" (나는 omitted)
- Single-word lines: "그냥." / "몰라." / "근데."
- Trailing off: "...인 거 같은데" / "...몰라" / "...그런 건가"

### Colloquial Endings
- ~함 ("그래서 걍 잠")
- ~임 ("이게 진짜 문제임")
- ~는 중 ("지금 후회하는 중")
- ~같은 거 ("사랑 같은 거")
- ~거든 ("근데 나도 아는 거든")

---

## Forbidden Patterns (Using These Exposes the AI)

### Absolute Prohibitions

| Forbidden | Reason | Alternative |
|-----------|--------|-------------|
| ~하며 | Literary/written style | ~하면서 / ~하고 |
| ~하였다 | Literary/written style | ~했다 / ~했음 |
| ~노라 | Archaic poetic style | Never needed |
| 심화되는 | Stiff Sino-Korean | 더 심해지는 |
| 귀결되다 | Report-style language | 결국 그렇게 되는 |
| 수렴하다 | Academic/thesis language | 거기로 가는 |

### Structural Prohibitions

- **Clean parallel couplets** are forbidden
  - X: "매번 미루기만 하고 / 시작할 생각은 없어"
  - O: "또 미뤘음 / 시작은 내일 한다고 했는데 그 내일이 언젠지 모르겠다"

- **Every line being a grammatically complete sentence** is forbidden
  - If it looks unnaturally clean, it smells like AI
  - There should be incomplete lines, single words, cut-off thoughts to feel natural

- **Overly balanced structure** is forbidden
  - No repeating symmetric patterns like 3-lines / twist / 3-lines
  - Asymmetry is natural

- **Two or more posts starting with the same word** is forbidden
  - When generating multiple posts, opening words must not overlap

- **Uniform comment length in lists** is forbidden (curated-list style)
  - If every list item has a comment of similar length (e.g., all 10-15 characters), it looks AI-generated
  - Real people write some comments long ("GPT는 매번 톤 달라지는데 이건 브랜드 톤 한번 세팅하면 기억해줌"), some short ("그냥 좋음"), some are just a reaction ("사기임")
  - X: all items have 1-sentence comments of similar length
  - O: item 1 has 2 sentences, item 3 has 1 word, item 5 has a comparison, item 7 has nothing

- **Overused self-deprecation patterns** are forbidden
  - "~없었으면 퇴사함" — was funny in 2024, now a cliché
  - "~없이 어떻게 살았지" — same pattern, overdone
  - "인생 바뀜" — too generic
  - If a self-deprecation phrase could be the top comment on ANY product review, it's too cliché
  - Instead, make self-deprecation SPECIFIC: "이거 쓰고 나서 팀장이 나보고 일 빨라졌다는데 비밀임" — this is specific and fresh

---

## Tone

### Self-Deprecating Humor
Not blaming others — "I know, but here I am doing it again":
- O: "다이어트 시작한다고 샐러드 시켰는데 드레싱을 3개 넣음"
- X: "요즘 사람들은 다이어트도 제대로 못 해"

### Emotional Disguise
Pretending to talk about romance/relationships, but actually writing about everyday objects or concepts:
- Obsession with Wi-Fi disguised as a love story
- The world outside the blanket disguised as a breakup
- Snoozing an alarm disguised as a reunion

### Raw Feel — First-Draft Principle

**Do not actually revise.** This is not "pretend you didn't revise" — literally write once and do not edit.

> "Writing roughly is closest to spoken language. The more you polish, the more it reads like written prose."
> — Insight validated in real Threads testing

- Resist the urge to refine. The moment you polish, the AI shows through
- Additions like "아 맞다 그리고" are fine
- The ending doesn't need to be clean. It can just stop
- Transitions between sentences don't need to be smooth. That's how real people write

---

## "Safe Post" Detection and Prevention (Viral vs Safe)

### Safe Post = Failure
"A post that isn't offensive and is somewhat relatable, but nobody wants to share it" — that's a safe post.
The kind of post that gets 50 likes on SNS. We must not produce this.

### How to Detect a Safe Post

Apply the following checklist. If **2 or more** items are true, the post is "safe" and must be reworked:

1. **Obvious observation test**: Could you find this exact take by scrolling for 30 seconds on any SNS? If yes, it's too obvious.
   - Example of obvious: "읽씹 싫다면서 나도 읽씹함" — a structure everyone has seen
2. **Meta-humor dependency test**: Does the humor come solely from the format "이걸 쓰면서 ~하는 중"? If the frame is the only joke, the post is hollow.
3. **Risk-avoidance test**: Is the topic safe AND the angle safe? A safe topic can work if the angle is sharp. A risky topic can work if the angle is grounded. Both being safe = bland.
4. **Predictability test**: After reading the first line, can you guess the last line? If the trajectory is obvious, there's no reason to keep reading.
5. **Tag test**: Would anyone actually tag a friend with "야 이거 너 아니냐"? If not, the post lacks shareability.

### Patterns of Viral Posts (The Goal)
- **Specific detail**: Not "알람" but "알람 5개 맞춰놓고 전부 스누즈 누르는"
- **Slight discomfort**: A little stinging, slightly embarrassing, a touch excessive
- **Unexpected angle**: Same topic, but approached from a direction nobody else has taken
- **Share impulse**: The kind of post where people tag someone and say "야 이거 너 아니냐"
- **Numbers/specificity**: Not "운동" but "헬스장 등록비 36만원 내고 3번 감"
