# Exploring the Seasonal Trends in Recipe Ratings

## Introduction
Food is a fundamental aspect of human life, essential for survival. While some people eat food simply out of necessity, others savor the experience of trying new tastes and combining flavors to create something extraordinary. Regardless of what ones relationship with food is, exploring and analyzing different aspects of what makes recipes great is necessary. People often rely on ratings to choose foods and recipes, but the accuracy of these reviews can be questioned due to external factors affecting the reviewers' mood. One such factor is the season, as a gloomy, cold climate may evoke more negative emotions comparted to a warm, sunny climate. Thus, I posed the question of **how the time of the year a recipe is posted affects their given rating**. To investigate the relationship between seasons and ratings, I analyzed the two datasets consisting of recipe and ratings.

The first dataset, `recipes`, consists of 83782 rows where each row is a recipe, and 12 columns with the following information:

| Columns         | Description                                                                                                                                  |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | Recipe name                                                                                                                                  |
| `id`              | Recipe ID                                                                                                                                    |
| `minutes`         | Minutes to make recipe                                                                                                                       |
| `contributor_id`  | User ID who submitted recipe                                                                                                                 |
| `submitted`       | Date recipe was submitted                                                                                                                    |
| `tags`            | Tags for recipe from Food.com                                                                                                                |
| `nutrition`       | Information in the form of [number of calories, total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] where PDV is percentage of daily value |
| `n_steps`         | Number of steps                                                                                                                              |
| `steps`           | In order text for recipe steps                                                                                                               |
| `description`     | User-provided description                                                                                                                    |
| `ingredients`     | List of ingredients required for the recipe                                                                                                  |
| `n_ingredients`   | Number of ingredients required for the recipe                                                                                               |

The second dataset, `interactions`, consists of 731927 rows where each row is a review with a rating, and 5 columns with the following information:

| Columns     | Description          |
| ----------- | -------------------- |
| `user_id`   | User ID              |
| `recipe_id` | Recipe ID            |
| `date`      | Date rating given    |
| `rating`    | Rating given         |
| `review`    | Text of review       |

Since I am trying to determine whether there is a relationship between seasons and recipes ratings, I will focus on the columns `name`, `minutes`, `submitted`, and `nutrition` from the `recipes` dataset, and `date` and `rating` from the `interactions` dataset. For now, I will also need to include columns such as `id` from the `recipes` dataset and `recipe_id` from the `interactions` dataset so I will be able to combine the two datasets. However, before I can begin separating out the relevant columns, I need to clean the data. 

## Data Cleaning and Exploratory Data Analysis

### Data cleaning

To clean my data, I followed the pillars of data cleaning which are performing data quality checks, identifing and handling missing values, and performing transformations.

1. I left merged the `recipes` and  `interactions` dataset by the `id` and `recipe_id` columns since they both represent recipe's IDs. We do not merge from the user ID columns because the same person who posted the recipe is probably not the same person who posted the review. Also, we want to keep multiple recipes with the same name so we perform a left merge. 

2. I filled all ratings of 0 with `np.nan` because ratings scale from 1 to 5 so a 0 rating is not possible. Also, ratings containing 0 values instead of simply being missing could inaccurately lower the average rating of a recipe.

3. I found the average rating per recipe and added it to the dataframe as the `average_rating` column to better capture the recipe's real rating since recipes could contain multiple ratings. 

4. Next, I check the data types of the columns and converted columns that appeared like lists but were actually strings into actual lists. Some columns like `tags`, `steps`, and `ingredients` needed to be transformed into a list of strings, while a column like `nutrition` needed to be transformed into a list of floats.

5. I converted columns that contain dates, `submitted` and `date`, into datetime data types so I can use them as Timestamp objects. I renamed `submitted` to `date_submitted` and `date` to `date_interacted` to better show that `submitted` is the date the recipe was submitted and `date` is the date the review/rating was posted or date someone interacted with the recipe. 

6. Next, I stripped the whitespaces from the `name` column.

7. I expanded the `nutrition` column into six new columns called `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates` to make it easier to explore every unique value in `nutrition`.

8. I added two columns that would better represent the season each review and recipe was posted. First, I added a column called `season_submitted` representing the season the recipe was submitted. Then, I added a column called `season_interacted` representing the season the review was submitted. Since, seasons are different depending on where in the world someone is located, I standardized the seasons to assume a North American climate due to the fact that the website the dataset was based off of, Food.com, is American and this research project takes place in America. Thus, I labeled the seasons winter if the year of `date_submitted` or `date_interacted` was December through Feburary, spring if the year was March through May, summer if the year was June through August, and fall if the year was September through November.

