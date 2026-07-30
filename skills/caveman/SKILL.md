---
name: caveman-compressor
description: Compress markdown, prompts, or text files into an ultra-concise "caveman" format while strictly preserving code, structure, and technical terms. Make sure to use this skill whenever the user asks to compress, shorten, minify, make instructions concise, or convert text/prompts to "caveman speak", even if they don't explicitly say "caveman".
---

# Caveman Compressor

Your task is to compress the provided text, markdown, prompts, or files into an ultra-concise "caveman" format. 

## STRICT RULES — NEVER MODIFY:
- Code blocks (``` fenced or indented): copy exactly
- Inline code (`backtick content`): copy exactly
- URLs and markdown links: preserve exactly
- File paths (`/src/...`, `./config.yaml`): preserve exactly
- Commands (`npm install`, `git commit`): preserve exactly
- Technical terms, library names, API names, version numbers, env vars

## COMPRESSION RULES:
- **Remove:** articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course/happy to), hedging (it might be worth/you could consider), connectives (however/furthermore/additionally).
- **Replace:** "in order to" → "to", "make sure to" → "ensure".
- **Compress:** fragments OK, short synonyms (big not extensive, fix not "implement a solution for").
- **Drop:** "you should", "remember to".
- **Merge:** redundant bullets.
- **Examples:** keep only one example per pattern.

## PRESERVE STRUCTURE:
- All heading text exact.
- Bullet nesting.
- Numbered lists.
- Tables (compress cell text only).
- YAML frontmatter.

## OUTPUT FORMAT
Return ONLY the compressed markdown body. No outer fence. No explanation. No conversational filler.
