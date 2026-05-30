This project explores the relationship between early-game advantages and match outcomes in professional League of Legends (LoL) matches across several major professional leagues. In particular, the analysis focuses on how securing early objectives and building resource advantages within the first 10 minutes relate to a team’s probability of winning.

Author: Sungsan Rhie

## Introduction

League of Legends is a competitive strategy game in which early-game objectives often influence the pace and direction of a match. Early-game objectives such as the first tower and first dragon can provide teams with map pressure, resource advantages, and strategic control that may influence the outcome of a match.

For this project, I use professional match data from Oracle's Elixir, focusing on matches from several major professional leagues during the 2026 season, including the LCK, LPL, LEC, LCS, LCP, and CBLOL. Note that LPL matches do not include per-minute early-game data in this dataset, so they are removed during cleaning and do not appear in the final analysis. The LPL is still discussed in the Missingness section, where this data-collection gap is itself the object of study.

This project investigates how strongly early-game objectives and resource advantages relate to match outcomes across major professional League of Legends leagues.

The dataset contains detailed gameplay statistics and match information from professional League of Legends matches played during the 2026 season across several major professional leagues.

The cleaned dataset contains 1,920 rows (960 matches, each represented by two team-level rows for the blue and red sides) and 10 columns. Below are the columns used throughout this project:

- `gameid`: A unique identifier for each match. This column is useful for distinguishing individual games and validating team-level rows.

- `side`: Indicates whether a team played on the blue side or red side during the match.

- `result`: Indicates the outcome of the match for a team. A value of `1` represents a win, while `0` represents a loss.

- `firsttower`: A binary column indicating whether a team secured the first tower of the game. A value of `1` means the team secured the first tower, while `0` means the opposing team secured it.

- `firstdragon`: A binary column indicating whether a team secured the first dragon objective. A value of `1` means the team secured the first dragon, while `0` means they did not.

- `golddiffat10`: The gold difference between a team and its opponent at 10 minutes. Positive values indicate an early gold advantage, while negative values indicate a deficit.

- `xpdiffat10`: The experience difference between a team and its opponent at 10 minutes. Positive values indicate an experience advantage.

- `csdiffat10`: The creep score (CS) difference between a team and its opponent at 10 minutes. Positive values indicate stronger early-game farming and lane pressure.

- `killsat10`: The total number of kills a team has at 10 minutes. This column is used as a feature for the prediction problem in later sections.

- `split`: The named portion of a competitive season (for example, "Spring" or "Summer"). This column is used in the missingness assessment section.

## Data Cleaning and Exploratory Data Analysis

The original dataset contains detailed match statistics from professional League of Legends matches across multiple regions and competitive levels. Since this project focuses on early-game objectives and match outcomes, only columns related to early-game performance and match results were selected for analysis.

To focus on high-level professional play, the dataset was filtered to include matches from several major professional leagues during the 2026 season, including the LCK, LPL, LEC, LCS, LCP, and CBLOL. Columns unrelated to the analysis, such as match URL information, were removed to simplify the dataset.

The cleaned dataset focuses on a smaller set of variables related to early-game objective control and resource advantages, including first tower, first dragon, gold difference, experience difference, and CS difference at 10 minutes.

In the original data, each match is saved as twelve separate rows. Ten of them are player rows (one for each of the ten players), and two of them are team rows (one summary row for each side). This project looks at team results, so I only need the two team rows from each match. The columns `firsttower` and `firstdragon` only have values on team rows, and they are empty on player rows. So when I drop rows with missing values in these columns, only the team rows are left. This is why the number of rows drops from 16,140 to 1,920. It is not a real loss of data. It just removes the player rows I do not need.

Some rows were also dropped because the early-game columns (`golddiffat10`, `xpdiffat10`, `csdiffat10`) were missing. These are missing for matches where Oracle's Elixir did not record per-minute data. In this dataset, those are the LPL matches. This is why the LPL is not in the cleaned data. I dropped these rows instead of filling them. There is no good way to guess a team's exact gold or experience lead at 10 minutes after the game. Filling them would create early-game numbers that never really happened.

After cleaning, the dataset has 1,920 rows (960 matches, each with two team rows for the blue and red sides). Last, I changed the binary columns `firsttower`, `firstdragon`, and `result` into integer types to make analysis and plotting easier.

