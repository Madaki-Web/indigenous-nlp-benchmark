# Indigenous Language AI Benchmark: Empirical NLP for Gbagyi

**Group 07 — Gbagyi (Gbagyi-Nkwa track)**  
**Course:** CSC 406 — Artificial Intelligence  
**Assignment:** Indigenous Language AI Benchmark  
**Date:** 2026-08-30

List Of Members Engaged in The work 
1. Nuhu Muhammad Datti U22 FNS CSC 1105
2. Mahmud Abubakar Madaki U22 FNS CSC 1091




## Abstract

This report describes a reproducible low-resource NLP pipeline for Gbagyi. Authentic public Gbagyi text was collected with Python `requests` from Biblica / YouVersion chapter pages that `robots.txt` permits, then normalised with Unicode-safe regular expressions and a custom tokenizer. The processed corpus contains **22,127 sentences**, **403,441 word tokens**, and a word vocabulary of **12,798**. Ordinary least squares on the log–log rank–frequency curve estimates a Zipf exponent **s = 1.4021** (R² = 0.9791). A from-scratch bigram model with Laplace (Add-1) smoothing attains a perplexity of **1400.8026** on the instructor’s unchanged file `tests/test_gbagyi_unseen.txt`. No sentences were fabricated.

## 1. Introduction

Gbagyi (ISO 639-3: `gbr`) is a Nupoid language of central Nigeria. Written resources are sparse, orthography is not fully standardised, and tone is often unmarked in published digital text. This assignment asks each group to build an inspectable baseline: scrape authentic text, tokenise it without English pretrained tools, test Zipf’s law, and estimate a smoothed n-gram language model.

The instructor’s Gbagyi unseen file is scripture-register prose (`shekwoi`, `jesun`, `yahudiyi`, `aduwa`). We therefore prioritised publicly readable Gbagyi Bible chapters that match that register, after verifying that `requests` receives server-rendered text and that `robots.txt` allows `/bible/` paths.

## 2. Objectives

1. Collect authentic Gbagyi documents with documented provenance.
2. Implement Unicode-safe cleaning and a custom tokenizer aligned with the unseen-test format.
3. Curate at least 30 attested Gbagyi function words without inventing glosses.
4. Estimate the Zipf exponent from the real corpus.
5. Train unigram and Add-1 bigram models from scratch and evaluate perplexity on the official test file.

## 3. Data sources and provenance

Raw JSONL stores every successful fetch, including English catalogue/encyclopedia pages. Those pages are **not** Gbagyi running text. The processed corpus is built only from authentic Gbagyi sentences after documented filters.

### Source classes

| Source class (URL pattern) | Source name | Language / version | Retrieval date | HTTP | Docs | Docs that contributed processed Gbagyi | Processed sentences | Filtering decision |
|---|---|---|---|---|---:|---:|---:|---|
| `https://www.bible.com/bible/1621/{BOOK}.{CH}.GAW` | Biblica Alkawali Woiwoyi (GAW 1621) chapter | Gbagyi (GAW / Alkawali Woiwoyi) | 2026-08-30 | 200 | 260 | 260 | 11532 | accepted as authentic Gbagyi scripture after verse extraction, English-UI filter, and held-out decontamination |
| `https://www.bible.com/bible/4607/{BOOK}.{CH}.GNB` | Biblica Gbagyi Contemporary Bible (GNB 4607) chapter | Gbagyi (GNB / Shekwoyi Ɓədagbma) | 2026-08-30 | 200 | 249 | 249 | 10595 | accepted as authentic Gbagyi scripture after verse extraction, English-UI filter, and held-out decontamination |
| `https://www.bible.com/versions/4607-gnb-gbagyi-nyizeyenya-baibwulu-shekwoyi-%C6%81%C9%99dagbma` | Bible.com version page: Gbagyi Contemporary Bible (GNB) | English version-catalogue page (not processed as Gbagyi) | 2026-08-30 | 200 | 2 | 0 | 0 | raw provenance only; bible.com version-catalogue pages excluded from processed Gbagyi text |
| `https://www.scriptureearth.org/00eng.php?iso=gbr` | Scripture Earth Gbagyi (gbr) index | English catalogue | 2026-08-30 | 200 | 2 | 0 | 0 | raw provenance only; English catalogue/encyclopedia excluded from processed corpus |
| `https://en.wikipedia.org/wiki/Gbagyi_people` | Wikipedia: Gbagyi people | English encyclopedia (not Gbagyi running text) | 2026-08-30 | 200 | 2 | 0 | 0 | raw provenance only; English catalogue/encyclopedia excluded from processed corpus |

