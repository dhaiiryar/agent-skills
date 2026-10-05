---
name: legible-output
description: Make AI output easier for humans to read, check, and understand. Use whenever the user asks to explain, summarize, teach, walk through, or help them understand something (code, a system, a paper, a plan, agent output, a concept), or says it is confusing, too dense, or hard to follow. Also use when they say "explain simply", "make this clearer", "ELI5", "STE", "explainer", "visualize this", "diagram this", "make a page for this", or "make a video about this". Be pushy - use it even if they only say "help me understand X".
---

# Legible Output

Goal: reduce the human's time-to-understanding. As models do more work, the human's job shifts to oversight and comprehension. Pick the output format that makes comprehension cheapest.

Based on Andrej Karpathy's tips for understanding LLM outputs.

## Format ladder

Pick the lowest rung that fully works. Escalate when the topic is spatial, sequential, interactive, or large.

1. **Writing** - plain controlled English (rules below). Default for short or simple topics.
2. **Diagram** - flows, architectures, relationships, state machines, comparisons. Easier to parse than prose.
3. **HTML page** - interactive explainer with sliders, tabs, animation, live demos. Use for anything with parameters, steps, or layers.
4. **Explainer video** - 3Blue1Brown style, narrated. Use when the user asks for it, or when motion and narration clearly beat a page.

Artifacts are cheap and discardable. Build a one-off app or page for a single explanation. Do not hold back because it "would not be reused."

## Rung 1: Writing in ASD-STE100 (80%)

ASD-STE100 is Simplified Technical English, built for aircraft maintenance manuals. Apply about 80% of it. Full strictness hurts flow.

Rules to apply:
- One term per concept. Pick one word and never swap it for a synonym. (Do not say "worker", then "agent", then "executor" for the same thing.)
- One meaning per word. Avoid words with several senses.
- Short sentences. Aim for 20 words or fewer for instructions, 25 or fewer for descriptions.
- One idea per sentence.
- Active voice. Subject, verb, object.
- Simple tenses. Prefer present. Avoid perfect and progressive forms when possible.
- Keep articles (a, an, the). Do not drop them.
- Imperative for steps. "Close the valve." not "The valve should be closed."
- Avoid idioms, phrasal verbs, figurative language, and slang.
- Limit noun strings to 3 nouns.
- Define each technical term on first use, then reuse it unchanged.
- Numbered list for sequential steps. Warnings go before the step they apply to.

Before sending, scan for: synonym drift, sentences over 25 words, passive voice, unexplained jargon.

Do not use full-spec features that hurt flow: the strict dictionary check, or rigid paragraph length caps.

## Rung 2: Diagrams

- Use Mermaid, inline SVG, or the Visualizer when available.
- One diagram, one idea. Split big systems into layers.
- Label every node and edge with the same terms used in the prose.
- Add 1-2 sentences under the diagram stating what to notice.

## Rung 3: HTML pages

- Output a single self-contained .html file.
- Include: a one-paragraph summary at top, the main visual or interactive element, then details on demand (tabs or expandable sections).
- Prefer interaction that exposes cause and effect: sliders, step-through buttons, hover tooltips, toggles.
- Keep terms consistent with the STE rules above.
- Follow the environment's publishing and frontend-design guidance when building and delivering the page.

## Rung 4: Explainer videos

Typical request: "Create a 3b1b style video explainer on X. Use my ElevenLabs API key for narration."

Workflow:
1. Write a short script first. 1 idea per scene. Aim 2-5 minutes total. Use STE-style narration.
2. Storyboard each scene: what appears, what moves, what the narrator says.
3. Render animation with Manim (`pip install manim --break-system-packages`) or another code-based tool. Check that ffmpeg and Cairo/Pango dependencies exist first.
4. Narration:
   - If the user gave an ElevenLabs key, use it. Never hardcode or echo the key.
   - Otherwise use a free local TTS (for example Piper or Kokoro) and say so.
5. Combine audio and video with ffmpeg, matching scene length to narration length.
6. Deliver the .mp4 with present_files.

Environment limits: the sandbox may block network access to TTS APIs or lack render dependencies. If so, do not fake it. Deliver the Manim project, the script, and a short run command, and state what the user must run locally.

## Choosing quickly

| Topic shape | Start at |
|---|---|
| Single concept, short | Writing |
| Flow, architecture, relationships | Diagram |
| Parameters, layers, step-through logic | HTML page |
| Intuition-heavy math or ML, user asks for video | Video |

When unsure, give the writing answer first, then add the diagram or page. Do not ask which format. Pick and deliver.

## Summary principle

As intelligence and code get cheap, ask for large, custom, throwaway artifacts that make understanding easier. Push the format boundary.