The head of the cleaned dataset is shown below:

| gameid           | side   |   result |   firsttower |   firstdragon |   golddiffat10 |   xpdiffat10 |   csdiffat10 | split   |   killsat10 |
|:-----------------|:-------|---------:|-------------:|--------------:|---------------:|-------------:|-------------:|:--------|------------:|
| LOLTMNT03_337058 | Blue   |        1 |            1 |             1 |           1979 |         1580 |           47 | Cup     |           3 |
| LOLTMNT03_337058 | Red    |        0 |            0 |             0 |          -1979 |        -1580 |          -47 | Cup     |           1 |
| LOLTMNT03_337069 | Blue   |        0 |            0 |             0 |            -73 |          438 |           25 | Cup     |           0 |
| LOLTMNT03_337069 | Red    |        1 |            1 |             1 |             73 |         -438 |          -25 | Cup     |           1 |
| LOLTMNT03_337081 | Blue   |        1 |            1 |             1 |           2301 |          547 |           21 | Cup     |           3 |

*The table above shows the head of the cleaned dataset. Each row is one team in one match, with its early-game objective results (`firsttower`, `firstdragon`), its resource differences at 10 minutes (`golddiffat10`, `xpdiffat10`, `csdiffat10`), and its match result (1 = win, 0 = loss). The binary columns have been converted to integer type.*

### Univariate Analysis

To better understand the distribution of early-game advantages in professional League of Legends matches, I analyzed the distribution of gold differences at 10 minutes.

In League of Legends, gold is one of the most important in-game resources because teams use gold to purchase items that increase combat strength and overall map control. As a result, gold differences between teams often reflect differences in early-game performance and strategic advantages.

<iframe src="assets/univariate_golddiff.html" width="800" height="600" frameborder="0"></iframe>

The distribution of gold differences at 10 minutes is centered near zero, though some matches show substantial early-game leads or deficits. This suggests that many professional matches remain relatively close during the early game, with teams often minimizing mistakes and responding quickly to small advantages before they snowball into larger gold leads.

I also looked at the distribution of team kills at 10 minutes. Unlike gold or experience differences, which are symmetric around zero by construction (one team's lead equals the other's deficit), team kills at 10 minutes is a count that reflects each team's own aggression in the early game.

<iframe src="assets/univariate_kills.html" width="800" height="600" frameborder="0"></iframe>

Most teams in major professional leagues record only a small number of kills by the 10-minute mark, reflecting professional players' preference for safe laning over high-risk aggression in the early game. Higher kill counts are less common, but when they appear they signal an unusually explosive early game.

### Bivariate Analysis

To investigate the relationship between early-game advantages and match outcomes, I analyzed how gold differences at 10 minutes differ between winning and losing teams across major professional League of Legends leagues.

I used a box plot to compare the distributions of early-game gold differences between winning and losing teams.

<iframe src="assets/bivariate_gold_by_result.html" width="800" height="600" frameborder="0"></iframe>

Winning teams generally tend to have larger positive gold differences at 10 minutes compared to losing teams. While some matches remain relatively close during the early game, the overall distribution suggests that teams with stronger early-game gold advantages are more likely to convert those leads into victories in professional League of Legends matches.

I also examined the joint relationship between gold and experience differences at 10 minutes.

<iframe src="assets/bivariate_gold_xp_scatter.html" width="800" height="600" frameborder="0"></iframe>

Gold and experience differences at 10 minutes appear strongly positively correlated. Teams that gain a gold lead also tend to gain an experience lead — this reflects the laning phase, where farming minions provides both gold and experience together. Color-coding by match result shows that wins concentrate in the upper-right region (positive gold and XP) while losses concentrate in the lower-left.

This plot also suggests that gold and XP differences carry overlapping information, which is worth keeping in mind when designing features for the prediction model later in the project.

### Interesting Aggregates

The table below compares average early-game statistics between teams that secured the first tower and teams that did not.

| First Tower         |   Avg Gold Diff |   Avg XP Diff |   Avg CS Diff |   First Dragon Rate |   Win Rate |
|:--------------------|----------------:|--------------:|--------------:|--------------------:|-----------:|
| Lost First Tower    |         -599.88 |      -403.18  |       -10.06  |               0.405 |      0.321 |
| Secured First Tower |          599.88 |       403.18  |        10.06  |               0.595 |      0.679 |

