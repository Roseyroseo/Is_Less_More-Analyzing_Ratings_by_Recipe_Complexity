# Is Less More? Analyzing Ratings by Recipe Complexity
 
by Rosey Gutierrez (rgutierrezmeza@ucsd.edu)

This is a project for the DSC80 course at the University of California San Diego

---

## Introduction

 In our fast-paced world people have come to appreciate convenience and appreciate time savings anywhere possible, so are recipe ratings perhaps biased towards quick-and-easy solutions to feeding ourselves and our families? Or do ratings provide a good measure of simply how enjoyable a meal is despite the painstaking labor some of the tastiest dishes require? This analysis aims to focus on the relationship between the complexity of recipes, how well scored they are, assess how much weight these properties hold in considering the overall rating of a recipe, and how well rated a simple recipe could be rated in comparison to a complex one.  
 
 The dataset used is comprised of two separate `.csv` files scraped from [food.com](https://www.food.com/) by the authors of [this paper.](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) 

 - `RAW_recipes.csv` contains recipes.
 - `RAW_interactions.csv` contains reviews and ratings submitted for the recipes in `RAW_recipes.csv`. 

 A description of each column in both datasets is given below.

RECIPES

|Column	          |Description|
|----------------:|----------:|
|'name'	          |Recipe name|
|'id'	          |Recipe ID|
|'minutes'	      |Minutes to prepare recipe|
|'contributor_id' |User ID who submitted this recipe|
|'submitted'	  |Date recipe was submitted|
|'tags'	          |Food.com tags for recipe|
|'nutrition'	  |Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”|
|'n_steps'	      |Number of steps in recipe|
|'steps'	      |Text for recipe steps, in order|
|'description'	  |User-provided description|

RATINGS

|Column	          |         Description|
|----------------:|-------------------:|
|'user_id'	      |             User ID|
|'recipe_id'      |	          Recipe ID|
|'date'	          | Date of interaction|
|'rating'         |	       Rating given|
|'review'         |         Review text|

---

## Data Cleaning and Exploratory Analysis 

Our process begins by merging both `.csv` files into a single dataset. We'll then proceed to fill all `0` recipe ratings with NaN since a rating of 0 implies that the recipe has yet to be rated, the lowest rating one can give is 1, thus it makes sense to treat 0 as a missing value as opposed to a low rating. Additionally, the average rating per recipe was calculated and added to a column named `avg_rating`. 

We have added a `taglist` column by processing the `tags` column to represent actual lists instead of one long string of tags, both columns were kept in order to access tags as needed. All date columns were transformed to `pd.datetime` objects, and `date` was renamed to `review_date` for clarity. Since the initial merge resulted in two `id` and `recipe_id` columns for the same recipe identifier, and the other column containing an identifier pertains to contributors and is aptly named `contributor_id`; we will drop `id` and change its type from float to int to save some memory.

By looking at some of the preliminary data distributions we can observe a particularly large outlier in the `minutes` column, the maximum in the data at 1051200 minutes. Using the recipe id of this particular data point and looking it up on the website it looks like it's a joke recipe, on ["How to Preserve a Husband"](https://www.food.com/recipe/how-to-preserve-a-husband-447963), which adds no real value and could massively skew our findings; therefore, since it has nothing to do with actual recipes it can be safely discarded from the dataset.

While there are still recipes remaining with values as high as 288000 minutes, causing a significant right skew in the data, these are legitimate recipes that are used for liqueur and other long-term preserved comestibles. Taking this into account, we will focus the analysis on recipes that can be made within 72 hours; this will allow for something like marinating overnight, slow-cooking & smoking to be well within the time-frame required for making a meal. Recipes for things like pickles and liquor are ultimately ingredients as opposed to meals, and these ingredients require large amounts of time and are out of the scope of what we are trying to predict, given that preservation and fermentation waiting time does not necessarily contribute to complexity the way being on top of cooking time in order to reach the perfect temperature / doneness might be. This approach will only drop 445 rows from our data out of over 230k. While there might still exist recipes that aren't necessarily meals, since they are not extreme time values, they do not need to be dropped.

This is an abbreviated preview of our `recipe_reviews` dataframe with only the recipe's id, minutes, number of steps, number of ingredients, rating and average rating. Both tag columns are omitted for the sake of presentation.  

|   recipe_id |   minutes |   n_steps |   n_ingredients |   rating |   avg_rating |
|------------:|----------:|----------:|----------------:|---------:|-------------:|
|      333281 |        40 |        10 |               9 |        4 |            4 |
|      453467 |        45 |        12 |              11 |        5 |            5 |
|      306168 |        40 |         6 |               9 |        5 |            5 |
|      306168 |        40 |         6 |               9 |        5 |            5 |
|      306168 |        40 |         6 |               9 |        5 |            5 |


<iframe src="assets/tagcountsplot.html" width=800 height=600 frameBorder=0></iframe>


Since the tags might yield some important categorical data for our analysis, we have identified some we might find useful which we can use to identify simple recipes from the get-go, and others that give us descriptive ways of identifying complexity. The plot showcases the popularity of simpler, less time-consuming recipes at a glance. Given the prevalence of the `easy` tag, we will give it its own column `is_easy`. 

Next, we will try to find relationships among the numerical variables to get a better sense of what we can combine to find meaningful combinations that help measure the complexity of a recipe. 

<iframe src="assets/correlationheatmap.html" width=800 height=600 frameBorder=0></iframe>

Immediately some obvious trends are visible, recipes with more steps tend to take longer to prepare, and the more ingredients a recipe has, the more steps need to be taken to prepare it. We can also observe negative correlation between recipes marked as being easy and the number of steps involved in preparing them, which makes sense since simpler recipes would take less work. There aren't any strong correlations with `avg_rating` other than a slight negative one with `minutes` which indicates that recipes that take less time have a relationship with higher average ratings. 

Let's use the tags we isolated previously and see if they give any numerical insights about average rating, preparation time, and number of steps in a pivot table:

| tag                   |   average_rating |   average_minutes |   average_steps |
|:----------------------|-----------------:|------------------:|----------------:|
| 15-minutes-or-less    |          4.71858 |           9.17745 |         5.52557 |
| 5-ingredients-or-less |          4.70803 |          49.7866  |         6.60927 |
| 3-steps-or-less       |          4.69791 |          52.6216  |         5.76725 |
| easy                  |          4.68376 |          61.3552  |         8.15145 |
| 30-minutes-or-less    |          4.68151 |          25.0457  |         9.25098 |
| 1-day-or-more         |          4.68113 |        1691.7     |        14.9182  |
| 4-hours-or-less       |          4.67671 |         102.551   |        13.1848  |
| 60-minutes-or-less    |          4.6681  |          45.6667  |        11.5558  |

While the difference is not exactly large, there is a trend showing that recipes requiring less overall effort do get higher ratings on average. With recipes that take a day or more outperforming those that might take over thirty minutes. This could be due to the time in those recipes involving just waiting as opposed to active prep and labor, thus requiring less effort despite being time-consuming.  

---

## Assessment of Missingness

As far as missing values in our data go, they can be mainly found in the `description`, `rating`, and `avg_rating` columns; which is reasonable since we've changed all ratings of 0 to missing to better represent unrated recipes, and not all recipes have a written description on the site. There's a single missing value for each `user_id`, `name`, and `review_date`, but these are trivial.

Given that the missing values in `rating` are a result of unrated recipes, the missingness is directly tied to the fact that these recipes lack any user-provided rating, and this column has a missingness mechanism of NMAR since the missing values are specifically tied to the values themselves (the lack of user ratings). Since `avg_rating` is a column directly calculated from `rating` its missingness mechanism is MAR since the missing values in it depend on the `rating` column. Lastly, the missingness in the `description` column is also NMAR for the same reason `rating` is, the values are specifically tied to a lack of provided user description, and their missing status is specifically tied to the values themselves. We can try to do some permutation tests in order to check dependency along other columns using the **difference in means** for missingness.

Permutation test results for `description` column: 

| Column                | Observed Statistic |           P-value |
|:----------------------|-------------------:|------------------:|
|recipe_id              |         -5472.2216 |             0.3860|
|contributor_id         |     -11471701.0962 |             0.6610|
|minutes                |            -4.9060 |             0.7420|
|n_steps                |             0.7304 |             0.2210|
|n_ingredients          |            -1.1347 |             0.0010|
|is_easy                |             0.1374 |             0.0060|

At a significance level of 0.01 we can say that the missingness of `description` does not depend on `recipe_id `, `contributor_id`, `minutes` or `n_steps`. These columns likely don't really influence the reason why users chose not to provide a description. On the other hand, `n_ingredients`, and `is_easy` show dependency with the missingness of `description`. This might suggest that recipes with fewer or more ingredients are more or less likely to have descriptions, and that recipes with the `easy` tag might influence whether or not users feel compelled to provide descriptions, whether they might feel it's unnecessary due to its simplicity, or they want to emphasize it. We'll do a ks test statistic to get more insights into `n_ingredients`

<iframe src="assets/desc_ing_ksplot" width=800 height=600 frameBorder=0></iframe>
KS Test Statistic: 0.1500, P-Value: 0.0105

The KS statistic of 0.1500 indicates that the largest difference between the cumulative distribution functions (CDFs) of the two groups is 15% of the data range.

The plot suggests that recipes with more ingredients are more likely to include a description.
The KS statistic measures the largest vertical gap between the two curves, which likely occurs around 1-6 ingredients, where the density of the missing group (in green) is visibly higher than the non-missing group (in blue). This difference could indicate that simpler recipes (with fewer ingredients) require less explanation, leading users to skip providing a description, and recipes with more ingredients might be perceived as more complex, prompting users to include descriptions for clarity, additional context and direction which is further validated by performing the same test on the `is_easy` column which shows a near identical density distribution and a KS Test statistic of 0.1374 with a slightly higher p-value of .0243.

---

## Hypothesis Testing

So far, `n_steps` has been the more varied of the column categories when trying to find a relationship with `ratings`, making it difficult to tell for sure if there is one at all. We'll try to perform a hypothesis test to see if we can uncover a more definitive metric that shows a relationship, or the absence of one. Additionally, `n_ingredients` had some correlation to `n_steps` so we will also check the relationship between ingredients and `rating`.

Under the null hypothesis the number of steps and ingredients has no influence on ratings. This implies that any observed difference in average ratings is due to random chance. We'll simulate this by shuffling the rating column to break the relationship between `n_steps` and `ratings`; then do the same with `n_ingredients` and `ratings` The alternative is that number of steps / ingredients does have an influence on ratings.

- **Null Hypothesis (H₀)**: The number of steps / ingredients has no influence on ratings, any observed difference in ratings is due to random chance.
- **Alternative Hypothesis (H₁)**: There is a relationship between the number of steps / ingredients in a recipe and the rating.

- **Test Statistic**: The test statistic is the difference in means *T* = *SimpleMean* - *ComplexMean*

Where:

- SimpleMean: Mean rating of recipes classified as "simple" (based on fewer steps / ingredients).
- ComplexMean: Mean rating of recipes classified as "complex" (based on more steps/ ingredients).

We'll group recipes by their number of steps by computing the median for the number of steps across recipes to create a somewhat even threshold between simple and complex recipes that's robust to outliers based on number of steps, and then we'll calculate the observed difference in means to see how often the null test statistics are as extreme of more extreme than the observed statistic.

<iframe src="assets/n_stepsRatings_hypothesisTest.html" width=800 height=600 frameBorder=0></iframe>

p-value: 0.039

At a significance level of 0.05 we are able to reject the null and acknowledge there is some relationship between `n_steps` and `rating`. In this case, we can see that the observed difference is not just due to random chance, and the number of steps does influence rating to an extent.

A hypothesis test for `n_ingredients` was performed in the same way, but with a resulting p-value of 0.134 we fail to reject the null, and we cannot say that number of ingredients has significant influence on rating. 

---

## Framing a Prediction Problem

Thus far, we've found that the number of steps, and time it takes to complete a recipe seem to play some part in higher average ratings. We want to try and predict the average rating of a recipe based on features that are known at the time the recipe is submitted. This is a regression problem since the response variable being used is `avg_rating`, since it is continuous and ranges from 1 to 5 as opposed to `rating` which is discrete 1-5; using `avg_rating` as the response variable will allow us to get a more precise range of predictions. We chose avg_rating as the response variable because it directly reflects the quality of a recipe as perceived by users, making it a meaningful and interpretable target.

The primary features used to predict `avg_rating` are `n_steps` (number of steps in the recipe) and `minutes` (time required to complete the recipe). These features are known at the time of submission, and we've established a correlation to higher average ratings based on our exploratory analysis; they can be measured as-is without future knowledge. We exclude any features or feedback generated after the recipe is submitted (e.g., user ratings, reviews) to ensure the model can be used in real-time prediction scenarios.

To evaluate the model, we will use the Root Mean Squared Error (RMSE) metric since it penalizes larger errors more heavily, which is crucial for accurately predicting ratings that follow a narrow range (1 to 5). RMSE provides a more interpretable measure of prediction error in the original units of `avg_rating` compared to other metrics like Mean Absolute Error (MAE). While metrics like R² can provide insight into the proportion of variance explained, RMSE directly informs us about how far predictions deviate from the true ratings, aligning with our goals.

---

## Baseline Model

|RMSE   |          Sample Prediction   |               Actual Value|
|:------|-----------------------------:|--------------------------:|
|0.4909 |   [4.68 4.68 4.63 4.68 4.68] | [5.   4.8  4.9  4.9  4.62]|


The baseline model is a linear regression model trained to predict the average rating (`avg_rating`) of a recipe based on two quantitative features: `n_steps` (number of steps in the recipe) and `minutes` (time required to complete the recipe). The features used are both quantitative, meaning no ordinal or nominal variables were included in this baseline model; and since no categorical variables were used, no encodings (e.g., one-hot or ordinal encoding) were necessary. Both features were standardized using StandardScaler before training the model to ensure consistent scaling.

The model was evaluated using Root Mean Squared Error (RMSE), which penalizes large prediction errors more heavily. The model achieved an RMSE of 0.4909, meaning that, on average, the model's predictions deviated from the actual ratings by approximately 0.49 on a scale of 1 to 5. Sample predictions (e.g., 4.68, 4.63) demonstrate that the model tends to predict values near the upper range of the target variable, with less variation compared to the actual values.

While the RMSE suggests moderate accuracy, the model is limited in its predictive ability because it's overly simplistic. It only uses two quantitative features and assumes a linear relationship between these features and `avg_rating`. Furthermore, the small variation in predictions suggests the model may not fully capture the underlying complexity of recipe ratings, which is to be expected as it does not take into account one of the most important (yet abstract and difficult to capture) factors in rating a recipe: subjective taste. 

---

## Final Model

We will add the `is_easy` column to our list of parameters and we will create two new features: 

- `log_minutes` : Log transformed `minutes` to reduce the effect of extreme outliers in prep time. 
- `steps_per_minute` : Combines `n_steps` and `minutes` to capture overall recipe simplicity and efficiency.

These additional features might offer some insights and could better capture the time and effort it takes to prepare a meal, which could factor into a better rating. 

Given these additional features, the final model we chose was a Random Forest Regressor, as it achieved the lowest Root Mean Squared Error (RMSE) of 0.4762, outperforming both a baseline Linear Regression model (RMSE: 0.4903) and the Lasso Regression model (RMSE: 0.4906). Random Forest was chosen due to its ability to capture complex, non-linear relationships between the features and the `average_rating`. The hyperparameters for the Random Forest model were tuned using `GridSearchCV` with 5-fold cross-validation to ensure the model generalizes well to unseen data. 

The best hyperparameters identified were:

- `max_depth`: 20, which balances capturing complexity while avoiding overfitting.
- `min_samples_split`: 10, ensuring splits occur only when sufficient data exists.
- `n_estimators`: 300, providing a robust ensemble of trees.

Our Final Model represents an improvement over the Baseline Model in terms of RMSE, which decreased from 0.4903 to 0.4762. This improvement demonstrates the value of incorporating additional engineered features (`log_minutes` and `steps_per_minute`), hyperparameter tuning, and using a non-linear model like Random Forest. While the improvement in RMSE is very modest, it shows how much of an impact an algorithm that can capture more complex patterns in the data can have.

---

## Fairness Analysis

We'll perform a fairness analysis of the **Random Forest Regression** model to determine whether its performance (measured by RMSE) differs across two groups. For this fairness analysis, we'll use the `is_easy` column to compare:

- **Group X**: Recipes marked as "easy" (`is_easy` = 1).
- **Group Y**: Recipes not marked as "easy" (`is_easy` = 0).

- **Null Hypothesis (H₀)**: The model's RMSE for "easy" recipes is roughly the same as for "non-easy" recipes. Any observed difference is due to random chance.
- **Alternative Hypothesis (H₁)**: The model's RMSE for "easy" recipes differs from its RMSE for "non-easy" recipes.
- **Test Statistic**: The absolute difference in RMSE between the two groups.

<iframe src="assets/NullDistGroupRMSE.html" width=800 height=600 frameBorder=0></iframe>

Observed RMSE Difference: 0.0099
P-Value: 0.2340

The absolute difference in RMSE between the two groups (is_easy=1 and is_easy=0) is 0.0099; this is a very small difference, indicating that the model performs similarly on 'easy' and 'non-easy' recipes in terms of RMSE. The p-value is greater than any statistically significant threshold, so we fail to reject the null hypothesis and can deduce that the model's performance **does not differ significantly** between the two groups. Based on these results, we can conclude that the model is fair with respect to this grouping. 