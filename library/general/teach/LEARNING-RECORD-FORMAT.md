# Learning Record Format

Learning records are one file each, named for what they capture. Create them lazily — only when the first record is written.

Use them in a persistent course to capture insights, demonstrated understanding, and stated prior knowledge that will affect future teaching. Distinguish the user's self-report from demonstrated understanding.

## Template

```md
# {Short title of what was learned or established}

{1-3 sentences: what was learned (or what prior knowledge was established), and why it matters for future sessions.}
```

A single paragraph is enough. State what the evidence supports and why it changes future teaching.

## Optional sections

Only include these when they add genuine value. Most records won't need them.

- **Status** frontmatter (`active | superseded by <later record>`) — useful when an earlier understanding turns out to be wrong and is replaced.
- **Evidence** — what the user said or demonstrated, such as an explanation, an exercise solution, or stated prior experience. Include it when the basis or depth of understanding would otherwise be unclear.
- **Implications** — what this unlocks or rules out for future sessions. Worth recording when non-obvious.

## When to write a learning record

Write one when any of these is true:

1. **The user demonstrated genuine understanding of something non-trivial** — not just exposure, but evidence they can use the concept correctly. Use the demonstrated depth to inform what to teach next.
2. **The user disclosed prior knowledge** — "I already know X." Record it so future sessions don't re-teach it. Also record the _depth_ claimed.
3. **A misconception was corrected** — the user shows or confirms that they understand the correction. These are high-value: they predict future stumbling blocks for related topics.

### What does _not_ qualify

- Material that was merely covered. Coverage is not learning. Wait for evidence.
- Anything already captured tersely in `GLOSSARY.md` as a term definition. Don't duplicate.
- Session-by-session activity logs. Learning records are not a journal — they are decision-grade insights.

## Supersession

When a later record contradicts an earlier one (the user's understanding deepened or corrected), mark the old record `Status: superseded by <later record>` rather than deleting it. The history of how understanding evolved is itself useful signal.
