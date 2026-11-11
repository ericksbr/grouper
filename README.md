<p align="center">
  <img src="images/grouper_logo.png" alt="Grouper logo" width="400"/>
</p>

# Grouper

Grouper groups similar public comments based on **word overlap** (not topic). It finds comments with shared wording, even if they're not identical. Upload a CSV, Excel, or Parquet file, run the notebook in Google Colab, and download your file with each comment labeled by grouping status and similarity level.

## Using the notebook

`grouper.ipynb` runs top to bottom in Colab, with a step-by-step explanation in each section:

1. **Act 1 — Analyze:** uploads your comments, finds exact copies and near copies, and groups them.
2. **Intermission — Save:** saves your results as a CSV (opens in Excel) and a Parquet file (for Python or R).
3. **Act 2 — Explore:** charts of the groups and families, and tools to compare comments side by side.

Grouper guesses which column holds the comment text and which holds a commenter ID. If it guesses wrong, you can pick the right columns from dropdown menus.

## How Grouper Works

Grouper finds comments that share wording. It compares words, not meaning. Real comments usually have several sentences, and Grouper scores each whole comment. Length matters: in a short comment, one changed word affects a larger share of the comparison, so a small edit can lower a score a lot.

Act 1 of the notebook runs these stages. This example follows them with six short comments:

- **A:** I oppose this plan to protect wolves. It harms their habitat and elk herds.
- **B:** I oppose this plan to protect wolves. It harms their habitat and elk herds.
- **C:** i OPPOSE this plan to protect wolves! it harms their habitat and elk herds.
- **D:** I STRONGLY oppose this plan to protect wolves. It harms their habitat and elk herds.
- **E:** I oppose this plan to protect wolves. It harms their habitat and deer herds.
- **F:** Wolves matter to our ecosystem. Please protect them for future generations.

**Stage 2: Find exact copies.** Grouper first looks for comments that match word for word, punctuation included. B matches A exactly, so B is an identical copy of A.

**Stage 3: Normalize, then look again.** Grouper lowercases each comment and removes punctuation, then looks for matches again. C matches A and B after this step, so C is also an identical copy and joins their group.

**Stage 4: Break the text into shingles, then fingerprint them.** A *shingle* is a run of words in a row (e.g., 5-word segments), taken from the normalized text. A has 10 5-word shingles:

<div align="center">

| A's shingles |
|:---:|
| i oppose this plan to |
| oppose this plan to protect |
| this plan to protect wolves |
| plan to protect wolves it |
| to protect wolves it harms |
| protect wolves it harms their |
| wolves it harms their habitat |
| it harms their habitat and |
| harms their habitat and elk |
| their habitat and elk herds |

</div>

Grouper fingerprints the set of shingles, not the whole comment. A fingerprint is a short list of numbers that stands in for a comment's shingles. It is built with MinHash, a well-established technique for finding near-duplicate documents at scale. MinHash runs each shingle through several hash functions, a kind of scrambling, and keeps the smallest result from each function. Comments with many of the same shingles end up with many of the same numbers. Grouper uses 256 numbers per fingerprint, so comparing fingerprints is much faster than comparing full texts. For a simple picture, imagine three numbers: if A's fingerprint is [17, 4, 52] and another comment's is [17, 9, 52], the two agree on two of three, which suggests similar text.

**Shingle length matters.** Short shingles (1–3 words) make small edits matter less. For D, the score is 0.79 with 3-word shingles, 0.75 with 5-word shingles, and 0.70 with 7-word shingles. But short shingles also match ordinary phrases, such as "I oppose this," across comments that have nothing else in common. Longer shingles (5–10 words) are more specific to a comment, but a single edit breaks more of them. For D, the score falls to 0.57 with 10-word shingles. Grouper uses 5-word shingles as a middle ground: long enough that common phrases don't match by chance, and short enough that one changed word doesn't break most matches.

