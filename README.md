<!-- # Recipe-Healthyness-Trends-by-season
For UCSD class DSC80 project 3 -->

# Introduction


In this notebook we are going to be analyzing a dataset that contains recipes and ratings from <a href="https://www.food.com/">food.com</a>. The dataset consists of `234,429` interactions with one recipe. Our goal is to answer the following question: 

**Do people eat more unbalanced food during the winter holiday season?**

We asked this question because we feel people tend to eat more unbalanced during these months since there are lots of holidays and special occasions which include unbalanced food many times. Therefore, in orther to check if this is true, we are going to analyze the data from <a href="https://www.food.com/">food.com</a>.

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

We are going to merge these two datasets on the `'recipe_id'` column.

Fill 0's with np.nan

Find the average rating per recipe, as a Series

Let's check how messy is our data, so we will start exploring column by column how everything looks.

First, let's check if the column `'id'` is unique. For that we need to use the recipes DataFrame which should only contain one row per `'recipe_id'`.

Since we aren't going to be using the `'name'` column neither the `'contributor_id'` column rather than for reference, there is no need to clean it thoroughly. Still, let's see how many recipe `'names'` and `'contributor_id'` are repeated.

Interestingly, we found out that `'microwave chocolate mug brownie'` is the most repeated recipe `'name'` in our data, and that `'contributor_id'` = `'37449'` has done `'3060'` reviews. Lets now check the `'minutes'` column.

There's a large max number of `'minutes'` and it seems strange since it's nearly impossible a recipe could take you **1 million** minutes to make. Lets visualize the `'minutes'` distribution and evaluate how to address this issue.

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

We can visualize that most of the data is centered in a reasonable place, but there are a few data points spread out all over until **1 million** minutes. 

**What are we going to do with these values?**

Since it's really ambiguous how to decide a cutoff to these values, we are going to use a common statistical practice called 1.5xIQR rule developed by statistics to remove outliers.

Exactly 24291 rows were cut off from the original dataframe.
Remaining Rows:  210138

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>


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

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

In the histogram above, we can observe that most of our data is concentrated in one point, and there are a bunch of other spread through until `45,000 calories`. Lets apply the same technique we used for `'minutes'`.

Exactly 11676 rows were cut off from the original dataframe.
Remaining Rows:  198462

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

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

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/file-name.html" width=800 height=600 frameBorder=0></iframe>

We have checked that our `'rating'` values are in order. With this we finish the cleaning procces of our DataFrame, here is the head of it.

Now let's observe the distribution of recipe's submissions throughout the years.



This is a really insightfull visualization since we can observe how the amount of submissions decreased over the years from thousands of recipes submitted per month to less than 50 per month. 

#### Bivariate Analysis

Sometimes the best food we eat is the one with the most calories. Although that's quite a shame, let's see if that's true in our data. For this we are going to compare the relationship between `'rating'` and `'calories'`. We are going to plot a horizontal bar chart to show the `mean of calories per star rating`.



This is quite surprising since it goes against our intuition. We thought food with more calories would have more ratings but apparently is completely opposite from what we thought. Even though almost all means are the same, we can see an inverse proportionate trend with respect to ratings. It will be interesting then to see how our question develops in the next steps.

Now lets observe the relationship between mean of calories consumed per month of the year.



There appears to be a trens in the data. The months February, March, September and October have quite a peak compared to the rest of the months. It could be quite interesting to anlyze more in depth what are the reasons of this.

Another bivariate analysis we can work on is the distribution of calories between `'balanced'` and `'unbalanced'` recipes. Lets plot a histogram with a marginal box plot to see the difference in distributions.



We can observe that there is a signifficant difference in the distributions of calories of balanced and unbalanced recipes. Most of the balanced recipes are centered around 200 while the unbalanced recipes are more skewed to the right.

#### Interesting Aggregates

One interesting aggregate is the mean of all columns for balanced and unbalanced recipes. For this we need to groupby recipe, but we already did that, so we are going to use our 'unique_id_df' to groupby the 'balanced' column.


We can compare the values of all the columns. First, we can see that the avg_rating for each one is quite similar which sounds fair. Then, the calories column do shows a good difference between them. The unbalanced recipes have a mean of 322.65 while the balanced recipes have a mean of 200.78. Finally the rest of the columns, unbalanced recipes is much greater to all of balanced recipes, with the carbohydrates column being the closest one to the blanced recipes mean.

Another aggregate we could do is is the mean of all columns for ratings of recipes. We are going to groupby ratings in our data DataFrame and aggregate all columns using the mean.

<table border="0" class="dataframe">
  <thead>
    <tr style="text-align: right;">
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
  </thead>
  <tbody>
    <tr>
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

