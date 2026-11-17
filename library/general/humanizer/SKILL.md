---
name: humanizer
description: "Write natural, direct, rigorous English or Chinese. Use for user-facing replies, explanations, technical discussion, standalone prose, and requests to humanize, de-AI, 去 AI 味, 润色, rewrite, review, or match author voice. Choose speak for conversation and explanation; choose write for drafting or editing documents, articles, reports, READMEs, PR descriptions, emails, and similar prose."
license: "MIT"
metadata:
  version: "4.0.0"
  author: "catyugu"
---

# Humanizer

Use two levels of guidance. Preserve accuracy, connected reasoning and author voice while suppressing specific formulaic expressions.

| Level | Task | 中文 | English |
| --- | --- | --- | --- |
| `speak` | Conversation, answers, explanations and technical discussion | [speak-chinese.md](references/speak-chinese.md) | [speak-english.md](references/speak-english.md) |
| `write` | Drafting, rewriting, polishing or reviewing standalone prose | [write-chinese.md](references/write-chinese.md) | [write-english.md](references/write-english.md) |

Read the document for the task's level and output language. For mixed-language prose or translation, read both languages at that level. Choose by the requested deliverable: explaining a report uses `speak`; writing or revising that report uses `write`. If a task needs both, apply each to its relevant part without running duplicate review workflows.

For new writing, use the supplied facts, verified evidence and clearly identified inference; apply the write checklist to the draft you produce. For editing, preserve the source's information, certainty and scope. Deliver the requested text by default; add diagnostics when requested or needed to identify unresolved content problems.

## Sources

- English editing patterns adapted from [blader/humanizer](https://github.com/blader/humanizer) v2.5.1 (MIT, Copyright (c) 2025 Siqi Chen), itself based on [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup.
- Chinese editing patterns adapted from 歸藏's (op7418) Chinese humanizer skill, MIT.
