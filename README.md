<p align="center">
  <img src="images/grouper_logo.png" alt="Grouper logo" width="400"/>
</p>

# Grouper

Grouper groups similar public comments based on **word overlap** (not topic). It finds comments with shared wording, even if they're not identical. Upload a CSV, Excel, or Parquet file, run the notebook in Google Colab, and download your file with each comment labeled by grouping status and similarity level.

## Labels
- **Ungrouped**: no other comment is similar enough
- **Grouped - Identical**: exactly matches another comment after normalization
- **Grouped - Highly similar**: best match in group has Jaccard ≥ 0.90
- **Grouped - Moderately similar**: best match in group has Jaccard in (0.50, 0.90)

## Groups
Comments are visited most-copied first. A comment that has not joined a group yet becomes a group's representative, and it claims the unassigned comments that are at least 50% similar to it. A comment joins the first representative that matches it. Every member is at least 50% similar to its group's representative, and each representative belongs to its own group, so no group chains through a middle comment. Groups are numbered by size, largest first; ungrouped comments are group size 1.

## Families
Each group is also described by its **medoid**, the member most similar to the rest of its group. Medoids are compared with each other, and groups whose medoids are at least 40% similar are linked into **families**. Families can reveal coordinated campaigns that use several related templates. Families are only computed across groups with 2+ members.

## Output columns
Added to your original columns: `group_id`, `group_size`, `group_label`, `is_identical`, `J_to_rep` (similarity to the group's representative), `max_j_in_group` (best match within the group), `best_match_id`, `is_group_rep`, `group_rep_id`, `family_id`, `family_size`, `family_anchor_row`. A blank `family_id` means the comment is not in a family.

## Note
Similarity is based on *word-level 5-gram shingles* (word overlap), not semantic topic similarity. Two comments discussing the same topic in different words may not group together.
