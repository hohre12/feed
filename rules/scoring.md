# Content Self-Scoring Rubric

Score your own generated content strictly using these criteria.
**Be harsh.** The moment you go easy on your own writing, quality drops.

---

## Scoring Dimensions

| Dimension | Key | What It Measures |
|-----------|-----|------------------|
| Twist | `twist` | Does the last line completely flip the context of what came before? Not a simple reversal — does it reach the level of "oh, THAT's what this was about?" |
| Relatability | `relatability` | Does the reader think "oh god, me too..." and instinctively nod? Not limited to a niche group — broad enough that most people get it? |
| Humor | `humor` | Not just a slight smile — does it make the reader actually laugh out loud? |
| Brevity | `brevity` | Is it tight with zero filler? So compressed that removing a single character would change the meaning? |
| Human Feel | `human-feel` | Would this pass as something a real person wrote on SNS? The instant it feels AI-generated, it fails. |

---

## Score Definitions (Apply Strictly)

### Twist (`twist`)

| Score | Criteria |
|-------|----------|
| 1-3 | No twist, or the twist is obvious, or "so what?" level. Reading the last line triggers no emotional shift. |
| 4-6 | There is a twist, but it's predictable. Reaction is "ah, I see" at most. No surprise. |
| 7-8 | Makes you go "oh?" and re-read. The meaning of the earlier lines changes on second reading. Funnier or sadder the second time. |
| 9-10 | Complete blindside. You stop and go "wow..." after reading. Every preceding sentence gains a double meaning. |

**Style-specific override:** Some styles have a fixed twist score. For example, `curated-list` style always receives `twist=7` because the twist comes from the list's personality/commentary rather than a narrative reversal. When a style specifies a fixed score, use it and do not re-evaluate that dimension.

### Relatability (`relatability`)

| Score | Criteria |
|-------|----------|
| 1-3 | The empathy point is unclear or too personal. Reaction: "what is this even about?" |
| 4-6 | Relatable, but feels like "someone else's story." Might get a like, but not a share. |
| 7-8 | "Oh god, this is literally me?" level. Makes the reader want to write their own experience in the comments. |
| 9-10 | Immediately brings a specific person's face to mind. The reader wants to tag them: "야 이거 너 아니냐" |

### Humor (`humor`)

This dimension is scored especially strictly. **A smile is not a laugh.**

| Score | Criteria |
|-------|----------|
| 1-3 | Not funny. The humor attempt failed, or there is no humor element at all. |
| 4-5 | Mildly amusing but no facial change. "ㅎ" level. A single puff of air through the nose. |
| 6 | Corners of the mouth go up. But no sound. One "ㅋ" level. |
| 7 | An actual smile forms. Facial muscles move. "ㅋㅋ" level. |
| 8 | Audible "ㅋㅋㅋ". If alone, you'd actually make a sound. |
| 9 | Screenshot it and send to a group chat. "야 이거 봐" level. |
| 10 | Can't breathe from laughing. Still funny on re-read. Still funny days later when you remember it. |

**Note:** A 10 almost never occurs. 9 is rare. Most "decent" humor lands at 6-7.

### Brevity (`brevity`)

| Score | Criteria |
|-------|----------|
| 1-3 | Overflowing with unnecessary modifiers, repetition, and explanation. Could be cut in half without losing meaning. |
| 4-6 | Clean enough, but one or two spots have filler. "Could this sentence be removed?" feeling in places. |
| 7-8 | Tight. Every word has a role. Removing one changes the meaning. |
| 9-10 | Not a single character can be touched. Peak compression. Short yet dense with meaning. |

### Human Feel (`human-feel`)

This is **the most important dimension.** Even if everything else scores well, a low score here makes the post worthless.

| Score | Criteria |
|-------|----------|
| 1-3 | Immediately identifiable as AI-written. The cleanliness and order itself is unnatural. |
| 4-6 | Well-written, but "too well-written." Suspiciously polished for an SNS post. |
| 7-8 | Natural. Would not raise suspicion on an actual timeline. |
| 9-10 | Perfectly human. You can almost feel a specific person's voice. Has personality. |

**Deduction triggers (apply -1 to -2 immediately upon detection):**
- Clean parallel structure (e.g., "A는 B하고, C는 D한다" — too balanced)
- Literary-style endings (e.g., "...인 걸." "...이었다." — too neat)
- Uniform sentence structure (all sentences similar in length or repeating a pattern)
- Overly correct grammar (spelling too perfect)
- Conjunction overuse ("그러나", "하지만", "그래서" appearing in every sentence)
- Lists/enumerations neatly organized with 3+ items (**Exception: curated-list style** — lists are the essence of that style, so this deduction does not apply. Instead, evaluate "does the list have personality?" — if items are just brand names without personal opinion/commentary, deduct.)

