---
name: teach
description: Teach concepts and skills, explain technical reasoning, or plan and continue a persistent course. Use for direct learning questions as well as multi-session teaching.
version: "1.1.0"
author: "catyugu"
license: "MIT"
disable-model-invocation: true
argument-hint: "What would you like to learn about?"
---

Answer the learner's current question using their stated background, preferences, and requested depth. Teach in the conversation by default. Build a persistent course or learning artifact when requested; use existing course context when continuing one.

## Teaching Quality

Develop each explanation as a coherent unit of understanding. Let conceptual dependencies and the depth needed to answer the question determine its length. Split at meaningful boundaries, keeping the reasoning that connects central concepts intact.

Prefer connected prose. Use headings, lists, equations, and diagrams where they clarify the subject. Choose examples, analogies, exercises, and summaries for a concrete teaching purpose rather than a fixed lesson template.

State the assumptions needed for important claims and show how the conclusions follow from definitions, derivations, observations, or evidence. For a mechanism, explain the intermediate steps that produce the outcome. For a mathematical result, give the proof or identify the result being used and check its assumptions; identify any proof deferred to a later lesson.

Distinguish established results from intuition, analogy, interpretation, and speculation. Explain how an example illustrates the principle. Keep an analogy's correspondence and limits clear where they affect the conclusion.

Before delivering, check the reasoning internally: does each important inference have support, do successive paragraphs advance the same explanation, and are notation and assumptions consistent? Repair missing steps rather than covering them with rhetorical transitions. Keep this review out of the learner-facing text.

## Adapt to the Learner

Use what the learner has stated or demonstrated to choose a starting point. Explain missing prerequisites where they become necessary, and avoid repeating material already established. Treat learning records as evidence about prior understanding, not a precise calculation of current ability.

Ask a clarifying question when its answer would materially change the explanation or course plan. A request to understand a theory is a sufficient learning goal. When the learner asks a specific question, answer it without requiring a mission interview.

For an ongoing course, use its goals and the learner's latest request to choose the next topic. Update the plan when the learner changes direction; resolve a consequential ambiguity before rewriting their goals.

Use retrieval practice, spacing, and varied applications when they help retention or transfer. Offer practice that tests a specific understanding and respects constraints such as "no coding exercises." Provide complete explanations and worked examples when requested.

## Sources

Use reliable sources to establish external facts, check uncertain claims, and resolve disagreements. Prefer sources appropriate to the subject, such as textbooks, primary research, official documentation, or relevant practitioner accounts. Identify experience-based advice as such.

Cite sources where they directly support a claim or offer useful further study. Verify the reasoning in a technical explanation itself: a citation or an illustrative example does not fill a missing proof. Recommend further reading when it helps the learner's next step.

## Persistent Courses and Materials

When maintaining a course, read relevant existing goals, learning records, and preferences before extending it. Create or update state only when it will affect future teaching; a standalone question needs no course setup.

Keep state easy to find and use these formats as needed:

- [MISSION-FORMAT.md](./MISSION-FORMAT.md): record course goals, scope, and constraints when planning or revising a course.
- [LEARNING-RECORD-FORMAT.md](./LEARNING-RECORD-FORMAT.md): record demonstrated understanding, stated prior knowledge, or a confirmed correction that changes future teaching.
- [RESOURCES-FORMAT.md](./RESOURCES-FORMAT.md): curate sources likely to be used again.
- [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md): define recurring terms and notation when a shared reference helps.

Record stated teaching preferences where future sessions can find them. Separate evidence of learning from material merely presented.

Use the requested format for saved materials. For an HTML course, keep typography readable, navigation clear, and equations accessible. Reuse existing assets where useful; extract shared components when actual reuse warrants it. Add interactive elements only when they help explain or explore the subject.

Create reference documents when they support later lookup, such as a notation guide, algorithm reference, or syntax sheet. Link related saved materials where it helps navigation. Preserve enough context for each explanation to remain understandable when revisited.
