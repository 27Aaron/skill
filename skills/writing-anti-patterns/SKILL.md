---
name: writing-anti-patterns
description: Catch and fix common AI-writing failures — empty claims, fabricated or unlabeled numbers, filler structure, unsourced facts, corporate tone, Chinese AI clichés, overwrought or hollow prose, and decorative visuals — and match a writer's voice from samples. Use when writing or editing copy, blog posts, articles, or any prose.
metadata:
  author: Aaron
  version: "2.0.0"
---

# Writing Anti-Patterns

Common failures when AI writes or edits text, each with a bad example and the fix. Apply them while drafting, then do a review pass before handing off. If a fix needs a fact you don't have, mark the gap or ask — never invent it.

## Content Emptiness

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 1 | Adjective pile-up, no numbers | "Achieved significant growth across key metrics" | State the number: "Revenue grew 34% YoY to $12M" |
| 2 | Opening-paragraph filler | "In today's rapidly evolving landscape..." | Delete the opener. Start with the first real claim. |
| 3 | Restating the heading as a sentence | Heading "Revenue Growth", body "Our revenue growth has been notable" | Body must add information the heading does not carry |
| 4 | Vague time references | "Recently launched", "in the near future" | Pin to a date or quarter: "Launched Q1 2026" |
| 5 | Synonyms masking repetition | Three paragraphs saying "we are growing" in different words | One claim, one proof, move on |

## Metric Fabrication

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 6 | Round numbers implying precision | "Exactly 10,000 users" when the source says "around 10K" | Match the source's precision: "approximately 10,000" |
| 7 | Fake decimal precision | "Market share: 23.7%" with no cited source | Verify the figure and cite its source, or mark it as unknown; rounding does not make an unsupported number valid |
| 8 | Metric-narrative disconnect | Chart shows flat revenue, text says "strong momentum" | Text must match what the chart shows |
| 9 | Invented comparison baselines | "3x faster than alternatives" with no benchmark | Name the alternative and the benchmark method, or remove |
| 10 | Mixing time periods | YoY growth next to QoQ growth as if comparable | Label every comparison window explicitly |
| 11 | Unlabelled derived figure | "About 60% less waiting" as a headline when the source gave 23 and 9 minutes | Keep the inputs beside it ("23 to 9 minutes, about 61% less") or mark it derived (推算); check the arithmetic |
| 12 | Hand-copied dynamic counts | "56.9k stars" pasted into the text, stale within weeks | Print a snapshot date next to it ("56.9k stars as of 2026-10"); a bare hand-copied count is an expiring fact with no expiry label |

## Structure

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 13 | Ghost outline fails | Reading only the headings produces a pile of topics, not an argument | Rewrite headings and order before polishing sentences |
| 14 | Heading as label, not assertion | "Q3 Results" | Assertion-evidence: "Q3 revenue beat guidance by 12%" |
| 15 | Action without result | "Managed a cross-functional team" | Action + Scope + Result: "Led 8-person team to ship v2.0, reducing churn 15%" |
| 16 | Template slots filled with filler | A features list of "Fast, Secure, Easy to use" | Name the specific capability and where it shows, or cut the section; don't pad slots (three real points do not become five) |
| 17 | Copy without a clear ask | Three sections of context, no "what we need from you" | The ask belongs above the fold, not implied |

## Source Gaps

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 18 | Unverified version numbers | "Compatible with v4.2" when latest is v5.1 | Check the official source before citing any version |
| 19 | "Latest" without a date | "Uses the latest framework" | "Uses Next.js 15 (as of 2026-04)" |
| 20 | Competitor comparison without market data | "Leading solution in the market" | Cite the ranking source, or use "one of the established solutions" |
| 21 | Assumed availability | "Available on all major platforms" | List the actual platforms verified |
| 22 | Stale positioning | Copy still uses an old single-tool label after the product became a broader utility | Rewrite the title, intro, and FAQ around the current category before adding feature bullets |
| 23 | Same fact differs across places | FAQ says one price or install path, the docs or another language version say another | Treat every page, FAQ, and translation as one fact set; search for the old value and update every match |
| 24 | Real identity leaks into examples | An example distilled from a real resume or proposal keeps client names, quote amounts, or dates that identify the source | Swap in public figures, public projects, or invented generic data before publishing; list the swapped signals in the handoff |

## Tone Contamination

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 25 | Chinese AI corporate speak | "赋能企业数字化转型", "打造一站式解决方案" | Say what it does: "帮公司把纸质流程搬到线上" |
| 26 | English AI corporate speak | "Leverage our platform to unlock synergies" | "Use the platform to share data between teams" |
| 27 | AI tone cliches (CN dashes and connectors) | "本质上是模型在做预测——这意味着..." / 大量破折号 | 删元评论框架，直接说结论。破折号换冒号或句号 |
| 28 | Caption restates the diagram | "六类来源 → 六道过滤 → 配比设计 → 训练分片，四步串联" | Cap 给出图意以外的判断："来源决定知识边界，过滤决定干净程度，配比决定能力侧重" |
| 29 | Caption restates the title | Title: "ORM vs PRM 对比" / Cap: "ORM 跟 PRM 的对比" | Cap 必须给出标题之外的信息：取舍判据、适用场景、或讲到这里要带出的下一步。同义重写浪费了 cap 的注意力位 |

