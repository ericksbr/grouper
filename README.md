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
Each group is anchored on one representative comment (the comment that started it), and every member is at least 50% similar to that representative. Comments are added greedily, most-connected first, so a group cannot grow by chaining through intermediate comments that are unlike the representative. Groups are numbered by size, largest first; ungrouped comments are group size 1.

## Families
Groups whose representatives are at least 30% similar are linked into **families**. Families can reveal coordinated campaigns that use several related templates. Families are only computed across groups with 2+ members.

## Output columns
Added to your original columns: `group_id`, `group_size`, `group_label`, `is_identical`, `J_to_rep` (similarity to the group's representative), `max_j_in_group` (best match within the group), `best_match_id`, `is_group_rep`, `group_rep_id`, `family_id`, `family_size`. A `family_id` of 0 means the comment is ungrouped.

## Note
Similarity is based on *word-level 5-gram shingles* (word overlap), not semantic topic similarity. Two comments discussing the same topic in different words may not group together.
