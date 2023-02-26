<!-- # Recipe-Healthyness-Trends-by-season
For UCSD class DSC80 project 3 -->

# Introduction


In this notebook we are going to be analyzing a dataset that contains recipes and ratings from <a href="https://www.food.com/">food.com</a>. The dataset consists of `234,429` interactions with recipes. Our goal is to answer the following question: 

**Do people eat more unbalanced food during the winter holiday season?**

We asked this question because we feel people tend to eat more unbalanced during these months since there are lots of holidays and special occasions which include unbalanced food many times. Therefore, in order to check if this is true, we are going to analyze the data from <a href="https://www.food.com/">food.com</a>.

For this we are going to be analyzing the columns `'id'`, `'date'` and `'nutrition'`. The column `'date'` stores the dates in which an interaction was made with a recipe. We will use this date as the date a person ate one of the recipes. Then we will use the column `'nutrition'` to classify each recipe `'id'` as a balanced or unbalanced recipe.

Other columns we might work with during the cleaning process are `'minutes'`,`'submitted'`,`'n_steps'`,`'n_ingredients'`. We will probably use these since they contain relevant information about the recipe that if it's extreme, it could raise some red flags about our data.

### Data Sets:

_Recipes_
<!-- start table -->
<div class="table-wrapper"><table><thead><tr><th style="text-align: left">Column</th><th style="text-align: left">Description</th></tr></thead><tbody><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'name'</code></td><td style="text-align: left">Recipe name</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'id'</code></td><td style="text-align: left">Recipe ID</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'minutes'</code></td><td style="text-align: left">Minutes to prepare recipe</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'contributor_id'</code></td><td style="text-align: left">User ID who submitted this recipe</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'submitted'</code></td><td style="text-align: left">Date recipe was submitted</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'tags'</code></td><td style="text-align: left">Food.com tags for recipe</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'nutrition'</code></td><td style="text-align: left">Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'n_steps'</code></td><td style="text-align: left">Number of steps in recipe</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'steps'</code></td><td style="text-align: left">Text for recipe steps, in order</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'description'</code></td><td style="text-align: left">User-provided description</td></tr></tbody></table></div>
<!-- end table -->

_Ratings_
<div class="table-wrapper"><table><thead><tr><th style="text-align: left">Column</th><th style="text-align: left">Description</th></tr></thead><tbody><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'user_id'</code></td><td style="text-align: left">User ID</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'recipe_id'</code></td><td style="text-align: left">Recipe ID</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'date'</code></td><td style="text-align: left">Date of interaction</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'rating'</code></td><td style="text-align: left">Rating given</td></tr><tr><td style="text-align: left"><code class="language-plaintext highlighter-rouge">'review'</code></td><td style="text-align: left">Review text</td></tr></tbody></table></div>



# Cleaning and EDA

<!-- Describe, in detail, the data cleaning steps you took and how they affected your analyses. The steps should be explained in reference to the data generating process. Show the head of your cleaned DataFrame -->
Steps:

    1. Merge these two datasets on the `'recipe_id'` column
    2. Fill 0's in `'rating'` with `'NaN'`, because food.com doesn't allow to rate a recipe with 0's
    3. Create a columns of the average rating per recipe

Let's check how messy is our data, so we will start exploring column by column how everything looks.

First, let's check if the column `'id'` is unique. For that we need to use the recipes DataFrame which should only contain one row per `'recipe_id'`.

Since we aren't going to be using the `'name'` column neither the `'contributor_id'` column rather than for reference, there is no need to clean it thoroughly. Still, let's see how many recipe `'names'` and `'contributor_id'` are repeated.

Interestingly, we found out that `'microwave chocolate mug brownie'` is the most repeated recipe `'name'` in our data, and that `'contributor_id'` = `'37449'` has done `'3060'` reviews. Lets now check the `'minutes'` column.

There's a large max number of `'minutes'` and it seems strange since it's nearly impossible a recipe could take you **1 million** minutes to make. Lets visualize the `'minutes'` distribution and evaluate how to address this issue.

<iframe src="assets/visualization_1.html" width=700 height=500 frameBorder=0></iframe>