Every GAW/GNB chapter URL is stored as its own JSONL record. The two scripture rows collapse those chapter URLs; the frozen file lists each URL in full.

### Non-chapter pages (full URLs)

| Source URL | Source name | Language / version | Retrieval date | HTTP | Contributed processed Gbagyi? | Sentences | Filtering decision |
|---|---|---|---|---|---|---:|---|
| https://www.bible.com/versions/1621-gaw-alkawali-woiwoyi | Bible.com version page: Alkawali Woiwoyi (GAW) | English version-catalogue page (not processed as Gbagyi) | 2026-08-30 | 200 | no | 0 | raw provenance only; bible.com version-catalogue pages excluded from processed Gbagyi text |
| https://www.bible.com/versions/4607-gnb-gbagyi-nyizeyenya-baibwulu-shekwoyi-%C6%81%C9%99dagbma | Bible.com version page: Gbagyi Contemporary Bible (GNB) | English version-catalogue page (not processed as Gbagyi) | 2026-08-30 | 200 | no | 0 | raw provenance only; bible.com version-catalogue pages excluded from processed Gbagyi text |
| https://www.bible.com/languages/gbr | Bible.com language index (gbr) | English navigation / language catalogue | 2026-08-30 | 200 | no | 0 | raw provenance only; English catalogue/encyclopedia excluded from processed corpus |
| https://en.wikipedia.org/wiki/Gbagyi_language | Wikipedia: Gbagyi language | English encyclopedia (not Gbagyi running text) | 2026-08-30 | 200 | no | 0 | raw provenance only; English catalogue/encyclopedia excluded from processed corpus |
| https://en.wikipedia.org/wiki/Gbagyi_people | Wikipedia: Gbagyi people | English encyclopedia (not Gbagyi running text) | 2026-08-30 | 200 | no | 0 | raw provenance only; English catalogue/encyclopedia excluded from processed corpus |
| https://www.scriptureearth.org/00eng.php?iso=gbr | Scripture Earth Gbagyi (gbr) index | English catalogue | 2026-08-30 | 200 | no | 0 | raw provenance only; English catalogue/encyclopedia excluded from processed corpus |

Wikipedia, Scripture Earth, and bible.com language/version catalogue pages remain in **raw JSONL for provenance**. None of those pages contribute processed Gbagyi sentences.

Attempted hosts: en.wikipedia.org, www.bible.com, www.scriptureearth.org.  
Documents written to JSONL: **515** (GAW chapters: 260; GNB chapters: 249).  
Unique source URLs: **515**.  
Retrieval date: **2026-08-30**.  
HTTP status: every stored JSONL record is a successful fetch (**HTTP 200**). Failed URLs were omitted; status is not an extra JSONL field because the autograder schema is `id`, `url`, `date_retrieved`, `raw_text`.

Copyright in the Gbagyi Bible text remains with Biblica, Inc. Records store URL provenance for this educational assignment and do not claim ownership of the scripture text.

## 4. Data collection methodology

`scrape_to_jsonl(url_list, output_path)` performs a polite GET for each URL:

- descriptive academic `User-Agent`
- 25-second timeout, three retries
- 0.7-second delay between requests
- `urllib.robotparser` check before fetch
- no CAPTCHA, login, or robots bypass

