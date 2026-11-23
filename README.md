# IR-TP1 – Document Preprocessing, Representation and Indexing

Information Retrieval – Work Session 1 (Master 2 – Computer Science).

## How to run
```bash
git clone <repo-url> && cd IR-TP1-Document-Preprocessing-and-Indexing
pip install -r requirements.txt
jupyter notebook tp1_preprocessing_indexing.ipynb      # Run All
```
The NLTK stop-word list is downloaded automatically on the first run (internet needed once). All outputs are written to `results/`.

## Structure
```
├── README.md
├── requirements.txt
├── tp1_preprocessing_indexing.ipynb   # all code, outputs and verification tests
└── results/                           # tokens, vocabularies, matrices, inverted indexes, comparison table
```

## Implementation choices
- **Three versions** of the collection, each built from the previous one: **A** = lowercase + punctuation removal (`re.sub(r'[^\w\s]', ' ', ...)` + `split()`); **B** = A + NLTK English stop-words (filtering coded by hand); **C** = B + NLTK `PorterStemmer`. Original texts are never modified.
- **Vocabulary:** unique terms in alphabetical order (reproducible).
- **Matrices:** `build_term_document_matrix` (binary) and `build_tf_matrix` (counts) are computed with plain loops; pandas is used only to display/export them (terms = rows, document IDs = columns).
- **Inverted index:** `term -> {doc_id -> [zero-based positions]}` built with `defaultdict` + `enumerate`; positions refer to the *processed* token list of each version. `search_term()` returns the document IDs (empty list for an unknown term).
- **Stemming and queries:** in version C the query must be stemmed too (`search_term(idx_C, stemmer.stem('retrieval'))`).
- **Verification (Part 5):** brute-force comparison for every term of A/B/C, the four required cases, position checks (`tokens[doc][pos] == term`, total positions = total tokens), manual vs automatic index for D1–D3, and matrix checks (binary only 0/1, binary == TF>0, TF = Counter result). All assertions pass.

## Main results
| Version | Vocabulary | Avg tokens/doc | Postings | Stored positions |
|---|---|---|---|---|
| A: lowercase + punctuation | 36 | 6.9 | 66 | 69 |
| B: A + stop-words | 30 | 6.1 | 59 | 61 |
| C: B + stemming | 24 | 6.1 | 57 | 61 |

Search example: `retrieval` returns D1, D2, D4, D8, D10 in B, but the stemmed query `retriev` returns D1, D2, D3, D4, D6, D7, D8, D10 in C.

## Interpretation (preprocessing effect)
Preprocessing shrinks the index step by step. Lowercasing and punctuation removal (A) already merge forms such as `Documents`/`DOCUMENTS`, giving 36 terms. Stop-word removal (B) deletes frequent, low-information words (`and`, `the`, `to`, `about`): the vocabulary drops to 30, tokens per document from 6.9 to 6.1, postings from 66 to 59 and stored positions from 69 to 61. Stemming (C) does not remove tokens (still 61 positions) but merges variants (`retrieval`/`retrieve`, `index`/`indexing`, `document`/`documents`), reducing the vocabulary to 24 and postings to 57. A smaller index is cheaper and increases recall, but not every step is harmless: stop-word removal shifts positions and breaks exact phrase queries, and stemming can conflate different meanings and produce non-words (`effici`), so precision may fall.
