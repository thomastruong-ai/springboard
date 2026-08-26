# SECOM data wrangling log

| step                            |   column_delta | note                              |   running_total |
|:--------------------------------|---------------:|:----------------------------------|----------------:|
| 0. raw                          |            590 | 41,951 missing cells              |             590 |
| 1. drop missing > 50%           |            -24 | sensors offline for most units    |             566 |
| 2. drop constant                |           -122 | zero variance on train            |             444 |
| 3. drop near-constant >= 99.5%  |              0 | too few varying units to estimate |             444 |
| 4. drop duplicates |r| > 0.9999 |             -8 | same physical measurement         |             436 |
| 5. median impute (train-fit)    |              0 | 0 missing values remain           |             436 |
| 6. add missing indicators       |             28 | preserves informative gaps        |             464 |
| 7. winsorize + outlier count    |              1 | no rows removed, tails bounded    |             465 |

Final modelling matrix: 1253 train + 314 test units x 465 columns.
Split boundary: 2008-10-02 20:54:00. Failure rate: 6.94% train / 5.41% test.