## Rhythm and Restraint

LLMs don't hold back unless told to: by default they use every device they know, all at once. Good writing is mostly ordinary sentences; the strong lines stand out because of them.

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 30 | Every sentence performs | Each sentence carries a metaphor, a parallel pair, or a punchline | Most sentences are plain and carry information. At most one device in five sentences; if every line is a highlight, none is |
| 31 | Atmosphere without increment | "海面很亮，风很轻，一切都安静下来" — mood stacked, nothing learned | Every detail must carry information or move the reader's understanding: say why it is bright, at what hour, tied to which event. Deletion test: if removing a fact collapses the paragraph, the piece runs on information — don't swap facts for scenery |
| 32 | Telegraphic plainness | "A推开门进去了。桌上三个杯子。她坐下。" — zero warmth, zero information | Plain ≠ telegraphic: "A翻了半天柜子找打火机，翻到第三个抽屉放弃了，顺手拿火柴凑合。" No device, but action and information carry it, and the tone comes through |
| 33 | Signature construction on repeat | "不是X，是Y" / "It's not just X — it's Y" four times in one piece | A distinctive construction appears once, at most. Any pattern in an example gets copied as a template, so watch the examples too |
| 34 | Ruminating on one sensation | The same smell, light, or feeling described three ways | One stroke per sense, then move on |
| 35 | Narration repeats the quote | A quote makes the point, then the next sentence explains it again | What a quote or dialogue already said, the narration doesn't repeat |
| 36 | Stretching thin material | Out of things to say, the paragraph keeps going in new words | When the material runs out, move to the next point or end |
| 37 | Elevated ending | "这不仅是一次旅行，更是一场心灵的洗礼" / "In the end, it's all about the journey" | End on the last real point: no elevation, no lyricism, no summary |

Self-check for #2, #25–27, #33, #37 (simplified and traditional):

```bash
grep -nE '本质是|本質是|这意味着|這意味著|值得注意的是|不仅.*而且|不僅.*而且|赋能|賦能|一站式|不是.{1,20}(而)?是|总之|總之|总而言之|總而言之|综上|綜上|[——–]' <file>
grep -niE "not just|isn't just|more than just|in conclusion|in summary|at the end of the day" <file>
```

## Charts and Images

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 38 | Chart with no insight title | Chart titled "Revenue by Quarter" | Title states the insight: "Revenue accelerated after Q2 price change" |
| 39 | Decorative chart that restates the text | Bar chart showing the same three numbers the paragraph just listed | If the text already communicates it, the chart must add a dimension (comparison, trend, distribution) |
| 40 | Icon or emoji as section marker | Sections led by decorative icons with no semantic value | Use typographic hierarchy (size, weight, spacing) instead |
| 41 | Label chrome over every heading | `01 · FEATURES` above each section when the sections are not a sequence | Number only real sequences (chapters, steps) |
| 42 | Visual brief leaks into reader copy | Caption says "21:9 ultra-wide screenshot with quiet background" | Keep image prompts and crop notes out of the text; captions state the insight |
| 43 | Real screenshot redrawn as fake UI | A product screenshot is AI-redesigned and no longer matches the shipped app | Preserve real screenshots |
| 44 | Missing image filled with atmosphere | Abstract texture used because no screenshot was provided | Mark the material gap or omit the image; do not substitute unrelated imagery |

## Matching a Voice

When asked to write in the user's voice or to match sample text:

1. **Samples beat instructions.** What the samples show outweighs what the instructions say; when they conflict, follow the samples. The tone of your own notes rubs off on the output, so think in the target voice, not in jargon.
2. **Read the samples along these axes**, as a picture of how the writer relates to the reader, not as a scorecard:
   - Sentence rhythm: mostly short or long? Short-short-long bursts?
   - Register: written, spoken, dialect; jargon or coined words
   - Distance: detached observer, or close to the subject?
   - Quotes vs narration: which one moves the piece?
   - Devices: which ones (repetition, parallelism, metaphor), and how dense
   - Where emotion sits: up front, at the end, held down throughout, or released at once
   - **How it advances**: new information, layered images, shifting emotion, or events. This is the axis most often misread — LLMs read every style as mood. Use the deletion test from #31.
3. **Attitude is the default state, not a device.** It runs through every sentence; devices are seasoning and get a quota. Rationing attitude like a device produces neat, cold text.
4. **Quote frequency, not adjectives.** "At most one device in five sentences" works; "use sparingly" is ignored.
5. **Revise one problem per pass**, and tell the three failure directions apart:
   - **Flat and cold** (neat but lifeless): the voice is missing. Strengthen the attitude and loosen the device quota.
   - **Overwrought** (every line shows off): tighten the quota and surround the strong lines with plain ones (#30).
   - **Empty** (mood without increment, form without content): not a density problem. Give the piece something to say (#31); it can't be written from atmosphere alone.

## Attribution

- Adapted from [Kami](https://github.com/tw93/Kami) `skills/kami/references/anti-patterns.md`, MIT License, Copyright (c) 2026 Tw93.
- Rhythm, restraint, and voice-matching guidance adapted from [TaysiTsuki](https://github.com/MidAutumnMoon/TaysiTsuki) `.agents/skills/writing-style/SKILL.md`, MIT License, Copyright (c) 2021 MidAutumnMoon.
