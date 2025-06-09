# A Rating Comparison of Vegetarian Recipes: Does Recipe Healthiness Affect Perceived Taste?

This project was completed as part of **DSC 80: The Practice and Application of Data Science** at UCSD during Spring 2025.

Author: Saya Egashira

## Introduction

### Research Question and Background

Food is a vital part of our lives—not just for nutrition but also for enjoyment. With growing interest in healthy eating and plant-based diets, terms like “vegetarian” and “healthy” have become increasingly common in recipe labels. But how do these labels influence how people perceive the taste of a dish?

A growing body of research suggests that health-related labels can negatively impact perceived tastiness. For example, consumers may assume that food labeled as “healthy” is less indulgent or satisfying, even if its actual flavor is unchanged. In addition, some studies suggest that meat-eaters may view vegetarian food as less healthy and less tasty, while vegetarians often believe their diet is both healthy and flavorful.

This leads us to several questions:

- How do the “vegetarian” and “healthy” labels interact with each other in recipes?
- Are vegetarian recipes actually less healthy?
- Are vegetarian recipes perceived as less tasty?
- Does adding a “healthy” label to a vegetarian recipe affect how well people rate?

In this project, I focus particularly on this final question. I analyze whether vegetarian recipes that are also tagged as “healthy” receive different user ratings than vegetarian recipes without the “healthy” tag. I use user-submitted ratings as a proxy for perceived tastiness, and aim to understand whether health-related labels bias perception on a popular online recipe platform.

### Dataset Description

To investigate our research question, we analyze datasets of recipes and ratings posted since 2008 on [food.com](https://www.food.com/). The two datasets we use were originally scraped and used for the paper on recommender systems [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Bodhisattwa Prasad Majumder et al.

The first dataset `RAW_recipes.csv` contains the information of 83,782 recipes with the following 12 features:

| Column | Description |
| ------ | ----------- |
| `'name'` | Recipe name|
|`'id'`|Recipe ID|
|`'minutes'`|Minutes to prepare recipe|
|`'contributor_id'`|User ID who submitted this recipe|
|`'submitted'`|Date recipe was submitted|
|`'tags'`|Food.com tags for recipe|
|`'nutrition'`|Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”|
|`'n_steps'`|Number of steps in recipe|
|`'steps'`|Text for recipe steps, in order|
|`'description'`|User-provided description|

The second dataset `RAW_interactions.csv` contians 731,927 reviews and ratings submitted for the recipes in the `RAW_recipes.csv` dataset.

| Column | Description |
| ------ | ----------- |
| `'user_id'` | User ID|
|`'recipe_id'`|Recipe ID|
|`'date'`|Date of interaction|
|`'rating'`|Rating given|
|`'review'`|Review text|

Given the datasets, we aim to investigate the ratings of vegetarian recipes with the merged dataset, especially focusing on the `'tags'` and `'nutrituion'` columns from `RAW_recipes.csv` and the `'rating'` column from `RAW_interactions.csv`.


## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

Here are the data cleaning steps I took to for the effective analysis:

1. **Merge Two Datesets into One:** I start by merging the two datasets and create a column containing the average rating per recipe. This is done by 1) left merging the recipes and interactions datasets together, 2) replacing 0 ratings with `np.nan`, 3) finding average rating per recipe, and then 4) merging the average rating Series back to the recipes dataset.

2. **Distribute the `'nutrition'` Information:** Even though the `'nutrition'` column seems to contain lists, they are actually strings that look like lists. According to the data dictionary, each value in the `'nutrition'` column contains information in the form `"[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV)]"`. To split up values, I completed the following steps: 1) apply `eval(x)` to each value in the `'nutrition'` column, 2) create a DataFrame from the resulting Series of lists with column names `'calories'`, `'total_fat_PDV'`, `'sugar_PDV'`, `'sodium_PDV'`, `'protein_PDV'`, `'sat_fat_PDV'`, and `'carbs_PDV'`, and then 3) concatenate with the merged dataset from step 1. Some of these columns would be used in the later analysis.

