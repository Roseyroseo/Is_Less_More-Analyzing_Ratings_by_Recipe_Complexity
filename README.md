# Is Less More? Analyzing Ratings by Recipe Complexity
 
by Rosey Gutierrez (rgutierrezmeza@ucsd.edu)

This is a project for the DSC80 course at the University of California San Diego

---

## Introduction

 In our fast-paced world people have come to appreciate convenience and appreciate time savings anywhere possible, so are recipe ratings perhaps biased towards quick-and-easy solutions to feeding ourselves and our families? Or do ratings provide a good measure of simply how enjoyable a meal is in spite of the painstaking labor some of the tastiest dishes require? This analysis aims to focus on the relationship between the complexity of recipes and how well scored they are and asses how much weight this particular aspect holds in considering the overall rating of a recipe, and how well rated a simple recipe could be rated in comparison to a complex one.  
 
 The dataset used is comprised of two separate .csv files scraped from [food.com](https://www.food.com/) by the authors of [this paper.](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) One contained recipes, and the other reviews for said recipes. 

---

## Data Cleaning and Exploratory Analysis 

The data cleaning process begins by filling all `0` ratings with NaN since a rating of 0 implies that the recipe has yet to be rated, the lowest rating one can give is 1, thus it makes sense to treat 0 as a missing value as oppossed to a low rating. Then, the average rating per recipe was calculated and added to a column named `avg_rating`. As far as missing values go, they can be mainly found in the `description`, `rating`, and `avg_rating` columns; which is reasonable since we've changed all ratings of 0 to missing to better represent unrated recipes, and not all recipes have a written description on the site. There's a single missing value for each `user_id`, `name`, and `review_date`, they will be left as is for the time being to preserve their other column values, but will drop them as necessary should the analysis require it.

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


Since the tags might yield some important categorical data for our analysis, we have identified some we might find useful like `easy` which we can use to identify simple recipes from the get-go, and others that give us descriptive ways of identifying simplicity. The plot showcases the popularity of simpler, less time-consuming recipes at a glance. Given the prevalence of the tag, we will give it its own boolean column. 

<iframe src="assets/correlationheatmap.html" width=800 height=600 frameBorder=0></iframe>

---

## Assessment of Missingness

---

## Hypothesis Testing

---

## Framing a Prediction Problem

---

## Baseline Model

---

## Final Model

---

## Fairness Analysis