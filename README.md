# Does Recipe Healthiness Affect Perceived Taste? A Rating Comparison of Vegetarian Recipes

This project was completed as part of **DSC 80: The Practice and Application of Data Science** at UCSD during Spring 2025.

Author: Saya Egashira

## Introduction

### Research Question and Background

### Dataset Description

We analyze datasets of recipes and ratings posted since 2008 on [food.com](https://www.food.com/). The two datasets we use were originally scraped and used for the paper on recommender systems [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Bodhisattwa Prasad Majumder et al.

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

### Univariate Analysis

<iframe
  src="assets/calorie_dist.html"
  width="800"
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
