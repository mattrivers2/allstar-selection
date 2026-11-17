# NBA All-Star Selection Classifier

Predicting which NBA players get selected as All-Stars from their season statistics, and comparing four classification models to see which one does it best.

Course project, University of Tampa (November 2026), with Joseph Pascarella.

## Overview

An All-Star is a player selected to play in the NBA's annual All-Star Game. This project builds and compares four classifiers on player statistics from 1980 to 2024:

- Logistic Regression
- Random Forest
- AdaBoost
- XGBoost

Each model is tuned with randomized search and stratified 5-fold cross-validation, then evaluated on 10 replicated train/test splits. All-Stars are only about 6% of player-seasons, so accuracy alone is misleading (a model that never predicts an All-Star would score about 94%). Models are judged on precision, recall and F1 for the All-Star class.

### Questions

1. Which model best separates All-Stars from other players?
2. Which statistics matter most for All-Star selection?
3. Where do the models disagree with actual selections, and why?

## Data

[NBA Stats (1980-2024)](https://www.kaggle.com/) by Rodney Carroll on Kaggle (data from Basketball-Reference). Three files are used:

| File | Contents |
|---|---|
| `Advanced.csv` | Advanced statistics (PER, TS%, usage, win shares, BPM, VORP, etc.) |
| `Player Per Game.csv` | Per-game statistics (points, rebounds, assists, shooting, etc.) |
| `All-Star Selections.csv` | Which players were selected each season |

The data files are not included in this repository. Download them from Kaggle and place them in the project folder. ABA seasons are dropped so the analysis covers the NBA only.

## Method

1. Load and merge the three files, then run exploratory data analysis (structure, missing values, duplicates, class balance).
2. Split into train and test sets with a stratified split.
3. For each model, tune hyperparameters with `RandomizedSearchCV` (10 iterations, stratified 5-fold CV, scored on F1).
4. Evaluate the tuned model on the held-out test set.
5. Repeat the full process (new split, tuning, evaluation) across 10 random seeds and report the mean, standard deviation, min and max of each metric.
6. Compare models and inspect each model's most influential features.

## Results

Test-set performance on the All-Star class, averaged over 10 splits:

| Model | Accuracy | Precision | Recall | F1 (mean) | F1 (SD) |
|---|---|---|---|---|---|
| Logistic Regression | 0.970 | 0.799 | 0.658 | **0.721** | 0.031 |
| XGBoost | 0.969 | 0.778 | 0.656 | 0.711 | 0.027 |
| Random Forest | 0.969 | 0.790 | 0.635 | 0.703 | 0.023 |
| AdaBoost | 0.966 | 0.752 | 0.631 | 0.686 | 0.026 |

### Findings

- **Logistic regression had the highest average F1**, but the top three models are within about one standard deviation of each other, so the differences are small. The simplest model performed as well as the more complex ones.
- **A single split can mislead.** XGBoost had the best F1 on the first split (0.750), but its 10-split average was lower (0.711). Every model's single-split F1 was higher than its replicated average.
- **Volume and production stats mattered most.** VORP and points per game were among the top features for the random forest, AdaBoost and XGBoost. Five statistics appeared in the top 10 of all four models: field goals per game, free throws per game, free throw attempts per game, minutes per game and win shares per 48 minutes.
- **Models missed about a third of All-Stars.** Recall was the weakest metric (0.63 to 0.66) while precision was higher (0.75 to 0.80). Selection also depends on things the data doesn't contain, such as fan voting and injury replacements.

## Limitations

- **Timing:** All-Stars are chosen about halfway through the season, but the models use full-season statistics.
- **Era:** the data starts in 1980, so results may not apply to earlier eras of the game.
- **Correlated features:** many statistics move together, which makes individual coefficients and importances harder to interpret.
- **Small positive class:** the test set contains only 226 All-Stars, so a handful of players can shift a ranking.

## Future work

- Use first-half-of-season statistics to match when selections are actually made.
- Add outside factors such as fan voting, injuries and team success.
- Try player tracking or advanced shooting data.
- Increase the number of replicated splits beyond 10.