9. I added a more general column called `season_category` to demonstrate the general climate of the different seasons of when the recipe was posted or based on `season_submitted`. Again, for the reasons explained in step 8, I used the North American climate and labeled seasons in summer and spring as warm while seasons in winter and fall as cold.

10. I added another column called `days_between` to show the how long it took between when a recipe was first posted and when the review was posted. I filtered out recipes where a review was posted before the recipe itself because the rating may not accurately represent the current version of the recipe. Such reviews could be based on earlier, potentially different, recipes or a mistaken entry and thereby should not be included. I also filtered out rows where no dates were present.

11. Then, I added two columns `dayofweek_recipe` to represent the day of the week a recipe was posted and `dayofweek_review` to represent the day of the week a review was posted.

12. Finally, I created a clean version of my dataframe by including the columns `name`, `minutes`, `date_submitted`, `date_interacted`, `rating`, `average_rating`, `num_calories`, `total_fat`, `sugar`, `season_submitted`, `season_category`, `season_interacted`, `days_between`, `dayofweek_recipe`, and `dayofweek_review` as these were the columns that would help me answer my research question. 

Below is the first five rows of the finalized dataframe, `df`:

| name                               |   minutes | date_submitted      | date_interacted     |   rating |   average_rating |   num_calories |   total_fat |   sugar | season_submitted   | season_category   | season_interacted   |   days_between | dayofweek_recipe   | dayofweek_review   |
|:-----------------------------------|----------:|:--------------------|:--------------------|---------:|-----------------:|---------------:|------------:|--------:|:-------------------|:------------------|:--------------------|---------------:|:-------------------|:-------------------|
| 1 brownies in the world best ever  |        40 | 2008-10-27 00:00:00 | 2008-11-19 00:00:00 |        4 |                4 |          138.4 |          10 |      50 | fall               | cold              | fall                |             23 | Mon                | Wed                |
| 1 in canada chocolate chip cookies |        45 | 2011-04-11 00:00:00 | 2012-01-26 00:00:00 |        5 |                5 |          595.1 |          46 |     211 | spring             | warm              | winter              |            290 | Mon                | Thurs              |
| 412 broccoli casserole             |        40 | 2008-05-30 00:00:00 | 2008-12-31 00:00:00 |        5 |                5 |          194.8 |          20 |       6 | spring             | warm              | winter              |            215 | Fri                | Wed                |
| 412 broccoli casserole             |        40 | 2008-05-30 00:00:00 | 2009-04-13 00:00:00 |        5 |                5 |          194.8 |          20 |       6 | spring             | warm              | spring              |            318 | Fri                | Mon                |
| 412 broccoli casserole             |        40 | 2008-05-30 00:00:00 | 2013-08-02 00:00:00 |        5 |                5 |          194.8 |          20 |       6 | spring             | warm              | summer              |           1890 | Fri                | Fri                |

### Univariate Analysis

I performed univariate analysis on the distribution of average ratings. 

<iframe
  src="assets/fig_rating-hist-plot.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

In this histogram, we can see that the graph is left skewed with a majority of the average ratings being between 4 and 5. This suggests that our data from food.com either includes only highly rated recipes or people on food.com, a majority of the time, tend to rate recipes higher than lower. 

### Bivariate Analysis

I performed bivariate analysis on the relationship between the average rating of when a recipe was posted and the average rating of when a review was posted.

<iframe
  src="assets/recipe_vs_rating-scatter-plot.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

In this scatter plot, we can see from looking at the x-axis that the season the review was posted is overall higher for spring and summer than winter and fall so it seems that when people post ratings in spring and summer, they tend to be more generous and rate higher than when people review in winter and fall. Also, from looking at the legend, we can see that generally the green and yellow dots which refer to spring and summer are higher than the other dots which refer to winter and fall, thus it appears that the seasons when recipes were posted in spring and summer recieve higher ratings than those posted in winter and fall. 

### Interesting Aggregates

I made a pivot table by grouping by the season the recipe and review was posted and then finding the mean of the average rating for each of these categories. It demonstrates how `season_submitted` or recipes that were posted in spring and summer seem to have higher average ratings than those posted in winter and fall. Also, it demonstrates how there is a similar trend for `season_interacted` as reviews that were posted during spring and summer seem to have higher average ratings than those posted in witner and fall.