Teams that secured the first tower generally had higher average win rates, gold differences, experience differences, and CS differences at 10 minutes. They also secured the first dragon more frequently on average. This suggests that securing the first tower is closely associated with broader early-game advantages across major professional League of Legends leagues.

## Assessment of Missingness

### NMAR Analysis

I believe the column `split` is **Not Missing At Random (NMAR)**. The `split` column shows the part of a season a match belongs to, like "Spring" or "Summer". In the full Oracle's Elixir dataset, before filtering to the six major leagues, this value is missing for many rows.

I think the reason is simple. Regional leagues are divided into splits, so their matches have a split value. But international tournaments and exhibition matches are not part of any split. For these matches, there is no split to record at all. The value is missing because that kind of event has no split in the first place.

This is NMAR because the missingness depends on the value itself, not on another column I can see. The dataset has no column that tells me the event type, so I cannot explain the missingness from the data alone. I have to reason about it using domain knowledge.

To make this MAR, I would want a column like `event_type` with values such as "Regular Split", "International Event", or "Exhibition". With that column, the missingness would be fully explained by an observed variable, and the mechanism would become MAR.

### Missingness Dependency

I analyze the missingness of `goldat25` (each team's total gold at the 25-minute mark). Initial inspection showed that LPL matches have `goldat25` missing in nearly 100% of rows, while other major leagues have missingness rates below 3%. Since this large gap reflects a difference in Oracle's Elixir data collection pipeline for the LPL rather than a feature of the games themselves, including LPL would let a single data-source effect dominate the analysis and mask other patterns. To focus on missingness that arises from match characteristics, I restrict this section to non-LPL major-league matches, where `goldat25` is missing in 1.88% of rows.

I perform two permutation tests at significance level **0.05**:
- **Test 1**: Does the missingness of `goldat25` depend on `gamelength`?
- **Test 2**: Does the missingness of `goldat25` depend on `playoffs`?

#### Test 1: `goldat25` missingness vs `gamelength`

**Null Hypothesis**: The mean of `gamelength` is the same for rows where `goldat25` is missing and rows where it is not.

**Alternative Hypothesis**: The means are different.

**Test statistic**: Absolute difference in mean `gamelength` between the two groups (used because `gamelength` is numerical).

<iframe src="assets/missingness_gamelength.html" width="800" height="600" frameborder="0"></iframe>

The observed mean difference of about 576 seconds is far larger than any value produced under the null (the largest simulated difference was around 73 seconds). The p-value of 0.0000 is much smaller than 0.05, so we **reject the null hypothesis**. The missingness of `goldat25` does depend on `gamelength`. This is consistent with the data-generating process: games that end before 25 minutes cannot have a recorded value at the 25-minute mark.

#### Test 2: `goldat25` missingness vs `playoffs`

**Null Hypothesis**: The distribution of `playoffs` is the same for rows where `goldat25` is missing and rows where it is not.

**Alternative Hypothesis**: The distributions are different.

**Test statistic**: Total Variation Distance (TVD), since `playoffs` is categorical (0/1).

<iframe src="assets/missingness_playoffs.html" width="800" height="600" frameborder="0"></iframe>

The observed TVD of about 0.021 falls comfortably inside the range produced under the null hypothesis. The p-value of approximately 0.32 is much larger than 0.05, so we **fail to reject the null hypothesis**. This suggests that the missingness of `goldat25` does not appear to depend on whether a match is a playoff game. The small observed difference in missing rates between regular-season and playoff matches (about 0.5 percentage points) is consistent with random variation.

## Hypothesis Testing

My hypothesis testing is: if a team already has the first tower, does also getting the first dragon raise its win rate even more?

I did not test "first dragon vs win rate" directly, because first dragon is such a strong sign of an early lead that it looks important in almost any group. I think the more interesting question is whether a second objective still adds value once a team is already ahead. First tower is one of the most valuable early objectives, so a team that already has it is in a strong spot. The question is whether the first dragon adds a lot on top of that, or only a little. This test measures that extra value, instead of the simple link between first dragon and winning.

This connects to my main question, which is about how early-game objectives relate to match results.

**Null Hypothesis (H0):** For teams that got the first tower, getting the first dragon does not change their win rate.

**Alternative Hypothesis (H1):** For teams that got the first tower, those that also get the first dragon win more often than those that do not.

**Test statistic:** The difference in win rate between first-tower teams with `firstdragon = 1` and first-tower teams with `firstdragon = 0`.

**Significance level:** 0.05.

I use a one-sided permutation test because my alternative has a direction (I expect an extra objective to help, not hurt). Under the null, I can shuffle the `firstdragon` labels within the first-tower group, since the label should not matter if the null is true. A permutation test fits here because it does not assume any specific distribution.

<iframe src="assets/hypothesis_null_distribution.html" width="800" height="600" frameborder="0"></iframe>

Among the 960 teams that already had the first tower, the win rate was 76.53% for teams that also got the first dragon, and 55.27% for teams that did not. That is a gap of about 21 percentage points. Across 5,000 shuffles under the null hypothesis, no simulated gap was as large as the real one. This gives a one-sided p-value of about 0.0000.

Since the p-value is far below 0.05, I reject the null hypothesis.

This means the first dragon still adds a lot to a team's win rate, even when the team already has the first tower. The two objectives are not the same thing. A team that gets both wins much more often than a team that gets only the first tower. This supports my main idea that each early-game objective adds real value to winning.

This result shows a link, not a cause. Teams that get both objectives may also be stronger in other ways that this dataset does not record, such as draft or jungle path.

## Framing a Prediction Problem

The prediction problem explored in this project is whether early-game performance metrics can predict the outcome of a professional League of Legends match.

This is a binary classification problem because the response variable, `result`, takes one of two possible values:
- 1 = win
- 0 = loss

The response variable was chosen because match outcome is the most important competitive indicator in professional League of Legends. Earlier sections of this project showed that early-game advantages such as gold and experience differences are associated with winning, making it reasonable to investigate whether these variables can also be used predictively.

To avoid data leakage, only information available within the first 10 minutes of a game was used for prediction. This ensures that the model only uses information that would realistically be available at the “time of prediction.”

The features considered for prediction include:
- `golddiffat10`
- `xpdiffat10`
- `csdiffat10`
- `killsat10`
- `firstdragon`
- `firsttower`

Since the dataset contains relatively balanced win and loss outcomes, accuracy was chosen as the primary evaluation metric because it is straightforward and interpretable for binary classification tasks.

## Baseline Model

For the baseline model, I used a Logistic Regression classifier to predict match outcomes.

The model used the following features:
- Quantitative (2):
  - `golddiffat10`
  - `xpdiffat10`
- Ordinal (0): none.
- Nominal (1):
  - `firstdragon`

I intentionally limited the baseline to a small set of three features (two quantitative resource advantages plus one binary objective). This keeps the baseline simple and reserves additional features such as `firsttower`, `csdiffat10`, and `killsat10` for the final model, where I can measure how much they improve performance beyond this minimal starting point.

Since `firstdragon` is categorical (binary), it was encoded using `OneHotEncoder` (with `drop='if_binary'`) within a sklearn Pipeline. Numerical features were passed through without transformation.

The model was evaluated using accuracy on a held-out test set in order to measure how well the model generalizes to unseen matches.

Logistic Regression was selected because it is interpretable and effective for binary classification problems involving structured numerical data.

The baseline model is evaluated two ways:

* Test-set accuracy on the held-out 20% split.
* 5-fold cross-validation accuracy on the training set.

I report both because the held-out test set has only about 384 rows, so a single-split accuracy can swing just from which rows happen to land in the test set. The cross-validation average over five folds is a more stable estimate of how the model generalizes, and it is the metric I use to compare the baseline against the final model.

This baseline is a reasonable starting point for predicting match outcomes from a small set of early-game features.

## Final Model

For the final model, I used a Gradient Boosting Classifier. I chose it because the relationship between early-game statistics and match outcomes is likely nonlinear and involves interactions between variables. Gradient Boosting builds trees one after another, where each new tree tries to fix the errors of the trees before it. This lets it capture patterns that the linear baseline model cannot.

To improve on the baseline model, I engineered two new features on top of the raw early-game features.

### gold_xp_product

This feature multiplies the gold difference and the experience difference at 10 minutes (then divides by 1000 to keep the scale small). It captures *simultaneous* gold and XP dominance. A team that leads in both gold and XP is in a much stronger position than a team that leads in only one. The linear baseline cannot express this kind of interaction between two features, but this product term makes it explicit.

### composite_resource_z

This feature combines the gold, experience, and CS differences at 10 minutes into one number. I first z-score each of the three columns (using the mean and standard deviation from the **training data only**, to avoid data leakage), then take their average. Since all three columns reflect laning and resource control, this single index gives a cleaner summary of a team's overall early-game economic advantage.

To optimize the model, I used GridSearchCV with 5-fold cross-validation to tune three hyperparameters:

- `n_estimators` (number of trees)
- `max_depth` (depth of each tree)
- `learning_rate` (how much each tree contributes)

The goal of tuning was to balance predictive performance against overfitting. The best parameters found were `n_estimators = 100`, `max_depth = 2`, and `learning_rate = 0.1`.

**Model comparison.** I compare the baseline and final models using **5-fold cross-validation accuracy**, computed on the same training rows, rather than the single held-out test split:

| Metric | Baseline | Final |
|---|---|---|
| 5-fold CV accuracy | 67.64% (± 1.81%) | **69.47% (± 1.22%)** |
| Single test-set accuracy | 69.79% | 67.19% |

On the cross-validation metric, the final model improves on the baseline by about 1.8 percentage points (67.64% → 69.47%) and is also more stable (its fold-to-fold standard deviation drops from 1.81% to 1.22%).

I rely on cross-validation rather than the single test split for this comparison because the held-out test set has only about 384 rows, which makes a single accuracy figure sensitive to which rows happen to fall in the split. In this case the two metrics disagree: the baseline's test-set accuracy (69.79%) is above its cross-validation average (67.64%), which means this particular split happened to favor the baseline, while the final model's test-set accuracy (67.19%) is below its cross-validation average (69.47%). Averaging over five folds removes this single-split luck and gives a more trustworthy estimate of how each model generalizes.

The model got better for a few reasons. First, I added more raw features (`csdiffat10`, `killsat10`, `firsttower`), which give the model more early-game information. Second, I added two new features: `gold_xp_product`, which shows when a team leads in both gold and XP at the same time, and `composite_resource_z`, which combines gold, XP, and CS differences into one score. A straight-line model cannot use these the way a tree model can. Third, Gradient Boosting can find patterns and feature interactions that a linear model cannot.

## Fairness Analysis

To evaluate fairness, I check whether the final model predicts match outcomes equally well for **blue-side** teams and **red-side** teams.

- **Group X**: blue-side teams (`side == 'Blue'`)
- **Group Y**: red-side teams (`side == 'Red'`)
- **Evaluation metric**: classification accuracy within each group.

**Null Hypothesis (H0)**: The model is fair. Its accuracy is the same for blue-side and red-side teams, and any observed difference is due to chance.

**Alternative Hypothesis (H1)**: The model is unfair. Its accuracy differs between blue-side and red-side teams.

**Test statistic**: the absolute difference in accuracy between the two groups, `|accuracy(Blue) - accuracy(Red)|`.

**Significance level**: 0.05.

I use a **two-sided** permutation test because I have no prior reason to expect the model to favor a particular side; the question is simply whether *any* accuracy gap exists. Under the null hypothesis, the side labels can be reshuffled across the test rows without changing the model's behavior, so a permutation test is appropriate.

The model's accuracy was about 63.16% for blue-side teams and 71.13% for red-side teams, an observed gap of about 7.98 percentage points, with the model being less accurate for blue-side teams. The two-sided permutation test produced a p-value of approximately 0.107.

Since the p-value of 0.107 is greater than the 0.05 significance level, I fail to reject the null hypothesis. There is no statistically significant evidence at the 0.05 level that the model is unfair with respect to side.

That said, the result is suggestive rather than fully reassuring. An 8-percentage-point accuracy gap is fairly large, and a p-value of about 0.107 means a gap this size would arise by chance roughly 11% of the time — not rare enough to call significant, but not clearly attributable to noise either. A larger test set would help determine whether the gap is real. This is worth flagging because the model never uses side as a feature, so any genuine gap would reflect side-correlated differences in the early-game features themselves rather than the model explicitly treating the two sides differently.