<table border="0" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>name</th>
      <th>id</th>
      <th>minutes</th>
      <th>contributor_id</th>
      <th>submitted</th>
      <th>tags</th>
      <th>n_steps</th>
      <th>steps</th>
      <th>description</th>
      <th>ingredients</th>
      <th>n_ingredients</th>
      <th>user_id</th>
      <th>date</th>
      <th>rating</th>
      <th>review</th>
      <th>avg_rating</th>
      <th>calories</th>
      <th>total_fat</th>
      <th>sugar</th>
      <th>sodium</th>
      <th>protein</th>
      <th>saturated_fat</th>
      <th>carbohydrates</th>
      <th>balanced</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1 brownies in the world    best ever</td>
      <td>333281</td>
      <td>40</td>
      <td>985201</td>
      <td>2008-10-27</td>
      <td>[60-minutes-or-less, time-to-make, course, main-ingredient, preparation, for-large-groups, desserts, lunch, snacks, cookies-and-brownies, chocolate, bar-cookies, brownies, number-of-servings]</td>
      <td>10</td>
      <td>['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting']</td>
      <td>these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!</td>
      <td>[bittersweet chocolate, unsalted butter, eggs, granulated sugar, unsweetened cocoa powder, vanilla extract, brewed espresso, kosher salt, all-purpose flour]</td>
      <td>9</td>
      <td>386585</td>
      <td>2008-11-19</td>
      <td>4.0</td>
      <td>These were pretty good, but took forever to bake.  I would send it ended up being almost an hour!  Even then, the brownies stuck to the foil, and were on the overly moist side and not easy to cut.  They did taste quite rich, though!  Made for My 3 Chefs.</td>
      <td>4.0</td>
      <td>138.4</td>
      <td>0.10</td>
      <td>0.50</td>
      <td>0.03</td>
      <td>0.03</td>
      <td>0.19</td>
      <td>0.06</td>
      <td>False</td>
    </tr>
    <tr>
      <td>1 in canada chocolate chip cookies</td>
      <td>453467</td>
      <td>45</td>
      <td>1848091</td>
      <td>2011-04-11</td>
      <td>[60-minutes-or-less, time-to-make, cuisine, preparation, north-american, for-large-groups, canadian, british-columbian, number-of-servings]</td>
      <td>12</td>
      <td>['pre-heat oven the 350 degrees f', 'in a mixing bowl , sift together the flours and baking powder', 'set aside', 'in another mixing bowl , blend together the sugars , margarine , and salt until light and fluffy', 'add the eggs , water , and vanilla to the margarine / sugar mixture and mix together until well combined', 'add in the flour mixture to the wet ingredients and blend until combined', 'scrape down the sides of the bowl and add the chocolate chips', 'mix until combined', 'scrape down the sides to the bowl again', 'using an ice cream scoop , scoop evenly rounded balls of dough and place of cookie sheet about 1 - 2 inches apart to allow for spreading during baking', 'bake for 10 - 15 minutes or until golden brown on the outside and soft &amp; chewy in the center', 'serve hot and enjoy !']</td>
      <td>this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.</td>
      <td>[white sugar, brown sugar, salt, margarine, eggs, vanilla, water, all-purpose flour, whole wheat flour, baking soda, chocolate chips]</td>
      <td>11</td>
      <td>424680</td>
      <td>2012-01-26</td>
      <td>5.0</td>
      <td>Originally I was gonna cut the recipe in half (just the 2 of us here), but then we had a park-wide yard sale, &amp; I made the whole batch &amp; used them as enticements for potential buyers ~ what the hey, a free cookie as delicious as these are, definitely works its magic! Will be making these again, for sure! Thanks for posting the recipe!</td>
      <td>5.0</td>
      <td>595.1</td>
      <td>0.46</td>
      <td>2.11</td>
      <td>0.22</td>
      <td>0.13</td>
      <td>0.51</td>
      <td>0.26</td>
      <td>False</td>
    </tr>
    <tr>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>50969</td>
      <td>2008-05-30</td>
      <td>[60-minutes-or-less, time-to-make, course, main-ingredient, preparation, side-dishes, vegetables, easy, beginner-cook, broccoli]</td>
      <td>6</td>
      <td>['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']</td>
      <td>since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008</td>
      <td>[frozen broccoli cuts, cream of chicken soup, sharp cheddar cheese, garlic powder, ground black pepper, salt, milk, soy sauce, french-fried onions]</td>
      <td>9</td>
      <td>29782</td>
      <td>2008-12-31</td>
      <td>5.0</td>
      <td>This was one of the best broccoli casseroles that I have ever made.  I made my own chicken soup for this recipe. I was a bit worried about the tsp of soy sauce but it gave the casserole the best flavor. YUM!  \nThe photos you took (shapeweaver) inspired me to make this recipe and it actually does look just like them when it comes out of the oven.  \nThanks so much for sharing your recipe shapeweaver. It was wonderful!  Going into my family's favorite Zaar cookbook :)</td>
      <td>5.0</td>
      <td>194.8</td>
      <td>0.20</td>
      <td>0.06</td>
      <td>0.32</td>
      <td>0.22</td>
      <td>0.36</td>
      <td>0.03</td>
      <td>False</td>
    </tr>
    <tr>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>50969</td>
      <td>2008-05-30</td>
      <td>[60-minutes-or-less, time-to-make, course, main-ingredient, preparation, side-dishes, vegetables, easy, beginner-cook, broccoli]</td>
      <td>6</td>
      <td>['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']</td>
      <td>since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008</td>
      <td>[frozen broccoli cuts, cream of chicken soup, sharp cheddar cheese, garlic powder, ground black pepper, salt, milk, soy sauce, french-fried onions]</td>
      <td>9</td>
      <td>1196280</td>
      <td>2009-04-13</td>
      <td>5.0</td>
      <td>I made this for my son's first birthday party this weekend. Our guests INHALED it! Everyone kept saying how delicious it was. I was I could have gotten to try it.</td>
      <td>5.0</td>
      <td>194.8</td>
      <td>0.20</td>
      <td>0.06</td>
      <td>0.32</td>
      <td>0.22</td>
      <td>0.36</td>
      <td>0.03</td>
      <td>False</td>
    </tr>
    <tr>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>50969</td>
      <td>2008-05-30</td>
      <td>[60-minutes-or-less, time-to-make, course, main-ingredient, preparation, side-dishes, vegetables, easy, beginner-cook, broccoli]</td>
      <td>6</td>
      <td>['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']</td>
      <td>since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008</td>
      <td>[frozen broccoli cuts, cream of chicken soup, sharp cheddar cheese, garlic powder, ground black pepper, salt, milk, soy sauce, french-fried onions]</td>
      <td>9</td>
      <td>768828</td>
      <td>2013-08-02</td>
      <td>5.0</td>
      <td>Loved this.  Be sure to completely thaw the broccoli.  I didn&amp;#039;t and it didn&amp;#039;t get done in time specified.  Just cooked it a little longer though and it was perfect.  Thanks Chef.</td>
      <td>5.0</td>
      <td>194.8</td>
      <td>0.20</td>
      <td>0.06</td>
      <td>0.32</td>
      <td>0.22</td>
      <td>0.36</td>
      <td>0.03</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 24 columns</p>