<iframe src="assets/visualization_2.html" width=700 height=500 frameBorder=0></iframe>

We can visualize that most of the data is centered in a reasonable place, but there are a few data points spread out all over until **1 million** minutes. 

**What are we going to do with these values?**

Since it's really ambiguous how to decide a cutoff to these values, we are going to use a common statistical practice called 1.5xIQR rule developed by statistics to remove outliers.

Exactly 24291 rows were cut off from the original dataframe.
Remaining Rows:  210138

<iframe src="assets/visualization_3.html" width=700 height=500 frameBorder=0></iframe>


Moving on, we want the column `'submitted'` and `'date'` to both be `DateTime` objects. We are going check if these are strings and if they are, make the necessary transformations.

As we can see above, these columns are both stored as string `objects`, so we are going to transform them.

We can check that data `types` have changed.

In the table above, we can see that the `'tags'`, `'nutrition'` and `'ingredients'` columns are also stored as `objects`, let's check if they are actually `lists`. If that's not the case, we are going to use some string methods to transform these columns elements into `lists`.

Now that we have converted our `string` into `lists` let's see how are the values of the `lists` from `'nutrition'` stored.

We need to do some cleaning here, so first we are going to transform this `strings` into `floats`. Then, these values are supposed to be percentages, so we are going to convert them into decimals.

Now, let's check that the values of these new columns make sense. For this we need to write some assumptions. First, none of these values should be negative. Let's check that!

Apparently, none of them is negative. Now, let's check their ranges.

As we can see, we have some values that are way out of reality for a single serving size. We need to explore more what's happening with this data. Let's check out all the recipes with extremely high calories.

As we can observe, most of these recipes are either cookies, cakes, breads or turkeys, which all of them are recipes that you usually cook for more than one person. These data will cause trouble to our hypothesis testing, so we need to find a way to cut it off. First lets visualize the distribution of `'calories'`, so we can decide how we are going to perform the cutoff.

<iframe src="assets/visualization_4.html" width=700 height=500 frameBorder=0></iframe>

<iframe src="assets/visualization_5.html" width=700 height=500 frameBorder=0></iframe>

In the histogram above, we can observe that most of our data is concentrated in one point, and there are a bunch of other spread through until `45,000 calories`. Lets apply the same technique we used for `'minutes'`.

Exactly 11676 rows were cut off from the original dataframe.
Remaining Rows:  198462

<iframe src="assets/visualization_6.html" width=700 height=500 frameBorder=0></iframe>

Since we have cleaned most of our data, lets check if there are any other columns with extreme values.

Most of these values are within a reasonable range, although the `'sodium'` column `max` seems pretty extreme. Lets explore the higher values of the `'sodium'` column and argue if these values are reasonable or not.

Most of this recipes are types of salts which is a reasonable explanation to their high levels of `'sodium'`. Also the turkey apparentley was brined and that requieres lots of `'sodium'` too, so it's also fine.

We have already gone through most of the columns, but we still need to check the `'user_id'` and `'rating'` columns. As you can see from the table above, there is only one row in the `'user_id'` column which contains `NaN`. Let's check it out.

Since this row doesn't provide any information to our data we are going to drop it.

Since this row doesn't provide any information to our data we are going to drop it.

As we can see, the ratings are stored as `floats`, but the reason being is because we can't store `np.NaN` as `ints`. Therefore, all ratings must be stored as `floats`. Now let's check if any of them has decimals or if they aren't in the range `[1,5]`.

We have checked that our `'rating'` values are in order. With this we finish the cleaning procces of our DataFrame, here is the head of it.

Now that we have our data cleaned, we can add a column that classifies our recipes as `'balanced'` or `'unbalanced'`. According to the FDA, meals should contain between `5%` to `20%` of the percentage of daily value (PDV). We are going to use this fact to classify our recipes in `'balanced'` and `'unbalanced'` categories.

Source: https://www.fda.gov/food/new-nutrition-facts-label/lows-and-highs-percent-daily-value-new-nutrition-facts-label