| season_interacted \ season_submitted | fall | spring | summer | winter |
|--------------------------------------|------|--------|--------|--------|
| fall                                 | 4.66 | 4.68   | 4.68   | 4.65   |
| spring                               | 4.68 | 4.70   | 4.68   | 4.66   |
| summer                               | 4.67 | 4.71   | 4.70   | 4.67   |
| winter                               | 4.67 | 4.67   | 4.68   | 4.65   |

I also grouped by the season category and found the mean to compare the differences between warm and cold seasons on how long it takes to prepare the recipes, the average rating of recipes, the number of calories of recipes, the total fat of recipes, the sugar of recipes, and the days between when a recipe was posted and a review was posted. What stood out to me was that in all these categories except the average rating, recipes posted in warmer seasons had a lower mean than colder seasons. For average ratings, recipes posted in warmer seasons, similar to what has been shown in the graphs, have a higher average rating than colder seasons. 

| season_category   |   minutes |   average_rating |   num_calories |   total_fat |   sugar |   days_between |
|:------------------|----------:|-----------------:|---------------:|------------:|--------:|---------------:|
| cold              |  125.488  |          4.66172 |        426.51  |     32.3507 | 64.7016 |        730.202 |
| warm              |   89.6232 |          4.69012 |        413.128 |     31.5245 | 63.0547 |        679.04  |

## Assessment of Missingness

In the merged dataset there are only three columns with missing values: `rating`, `review`, and `description`. 

### NMAR Analysis

A column in the dataset that I think is Not Missing At Random (NMAR) is the `review` column. Since people are typically more likely to submit a review and describe how they are feeling when they feel strongly about something such as when they are disappointed with a recipe or when they are pleased with a recipe, the fact that the review is missing could depend on the actual value missing. Thus, if a person was neutral about a recipe they will feel less inclined to spend their time leaving a description of how the recipe made them feel. Additional data I could obtain to explain the missingness and make it Missing At Random (MAR) is the `rating` column to see if the values that are missing are associated with the rating they left. Also, the time of day the review was posted could make it MAR. A rating is more likely to be missing if the time of day was late at night or early in the morning since the reviewer could be tired than if it was in the middle of the day. 

### Missingness Dependency

I decided to test if the missingness of `rating` depends on the `season_submitted` column, which is the season the recipe was submitted. For this permutation test, I used a significance level of 0.05 and test statistic of total variation distance (TVD) because I needed to measure the distance between two categorical distributions.

**Null Hypothesis:** The distribution of the season when the recipe was submitted when rating is missing is the same as the distribution of the season when the recipe was submitted when the rating is not missing.

**Alternative Hypothesis:** The distribution of the season when the recipe was submitted when rating is missing is not the same as the distribution of the season when the recipe was submitted when the rating is not missing.

The graph below shows the distribution of when the rating is missing versus when the rating is not missing for the seasons a recipe was submitted.

<iframe
  src="assets/missingness-bar-plot.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

I ran the permuation test by shuffling the `season_submitted` column 500 times to generate different TVDs and compare these to my observed TVD of 0.039, which is shown by the red line in the graph below. Since the p-value of 0.0 is less than the alpha 0f 0.05, I reject the null hypothesis. The distribution of seasons when a recipe was posted when a rating is missing is not the same as the distribution when the rating is there. Thus, the permutation test does provide proof that the missingness of rating is dependent on the season the rating was submitting, making it MAR.  

<iframe
  src="assets/missingness-emp-dist.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

Next, I wanted to test if the missingness of `rating` depends on the `minutes` column, which is how long it takes to make a recipe. For this permutation test, I used a significance level of 0.05. To determine what test statistic to use, I created a graph to compare the null and non-null rating distribution on how long it took to make the recipe. 

<iframe
  src="assets/missingness-min-dist-1.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

In the initial graph above, it is difficult to determine the effect missing ratings have on the time it took to make a recipe due to the presense of outliers. Some data points indicate that it took over 6000 minutes or 250 days to create, which seems implausible. To address this, I removed these extreme values and created a new graph for better clarity.

<iframe
  src="assets/missingness-min-dist-2.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

In the new graph, the distribution of recipe times, whether the rating was missing or not, appear to be similar. Therefore, I decided to use absolute difference in means as my test statistic which is typically used if the two distributions have similar shapes.

**Null Hypothesis:** The distribution of minutes take to make recipe when rating is missing is the same as the distribution of minutes take to make recipe when the rating is not missing.

