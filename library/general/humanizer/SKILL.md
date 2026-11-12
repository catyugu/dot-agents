---
name: humanizer
description: "Humanize text in English or Chinese: strip the statistical tells of AI writing and put a real voice back. Use for any request to humanize, de-AI, de-slop, un-ChatGPT, 去 AI 味 or 润色 a draft — essays, blog posts, PR descriptions, READMEs, docs, memos, emails, tweets, resume bullets — when text has to pass as human-written, and when reviewing prose before publishing."
version: "3.1.0"
author: "Siqi Chen (@blader); 歸藏 (op7418); merged by catyugu"
license: "MIT"
---

# Humanizer

Humanizing a text does two things: remove the tells of AI writing, then put a voice back.

The full guide, the tell checklist and a worked example exist in one file per language:

- English text → `references/english.md`
- 中文文本 → `references/chinese.md`

Read the version for the language of the text before rewriting anything. Text that switches languages, or a translation between them, usually needs both.

Load it when the user hands you a text to humanize, wants a draft reviewed for AI tells, or asks you to match a voice sample.

## Sources

- English version ported from [blader/humanizer](https://github.com/blader/humanizer) v2.5.1 (MIT, Copyright (c) 2025 Siqi Chen), itself based on [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup.
- Chinese version adapted from 歸藏's (op7418) Chinese humanizer skill, MIT.