3. **Create Boolean Columns `'is_vegetarian'` and `'is_healthy'`:** Since our focus of analysis is on vegetarian and healthy recipes, I create two Beelean columns: `'is_vegetarian'` and `'is_healthy'`, which contain `True` for vegetarian and healthy recipes, respectively, and `False` for the others. The True/False values are determined by checking if the `'tags'` value of each recipe contain "vegetarian" and "healthy".

4. **Create a Catagorical Column `'tag_combo'`:** Next, I create a new column `'tag_combo'` that shows if the recipes are "vegetarian only", "healthy only", "both", or "neither" based on the `'is_vegetarian'` and `'is_healthy'` columns generated in the previous step. This column would be particularly helpful in the later analysis where we want to differenciate "vegetarian only" recipes and "both vegetarian and healthy" recipes.

<!-- 5. **Drop Unnecessary Columns (Optional):** Finally, in order to avoid carrying the huge entire dataset throughout the analysis, I may drop columns that are irrelevant for this analysis: `'name'`, `'id'`, `'contributor_id'`, `'submitted'`, `'tags'`, `'nutrition'`, `'description'`, `'steps'`, and `'ingredients'`. These are either the identification variables or descriptive strings, neither of which would be useful for our analysis. All the information useful and contained in the descriptive text columns have already been extracted into the new columns by step 4. -->


The cleaned DataFrame consists of 83782 recipes (rows) with 23 features (columns). Below is the first 5 rows of my cleaned DataFrame. Since the number of columns is too many and some columns contain long string objects, I chose to display only a subset of columns from the full cleaned DataFrame based on the relevance to our questions. Scroll right to view more columns.

| name                                 |   minutes |   n_steps |   n_ingredients |   rating |   calories |   total_fat_PDV |   sugar_PDV |   sodium_PDV |   protein_PDV |   sat_fat_PDV |   carbs_PDV | is_vegetarian   | is_healthy   | tag_combo   |
|:-------------------------------------|----------:|----------:|----------------:|---------:|-----------:|----------------:|------------:|-------------:|--------------:|--------------:|------------:|:----------------|:-------------|:------------|
| 1 brownies in the world    best ever |        40 |        10 |               9 |        4 |      138.4 |              10 |          50 |            3 |             3 |            19 |           6 | False           | False        | neither     |
| 1 in canada chocolate chip cookies   |        45 |        12 |              11 |        5 |      595.1 |              46 |         211 |           22 |            13 |            51 |          26 | False           | False        | neither     |
| 412 broccoli casserole               |        40 |         6 |               9 |        5 |      194.8 |              20 |           6 |           32 |            22 |            36 |           3 | False           | False        | neither     |
| millionaire pound cake               |       120 |         7 |               7 |        5 |      878.3 |              63 |         326 |           13 |            20 |           123 |          39 | False           | False        | neither     |
| 2000 meatloaf                        |        90 |        17 |              13 |        5 |      267   |              30 |          12 |           12 |            29 |            48 |           2 | False           | False        | neither     |

### Univariate Analysis

For the univariate analysis, I examined the distribution of `'rating'` and `'tag_combo'`.

