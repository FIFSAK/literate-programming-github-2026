# LLM review of spec.md

This is the last of four review rounds. After each of the first three rounds I
fixed the problems the review found and ran the review again with the same
prompt. This round found no wrong expected outputs and no contradictions; the
four remaining findings are addressed in this commit, as noted after the output.

## Model and prompt

Prompting **Claude Opus 5.5** (`claude-opus-5-5`, a subagent in Claude Code,
default effort), with no files edited by the model, with the following prompt:

```
Review anu_kurmanov/spec.md (in the current directory). It is a specification for this problem:

"A news site shows a "Trending words" box beside each article. Write a Python function that finds the 10 most common words in an article's text, along with how many times each one appears. Articles may be in English, Russian, or Kazakh. Implementation: Python 3. Name the function `top_words` and put it in `top_words.py`. Assignment: Tighten this specification to produce one plausible result. Then, for each decision your spec makes, describe at least one input and its expected output, usually as a table in your spec. A useful example gets a different output under some other plausible reading of the original spec."

Find errors (including wrong expected outputs in the tables), contradictions, and underspecified behavior that would let two careful implementers produce different results. Do not edit any files. Report a numbered list of findings, each with the location, the problem, and a suggested fix; put the most serious first. If something is fine, don't list it.
```

produced the following output. Strikethrough indicates LLM output that I deemed
incorrect or irrelevant. The only edit is that the absolute path to the file is
shortened to `anu_kurmanov/spec.md`.

## Output

> I reviewed anu_kurmanov/spec.md without editing anything. The spec is in very
> good shape. I found no wrong expected outputs, no contradictions and nothing
> that would let two careful implementers get different results. What's left is
> three wording problems and one gap in the examples.
>
> How I checked: I wrote a reference implementation that follows the spec
> literally (type check, `str.__str__`, NFC, scan, then per word: swap ’ for '
> and ‐ for -, `lower()`, NFC; then sort by count descending and word ascending,
> keep 10). I ran every table row through it on Python 3.14. All rows match, and
> every `raises TypeError` row raises. The prose claims also hold:
>
> - The `Shout` subclass example returns `[("hi", 2)]` with a plain `str`.
> - Lowering `ΑΣ.Β` as a whole text gives `σ`, while lowering the word `ΑΣ`
>   gives `ς`.
> - `'`/`’` are case-ignorable and `-`/`‐` are not, so it doesn't matter whether
>   the ’ swap happens before or after lowering.
> - U+02BC is a letter (Lm); U+00AD, U+2011 and U+20DD are not letters and not
>   Mn/Mc marks.
> - No character is both `isalpha()` and a mark, so "letter" and "mark" never
>   overlap.
> - Composing U+0302 onto `á`, and U+0301 onto Cyrillic `о`, has no precomposed
>   result.
>
> Findings, most serious first (all minor):
>
> 1. **Section 6, first paragraph: the note about letters below U+0400 is
>    inaccurate.** It says letters below U+0400 are "Basic Latin through Latin
>    Extended-B". Latin Extended-B ends at U+024F. The range below U+0400 also
>    includes IPA Extensions, Spacing Modifier Letters (for example `ʼ` U+02BC,
>    which section 4 makes a letter) and Greek (U+0370–U+03FF), which section 3
>    uses in its examples. The sorting rule itself (plain code-point order) is
>    right; only the description is misleading. Fix: write "Letters below U+0400
>    (Latin up to Latin Extended-B, IPA, modifier letters and Greek) sort before
>    Cyrillic", or drop the note and keep "compares code points".
>
> 2. **Section 8, last bullet: the performance rule can't be tested and is
>    vague.** ~~"O(n log n) time or better" has no example.~~ "Calling
>    `list.count` or `str.count` once per word is not allowed" doesn't say
>    whether "word" means each occurrence or each distinct word. It also bans a
>    technique instead of stating a requirement, which is odd under "Out of
>    scope". Fix: ~~either give a measurable requirement (for example, "an
>    article of 1,000,000 words, 100,000 of them distinct, finishes in under N
>    seconds on the reference machine") or~~ drop the ban on specific calls and
>    keep only the complexity bound.
>
> 3. **Section 7, first paragraph: one sentence reads wrongly on its own.**
>    "...the result is still exactly 10 entries" sits right after "If there are
>    fewer than 10 distinct words, all of them are returned". Read alone, it
>    clashes with `min(10, number of distinct words)`. Fix: "If more than 10
>    distinct words exist and several tie at 10th place, ... the result has
>    exactly 10 entries."
>
> 4. **Sections 1 and 4.4: no example covers Kazakh Latin apostrophe
>    spellings.** The intro says Kazakh Latin is supported, but the 2017 Kazakh
>    Latin alphabet wrote letters like ш as `s'`. Under section 4.4 a word-final
>    apostrophe is a separator, so a word ending in such a letter loses it. The
>    rule is clear, so implementers won't disagree; the gap is that the spec
>    doesn't mention this. Fix: add a row, for example `"as' as"` →
>    `[("as", 2)]`, and one sentence saying apostrophe-based Kazakh Latin
>    spellings get no special handling. Or narrow "Kazakh Latin" to the 2021
>    diacritic alphabet.
>
> There is nothing more serious to report.

## What I changed in response

1. Accepted. Section 6 now lists Latin up to Latin Extended-B, IPA, modifier
   letters, and Greek as the letters below U+0400.
2. Partly accepted. I removed the ban on `list.count` / `str.count` and kept
   only the O(n log n) bound. I struck out the benchmark suggestion: the spec
   puts performance out of scope, and a time limit on an unnamed "reference
   machine" can't be checked either. A complexity bound is a normal
   requirement and doesn't need a table example.
3. Accepted. Section 7 now says the "exactly 10 entries" case applies only when
   there are more than 10 distinct words.
4. Accepted. Section 4.4 now says 2017-style apostrophe spellings get no
   special handling, with three rows: `"as' as"`, `"s'ah sah"`, and
   `"Şaş şaş"`.

## Earlier rounds (summary)

- **Round 1** (9 findings): the joiner rule disagreed with its summary sentence;
  "ignored" for a stray combining mark was ambiguous; no rule for Turkic
  `İ`/`ı`; results depend on the interpreter's Unicode version; soft hyphen,
  non-breaking hyphen, and `ʼ` were unspecified; invisible decomposed
  characters were stored raw; several decisions had no example; the column
  description didn't match the `TypeError` rows; the performance target was
  subjective. All fixed.
- **Round 2** (6 findings): "linear time" conflicted with sorting; no second NFC
  after `lower()` (`J̌` vs `ǰ`), and per-word vs whole-text lowering (Greek
  final sigma); `str` subclasses were underspecified; wrong prose for the `á̂`
  row; the joiner condition was worded loosely; enclosing marks (`Me`) had no
  example. All fixed.
- **Round 3** (7 findings): "single joiners" could mean one per word; loose
  restatements of the joiner rule in 4.3/4.4; "all Latin before Cyrillic" was
  false for letters such as `ạ`; the `"ёж еж"` ordering row didn't distinguish
  code point order from Russian collation; joiners change Greek final sigma;
  an inaccurate final-sigma explanation; "before anything else" conflicted with
  the type check. All fixed.

Every expected output in spec.md was also checked against a reference
implementation that follows the spec: all 79 table rows match.