**Alternative Hypothesis:** The distribution of minutes take to make recipe when rating is missing is not the same as the distribution of minutes take to make recipe when the rating is not missing.

I ran the permuation test by shuffling the `rating` column 500 times to generate new absolute difference in mean values and compare these to my observed absolute difference in mean of 51.480, which is shown by the red line in the graph below. Since the p-value of 0.106 is greater than the alpha 0f 0.05, I fail to reject the null hypothesis. The distribution of minutes when a rating is missing is the same as when a rating is not missing. Thus, the permutation test does not provide proof that the missingness of rating is dependent on how long it takes to make a recipe, making it Missing Completely At Random (MCAR). 

<iframe
  src="assets/missingness-min-emp-dist.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

## Hypothesis Testing

I am curious on if the time of the year affects a recipe's average rating so I will be testing whether the average rating is greater for warmer seasons consisting of summer and spring than colder seasons consisting of winter and fall. This investigation is significant because it will allow me to understand if seasonal trends do have an affect on the ratings of recipes. I will be using the columns `season_category` and `average_rating` for my test.

**Null Hypothesis:** In the population, the average rating of recipes submitted in warmer seasons (summer and spring) and colder seasons (winter and fall) have the same distribution, and the observed differences in our samples are due to random chance.

**Alternative Hypothesis:** In the population, warmer seasons (summer and spring) have higher average ratings than colder seasons (winter and fall), and the observed difference in our samples cannot be explained by random chance alone.

**Test Statistic:** difference in group means (warm - cold)

**Significance Level:** 0.05

I decided to do a permuation test since I have two groups, the warmer seasons and colder seasons, and I am trying to determine if they look like they were drawn from the same population. I generated new data by shuffling the group labels of warm and cold and computing the test statistic of difference in group means for each shuffle. I choose difference in group means because it allows me to measure how different the two numerical distributions are by doing the mean average rating for warmer seasons minus the mean average rating for colder seasons.

<iframe
  src="assets/hyp_test-emp-dist.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

Since my p-value of 0.0, as shown by the red line in the distribution above, is less than my significance level of 0.05, I reject the null hypothesis that the average rating of recipes submitted in warmer seasons and colder seasons have the same distribution. Thus, I can say that the difference between the two groups of warmer and colder seasons is statisically significant and that the average ratings of recipes that were submitted in warmer seasons are higher than those that were submitted in colder seasons. I can infer that there is a correlation between the time of year when recipes were submitted and the ratings they receive. However, it is important to note that while there is an association between the season a recipe was submitted and its rating, this does not necessarily imply that submitting a recipe during a specific season will cause it to receive a higher rating.

## Framing a Prediction Problem

I plan on predicting the average rating for a recipe which would typically be a regression problem, however, since a majority of the ratings fall between 4 and 5, I separated the average predictions into categories. An average rating of 0 to 1.9 is changed to 1, 2 to 2.9 is changed to 2, 3 to 3.9 is changed to 3, 4 to 4.9 is changed to 4, and everything larger becomes 5, thus making it a classification problem. More specifically, this is a multiclass classification because it involves predicting one of five possible classes unlike binary classification which has two possible values.

I choose `average_rating` as the response variable because I want to be able to predict the rating a recipe may receive depending on certain features. At this time of the prediction, I would train my model on features from the columns in the dataframe that I already have such as `minutes`, `days_between`, `season_category`, `season_submitted`, `num_calories`, `total_fat`, and `sugar`. 

The metric I am choosing to evaluate my model is F1-score with weighted averaging because as I mentioned before and showed in the graph in univariate analysis, a majority of the ratings fall between 4 and 5. By using this metric, the unbalanced ratings become balanced as it calculates the F1-score for each class independently then takes the weighted average.  

## Baseline Model

For my baseline model, I began by dropping rows from the `mintues` and `average_rating` columns that contained missing values as this information was not available at the time of prediction. I also dropped the `rating` column from the df because I am predicting the average rating which is based on the rating so having both columns would be redundant. I chose three features based on the `minutes`, `days_between`, and `sugar` columns in my df. I chose three quantitative columns for my model because as seen in previous sections like data analysis or missingness analysis, the `minutes` and `days_between` seem to have an association with a recipes's average rating. I included the `sugar` column as well because I wanted to determine if the sugar a recipe has would also affect a recipe's rating.