**Stage 5: Score, then group.** Grouper scores a candidate pair by counting shared shingles and dividing by all the different shingles the two comments have between them. Scores run from 0 to 1. In the tables below, ✓ means the sentence's shingles all match, and ✗ means some of them differ.

<div align="center">

**D compared with A**

| A | | D |
|:---|:---:|:---|
| I oppose this plan to protect wolves. | ✗ | I STRONGLY oppose this plan to protect wolves. |
| It harms their habitat and elk herds. | ✓ | It harms their habitat and elk herds. |

</div>

D shares 9 shingles with A, and the two comments have 12 different shingles between them. Score: 9 ÷ 12 = 0.75, labeled "Grouped - Moderately similar."

<div align="center">

**E compared with A**

| A | | E |
|:---|:---:|:---|
| I oppose this plan to protect wolves. | ✓ | I oppose this plan to protect wolves. |
| It harms their habitat and elk herds. | ✗ | It harms their habitat and deer herds. |

</div>

E shares 8 shingles with A, and the two comments have 12 different shingles between them. Score: 8 ÷ 12 ≈ 0.67, labeled "Grouped - Moderately similar."

<div align="center">

**F compared with A**

| A | | F |
|:---|:---:|:---|
| I oppose this plan to protect wolves. | ✗ | Wolves matter to our ecosystem. |
| It harms their habitat and elk herds. | ✗ | Please protect them for future generations. |

</div>

F shares no shingles with A. Score: 0, so F stays "Ungrouped."

Grouper visits the most-copied comments first. A is the most-copied, so it becomes the group's representative. Any comment that scores 0.50 or higher with the representative joins its group. A representative always belongs to its own group, so groups don't chain together through a middle comment. Groups are numbered by size, largest first; ungrouped comments are group size 1. Grouper can limit how many representatives a fingerprint bucket allows in its first pass. A second pass gives any leftover comment its own group, so no comment is dropped.

**Families (Stage 7).** Each group is described by its medoid, the member most similar to the rest of its group. Grouper compares medoids across groups. Groups whose medoids score 0.40 or higher are linked into one family. Families can reveal coordinated campaigns that use several related templates. F is ungrouped, so it isn't in a family.

Grouper compares wording, not meaning. A single changed word can move a comment to a different group level, even when the two comments say nearly the same thing.

## Labels
- **Ungrouped**: no other comment is similar enough
- **Grouped - Identical**: exactly matches another comment after normalization
- **Grouped - Highly similar**: best match in group has Jaccard ≥ 0.90
- **Grouped - Moderately similar**: best match in group has Jaccard in (0.50, 0.90)

## Output columns
Added to your original columns: `group_id`, `group_size`, `group_label`, `is_identical`, `J_to_rep` (similarity to the group's representative), `max_j_in_group` (best match within the group), `best_match_id`, `is_group_rep`, `group_rep_id`, `family_id`, `family_size`, `family_anchor_row`. A blank `family_id` means the comment is not in a family.

The `*_id` and `*_row` columns that point to other comments (`best_match_id`, `group_rep_id`, `family_anchor_row`) hold **row positions** in your input file, not comment IDs.

## What changed from the original version

The original notebook is kept in `archive/grouper_original.ipynb` for reference, and is no longer maintained. The current `grouper.ipynb` differs in these ways:

- **Interface:** a step-by-step, narrative layout with Act 1 (analyze), Intermission (save), and Act 2 (explore). Column detection can be corrected from dropdowns.
- **Output:** the same grouping columns, plus a Parquet copy of the results.
- **Visit order:** comments are visited most-copied first (the original visited most-connected first).
- **Representative cap:** each fingerprint bucket allows five representatives in the first pass, with a second pass for leftover comments.
- **Families:** linked by medoids rather than group representatives. The threshold (`FAMILY_J = 0.40`) is unchanged.
- **Exploration:** saved charts and browsable families and groups.

## Note
Similarity is based on *word-level 5-gram shingles* (word overlap), not semantic topic similarity. Two comments discussing the same topic in different words may not group together.
