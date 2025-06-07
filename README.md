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
- Does adding a “healthy” label to a vegetarian recipe affect how people rate its tastiness?

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

<iframe
  src="assets/rating_dist.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis

### Interesting Aggregates


## Assessment of Missingness

### NMAR Analysis

### Missingness Dependency


## Hypothesis Testing


## Framing a Prediction Problem


## Baseline Model


## Final Model


## Fairness Analysis
