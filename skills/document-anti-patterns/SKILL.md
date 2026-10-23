---
name: document-anti-patterns
description: Catch and fix common quality failures in AI-generated professional documents — empty claims, fabricated metrics, template filler, unsourced facts, AI corporate tone, visual excess, image-slot, slide, and landing-page mistakes. Use when drafting or reviewing reports, resumes, proposals, one-pagers, slide decks, charts, or landing pages.
metadata:
  author: Aaron
  version: "1.0.0"
---

# Anti-Patterns: AI Document Quality

Common failures when AI generates professional documents. Organized by failure type, each with a bad example and the fix.

The rules below apply to every document. Also read the reference for the document type:

| Document has | Read |
| --- | --- |
| Charts, icons, images, or screenshots | [visuals.md](references/visuals.md) |
| Slides | [slides.md](references/slides.md) |
| A landing page or localized site | [landing-pages.md](references/landing-pages.md) |

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
| 11 | Unlabelled derived figure | "About 60% less waiting" as a slide headline when the source gave 23 and 9 minutes | Keep the inputs beside it ("23 to 9 minutes, about 61% less") or mark it derived (推算); check the arithmetic |

## Structure Mimicry

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 12 | Resume bullet without result | "Managed a cross-functional team" | Action + Scope + Result: "Led 8-person team to ship v2.0, reducing churn 15%" |
| 13 | Template slots filled with filler | Skills section listing "Communication, Teamwork, Problem-solving" | Name the specific skill and where it was applied, or cut the section |
| 14 | Equity report without variant perception | "Company is well-positioned for growth" | State what the market gets wrong and why your thesis differs |
| 15 | One-pager without a clear ask | Three sections of context, no "what we need from you" | The ask belongs above the fold, not implied |
| 16 | Slide title as label, not assertion | "Q3 Results" | Assertion-evidence: "Q3 revenue beat guidance by 12%" |

## Source Gaps

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 17 | Unverified version numbers | "Compatible with v4.2" when latest is v5.1 | Check the official source before citing any version |
| 18 | "Latest" without a date | "Uses the latest framework" | "Uses Next.js 15 (as of 2026-04)" |
| 19 | Competitor comparison without market data | "Leading solution in the market" | Cite the ranking source, or use "one of the established solutions" |
| 20 | Assumed availability | "Available on all major platforms" | List the actual platforms verified |
| 21 | Maintainer identity leaks into demo content | A demo or example distilled from a real resume or proposal keeps job-search phrases, client names, quote amounts, or engagement periods that identify the source | Swap in public figures, public projects, or invented generic data before the content lands anywhere shareable; list the swapped signals in the handoff |

## Tone Contamination

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 22 | Chinese AI corporate speak | "赋能企业数字化转型", "打造一站式解决方案" | Say what it does: "帮公司把纸质流程搬到线上" |
| 23 | English AI corporate speak | "Leverage our platform to unlock synergies" | "Use the platform to share data between teams" |
| 24 | Caption restates the flow diagram | "六类来源 → 六道过滤 → 配比设计 → 训练分片，四步串联" | Cap 给出图意以外的判断："来源决定知识边界，过滤决定干净程度，配比决定能力侧重" |
| 25 | AI tone cliches (CN dashes and connectors) | "本质上是模型在做预测——这意味着..." / 大量破折号 | 删元评论框架，直接说结论。破折号换冒号或句号。自检: `grep -nE '本质是\|这意味着\|值得注意的是\|不仅.*而且\|[——–]'` |
| 26 | Sans font stack missing CJK fallback | `font-family: Inter` 用在含中文的 th / h3 | CJK 回退到系统 sans (PingFang) 跟 serif 主调冲突。任何可能渲染 CJK 的元素用 `var(--serif)` |
| 27 | Caption restates the slide title | Title: "ORM vs PRM 对比" / Cap: "ORM 跟 PRM 的对比" | Cap 必须给出标题之外的信息：取舍判据、适用场景、或讲到这里要带出的下一步。同义重写浪费了 cap 的注意力位 |

## Attribution

Adapted from [Kami](https://github.com/tw93/Kami) `skills/kami/references/anti-patterns.md`, MIT License, Copyright (c) 2026 Tw93.