Let's create a DataFrame with `unique id's`. This will be handy for the future when we start doing some `EDA`.

#### Univariate Analysis

We want to explore what's the distribution of interactions among all recipies. We think this is an interesting analysis since we can start drawing lots of experiments from it.

<iframe src="assets/visualization_7.html" width=700 height=500 frameBorder=0></iframe>

<iframe src="assets/visualization_8.html" width=700 height=500 frameBorder=0></iframe>

We have checked that our `'rating'` values are in order. With this we finish the cleaning procces of our DataFrame, here is the head of it.

Now let's observe the distribution of recipe's submissions throughout the years.



This is a really insightfull visualization since we can observe how the amount of submissions decreased over the years from thousands of recipes submitted per month to less than 50 per month. 

#### Bivariate Analysis

Sometimes the best food we eat is the one with the most calories. Although that's quite a shame, let's see if that's true in our data. For this we are going to compare the relationship between `'rating'` and `'calories'`. We are going to plot a horizontal bar chart to show the `mean of calories per star rating`.

<iframe src="assets/visualization_9.html" width=700 height=500 frameBorder=0></iframe>

This is quite surprising since it goes against our intuition. We thought food with more calories would have more ratings but apparently is completely opposite from what we thought. Even though almost all means are the same, we can see an inverse proportionate trend with respect to ratings. It will be interesting then to see how our question develops in the next steps.

Now lets observe the relationship between mean of calories consumed per month of the year.

<iframe src="assets/visualization_10.html" width=700 height=500 frameBorder=0></iframe>

There appears to be a trens in the data. The months February, March, September and October have quite a peak compared to the rest of the months. It could be quite interesting to anlyze more in depth what are the reasons of this.

Another bivariate analysis we can work on is the distribution of calories between `'balanced'` and `'unbalanced'` recipes. Lets plot a histogram with a marginal box plot to see the difference in distributions.

<iframe src="assets/visualization_11.html" width=700 height=500 frameBorder=0></iframe>

We can observe that there is a signifficant difference in the distributions of calories of balanced and unbalanced recipes. Most of the balanced recipes are centered around 200 while the unbalanced recipes are more skewed to the right.

#### Interesting Aggregates

One interesting aggregate is the mean of all columns for balanced and unbalanced recipes. For this we need to groupby recipe, but we already did that, so we are going to use our 'unique_id_df' to groupby the 'balanced' column.

<table border="0" class="dataframe" style="width:90%">  <thead>    <tr style="text-align: right;">      <th></th>      <th>minutes</th>      <th>n_steps</th>      <th>n_ingredients</th>      <th>avg_rating</th>      <th>calories</th>      <th>total_fat</th>      <th>sugar</th>      <th>sodium</th>      <th>protein</th>      <th>saturated_fat</th>      <th>carbohydrates</th>    </tr>    <tr>      <th>balanced</th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>      <th></th>    </tr>  </thead>  <tbody>    <tr>      <th>False</th>      <td>37.288474</td>      <td>9.720619</td>      <td>9.037901</td>      <td>4.630582</td>      <td>322.656586</td>      <td>0.237330</td>      <td>0.469933</td>      <td>0.229876</td>      <td>0.268985</td>      <td>0.295157</td>      <td>0.101728</td>    </tr>    <tr>      <th>True</th>      <td>39.719638</td>      <td>10.237726</td>      <td>9.284238</td>      <td>4.611692</td>      <td>200.788889</td>      <td>0.111835</td>      <td>0.117106</td>      <td>0.119289</td>      <td>0.117946</td>      <td>0.108475</td>      <td>0.088152</td>    </tr>  </tbody></table>

We can compare the values of all the columns. First, we can see that the avg_rating for each one is quite similar which sounds fair. Then, the calories column do shows a good difference between them. The unbalanced recipes have a mean of 322.65 while the balanced recipes have a mean of 200.78. Finally the rest of the columns, unbalanced recipes is much greater to all of balanced recipes, with the carbohydrates column being the closest one to the blanced recipes mean.

Another aggregate we could do is is the mean of all columns for ratings of recipes. We are going to groupby ratings in our data DataFrame and aggregate all columns using the mean.

<table border="0" class="dataframe" style="width:90%">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>minutes</th>
      <th>n_steps</th>
      <th>n_ingredients</th>
      <th>calories</th>
      <th>total_fat</th>
      <th>sugar</th>
      <th>sodium</th>
      <th>protein</th>
      <th>saturated_fat</th>
      <th>carbohydrates</th>
      <th>balanced</th>
    </tr>
    <tr>
      <th>rating</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1.0</th>
      <td>38.037634</td>
      <td>10.156362</td>
      <td>8.717742</td>
      <td>314.147939</td>
      <td>0.224610</td>
      <td>0.569194</td>
      <td>0.250883</td>
      <td>0.224888</td>
      <td>0.292706</td>
      <td>0.109933</td>
      <td>0.012097</td>
    </tr>
    <tr>
      <th>2.0</th>
      <td>38.960128</td>
      <td>10.219564</td>
      <td>9.004785</td>
      <td>328.388357</td>
      <td>0.236273</td>
      <td>0.508814</td>
      <td>0.235986</td>
      <td>0.258788</td>
      <td>0.309495</td>
      <td>0.110404</td>
      <td>0.009038</td>
    </tr>
    <tr>
      <th>3.0</th>
      <td>37.530426</td>
      <td>9.578262</td>
      <td>9.008114</td>
      <td>323.329665</td>
      <td>0.230914</td>
      <td>0.451915</td>
      <td>0.223435</td>
      <td>0.278932</td>
      <td>0.289329</td>
      <td>0.104074</td>
      <td>0.013185</td>
    </tr>
    <tr>
      <th>4.0</th>
      <td>36.021115</td>
      <td>9.325153</td>
      <td>8.938727</td>
      <td>319.878731</td>
      <td>0.228987</td>
      <td>0.417097</td>
      <td>0.223185</td>
      <td>0.282177</td>
      <td>0.280962</td>
      <td>0.101275</td>
      <td>0.012066</td>
    </tr>
    <tr>
      <th>5.0</th>
      <td>35.875719</td>
      <td>9.593679</td>
      <td>8.884717</td>
      <td>317.598039</td>
      <td>0.237551</td>
      <td>0.449384</td>
      <td>0.239469</td>
      <td>0.263811</td>
      <td>0.297464</td>
      <td>0.098186</td>
      <td>0.012037</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 11 columns</p>



It's interesting to see that that there is an inverse relationship between `'rating'` and `'minutes'`. Same happens with `'rating'` and `'n_steps'`. The rest of the columns doesn't seem to have much trends at all.


Here is the final resulting cleaned dataframe:


<iframe src="assets/datahead.html" width=1000 height=600 frameBorder=0></iframe>

<!-- <iframe src="assets/df2.html" width=1000 height=600 frameBorder=0></iframe> -->




# Assessment of Missingness

<!-- to markdown -->
**Percent Data Missing**

|             |         0 |

|:------------|----------:|
| description | 0.0524032 |
| rating      | 5.94474   |
| avg_rating  | 1.06822   |

dtype: float64
<!-- end -->

This summary shows us that of the relevant columns we are still working with, there are 3 columns with missing data. 2 are from the original dataset but we added `'avg_rating'`. We will look deeper into `'rating'` and `'description'`. But we should notice that none of them have relatively many datapoints missing.

First we can quickly explain `'avg_rating'`

Looking at the other Columns with missing data, we know avg_rating was calculated by aggregating the ratings for each recipe, so any missingness here is __Missing by Design__ based on missingness in the rating column. We can perfectly and exactly determine the missingness of `'avg_rating'` by checking if any interactions are missing a `'rating'` for each recipe (grouping by `'id'`). 

#### NMAR Analysis

We do not believe that the missingness mechanism for either is not missing at random, or NMAR. That would mean that some quality of the missing data (the value itself) determines the probability of its missingness. 

- For `'description'` there is no obvious or reasonable quality such as sentiment or length that would relate to its missingness. 

- `'Rating'` is more complicated. Users interacting with a recipe can either leave a review with a rating, or just a comment with no rating. This means some users may be commmenting without having even tried the recipe, that means for that interaction no rating value exists or should exist. This rules out NMAR.

Here is an example of an interaction without a rating, it happens to be for someone who tried the recipe and loved it. We can imagine this is also likely for someone who hasn't tried it or tried it and did not like it. 

<!-- <img src="assets/NoRating.png" height=100> -->
![example of missing rating](assets/NoRating.png){style="display: block; margin: 0 auto" }

#### Missingness Dependency

Neither `'description'` nor `'rating'` is closely related to the guiding question of unhealthy foods during the holiday season. However we will still investigate if they are missing dependent on another column (MAR) or if they are missing completely at random (MCAR). 

First we will explore missingness in `'description'`:

We believe it is plausible that if a `'description'` is missing then it is likely the author did not put in much effort to their post and the recipe will not be as good. This line of thinking would mean the `'avg_rating'` on that recipe would probably be lower for those that are missing. That would make `'description'` MAR on `'avg_rating'`. We can test this hypothesis with a permutation test. 
<br>
<br>

**Avg_rating**

We perform a Permutation Test to see if missingness of description column is conditional on `'avg_rating'`. We will end up repeating this process for many columns following. Description is unique to a recipe but duplicated for each interaction, we want to know how it relates to the recipes so we must retrieve that data again by grouping the interviews for each recipe together. We need to work with only the non-null rows of the data in avg_rating otherwise no statistic could be computed.

#### Choosing a Test Statistic

`'avg_rating'` is a quantitative variable which gives us two main options. We should use the difference of means for missing and not missing `'description'` if the distribution of `'avg_rating'` is roughly the same shape but shifted. If the shape is different and distributions are centered in the same place we should use the Kolmogorov-Smirnov statistic. 

<iframe src="assets/visualization_12.html" width=700 height=500 frameBorder=0></iframe>

As we can see the distributions are centered at roughly the same location but their shapes are pretty different. This means we will use the Kolmogorov-Smirnov test statistic and run a permutation test to see if `'description'` is MAR on `'avg_rating'`.

* Permutation Test for MAR of Description on `'avg_rating'`
* Test Statistic: Kolmogorov-Smirnov
* Alpha level: 5%

_Results_
z-score:  3.3333333333333335 <br>
p-value:  0.9380583291749669 <br>
Reject Null:  False <br>

##### Interpret Results

We can interpret the results of the Kolmogorov-Smirnov permutation test as follows. Under the assumption that `'description'` is unrelated to the distribution of `'avg_rating'`, the chance of seeing distributions as, or more, different than our two observed `'avg_rating'` distributions is 93.8%.

This is above our alpha threshold of 5% so we **fail to reject the null**. This means that the missingness of `'description'` is unlikely to be related and or condional to the distribution of `'avg_rating'` and that we have no evidence to say that it does (we also have no evidence to say it is unrelated for sure).

This would mean `'description'` is not MAR upon `'avg_rating'`.

This also indicates that our intuition was incorrect, lets test another column, `'user_id'`.
<br>
<br>

**User_id**

User_id is a categorical variable which means that we should use the total variation distance for a test statistic. This is calculated as the sum of the differences by category, divided by 2. It describes the distance between two categorical distributions. 

* Permutation Test for MAR of Description on `'User_id'`
* Test Statistic: TVD
* Alpha level: 5%

The last test statistic was calculated using a python library and immediately gave the results. Now we will run the permutation test with our own simulation. To do that we calculate many TVDs, one for each shuffle of `'user_id'`, this is done because under the null the missingness of `'description'` does not depend on `'user_id'` so the pairings are arbitrary. We will then compare our observed TVD to these. The observed test statistic is the TVD calculated on the actual sample from the data without any shuffling.

<iframe src="assets/visualization_13.html" width=700 height=500 frameBorder=0></iframe>

The graph gives an idea of how usual or abnormal our observed TVD is compared to permutations under the null. It shows a histogram of all the TVDs when `'user_id'` doesn't influence `'description'` missingness. We get an exact measure of how many TVDs are as or more extreme when we calculate the p-value.

##### Results

We can interpret the results of the TVD permutation test as follows. Under the assumption that `'description'` is unrelated to the distribution of `'user_id'`, the chance of seeing distributions as, or more, different than our two observed `'user_id'` distributions is 35.9%.

This is above our alpha threshold of 5% so we **fail to reject the null**. This means that the missingness of `'description'` is not likely to be related and or condional on `'user_id'`. Consequently we cannot say `'description'` is MAR upon `'user_id'`.

Lets try another, there might be one or more that are dependent, if we find even one that is MAR we would consider the missingness in `'description'` to be MAR, if we _fail_ to reject the null for every column then we default to MCAR. Lets test `'calories'` against `'description'`.
<br>
<br>

**Calories**

Need to choose a test statistic, so need to see if the distribtutions of `'calories'` with and without `'description_missingness'` are shifted versions (diff of means) or same center and diff shape (Kolmogorov-Smirnov)

<iframe src="assets/visualization_14.html" width=700 height=500 frameBorder=0></iframe>

We see that the means are closely aligned, The shapes are also pretty similar but the height of the curve and the level it dips are different so we will try Kolmogorov-Smirnov statistic.

z-score:  202.3
p-value:  0.5062467217248188
Reject Null:  False

That did not result in significant results, we **fail to reject the null**. Before we interpret and conclude, lets test the difference in means as well. 

<iframe src="assets/visualization_15.html" width=700 height=500 frameBorder=0></iframe>

Once again, the graph gives an idea of how usual or abnormal our observed difference of means is compared to permutations under the null, we get an exact measure of how many differences are as or more extreme when we calculate the p-value:

##### Interpret Results

We can interpret the results of both permutation tests. Under the null assumptions where missingness of `'description'` is unrelated to the distribution of `'calories'`, the chance of seeing a mean difference or Kolmogorov-Smirnov, or more, different than observed is 17.2% and 50.6% respectively.

These are above our alpha threshold of 5% so we **fail to reject the null** twice. This means that the missingness of `'description'` is not likely to be related and or condional on `'calories'`.

This would mean `'description'` is not MAR upon `'calories'`.

As we may have guessed from visualizing the distributions, the difference was not significant and we **fail to reject the null hypothesis**. `'description'` is not MAR on `'calories'`

We will skip checking each column individually but that is the process to check for MAR vs MCAR.



Next we will explore missingness in `'ratings'`:

This means we need the whole data dataframe since this involves each interaction not only each unique recipe.

We will check `'calories'` first. Its a stretch but since we have a hunch about unhealthy foods being more popular in the holiday season, maybe more calories relates to lower ratings and therefore missing ratings. \*Note even a rejected null could not prove this or any theory because it is not evidence of causation.

<iframe src="assets/visualization_16.html" width=700 height=500 frameBorder=0></iframe>

<iframe src="assets/visualization_17.html" width=700 height=500 frameBorder=0></iframe>

Once again, the graph gives an idea of how usual or abnormal our observed difference of means is compared to permutations under the null, we get an exact measure of how many differences are as or more extreme when we calculate the p-value:


z-score:  387.5
p-value:  8.96823731655477e-08
Reject Null:  True

##### Interpret Results

We can interpret the results of the Kolmogorov-Smirnov permutation test as follows. Under the assumption that `'ratings'` is unrelated to the distribution of `'calories'`, the chance of seeing distributions as, or more, different than our two observed `'calories'` distributions rounds to 0%.

This is below our alpha threshold of 5% so we **reject the null**. This means that the missingness of `'ratings'` is likely related and or condional to the distribution of `'calories'`.

This means `'ratings'` is MAR upon `'calories'`.

We will check another column against `'ratings'`.


**total_fat**

* Permutation Test for missingness of Rating on `'total_fat'`
* Test Statistic: Kolmogorov-Smirnov
* Alpha level: 5%

<iframe src="assets/visualization_18.html" width=700 height=500 frameBorder=0></iframe>

z-score:  0.37
p-value:  2.7111280133990385e-06
Reject Null:  True


We got another significant result for MAR on `'rating'`. Lets see one more column unrelated to the original `'nutrition'` column to see if it is also MAR.

**name**

* Permutation Test for missingness of Rating on `'name'`
* Test Statistic: TVD
* Alpha level: 5%

The observed test statistic is the TVD calculated on the actual sample from the data without any shuffling.

<iframe src="assets/visualization_19.html" width=700 height=500 frameBorder=0></iframe>

##### Interpret Results

For a third time we **reject the null** for a missingness of `'rating'` conidioned on another column. Making `'rating'` dependent on many other columns. 

While assessing missingness for the recipes and interactions data we found that missing descriptions are not missing at random by `'avg_rating'`, `'user_id'`, or `'calories'`. We did not find any columns that it was MAR for. For the `'rating'` column we found the opposite, it was missing at random on `'calories'`, `'total_fat'`, and `'name'`. We are not using either column for our hypothesis test so no imputation or drops are necessary.









# Hypothesis Testing

Guiding Question: **Do people eat more unhealthy food during the winter holiday season?**

Interpretation according to available data: Do people interact more with unbalanced recipes from October to December than the rest of the year? Remember unbalanced recipes are our equivalent of unhealthy.


* Variables: `'month'` and `'balanced'`

* Test Statistic: _proportion of recipes that are **not** balanced_

* alpha level: 0.01 significance level

**Hypotheses**

* _Null Hypothesis:_ proportion of unbalanced recipes from October to December is **equal to** the proportion of all unbalanced recipes
* _Alternative Hypothesis:_ proportion of unbalanced recipes from October to December is **greater than** the proportion of all unbalanced recipes

Lets check how the proportion of unbalanced recipes varies by year in case we should split up our hypothesis test. 

<iframe src="assets/visualization_20.html" width=700 height=500 frameBorder=0></iframe>

We notive that there is a dip followed by a spike in the last three years, this data could be more variable since as we saw in the EDA there are much fewer recipes interacted with in the last few years. For this reason we will continue with the hypothesis test on all of the data together regardless of year. 

_Now we perform the hypothesis test:_

Just like with the permutation tests for missingness earlier, we have a distribution of test statistics, in this case proportions of unbalanced recipes. We can compare the proportion of unbalanced recipes from only the Holiday months which we are interested in compared to the general 

<iframe src="assets/visualization_21.html" width=700 height=500 frameBorder=0></iframe>

##### Interpret Results

Under the assumption that `'balanced'` or healthiness of a recipe is unrelated to the `'month'`, the chance of seeing a proportion of unbalanced recipes __in any three month period__ as or more extreme than the observed proportion of unbalanced recipes in the holiday season is 42.2%.

This is above our alpha threshold of 1% so we **fail to reject the null**. Our observed proportion was not unlikely under the null, which means our observation is consistent with the hypothesis that being the winter holiday season does not relate to unbalanced recipe interactions. This also means that our intuition was not supported and we cannot say based on the data that people eat more unhealthy food during the winter holiday season. 

This was not the result we expected, however it does make us wonder if any one specific month or week would be significant. That any many more questions will have to be explored another time. 





# Conclusion

We really enjoyed this project. We would like to encourage other students (everyone is a student) to check out online resources and try to do a project like this on your own with the same or another dataset! Real life data is messy, cleaning was paramount, little changes to outlier removal dramatically changed p-value results in our hypothesis test. It is always good to document your process for replicability and so you ensure your steps are logical. Lastly doing it with a friend makes it extra fun!

Sincerely, 
Eduardo and Trey

PS
Check out our other work at our github profiles linked below or send us a message [trey11sport@gmail.com](mailto:trey11sport@gmail.com) [eduardo.spiegel21@gmail.com](mailto:eduardo.spiegel21@gmail.com)


**Other Possible Questions:**
- What are the healthy/unhealthy food recipe trends over the year?
- More specifically: Have low fat low sugar recipes become more or less common over time? What about seasonally?
- Which ingredient is the most unique with the highest rating?
- Are healthy foods rated higher or lower?
- Over time have there been a higher or lower proportion of healthy recipes posted?
- can we find trends on when a specific recipe is or will be popular, what about general trends accross recipes?
- Are recipes that do not require an oven less healthy?
- Does the sentiment of a review correlate positively with the rating?
- Which ingredients have become more popular since COVID