<iframe src="assets/datahead.html" width=1000 height=600 frameBorder=0></iframe>





# Assessment of Missingness

<!-- to markdown -->
description    0.052403
rating         5.944745
review         0.022171
avg_rating     1.068220
dtype: float64
<!-- end -->

This summary shows us that of the relevant columns we are still working with, there are 4 columns with missing data. 3 are from the original dataset but we added `'avg_rating'`. We will look deeper into `'rating'` and `'description'`. But we should notice that none of them have many datapoints missing.

First we can quickly explain `'avg_rating'`

Looking at the other Columns with missing data, we know avg_rating was calculated by aggregating the ratings for each recipe, so any missingness here is __Missing by Design__ based on missingness in the rating column. We can perfectly and exactly determine the missingness of `'avg_rating'` by checking if any interactions are missing a `'rating'` for each recipe (grouping by `'id'`). 

#### NMAR Analysis

We do not believe that the missingness mechanism for either is not missing at random, or NMAR. That would mean that some quality of the missing data (the value itself) determines the probability of its missingness. 

- For `'description'` there is no obvious or reasonable quality such as sentiment or length that would relate to its missingness. 

- `'Rating'` is more complicated. Users interacting with a recipe can either leave a review with a rating, or just a comment with no rating. This means some users may be commmenting without having even tried the recipe, that means for that interaction no rating value exists or should exist. This rules out NMAR.

Here is an example of an interaction without a rating, it happens to be for someone who tried the recipe and loved it. We can imagine this is also likely for someone who hasn't tried it or tried it and did not like it. 

<img src="assets/NoRating.png" height=5>

![example of missing rating](assets/NoRating.png)

#### Missingness Dependency

Neither `'description'` nor `'rating'` is closely related to the guiding question of unhealthy foods during the holiday season. However we will still investigate if they are missing dependent on another column (MAR) or if they are missing completely at random (MCAR). 

First we will explore missingness in `'description'`:

We believe it is plausible that if a `'description'` is missing then it is likely the author did not put in much effort to their post and the recipe will not be as good. This line of thinking would mean the `'avg_rating'` on that recipe would probably be lower for those that are missing. That would make `'description'` MAR on `'avg_rating'`. We can test this hypothesis with a permutation test. 

**Avg_rating**

Perform Permutation Test to see if missingness of description column is conditional on `'avg_rating'`. We will end up repeating this process for many columns following.


