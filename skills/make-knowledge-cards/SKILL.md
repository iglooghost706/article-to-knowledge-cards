---
name: make-knowledge-cards
description: "Convert a pasted article or a local Markdown/TXT file into 5-8 concise knowledge cards. Use this skill whenever a user wants to summarize, distill, or turn reading material such as notes, blog posts, docs, or textbook passages into a study deck. Each card isolates one key concept with a title, the core knowledge, a plain-language explanation, and an example or a self-test question. Does not support web scraping, PDF, Anki, or a graphical UI."
agent_created: true
---

# Make Knowledge Cards

## Overview

Turn long-form text into a small, high-signal deck of study cards. The skill
distills an article down to what actually matters, expresses each idea once, and
never invents information. Output is a single Markdown document — easy to read,
copy, or paste anywhere.

## When to Use

- The user pastes an article, essay, notes, or documentation and asks to
  "summarize", "make cards", "turn this into flashcards", "extract the key
  points", or "help me study this".
- The user points to a local `.md` or `.txt` file and wants it distilled.
- The user wants a compact, reviewable version of dense reading material.

## Scope and Unsupported Inputs

Supported:
- Text pasted directly into the conversation.
- Local files with a `.md` or `.txt` extension (read the file's contents).

Explicitly NOT supported — say so and stop, do not attempt a workaround:
- Web pages / URLs (no scraping).
- PDF (and other binary document formats).
- Anki (or any spaced-repetition / proprietary export format).
- Any graphical user interface or interactive app.

If the user provides an unsupported input, name the limitation clearly and ask
for a Markdown or plain-text version instead.

## Card Schema

Every card has exactly four fields. Keep field names identical so output is
machine- and human-readable:

- **标题 (Title)** — A short label naming the single concept on the card.
- **核心知识 (Core Knowledge)** — One dense, factual statement or pointer. No
  filler. This is the "answer" the learner should be able to recall.
- **简明解释 (Concise Explanation)** — One to three plain-language sentences
  that make the core knowledge understandable. Avoid jargon unless the source
  uses it.
- **例子或自测问题 (Example or Self-Test Question)** — Exactly one of:
  - a concrete example that illustrates the concept, OR
  - a self-test question that checks whether the learner understood it.

## Extraction Workflow

Follow these steps in order.

1. **Acquire the source.**
   - Pasted text: use it verbatim.
   - Local file: read the `.md`/`.txt` file and use its text.
   - If the source is a URL, PDF, or other unsupported type, stop and explain
     the limitation.

2. **Separate signal from noise.** Identify the genuinely important ideas. Drop:
   - introductory filler, anecdotes, and motivational padding;
   - summaries that merely restate later content;
   - examples the source itself calls illustrative unless they carry unique
     information;
   - metadata (author, date, headers) unless it is the actual subject.

3. **De-duplicate.** If the source makes the same point in two places, merge
   them into ONE card. Never produce two cards that say the same thing.

4. **One concept per card.** Each card must cover a single, distinct idea. If a
   concept has two equal-weight sub-points that cannot be expressed in one
   sentence, split into two cards or pick the more important one.

5. **Count honestly.** Target 5-8 cards. Produce the number that the material
   genuinely supports:
   - rich source → 5 to 8 cards;
   - thin source → fewer than 5 is correct and expected;
   - never invent extra cards or repeat ideas just to hit a number.

6. **Fill the four fields per card.** Write the core knowledge strictly from the
   source. Use the source's own terms, names, and numbers. For the last field,
   prefer a self-test question when the concept is definitional, and a concrete
   example when it is procedural or abstract.

7. **Verify faithfulness.** Before outputting, re-check every card: is each claim
   present in the source? If anything was added from general knowledge, remove or
   clearly mark it as not-from-source. The default is to remove it.

8. **Output as Markdown.** Use the template below. No extra prose, no GUI, no
   Anki export.

## Quality Rules (Hard Constraints)

- **No fabrication.** Never add facts, figures, names, dates, or claims absent
  from the source. Faithful distillation only.
- **No duplicates.** Each idea appears on exactly one card.
- **One concept per card.** A card with two unrelated points is a failed card.
- **Honest count.** Emit the number the material supports; do not pad.
- **Plain language.** Explanations should be understandable to someone seeing the
  topic for the first time.
- **Source-faithful terms.** Preserve the source's key terminology; do not
  silently "improve" or rename concepts.

## Output Format

Output a single Markdown document. Use this exact structure:

```markdown
# 知识卡片：<source title or short topic>

> 来源：<pasted text / filename> ｜ 卡片数：<N>

## 卡片 1：<标题>
- **核心知识**：<one dense factual statement>
- **简明解释**：<1-3 plain-language sentences>
- **例子或自测问题**：<one example, or one question>

## 卡片 2：<标题>
- **核心知识**：...
- **简明解释**：...
- **例子或自测问题**：...

<!-- repeat for each card; number sequentially -->
```

Keep the "来源" line so the reader knows what was distilled. If the material was
too thin for 5 cards, say so briefly in that line (e.g. "卡片数：3（素材较少）").
If heavy duplication was merged, note it there too (e.g. "卡片数：3（已合并重复表述）").
This makes the honest count self-explanatory and reproducible.

## Examples

### Example 1 — Pasted technical article

Input (user pastes):
> "HTTP caching uses headers. `Cache-Control` sets max-age in seconds. `ETag`
> lets the server validate a cached response with a conditional request. A 304
> means 'use your cache'. `Vary` tells caches that the response depends on a
> request header like `Accept-Encoding`."

Output:
```markdown
# 知识卡片：HTTP 缓存基础

> 来源：粘贴文本 ｜ 卡片数：4

## 卡片 1：Cache-Control 控制最大年龄
- **核心知识**：`Cache-Control` 用 max-age（秒）定义响应可被缓存的时长。
- **简明解释**：客户端在该秒数内可复用缓存，无需重新请求。
- **例子或自测问题**：自测：max-age=60 表示缓存多久内有效？

## 卡片 2：ETag 用于条件校验
- **核心知识**：`ETag` 是资源的标识，服务器用它做条件请求校验缓存。
- **简明解释**：客户端带上 ETag 询问"是否变化"，服务器比对后决定。
- **例子或自测问题**：自测：什么情况下服务器会返回 304？

## 卡片 3：304 表示复用缓存
- **核心知识**：状态码 304 表示资源未变，客户端应使用本地缓存。
- **简明解释**：这避免了重复传输相同内容，节省带宽。
- **例子或自测问题**：例子：浏览器收到 304 后直接渲染已存的页面。

## 卡片 4：Vary 描述依赖的请求头
- **核心知识**：`Vary` 告知缓存响应取决于某个请求头（如 Accept-Encoding）。
- **简明解释**：不同请求头可能得到不同响应，需分别缓存。
- **例子或自测问题**：自测：为什么压缩与否会影响缓存键？
```

### Example 2 — Thin source (honest count)

Input (user pastes):
> "Sleep is important for memory."

Output:
```markdown
# 知识卡片：睡眠与记忆

> 来源：粘贴文本 ｜ 卡片数：1（素材较少）

## 卡片 1：睡眠有助于记忆
- **核心知识**：睡眠对记忆很重要。
- **简明解释**：原文仅给出结论，未解释机制或提供证据。
- **例子或自测问题**：自测：原文是否说明了"为什么"睡眠有助于记忆？
```

The skill emits one card rather than padding to five, because the source
contains only a single distinct claim.