YouVersion chapter HTML is parsed with BeautifulSoup. Verse-like nodes and, when present, the `__NEXT_DATA__` payload are preferred over raw page text so that menus, player chrome, and English UI strings are not treated as Gbagyi. Empty or near-empty extractions are discarded. Exact-normalised document bodies are deduplicated.

Each JSONL line is:

```json
{"id": 1, "url": "https://...", "date_retrieved": "2026-08-30", "raw_text": "..."}
```

The autograder requires **integer** `id` values; we follow that contract.

## 5. Corpus statistics

| Metric | Value |
|---|---|
| Raw documents | 515 |
| Raw segmented sentences | 23,077 |
| Filtered sentences | 778 |
| Duplicate tokenized sentences | 172 |
| Held-out exact matches removed | 2 |
| Held-out high-overlap (containment) removed | 6 |
| Final authentic Gbagyi sentences | 22,127 |
| All tokens (incl. punctuation) | 466,282 |
| Word tokens | 403,441 |
| Word vocabulary | 12,798 |
| Token vocabulary (incl. punct.) | 12,834 |
| Mean sentence length (tokens) | 21.07 |
| Median sentence length | 19.00 |
| Min / max sentence length | 1 / 96 |
| Unique URLs | 515 |

### Most frequent word types

| Rank | Word | Frequency |
|---|---|---|
| 1 | n | 33527 |
| 2 | ɓa | 14607 |
| 3 | wo | 11033 |
| 4 | wa | 8730 |
| 5 | nu | 8415 |
| 6 | yi | 7284 |
| 7 | fye | 6171 |
| 8 | lo | 5968 |
| 9 | mi | 5674 |
| 10 | zhin | 5555 |

JSONL validation errors: 0.  
Processed-corpus validation errors: 0.

## 6. Preprocessing methodology

Pipeline: HTML/XML removal → control-character removal → Unicode NFC → whitespace normalisation → sentence segmentation → custom tokenisation.

NFC is used so that **ɓ**, **ə**, subdot letters, and any combining marks stay intact. We do not call `unidecode` and we do not fold to ASCII.

Sentencehood is not “one HTML line”. Blocks are split on `. ! ?` after cleaning. Bible verses without terminal punctuation are kept as one sentence.

**Processed-corpus filters (reproducible in `build_processed_corpus`):**

1. Wikipedia, Scripture Earth, and `bible.com/languages/` pages contribute **zero** processed sentences.
2. `bible.com/versions/` pages contribute **zero** processed sentences (raw provenance only).
3. English-majority sentences and catalogue/template phrases (`bible versions`, Biblica ministry boilerplate, Wikipedia help text, etc.) are rejected on every source, including chapter pages.
4. Exact tokenized identity with `tests/test_gbagyi_unseen.txt` is removed from training (**held-out decontamination**, not lexicon mining).
5. High-overlap contamination: a training line is dropped if its word sequence and a test sentence stand in contiguous containment either way and the shorter sequence has ≥ 4 word tokens. Shared function words alone are not enough. Related but non-identical verses (for example `aɓi` vs `aɓeye` in otherwise similar clauses) are kept.

The instructor test file is never edited. If decontamination raises perplexity, the higher number is the honest result.

## 7. Tokenization methodology

`custom_tokenizer` is implemented with `re` only (`TOKEN_RE`). It:

- lowercases with Python 3 `str.lower()` (Ɓ → ɓ)
- detaches punctuation (`yi .`, `yi ,`)
- keeps internal hyphens (`bui-bui`, `zaho-zahoyi`, `tnu-tnu`)
- emits a single-space string with no leading/trailing space

This matches the instructor file format, for example:

Input: `Fye zhin bugba bui-bui ntu ge lada dnagmayi shi lo.`  
Output: `fye zhin bugba bui-bui ntu ge lada dnagmayi shi lo .`

Implosive **ɓ** present in processed corpus: **True**.

## 8. Stop-word methodology

