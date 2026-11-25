# Best Time to Post on TikTok: Posting Hour vs. Engagement

## Project goal

Find out whether the hour of day a TikTok video is posted is related to how well it performs, and if so, which hours do best.

## Main finding

In this dataset, **no posting time block reliably produces higher engagement**. Median engagement rate was similar across 4-hour blocks (about 0.049 to 0.072) and the differences were not statistically significant (Kruskal-Wallis p = 0.145). The best-performing block also changed from year to year.

## Data

- **File:** `tiktok_data.csv` (loaded from `../BIT3484/data/`)
- **Unit of analysis:** one row per TikTok video
- **Columns used:**

| Column | Meaning |
|---|---|
| `id_video` | Video ID |
| `duration_seconds` | Length of the video |
| `human_time` | Timestamp the video was posted |
| `author_followers` | Creator's follower count |
| `author_likes` | Creator's total likes |
| `views` | Views on the video |
| `likes` | Likes on the video |

- **Columns dropped:** `text_part`, `hashtags`, `id_sound` (not needed for a posting-time analysis).

## Cleaning steps

1. Dropped the three unused columns above.
2. Dropped rows with missing values (`dropna`, result saved back into `tiktok_df`).
3. Converted `human_time` to a real datetime and extracted `hour`, `weekday` and `year`.
4. **Kept only videos posted 2022-2025** (the dataset also contains 2015-2021, but there were very few videos in the early years and TikTok behavior changed a lot over time). Result: 518 videos.
5. **Removed videos that share an exact timestamp with another video** (`duplicated(keep=False)`). Large groups of videos had the identical second, for example 158 videos at 2025-05-28 22:55:03 and 24 at 2022-11-10 08:37:34. Different videos can't all be posted in the same second, so these are almost certainly a collection or placeholder artifact, not real posting times. This removed 254 rows.
6. Final analysis set: **264 videos** (2022: 56, 2023: 68, 2024: 89, 2025: 51).

## Metrics