The following histogram of `'rating'` shows that ratings are heavily skewed toward 5, implying that users predominantly give high ratings to most of recipes on [food.com](https://www.food.com/). Only a few recipes receive ratings below 3, suggesting users' potential bias or reluctance to rate poorly.

<iframe
  src="assets/rating_dist.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The distribution of `'tag_combo'` is displayed in the next bar plot. Most recipes are tagged as "neither" vegetarian nor healthy. "Vegetarian only" recipes are fewer than "healthy only" recipes, but the discrepancy does not seem to be significant (9.3K vs. 12.5K) compared to the number of "neither" recipes (58.4K). Recipes that are "both" vegetarian and healthy is the rarest category.

<iframe
  src="assets/tag_combo_dist.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis

I conducted a few bivariate analysis and chose to include two results that are the most relevant to the original research question. 

The first one is the following box plot comparing ratings across different tag combination (vegetarian only, healthy only, both, neither). This visualization reveals that recipes tagged as "both" vegetarian and healthy had slightly lower first quartile value of rating compared to "vegetarian only" recipes. "Healthy only" recipes also has the lower first quartile value than "vegetarian only" and "neither" recipes. This suggests that recipes tagged as "healthy" are likely to get lower ratings while "vegetarian" tag may not have a same effect on lowering ratings as the "healthy" tag.

<iframe
  src="assets/tag_rating.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The next figure aims to show the difference in the rating distribution of recipes having the vegetarian tags and healthy tags, respectively. While there is no significant difference observed from the following bar charts, we can still tell that healthy-tagged recipes have a slightly lower chance of obtaining the highest rating.

<iframe
  src="assets/rating_dist_by_tag.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

I grouped the DataFrame by `'tag_combo'` column to obtain the mean rating for tag-combination each group; vegetaran, healthy, both, or neither. The results corresponds to the observation from the bivariate analysis plots above, healthy recipes seems to be likely to gain lower ratingss than recipes without tagged as healthy. "Vegetarian only" recipes attained slightly higher mean rating than recipes that are neither vegetarian nor healthy, but we cannot tell from the table if this difference is statistically significant.

| tag_combo       |   mean_rating |
|:----------------|--------------:|
| vegetarian only |       4.63544 |
| neither         |       4.62967 |
| healthy only    |       4.60457 |
| both            |       4.60167 |

The following table was also created by grouping the DataFrame by `'tag_combo'`, but I applied the aggregate function `mean()` to all the values obtained from the original `'nutrition'` column. Interestingly, mean `'sugar_PDV'` and mean `'sodium_PDV'` were the highest for "healthy only" recipes, which contradicts to the intuition that sugarly or salty foods are not good for health.

| tag_combo       |   calories |   total_fat_PDV |   sugar_PDV |   sodium_PDV |   protein_PDV |   sat_fat_PDV |   carbs_PDV |
|:----------------|-----------:|----------------:|------------:|-------------:|--------------:|--------------:|------------:|
| vegetarian only |    349.345 |         27.1218 |     57.484  |      21.5624 |       18.1318 |      30.5562  |     12.8915 |
| neither         |    466.131 |         38.9174 |     66.6202 |      30.2154 |       37.5433 |      49.4096  |     12.9166 |
| healthy only    |    358.779 |         13.1402 |     86.1338 |      31.8353 |       28.5104 |      13.3847  |     17.8852 |
| both            |    293.988 |         11.911  |     69.9569 |      16.8774 |       15.8794 |       9.05052 |     16.0244 |


## Assessment of Missingness

### NMAR Analysis

The cleaned DataFrame has only three columns that contain missing values; 1 missing value in `'name'`, 70 missing values in `'description'`, and 2609 missing values in `'rating'`. The missingness in `'name'` would likely MCAR (missing completely at random) because there is only a single missing value. The missing value might have come from a data error entry error considering the extreme rarity (1 out of 83782), which suggests an isolated data processing error rather than a systematic pattern. However, the missingness of `'name'` can be considered as NMAR (not missing at random) if [food.com](https://www.food.com) has a system that deletes inappropriate recipe names or it allows users to save a recipe without the recipe name if they do not intend to publish. The missingness in `'description'` column would be MAR (missing at random) because the choice of writing description would likely depend on the contributor, which is included in our dataset as the `'contributor_id'` variable. The missingness of `'rating'` would also be MAR (missing at random). People would be less likely to use recipe with large number of `'minutes'` or `'n_steps'`, which indirectly reduces the chance of such kinds of recipe to receive reviews.

### Missingness Dependency

I investigated whether the missingness in the `'rating'` column depends on the column `'is_vegetarian'` and `'is_healthy'`, respectively.

First, I performed the missingness permutation test to check if missingness in `'rating'` depends on `'is_vegetarian'`.

**Null Hypothesis:** The missingness of `'rating'` does not depend on whether the recipe is tagged "vegetarian" or not.

**Alternative Hypothesis:** The missingness of `'rating'` depends on whether the recipe is tagged "vegetarian" or not.

**Significance Level:** 0.05

<iframe
  src="assets/missing_perm_rating_is_vegetarian.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The observed p-value was 0.1096. Since it is greater than hte significance level 0.05, we keep the null hypothesis and claim that there is no evidence that missingness of `'rating'` depends on `'is_vegetarian`'.

Next, I performed the missingness permutation test to check if missingness in `'rating'` depends on `'is_healthy'`.

- **Null Hypothesis:** The missingness of `'rating'` does not depend on whether the recipe is tagged "healthy" or not.
- **Alternative Hypothesis:** The missingness of `'rating'` depends on whether the recipe is tagged "healthy" or not.
- **Significance Level:** $\alpha=0.05$

<iframe
  src="assets/missing_perm_rating_is_healthy.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The observed p-value was 0.0124. Since it is less than hte significance level 0.05, we reject the null hypothesis and claim that missingness of `'rating'` depends on `'is_healthy`'.


## Hypothesis Testing

As mentioned in the introduction, I am curious about whether the distributions of vegetarian recipe ratings differ depending on whether they are tagged as "healthy" or not. I formulated the following hypotheses.

- **Null Hypothesis:** There is no difference in average rating between "vegetarian only" recipes and recipes "both" vegetarian and healthy.
- **Alternative Hypothesis:** "vegetarian only" recipes have a higher average rating than recipes "both" vegetarian and healthy.
- **Significance Level:** $\alpha=0.05$

<iframe
  src="assets/perm_rating_vegetarian only_both.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

I used a permutation test with the difference in average ratings as the test statistic, specifically, I used (average rating of "vegetarian only" recipes) $-$ (average rating of recipes that are "both" vegetarian and healthy), based on the observation from the exploratory analysis. The resulting p-value was 0.0020, which is below the significance level $\alpha=0.05$. Thus, I reject the null hypothesis, providing evidence that "vegetarian-only" recipes tend to have higher ratings than those tagged as "both" vegetarian and healthy.


## Framing a Prediction Problem

My model will try to predict the rating that a person would post when a recipe features are given. I treat this as a multiclass classification problem where the outcome should be one of [1,2,3,4,5], which corresponds to the users' rating scale. For this purpose, the target variable `'rating'` will be converted from floats to integer categories by rounding before working on the prediction modeling process. The rows with missing rating values should be dropped when modeling. I will use the macro-averaged F1-score, instead of accuracy, as the primary evaluation metric. This addresses critical limitations of accuracy in our imbalanced dataset, where most of ratings are 4~5. While accuracy would misleadingly reward a model that only predicts the majority class, the F1-score's balance of precision and recall will assure all rating categories are weighted equally. This is particularly important because misclassifications have asymmetric impacts (e.g., predicting 5 stars for a 1-star recipe is far worse than confusing 4 and 5 stars). This approach will allow the model to perform meaningfully across the entire rating spectrum rather than exploiting class imbalances.


## Baseline Model

For the baseline model, I tried to predict `'rating'` using a Random Forest classifier based on three features: `'is_vegetarian'`, `'is_healthy'`, and `'calories'`. The numerical feature `'calories'` were scaled using a `StandardScaler`. Transformations on `'is_vegetarian'` and `'is_healthy'` would not be required since they are already stored as Boolean values (1 for True, 0 for False). The model employed class weighting (`class_weight='balanced'`) to partially address the severe rating imbalance in the dataset, where 5-star ratings comprise about 70% of observations. This approach would automatically adjust weights so the model pays proportionate attention to minority classes. The baseline model achieved a macro-averaged F1-score of 0.18 ([0.01,0.02,0.05,0.26,0.55] for classes [1,2,3,4,5], respectively), indicating a poor performance across all rating categories. The model failed almost completely on 1-3 star ratings, demonstrating that this baseline model performs especially poorly for distinguishing lower ratings. To improve the model, I plan to try different `class_weight` for a better balance between classes and incorporate additional features from the dataset and engineer new features.


## Final Model




## Fairness Analysis