We curated **35** Gbagyi function words (**30** attested, **5** uncertain). Primary linguistic sources are the Gbagyi noun-phrase field manuscript (*The Structure of Noun Phrases in Gbagyi*, Niger State speaker interviews) and independently attested forms in collected Biblica GAW/GNB chapter text. We did **not** invent lexemes and we did **not** use `tests/test_gbagyi_unseen.txt` as a lexicon or linguistic source. That file is used only as a held-out evaluation set and as an exclusion list for train/test decontamination.

Where a spelling has more than one grammatical function (for example `o` as preposition vs coordinator; `ye` as demonstrative vs verb/particle; `ma` as particle vs the verb ‘give birth’), the table records the ambiguity instead of collapsing the senses. Uncertain items (`na`, `ga`, `a`, `ku`, `ma`) are labelled **uncertain** rather than given a forced gloss.

Stop words are **not** deleted from `cleaned_corpus_group_07.txt`. Removing them would make language-model evaluation incomparable with the official tokenized test format.

| Gbagyi | English | Category | Confidence | Source |
|---|---|---|---|---|
| mi | I / my | pronoun / possessive | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| omi | my | possessive determiner | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| wo | his / him / her | pronoun / possessive | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| wa | he / she (3sg subject) | pronoun | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| wu | he / she | pronoun | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ɓa | they (3pl) | pronoun | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ye | this (near demonstrative). In scripture orthography the same spelling also occurs as a high-frequency verb/particle; those uses are not collapsed here. | demonstrative / function word | attested (demonstrative); other uses vary | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| yi | that (distant demonstrative) / anaphoric particle | demonstrative / particle | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ho | the (optional determiner) | determiner | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| lo | tense/aspect or predicative particle | auxiliary / particle | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| o | on / in (preposition); also coordinating 'and' in some examples | preposition / conjunction | attested (polysemous: preposition and coordinator) | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| n | relativizer / linker 'that'; also a high-frequency clitic linker | relativizer / conjunction | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| nu | determiner / copular-focus particle | determiner / particle | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| zhni | be / become (copula) | auxiliary / copula | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| zhin | be / become (copula) | auxiliary / copula | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| kwo | it (3sg inanimate / resumptive) | pronoun | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ge | quotative / complementizer 'that' (naming, reported speech) | conjunction / complementizer | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| nya | of (associative / genitive) | preposition / genitive marker | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| to | not / negative particle | negation / particle | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| ntu | so that / in order that (purpose) | conjunction | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| ntuge | because / so that (purpose-reason) | conjunction | attested | Biblica (1997/2025), Gbagyi Contemporary Bible (GNB), form attested in collected |
| gmanyi | one / some (quantifier) | determiner / quantifier | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| vnyanya | all / whole | quantifier / determiner | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ama | but | conjunction | attested (Hausa loan used as a Gbagyi conjunction in these publications) | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| sai | then / only / except | conjunction / particle | attested (Hausa loan used as a Gbagyi particle in these publications) | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| har | until / even | conjunction / preposition | attested (Hausa loan used as a Gbagyi function word in this publication) | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| shi | then / and then (sequential) | conjunction / particle | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| ma | and (coordinator in some constructions) / 'give birth to' as a content verb — listed here only as the high-frequency coordinator/particle sense when not the main verb | conjunction / particle | uncertain (polysemous) | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| ga | clause-final / focus particle (function word; exact force varies by dialect) | particle | uncertain | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| na | associative / high-frequency linker (exact sense varies) | particle / linker | uncertain | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| nyi | locative / relational particle (often clause-final) | particle | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ɓe | come (light verb / motion; also appears in serial constructions) | auxiliary / light verb | attested | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ɓei | past / sequential auxiliary appearing before zhin in GAW genealogies | auxiliary | attested | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |
| a | they / impersonal plural prefix or pronoun (context-dependent) | pronoun / agreement | uncertain (prefix vs independent pronoun) | Unpublished field manuscript, The Structure of Noun Phrases in Gbagyi (Niger Sta |
| ku | to / and (light preposition; also in the Hausa-origin phrase ku gode 'give thanks') | preposition / particle | uncertain | Biblica (1997), Alkawali Woiwoyi (GAW), form attested in collected chapter text. |

Limitation: several high-frequency particles lack a single published one-word English equivalent. We report uncertainty rather than inventing precision.

## 9. Zipf analysis

Word types were ranked by descending frequency. We fitted

`log(f) = C − s log(r)`

by `numpy.polyfit` on natural logs.

| Quantity | Estimate |
|---|---|
| s | 1.402137 |
| C | 12.890745 |
| R² | 0.979136 |
| Types used (N) | 12798 |

The figure `zipf_rank_frequency.png` shows the observed log–log cloud and the OLS line. A classical Zipf exponent near 1 is a large-corpus ideal. Our estimate should be read as a **property of this written, scripture-heavy sample**, not as a universal constant of spoken Gbagyi.

Orthographic variation (GAW `zhin` vs GNB `zhni`; `Yesu` vs `Yeisu`; optional **ə**) splits what may be one lemma into several types and therefore flattens or steepens the tail depending on how variation is distributed. Unicode distinctions (ɓ vs b) also expand V. We do not claim a dialectological finding beyond that measurement caveat.

## 10. Unigram model

Unigram counts are dictionary frequencies on the processed corpus. The MLE is `P(w) = count(w) / N` with `N = 466,282` tokens and `V_unigram = 12,834` types (all tokens, including punctuation). Add-1 unigram smoothing is implemented for comparison but is not used for the official perplexity number.

## 11. Bigram model

`BigramModel.fit` counts adjacent token pairs inside each sentence (no sentence-boundary symbols, matching the instructor template). It returned **444,155** bigram tokens. `vocab_size` is the number of unique unigram types in training, **12,834**.

## 12. Laplace (Add-1) smoothing

```
P(w2 | w1) = (count(w1, w2) + 1) / (count(w1) + V)
```

If `w1` is unseen, `count(w1) = 0` and `P = 1/V`. A probe with a fake context yields **0.00007792**, which equals `1/V = 0.00007792`.

## 13. Perplexity

Evaluation file: `tests/test_gbagyi_unseen.txt` (unchanged).

`PP = exp( −(1/N) ∑ log P(w_i | w_{i-1}) )`

which is identical to the assignment’s base-2 form when logs match. An independent `log2` recomputation produced **1400.802620** versus **1400.802620**.

| Quantity | Value |
|---|---|
| Test sentences | 15 |
| Test tokens | 142 |
| Predicted bigrams (N) | 127 |
| Training V | 12834 |
| Smoothing | Laplace / Add-1 |
| Bigram perplexity | **1400.802620** |

## 14. Results

- Authentic JSONL documents: 515
- Final Gbagyi sentences: 22,127
- Word vocabulary: 12,798
- Zipf s = 1.4021, R² = 0.9791
- Blind-test bigram perplexity = 1400.8026

The sentence count meets the 2,500-sentence assignment floor **from collected public Gbagyi text**, not from synthesis.

## 15. Discussion

The corpus is large enough for a classroom benchmark but narrow in genre: published Christian scripture in two Biblica orthographies. That is appropriate for the hidden test file, which is written in the same register, and it is also a limitation for any claim about “Gbagyi in general”. High-frequency items are function words and names (`wa`, `zhin`/`zhni`, `n`, `nu`, `yesu`/`yeisu`). Zipf’s law holds only approximately; R² = 0.979 on a still-small type inventory.

Diacritics and the implosive **ɓ** are preserved. Tone is largely unmarked in these digital editions, so tonal minimal pairs, if they exist in speech, are merged in writing. Mixing GAW and GNB increases lexical fragmentation and is scientifically honest: both are legitimate Gbagyi publications.

## 16. Limitations

1. **Bible-register concentration.** After English/catalogue filtering, essentially all processed sentences are New Testament translations in two Biblica orthographies (GAW and GNB). This matches the hidden test register and is also a serious domain limitation: the Zipf exponent, vocabulary, and perplexity describe written Christian Gbagyi, not spoken Gbagyi or other genres.
2. Two official orthographies inflate vocabulary (`zhin`/`zhni`, `Yesu`/`Yeisu`).
3. Sentence splitting on punctuation is an approximation; some verses contain coordinated clauses.
4. Stop-word glosses for a few particles remain uncertain; 5 of 35 entries are labelled uncertain.
5. Held-out decontamination uses exact match plus ≥4-token contiguous containment. Near-paraphrases that do not contain a test word sequence are retained.
6. The instructor autograder’s `BigramModel` import path still points at Group 01 Nupe; we did not move Gbagyi files into Nupe folders.
7. GitHub Actions uses a shallow clone, so the collaboration test may fail on CI even with a genuine multi-author history.

## 17. Reproducibility

From the repository root, with `requirements.txt` installed (Python 3.10 in CI):

```text
python submissions/group_07_gbagyi/HW1_assignment.py
pytest tests/autograder_eval.py -v --tb=short
```

The notebook imports the same module. Collection respects robots.txt and will skip a URL if a site later disallows it. Randomness is not used in preprocessing, Zipf, or perplexity.

## 18. Conclusion

Group 07 delivers an inspectable Gbagyi baseline: provenanced JSONL, a validated one-sentence-per-line corpus, a documented custom tokenizer, an attested stop-word table, a measured Zipf fit, and a from-scratch Add-1 bigram model evaluated on the official unseen file. The numerical results above are generated from those artefacts.

## References

1. Biblica, Inc. (1997). *Alkawali Woiwoyi* (GAW). Retrieved from https://www.bible.com/versions/1621-gaw-alkawali-woiwoyi
2. Biblica, Inc. (1997/2025). *Gbagyi Nyizeyenya Baibwulu: Shekwoyi Ɓədagbma* (GNB). Retrieved from https://www.bible.com/versions/4607-gnb-gbagyi-nyizeyenya-baibwulu-shekwoyi-%C6%81%C9%99dagbma
3. Blench, R. (2013). *The Nupoid languages of west-central Nigeria: Overview and comparative wordlist*. Working document. https://rogerblench.info/Language/Niger-Congo/VN/Nupoid/Nupoid%20Overview%202013.pdf
4. Dalhatu, A. M. (2019). Gbagyi syllable and phonotactics. *Journal of the Nigerian Languages Project*, 1. https://jnlp.com.ng/index.php/home/article/view/2
5. Hyman, L. M., & Magaji, D. J. (1970). *Essentials of Gwari grammar* (Occasional Publication 27). Ibadan: Institute of African Studies.
6. Jurafsky, D., & Martin, J. H. (2025). *Speech and language processing* (3rd ed. draft). https://web.stanford.edu/~jurafsky/slp3/
7. Rosendall, E. P. (1998). *Aspects of Gbari grammar* (Master’s thesis). University of Texas at Arlington.
8. Rosendall, H. J. (1992). *A phonological study of the Gwari lects*. Dallas: SIL.
9. Scripture Earth. Gbagyi (`gbr`) resource index. https://www.scriptureearth.org/00eng.php?iso=gbr
10. Wikipedia. (2026). Gbagyi language. https://en.wikipedia.org/wiki/Gbagyi_language
11. Wikipedia. (2026). Gbagyi people. https://en.wikipedia.org/wiki/Gbagyi_people
12. Zipf, G. K. (1949). *Human behavior and the principle of least effort*. Addison-Wesley.
13. YouVersion / Bible.com `robots.txt` (checked 2026-08-30). `/bible/` chapter paths are allowed; `/bible/*/*/notes` is disallowed.
14. Field examples of Gbagyi noun-phrase function words as reported in *The Structure of Noun Phrases in Gbagyi* (Niger State interview data; demonstratives *yè/yî*, determiner *hò*, relativiser *ń*, preposition *o*).