Description is unique to a recipe but duplicated for each interaction, we want to know how it relates to the recipes so we must retrieve that data again by grouping

need to work with only the non-null rows of the data in avg_rating otherwise no statistic could be computed

#### Choosing a Test Statistic

`'avg_rating'` is a quantitative variable which gives us two main options. We should use the difference of means for missing and not missing `'description'` if the distribution of `'avg_rating'` is roughly the same shape but shifted. If the shape is different and distributions are centered in the same place we should use the Kolmogorov-Smirnov statistic. 

plot distribution of Avg_rating based on missingness of description



As we can see the distributions are centered at roughly the same location but their shapes are pretty different. This means we will use the Kolmogorov-Smirnov test statistic and run a permutation test to see if `'description'` is MAR on `'avg_rating'`.

* Permutation Test for MAR of Description on `'avg_rating'`
* Test Statistic: Kolmogorov-Smirnov
* Alpha level: 5%

z-score:  3.3333333333333335
p-value:  0.9380583291749669
Reject Null:  False

##### Interpret Results

We can interpret the results of the Kolmogorov-Smirnov permutation test as follows. Under the assumption that `'description'` is unrelated to the distribution of `'avg_rating'`, the chance of seeing distributions as, or more, different than our two observed `'avg_rating'` distributions is 93.8%.

This is above our alpha threshold of 5% so we **fail to reject the null**. This means that the missingness of `'description'` is unlikely to be related and or condional to the distribution of `'avg_rating'` and that we have no evidence to say that it does (we also have no evidence to say it is unrelated for sure).

This would mean `'description'` is not MAR upon `'avg_rating'`.

This also indicates that our intuition was incorrect, lets test another column, `'user_id'`.

**User_id**

User_id is a categorical variable which means that we should use the total variation distance for a test statistic. This is calculated as the sum of the differences by category, divided by 2. It describes the distance between two categorical distributions. 

* Permutation Test for MAR of Description on `'User_id'`
* Test Statistic: TVD
* Alpha level: 5%


We will calculate many TVDs, one for each shuffle of `'user_id'`, this is done because under the null the missingness of `'description'` does not depend on `'user_id'` so the pairings are arbitrary. We will then compare our observed TVD to these. 

The observed test statistic is the TVD calculated on the actual sample from the data without any shuffling.

<!-- plot -->

The graph gives an idea of how usual or abnormal our observed TVD is compared to permutations under the null. It shows a histogram of all the TVDs when `'user_id'` doesn't influence `'description'` missingness. We get an exact measure of how many TVDs are as or more extreme when we calculate the p-value next:

##### Interpret Results

We can interpret the results of the TVD permutation test as follows. Under the assumption that `'description'` is unrelated to the distribution of `'user_id'`, the chance of seeing distributions as, or more, different than our two observed `'user_id'` distributions is `f"{p_value}"`%.

This is above our alpha threshold of 5% so we **fail to reject the null**. This means that the missingness of `'description'` is not likely to be related and or condional on `'user_id'`.

This would mean `'description'` is not MAR upon `'user_id'`.

Lets try another, there might be one or more that are dependent, if we find even one that is MAR we would consider the missingness in `'description'` to be MAR, if we _fail_ to reject the null for every column then we default to MCAR. Lets test `'calories'` against `'description'`.

**Calories**

Need to choose a test statistic, so need to see if the distribtutions of `'calories'` with and without `'description_missingness'` are shifted versions (diff of means) or same center and diff shape (Kolmogorov-Smirnov)

We see that the means are closely aligned, The shapes are also pretty similar but the height of the curve and the level it dips are different so we will try Kolmogorov-Smirnov statistic.

z-score:  202.3
p-value:  0.5062467217248188
Reject Null:  False

That did not result in significant results, we **fail to reject the null**. Before we interpret and conclude, lets test the difference in means as well. 



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


z-score:  0.37
p-value:  2.7111280133990385e-06
Reject Null:  True


We got another significant result for MAR on `'rating'`. Lets see one more column unrelated to the original `'nutrition'` column to see if it is also MAR.

**name**

* Permutation Test for missingness of Rating on `'name'`
* Test Statistic: TVD
* Alpha level: 5%

The observed test statistic is the TVD calculated on the actual sample from the data without any shuffling.


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


We notive that there is a dip followed by a spike in the last three years, this data could be more variable since as we saw in the EDA there are much fewer recipes interacted with in the last few years. For this reason we will continue with the hypothesis test on all of the data together regardless of year. 

_Now we perform the hypothesis test:_

Just like with the permutation tests for missingness earlier, we have a distribution of test statistics, in this case proportions of unbalanced recipes. We can compare the proportion of unbalanced recipes from only the Holiday months which we are interested in compared to the general 


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


