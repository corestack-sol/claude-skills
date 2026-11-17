---
name: blog-article-writer
description: Writes original, publication-ready blog/guide articles that synthesize external sources (news, studies, reports) with proper attribution, without copying copyrighted text. Use when asked to write a blog post, guide, or article that draws on outside sources — especially when citing news outlets, studies, or third-party data — and the user needs copyright-safe, professionally-toned copy ready to publish.
---

# Blog Article Writer

Turns research (your own notes, a NotebookLM notebook, or raw source material the user pastes in) into original articles that are safe to publish: no copied prose, every fact traceable to a named source, tone matched to the publishing platform.

This exists because "write an article based on these sources" is an easy way to accidentally launder copyrighted text — a model asked to "summarize this article" will often reproduce whole sentences nearly verbatim unless explicitly told not to. This skill makes the non-copying rule and the attribution rule mechanical, not a vague reminder.

## When to Use This Skill

Trigger when the user asks to:
- Write a blog post / guide / article based on external sources (articles, studies, reports, interviews)
- Turn research (their own notes, a NotebookLM notebook, search results) into publishable copy
- Do this for a platform with its own voice/brand (needs tone-matching, not generic blog voice)

Not for: pure opinion pieces with no external sourcing, marketing copy for the platform's own product (that's `copywriting`), or SEO-structure work with no sourcing question involved (that's `seo`/`content-strategy`).

## Core Rules — Copyright & Attribution (non-negotiable)

1. **Never copy more than ~8-10 consecutive words verbatim from a source without quotation marks.** If a phrase is that distinctive, either quote it explicitly (`"exact phrase," according to X`) or rewrite the underlying fact in different words and sentence structure entirely.
2. **Every specific fact, figure, or claim gets an inline, named attribution in the prose itself** — "según AMPI Villahermosa", "La Jornada reportó", "de acuerdo con el estudio de X" — not a footnote, not a generic "studies show". The reader should never be able to mistake a sourced fact for something the platform itself investigated.
3. **Never invent a fact, quote, or statistic not present in the source material.** If a plausible-sounding number isn't in the sources, it doesn't go in the article — say less, not more.
4. **Synthesize, don't transcribe.** Read the source's point, then write the sentence from scratch in your own structure. If you catch yourself with the source open and typing what it says almost word-for-word, stop and rewrite from memory of the *fact*, not the sentence.
5. **End-of-article links are an editorial decision, not a default.** Ask: does linking out fit this platform's tone and keep it looking professional/authoritative, or does it read as thin/aggregated content? State the decision and the one-line reason; don't silently default either way.
6. **When multiple sources disagree or one is thin, say so** — a synthesized article that pretends single-source claims are consensus is itself a small dishonesty, same class of problem as fabricating data.

## Workflow

1. **Get the source material.** Three paths, pick what fits:
   - User already has notes/research → use as-is.
   - Nothing exists yet → research it (web search, or the `notebooklm` skill if the user wants a reusable, queryable source base — good for recurring topics on the same platform).
   - User points at a specific article/report → read it directly, don't paraphrase from a search-result snippet if the full text is reachable (snippets lose nuance and increase the risk of accidentally reproducing a search engine's own summary phrasing).
2. **Compile attributed notes before drafting** — one file/scratch note per research pass, each fact tagged with its source by name. This is the artifact that makes rule #2 mechanical later: if a fact in your notes has no source tag, it can't go in the article.
3. **Draft each article** applying Core Rules 1-4. Write for the platform's actual voice (check its existing published content for tone — formal vs. conversational, short vs. long paragraphs — don't default to generic "blog voice").
4. **Decide on end-links** (Core Rule 5) and state the decision to the user before publishing, don't bury it.
5. **Self-check before handing off — mandatory, every time, no exceptions.** This is two separate checks, not one:
   - **Truth check**: pick 2-3 sentences with the strongest claims and verify each traces to a named source. Live-fetch the actual source page if you have a fetch tool available and haven't read it directly yet — don't settle for a search-summary or a research tool's paraphrase of it.
   - **Wording check** (the one that's easy to skip and shouldn't be): for every sentence built from a research tool's output (NotebookLM answer, search-summary), open the real source next to it and compare word-by-word. A tool that answers "what does this source say" will often hand you phrasing that sits right at — or over — the copying line, because close paraphrase is what makes it accurate. Passing the truth check says nothing about the wording check; do both, separately, every article, not just when something feels off.
6. **Hand off in the format the target actually needs** — if it's going into a JSON-backed blog (common pattern: `id`, `slug`, `titulo`, `resumen`, `contenido`, `categoria`, `fecha`), match the existing schema exactly rather than inventing a new shape.

## Example

**Source fact:** An AMPI Villahermosa press statement said fraud losses "pueden ascender hasta los 300 mil pesos" and blamed unregistered informal sellers.

**Wrong (too close to source, uncredited):**
> Las pérdidas económicas para las víctimas podrían ascender hasta los 300 mil pesos, cometidas por inmobiliarias que no están registradas.

This is a near-verbatim lift with no attribution — exactly the failure mode this skill exists to prevent.

**Right (synthesized, attributed):**
> Según AMPI Villahermosa, las pérdidas de las víctimas en estos casos han llegado hasta los 300 mil pesos — y la asociación atribuye buena parte del problema a vendedores que operan sin registro.

Same fact, original sentence, source named inline, and the reader knows exactly whose claim it is.

## Common Pitfalls

- Asking the model to "summarize this article" without the non-copying/attribution rules stated explicitly — the default behavior leans toward close paraphrase, which is still infringement risk.
- Treating a search-engine-generated summary as a citable source in itself — cite the underlying outlet/study, not the search tool that surfaced it.
- Padding an article with a source recap instead of new synthesis, or the opposite (over-condensing until the attribution reads as an afterthought).
- Adding a platform-tie-in claim ("we verify every listing") that isn't actually true of the platform — the no-fabrication rule (Core Rule 3) applies to claims about the platform itself, not just the cited sources.
- Publishing without checking the platform's existing content schema/voice first — a technically correct article that doesn't fit the site's shape or tone still isn't done.
- Taking a research tool's answer (NotebookLM, an AI search summary) as the draft itself instead of as a set of facts to re-write from scratch. Real incident: a NotebookLM answer phrased a point close enough to its source that it copied an 8+ word run near-verbatim into a published article — caught only because the user asked "did you verify this," not by the drafting process itself. The truth check (is the fact real) and the wording check (is my sentence too close to the source's sentence) are different questions — verifying one is not evidence for the other.

## References

- [`notebooklm`](../notebooklm/SKILL.md) — for building a reusable, queryable source base when the topic will come up again (recurring platform content, not a one-off article)
