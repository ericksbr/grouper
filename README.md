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

## Labels
- **Ungrouped**: no other comment is similar enough
- **Grouped - Identical**: exactly matches another comment after normalization
- **Grouped - Highly similar**: best match in group has Jaccard ≥ 0.90
- **Grouped - Moderately similar**: best match in group has Jaccard in (0.50, 0.90)

## Groups
Comments are visited **most-copied first**, so the most common letter becomes a group's representative. A comment that has not joined a group yet becomes a representative, and it claims the unassigned comments that are at least 50% similar to it. A comment joins the first representative that matches it. Every member is at least 50% similar to its group's representative, and each representative belongs to its own group, so no group chains through a middle comment. Groups are numbered by size, largest first; ungrouped comments are group size 1.

To keep the search manageable, each fingerprint bucket allows only five representatives in the first pass. A second pass lets every leftover comment start its own group, so no comment is dropped.

## Families
Each group is described by its **medoid**, the member most similar to the rest of its group. Medoids are compared with each other, and groups whose medoids are at least 40% similar are linked into **families**. Families can reveal coordinated campaigns that use several related templates. Families are only computed across groups with 2+ members.

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