As I mentioned in the framing a prediction problem section, I made average ratings into categories and added these values to a new column called `avg_rating_cat`. I then split the data into training and test data using the above columns as my features and `avg_rating_cat` as the value I am predicting. After looking at the values contained in each of my features, I realized that the `minutes` and `days_between` features contain a majority of values closer to zero with some outliers. To account for this I converted both continuous numerical columns into discrete categories. I transformed the `minutes` column using custom binning where I categorized them into five descriptive categories (very short, short, medium, long, very long) based on how long it takes to make a given recipe. Similarly, I transformed the `days_between` column using custom binning where I categorized them into the same description categories as `minutes` based on how long it takes to post a review for a recipe. For both these features, I then used ordinal encoding to map these categories into numbers in a way that preserves order where 0 was very short up till 4 for very long. I chose this approach because shorter recipes and quicker review times are generally more practical and appealing for users as they are more efficient and convenient making it more valuable than longer recipes and extended review times which are less attractive to people because they require more time and effort. I used `FunctionTransfomer` to apply these transformations. For the `sugar` feature, I used `StandardScalar` to standardize the numerical data. I then used a `DecisionTreeClassifier` to classify my model. 

The performance of this model was not that good as I got an overall F1-score of 0.534 suggesting that there is room for improvement by adding more features that are more correlated to predicting average rating.

## Final Model 

I added four features to my final model which were `num_calories`, `total_fat`, `season_category`, and `season_submitted` as well as the features from the baseline model, `minutes`, `days_between`, and `sugar`. 

The column `num_calories` is the total number of calories a recipe contains. As shown in the pivot table created in the framing a prediction model section, it seems that the number of calories a recipe has is correlated with the average rating the recipe gets. Thus, I believe adding this feature improved my model because people tend to look for healthier receipes so those with lower amounts of calories could be predicted to receive better ratings. I used `StandardScalar` to transform the feature and make it more generalizable. 

The column `total_fat` is the total amount of fat put into a recipe. Similar to `num_calories`, the pivot table showed that a lower number of calories are typically associated with a higher rating. Also, health consious people are more likely to try and rate foods that have a lower amount of fat in it. For these reasons, I believe that this feature would improve the performance of my model. I transformed this feature using `StandardScalar`. 

The column `season_category` is the weather type associated with the season a receipe was posted during with values being either warm or cold. As shown in the hypothesis testing section, I found that recipes posted during warmer seasons tend to receive higher ratings. So, I thought adding this feature would improve my model's performance due to the correlation, and a person may be happier during warmer weather than cold weather encouraging them to rate recipes more generously. I used `OneHotEncoder` to transform the categorical values into binary features. 

Similarly, the column `season_submitted` indicates the season in which a recipe was posted. Based on findings from hypothesis testing and bivariate analysis which indicated that there is a relationship between the season a recipe was posted and the review, I believed that adding this feature would improve my model's predictive capabilities. I also used `OneHotEncoder` to transform the catergorical values of spring, summer, winter, and fall into binary features.

I choose to use a `DecisionTreeClassifier` and performed k-fold cross-validation using `GridSearchCV` to find the best combination of hyperparameters with the best average validation performance. For my model, the hyperparameters that performed the best were `criterion` of gini, `max_depth` of 54, and `min_samples_split` of 2. My F1-score for my final model is 0.796 showing that there was an improvement over my baseline model which had an F1-score of 0.534. 

## Fairness Analysis

To assess the fairness of my model, I aim to answer the question of whether my model performs better for warmer or colder seasons. Thus my two groups will be based on the `season_category` column, and I will be determining if my model performs better for recipes that were posted during warmer seasons than recipes that were posted during colder seasons. I performed a permutation test using accuracy parity as my evaluation method because the classes have roughly equal number in both classes so they are balanced. I randomly shuffled the `season_category` column 1000 times and computing the accuracy parity each time. 

**Null Hypothesis:** The classifier's accuracy is the same for both recipes posted in warm seasons and cold seasons, and any differences are due to chance.

**Alternative Hypothesis:** The classifier's accuracy is higher for warmer seasons.

**Test Statistic:** difference in accuracy (warm - cold)

**Significance Level:** 0.05

<iframe
  src="assets/fairness-emp-dist.html"
  width="800"
  height="416"
  frameborder="0"
></iframe>

As shown in the empirical distribution above, I got a p-value of 0.008 which is less than my significance level of 0.05 so I reject the null hypothesis that the classifier's accuracy is fair. The model predicts more accurately for recipes that were submitted during warmer seasons than colder seasons. There are biases present in my model when it comes to the time of year a recipe was posted. 