**Bonus triggers (apply +1 upon detection):**
- Incomplete sentences ("아 그거... 진짜")
- Colloquial abbreviations ("걍", "ㅇㅇ", "ㄹㅇ", "~함", "~임")
- Raw, unpolished feel (rough style that looks unedited)
- Irregular sentence lengths (long-short-long variation)
- Typos at a level that feels naturally casual
- Raw emotional expression ("ㅠㅠ", "ㅋㅋㅋㅋ", "ㄹㅇ미침")

---

## Passing Criteria

### Minimum Threshold (below this = fail)
- **All dimensions must score 7 or above** to pass
- If any single dimension is below 7, **auto-regenerate** (up to 2 retries)

### Quality Target (aim for this)
- **Target average: 8.0+** — this is what a content generation tool should produce
- Average 7.0-7.4 → PASS but **auto-improve once** (content is acceptable but not good enough)
- Average 7.5-7.9 → PASS, publishable
- Average 8.0+ → EXCELLENT, confidently publish

### Forced Regeneration Conditions
- `human-feel` below 7 → **mandatory regeneration** (no matter how high other scores are)
- Both `humor` and `twist` at 6 or below → regenerate (core entertainment value missing)
- Average below 7.5 after first generation → **auto-improve once** before showing to user

### Regeneration Rules
- When regenerating, explicitly target the low-scoring dimensions for improvement
- If still failing after 2 regenerations, select the version with the highest overall score, but mark it as having failed the quality target
- Maximum 3 total attempts (1 initial + 2 regenerations)

---

## Self-Scoring Protocol

### Attitude
- **Score strictly.** Do not be generous with your own writing.
- If you are giving 8+ on every dimension, **re-read the scoring criteria.** You are being too lenient.
- **Target average is 8.0+.** This is a content generation tool — "acceptable" is not good enough.
- **7 means "minimum pass."** 8 means "good." 9 means "viral potential."
- If you're averaging below 7.5, the content needs improvement, not lenient scoring.

### Verification Questions (Mandatory Before Scoring — Do Not Skip)

Before scoring, answer these 5 questions **honestly.** The answers set the ceiling for your scores.

1. **If you saw this post on your timeline, would you stop scrolling?**
   -> "I'd probably scroll past" -> overall average capped at 7
2. **Would you personally screenshot this and send it to a friend?**
   -> "No" -> `humor` capped at 7, `twist` capped at 7
3. **Does this look like AI wrote it? Even 1%?**
   -> "Even a little bit, yes" -> `human-feel` capped at 6
4. **Did you go "oh?" and re-read it?**
   -> "I read it through in one pass" -> `twist` capped at 6
5. **Did you actually laugh out loud? (Not just smile — laugh)**
   -> "I just smiled" -> `humor` capped at 6
   -> "Not even a smirk" -> `humor` capped at 5

### Safe Post Deduction

"A post that isn't offensive and is somewhat relatable, but nobody wants to share it" is a safe post.

**Detection criteria** — the post is "safe" if 2 or more of the following are true:
- It's an obvious observation anyone could make (e.g., "읽씹 싫다면서 나도 읽씹함" — seen too many times)
- It relies solely on meta-humor (e.g., "이거 쓰면서 ~하는 중" — the frame itself is the only joke)
- Both the topic and the angle avoid any risk (neither provocative nor sharp)
- The ending is predictable from the first line
- Nobody would tag a friend to share it

**Implementation rules for safe-post deduction:**
1. Run the 5 detection criteria above after initial scoring
2. If 2+ criteria are met, apply the following deductions:
   - `humor` -1 point
   - `relatability` -1 point (because obvious ≠ relatable — it becomes "ugh, this again")
3. If the overall average is 7.5 or above after deductions, **mandatory re-review** — a safe post should not score this high
4. Log which detection criteria triggered so the regeneration can specifically address them

---

## Output Format

Display the scorecard below the generated content in the following format:

```
---
| Dimension | Score | Reasoning |
|-----------|-------|-----------|
| Twist (twist) | 8 | The meaning of "출근" completely flips in the last line |
| Relatability (relatability) | 7 | Relatable for office workers, but scope is limited |
| Humor (humor) | 7 | Smile-level, not laugh-out-loud |
| Brevity (brevity) | 8 | Zero filler, every line serves a purpose |
| Human Feel (human-feel) | 7 | Natural, but sentence structure is slightly uniform |
| **Average** | **7.4** | |
| **Verdict** | **PASS** | |
---
```

**The Reasoning column is mandatory.** Do not just write a number. Explain in one line why that score was given.
If the verdict is FAIL, specify which dimensions fell short and what must be improved in the regeneration.
