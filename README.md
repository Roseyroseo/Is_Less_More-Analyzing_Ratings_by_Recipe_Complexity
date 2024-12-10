# Is Less More? Analyzing Ratings by Recipe Complexity
 
by Rosey Gutierrez (rgutierrezmeza@ucsd.edu)

This is a project for the DSC80 course at the University of California San Diego

---

## Introduction

 In our fast-paced world people have come to appreciate convenience and appreciate time savings anywhere possible, so are recipe ratings perhaps biased towards quick-and-easy solutions to feeding ourselves and our families? Or do ratings provide a good measure of simply how enjoyable a meal is in spite of the painstaking labor some of the tastiest dishes require? This analysis aims to focus on the relationship between the complexity of recipes and how well scored they are and asses how much weight this particular aspect holds in considering the overall rating of a recipe, and how well rated a simple recipe could be rated in comparison to a complex one.  
 
 The dataset used is comprised of two separate .csv files scraped from [food.com](https://www.food.com/) by the authors of [this paper.](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) One contained recipes, and the other reviews for said recipes. 

---

## Data Cleaning and Exploratory Analysis 

The data cleaning process begins by filling all `0` ratings with NaN since a rating of 0 implies that the recipe has yet to be rated, the lowest rating one can give is 1, thus it makes sense to treat 0 as a missing value as oppossed to a low rating. Then, the average rating per recipe was calculated and added to a column named `avg_rating`. 

We have added a `taglist` column by processing the `tags` column to represent actual lists instead of one long string of tags, both columns were kept in order to access tags as needed. All date columns were transformed to datetime objects, and `date` was renamed to `review_date` for clarity. Since the initial merge resulted in two `id` and `recipe_id` columns for the same recipe identifier, and the other column containing an identifier pertains to contributors and is aptly named `contributor_id`; we will drop `id` and change its type from float to int to save some memory.

By looking at some of the preliminary data distributions we can observe a particularly large outlier in the `minutes` column, the maximum in the data at 1051200 minutes. Using the recipe id of this particular data point and looking it up on the website it looks like it's a joke recipe, on ["How to Preserve a Husband"](https://www.food.com/recipe/how-to-preserve-a-husband-447963), which adds no real value and could massively skew our findings; therefore, since it has nothing to do with actual recipes it can be safely discarded from the dataset.

While there are still recipes remaining with values as high as 288000 minutes, causing a significant right skew in the data, these are legitimate recipes that are used for liqueur and other long-term preserved comestibles. Taking this into account, we will focus the analysis on recipes that can be made within 72 hours; this will allow for something like marinating overnight, slow-cooking & smoking to be well within the time-frame required for making a meal; and recipes for things like pickles and liquor are ultimately ingredients as opposed to meals. These ingredients require large amounts of time and are out of the scope of what we are trying to predict, given that preservation and fermentation waiting time does not necessarily contribute to complexity the way being on top of cooking time in order to reach the perfect temperature / doneness might be. This approach will only drop 445 rows from our data out of over 230k. While there might still exist recipes that aren't necessarily meals, since they are not extreme time values, they do not need to be dropped.

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

While the difference is not exactly large, there is a trend showing that recipes requiring less overall effort, do get higher ratings on average, with recipes that take a day or more outperforming those that might take over thirty minutes. This could be due to the time in those recipes involving just waiting as oppossed to active prep and labor, thus requiring less effort despite being time-consuming.  

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
The KS statistic measures the largest vertical gap between the two curves, which likely occurs around 1-6 ingredients, where the density of the missing group (in green) is visibly higher than the non-missing group (in blue). This difference could indicate that simpler recipes (with fewer ingredients) require less explanation, leading users to skip providing a description, and recipes with more ingredients might be perceived as more complex, prompting users to include descriptions for clarity and additional context and direction which is further validated by performing the same test on the `is_easy` column which shows a near identical density distribution and a KS Test statistic of 0.1374 with a slightly higher p-value of .0243.

---

## Hypothesis Testing

So far, `n_steps` has been the more varied of the column categories when trying to find a relationship with `ratings`, making it difficult to tell for sure if there is one at all. We'll try to perform a hypothesis test to see if we can uncover a more definitive metric that shows a relationship, or the absence of one. 

### Null Hypothesis

Under the null hypothesis $H_0$ : The number of steps has no influence on ratings. This implies that any observed difference in average ratings is due to random chance. We'll simulate this by shuffling the rating column to break the relationship between `n_steps` and `ratings`. The alternative is that number of steps does have an influence on ratings.

### Test Statistic:
The test statistic is the difference in means:

$$T = \bar{R}_{\text{simple}} - \bar{R}_{\text{complex}}\$$

Where:

- $\bar{R}_{\text{simple}}$: Mean rating of recipes classified as "simple" (based on fewer steps).
- $\bar{R}_{\text{complex}}$: Mean rating of recipes classified as "complex" (based on more steps).

We'll group recipes by their number of steps by computing the median for the number of steps across recipes to create a somewhat even threshold between simple and complex recipes that's robust to outliers based on number of steps, and then we'll calculate the observed difference in means to see how often the null test statistics are as extreme of more extreme than the observed statistic.

<iframe src="assets/n_stepsRatings_hypothesisTest.html" width=800 height=600 frameBorder=0></iframe>

At a significance level of 0.01 we'd fail to reject the null, but at a 0.05 significance level we are able to reject the null and say that there is some relationship between `n_steps` and `rating`. In this case, we can see that the observed difference is not just due to random chance, and the number of steps does influence rating though it might not be the strongest predictor.  

---

## Framing a Prediction Problem

---

## Baseline Model

---

## Final Model

---

## Fairness Analysis