- `engagement_rate = likes / views` (share of viewers who liked the video). This is the main outcome.
- `views_per_follower = views / author_followers` (reach relative to the creator's audience size). Created but not used in the final results.
- Medians are used instead of means because views and engagement are heavily right-skewed.
- `hour_block`: hours grouped into six 4-hour blocks (0-3, 4-7, 8-11, 12-15, 16-19, 20-23) because single hours had too few videos (often 0 to 5).

## Methods

1. Pearson and Spearman correlation between `hour` and `engagement_rate`.
2. Median engagement rate by hour, by year, and by 4-hour block (line charts).
3. Kruskal-Wallis test (a rank-based test that doesn't assume normal data) to check whether the blocks differ more than chance would explain.
4. Multiple linear regression (OLS with robust HC3 standard errors) of `log(engagement_rate)` on hour block, weekday, year, `log(author_followers)` and video duration, so the posting-time effect is measured after adjusting for creator size, video length and calendar effects.
5. A nested F-test comparing the model with and without the hour blocks, to get one p-value for the whole hour effect.

## Results

**Engagement rate by 4-hour block (all years pooled, 2022-2025)**

| Hour block | Videos | Median engagement rate |
|---|---|---|
| 0-3 | 54 | 0.0669 |
| 4-7 | 44 | 0.0660 |
| 8-11 | 28 | 0.0712 |
| 12-15 | 38 | 0.0679 |
| 16-19 | 46 | 0.0488 |
| 20-23 | 54 | 0.0724 |

- Kruskal-Wallis: H = 8.21, **p = 0.145** (not significant at 0.05).
- Five of six blocks have almost identical medians. 16-19 is the lowest, but the test does not show that difference is real.
- **Peak block by year:** 2022 = 12-15, 2023 = 20-23, 2024 = 8-11, 2025 = 4-7. There is no consistent best block.
- **Correlation of hour with engagement rate** (2022-2025, computed before removing duplicate timestamps): Pearson = -0.229, Spearman = -0.130. Both are weak and negative, and the hour explains very little of the variation. *These were not rerun after the duplicate-timestamp cleaning, so rerun them and update this line.*

**Regression (264 videos, outcome = log engagement rate, reference block = 0-3)**

| Hour block | Coefficient | p-value | Approx. change vs 0-3 |
|---|---|---|---|
| 4-7 | -0.025 | 0.904 | -2% |
| 8-11 | +0.185 | 0.431 | +20% |
| 12-15 | +0.081 | 0.726 | +8% |
| 16-19 | -0.403 | 0.073 | -33% |
| 20-23 | -0.103 | 0.635 | -10% |

- No hour block is statistically significant. The 95% confidence interval for every block includes 0 (for 16-19: about 57% lower to 4% higher).
- **Nested F-test for the hour blocks as a group: F = 1.58, p = 0.165.** Adding posting hour does not significantly improve the model.
- The whole model explains almost nothing: R-squared = 0.057, adjusted R-squared = -0.004, overall F-test p = 0.527.
- Creator size (`log_followers`, p = 0.908), video duration (p = 0.291), weekday and year were also not significant.

## Graphs

<img width="1281" height="701" alt="engagement_by_block_year" src="https://github.com/user-attachments/assets/dd42f39d-bc50-48f8-829f-bb3f107a79e4" />

<img width="1165" height="701" alt="engagement_by_block" src="https://github.com/user-attachments/assets/d486a832-8aba-44e7-9dea-990f8dd7222d" />

<img width="2083" height="584" alt="hourly_counts_before_after" src="https://github.com/user-attachments/assets/bf4b94a3-c53d-4ab9-ba07-55c73723fb83" />

<img width="1063" height="701" alt="regression_hour_effects" src="https://github.com/user-attachments/assets/e6a84de8-352e-40f5-b8e9-11ffe753d2a5" />

## Conclusion

After removing suspect timestamps and limiting the data to 2022-2025 (264 videos), posting hour showed no statistically reliable relationship with engagement rate. Median engagement was nearly the same in five of the six 4-hour blocks, the one lower block (16-19) was not significantly different, and the best block changed every year. Based on this data, **there is no evidence for a single best time to post**. Creators should not expect posting time alone to meaningfully change engagement, and content and audience likely matter more.

A regression that adjusted for creator size, video length, weekday and year gave the same answer: the hour blocks were not jointly significant (nested F-test p = 0.165), and the only block that stood out, 16-19 (about 33% lower engagement than 0-3, p = 0.073), was not significant either.

This does not prove posting time never matters. It means this dataset is too small and noisy to detect a small effect.

## Limitations

- **Small sample:** 28 to 54 videos per block, and often 0 to 5 per single hour, so one unusual video can move a result.
- **Time zone unknown:** if `human_time` is UTC, the hour is not the viewers' local hour, which could hide a real effect. Confirm the data source's time zone.
- **Data collection artifacts:** many videos shared identical timestamps. Removing them (254 of 518 rows, about half) shrinks the sample and could remove some real videos.
- **Not a random sample of TikTok:** the dataset appears to be ordered by view count and skews toward popular videos, so results may not apply to an average account.
- **Engagement rate is only one definition of success.** Reach (views per follower) may behave differently.
- **Multiple comparisons:** looking at many blocks and years makes it easy to find a pattern by chance. The 16-19 dip was noticed after seeing the data, so treat it as a hint only.
- **Correlation is not causation,** and a straight-line correlation can miss curved patterns.

## Next steps

- Repeat the block test using `views_per_follower` as the outcome.
- Test by weekday and by weekday x time block.
- Confirm the time zone and convert to a consistent local time if possible.
- Collect more videos, especially from average accounts, and keep timestamps unmodified.

## How to run

1. Install the packages: `pip install -r requirements.txt`
2. Put `tiktok_data.csv` in the data folder, or change the `DATA` path in the first notebook cell.
3. Run the notebook cells **in this order** (later cells depend on earlier ones):
   1. Imports and data load
   2. Drop unused columns and `dropna` (assign the result back to `tiktok_df`)
   3. Timestamp handling (`hour`, `weekday`, `year`) and the 2022-2025 filter
   4. Remove duplicate timestamps
   5. Create `engagement_rate` and `views_per_follower`
   6. Create `hour_block`
   7. Correlations, charts and the Kruskal-Wallis test

## Tools

Python, pandas, NumPy, matplotlib, SciPy.
