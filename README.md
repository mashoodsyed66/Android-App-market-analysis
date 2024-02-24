
<center>
    
# Google Play Store apps and reviews

</center>


## 1. Introduction
<p>Mobile apps are everywhere. They are easy to create and can be lucrative. Because of these two factors, more and more apps are being developed. In this notebook, we will do a comprehensive analysis of the Android app market by comparing over ten thousand apps in Google Play across different categories. We'll look for insights in the data to devise strategies to drive growth and retention.</p>

![Google-Play-Logo-700x394](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/21889f5a-7cd9-4f67-9664-faf97465c7f3)

<p>Let's take a look at the data, which consists of two files:</p>
<ul>
<li><code>apps.csv</code>: contains all the details of the applications on Google Play. There are 13 features that describe a given app.

**Features:**  App, Category, Rating, Reviews, Size, Installs, Price, Content Rating, Genres, Last Updated, Current Ver, Android Ver</li>
<li><code>user_reviews.csv</code>: contains 100 reviews for each app, <a href="https://www.androidpolice.com/2019/01/21/google-play-stores-redesigned-ratings-and-reviews-section-lets-you-easily-filter-by-star-rating/">most helpful first</a>. The text in each review has been pre-processed and attributed with three new features: Sentiment (Positive, Negative or Neutral), Sentiment Polarity and Sentiment Subjectivity.</li>
</ul>

|   Unnamed: 0 | App                             | Category     |   Rating |   Reviews |   Size | Installs    | Type   | Price   | Content Rating   | Genres         | Last Updated      | Current Ver       | Android Ver   |
|--------------|---------------------------------|--------------|----------|-----------|--------|-------------|--------|---------|------------------|----------------|-------------------|-------------------|---------------|
|         2093 | OLX - Buy and Sell              | SHOPPING     |      4.2 |    857923 |     18 | 50,000,000+ | Free   | 0       | Everyone         | Shopping       | July 31, 2018     | 11.7.3.0          | 4.1 and up   |
|         2208 | InstaBeauty -Makeup Selfie Cam | PHOTOGRAPHY  |      4.3 |    654419 | NaN    | 50,000,000+ | Free   | 0       | Everyone         | Photography    | February 1, 2018  | Varies with device| 4.0.3 and up |
|         6800 | CT Abdomen Pelvis               | MEDICAL      |      4   |        36 |     36 | 10,000+     | Free   | 0       | Everyone         | Medical        | January 20, 2018  | 2                 | 4.0 and up   |
|         3505 | Guns'n'Glory Heroes Premium     | FAMILY       |      4.3 |       923 | NaN    | 10,000+     | Paid   | $2.99   | Everyone         | Strategy       | November 9, 2017  | Varies with device| Varies with device|
|         6310 | DRAGON QUEST IV                 | FAMILY       |      4.7 |      1647 |     16 | 10,000+     | Paid   | $14.99  | Everyone         | Role Playing   | February 28, 2017 | 1.0.5             | 2.3 and up   |


## 2. Data cleaning
<p>Data cleaning is one of the most essential subtask any data science project. Although it can be a very tedious process, it's worth should never be undermined.</p>
<p>By looking at a random sample of the dataset rows (from the above task), we observe that some entries in the columns like <code>Installs</code> and <code>Price</code> have a few special characters (<code>+</code> <code>,</code> <code>$</code>) due to the way the numbers have been represented. This prevents the columns from being purely numeric, making it difficult to use them in subsequent future mathematical calculations. Ideally, as their names suggest, we would want these columns to contain only digits from [0-9].</p>
<p>Hence, we now proceed to clean our data. Specifically, the special characters <code>,</code> and <code>+</code> present in <code>Installs</code> column and <code>$</code> present in <code>Price</code> column need to be removed.</p>
<p>It is also always a good practice to print a summary of your dataframe after completing data cleaning. We will use the <code>info()</code> method to acheive this.</p>

```python
chars_to_remove = ['+', ',', '$']
cols_to_clean = ['Installs', 'Price']

for col in cols_to_clean:
    for char in chars_to_remove:
        apps[col] = apps[col].apply(lambda x: x.replace(char, ''))

print(apps.info())

```

![2 appsinfo](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/de5e63ea-b595-4dfb-8cff-d8b3aa30b5bb)

From below image we can see that the unwanted characters have been removed and the data have been cleaned.

![2](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/a60846bc-a91e-4ef6-8ce7-fef6da5f3ea9)

## 3. Correcting data types
<p>From the previous task we noticed that <code>Installs</code> and <code>Price</code> were categorized as <code>object</code> data type (and not <code>int</code> or <code>float</code>) as we would like. This is because these two columns originally had mixed input types: digits and special characters. To know more about Pandas data types, read <a href="https://datacarpentry.org/python-ecology-lesson/04-data-types-and-format/">this</a>.</p>
<p>The four features that we will be working with most frequently henceforth are <code>Installs</code>, <code>Size</code>, <code>Rating</code> and <code>Price</code>. While <code>Size</code> and <code>Rating</code> are both <code>float</code> (i.e. purely numerical data types), we still need to work on <code>Installs</code> and <code>Price</code> to make them numeric.</p>


 ```python
import numpy as np

# Convert Installs to float data type
apps['Installs'] = apps['Installs'].astype(float)

# Convert Price to float data type
apps['Price'] = apps['Price'].astype(float)

# Checking dtypes of the apps dataframe
#print(apps.dtypes)
apps.head()
```
![3](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/ee9fd2e8-e499-43c7-a041-c93664da4ca0)

## 4. Exploring app categories
<p>With more than 1 billion active users in 190 countries around the world, Google Play continues to be an important distribution platform to build a global audience. For businesses to get their apps in front of users, it's important to make them more quickly and easily discoverable on Google Play. To improve the overall search experience, Google has introduced the concept of grouping apps into categories.</p>
<p>This brings us to the following questions:</p>
<ul>
<li>Which category has the highest share of (active) apps in the market? </li>
<li>Is any specific category dominating the market?</li>
<li>Which categories have the fewest number of apps?</li>
</ul>
<p>We will see that there are <code>33</code> unique app categories present in our dataset. <em>Family</em> and <em>Game</em> apps have the highest market prevalence. Interestingly, <em>Tools</em>, <em>Business</em> and <em>Medical</em> apps are also at the top.</p>

```python
import plotly
plotly.offline.init_notebook_mode(connected=True)
import plotly.graph_objs as go

num_categories = apps.Category.nunique()

num_apps_in_category = apps['Category'].value_counts()

sorted_num_apps_in_category = num_apps_in_category.sort_values(ascending=False)

data = [go.Bar(
        x=num_apps_in_category.index,
        y=num_apps_in_category.values,
)]

plotly.offline.iplot(data)
```

![4](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/a162e6c8-5028-4e0a-a212-408e721df9e7)


## 5. Distribution of app ratings
<p>After having witnessed the market share for each category of apps, let's see how all these apps perform on an average. App ratings (on a scale of 1 to 5) impact the discoverability, conversion of apps as well as the company's overall brand image. Ratings are a key performance indicator of an app.</p>
<p>From our research, we found that the average volume of ratings across all app categories is <code>4.17</code>. The histogram plot is skewed to the left indicating that the majority of the apps are highly rated with only a few exceptions in the low-rated apps.</p>

```python
avg_app_rating = apps['Rating'].mean()

data = [go.Histogram(
        x=apps['Rating']
)]

layout = {'shapes': [{
              'type': 'line',
              'x0': avg_app_rating,
              'y0': 0,
              'x1': avg_app_rating,
              'y1': 1000,
              'line': {'dash': 'dashdot'}
          }]
          }

plotly.offline.iplot({'data': data, 'layout': layout})
```
![5](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/29440bfd-b358-41ef-ba9b-9ab921f9ed88)


## 6. Size and price of an app
<p>Let's now examine app size and app price. For size, if the mobile app is too large, it may be difficult and/or expensive for users to download. Lengthy download times could turn users off before they even experience your mobile app. Plus, each user's device has a finite amount of disk space. For price, some users expect their apps to be free or inexpensive. These problems compound if the developing world is part of your target market; especially due to internet speeds, earning power and exchange rates.</p>
<p>How can we effectively come up with strategies to size and price our app?</p>
<ul>
<li>Does the size of an app affect its rating? </li>
<li>Do users really care about system-heavy apps or do they prefer light-weighted apps? </li>
<li>Does the price of an app affect its rating? </li>
<li>Do users always prefer free apps over paid apps?</li>
</ul>
<p>We find that the majority of top rated apps (rating over 4) range from 2 MB to 20 MB. We also find that the vast majority of apps price themselves under \$10.</p>

```python
%matplotlib inline
import seaborn as sns
import matplotlib.pyplot as plt
import warnings

warnings.filterwarnings("ignore")

apps_with_size_and_rating_present = apps.dropna(subset=['Rating', 'Size'])

large_categories = apps_with_size_and_rating_present.groupby('Category').filter(lambda x: len(x) >= 250)

plt1 = sns.jointplot(x=large_categories['Size'], y=large_categories['Rating'])

paid_apps = apps_with_size_and_rating_present[apps_with_size_and_rating_present['Type'] == 'Paid']

plt2 = sns.jointplot(x=paid_apps['Price'], y=paid_apps['Rating'])
```

![output_11_0](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/1f7e0216-700e-465e-9de6-4cc2bd8246c5)
![output_11_1](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/fc68ce62-9410-4cad-86b0-6484c25d5425)


## 7. Relation between app category and app price
<p>So now comes the hard part. How are companies and developers supposed to make ends meet? What monetization strategies can companies use to maximize profit? The costs of apps are largely based on features, complexity, and platform.</p>
<p>There are many factors to consider when selecting the right pricing strategy for your mobile app. It is important to consider the willingness of your customer to pay for your app. A wrong price could break the deal before the download even happens. Potential customers could be turned off by what they perceive to be a shocking cost, or they might delete an app they’ve downloaded after receiving too many ads or simply not getting their money's worth.</p>
<p>Different categories demand different price ranges. Some apps that are simple and used daily, like the calculator app, should probably be kept free. However, it would make sense to charge for a highly-specialized medical app that diagnoses diabetic patients. Below, we see that <em>Medical and Family</em> apps are the most expensive. Some medical apps extend even up to \$80! All game apps are reasonably priced below \$20.</p>

```python
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
fig.set_size_inches(15, 8)

popular_app_cats = apps[apps.Category.isin(['GAME', 'FAMILY', 'PHOTOGRAPHY',
                                            'MEDICAL', 'TOOLS', 'FINANCE',
                                            'LIFESTYLE','BUSINESS'])]
ax = sns.stripplot(x = popular_app_cats['Price'], y = popular_app_cats['Category'], jitter=True, linewidth=1)
ax.set_title('App pricing trend across categories')

apps_above_200 = apps[apps['Price']>200]
apps_above_200[['Category', 'App', 'Price']]
```
![output_13_1](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/87a6d913-eb74-4e47-b2ac-26ecacb1eabd)


## 8. Filter out "junk" apps
<p>It looks like a bunch of the really expensive apps are "junk" apps. That is, apps that don't really have a purpose. Some app developer may create an app called <em>I Am Rich Premium</em> or <em>most expensive app (H)</em> just for a joke or to test their app development skills. Some developers even do this with malicious intent and try to make money by hoping people accidentally click purchase on their app in the store.</p>
<p>Let's filter out these junk apps and re-do our visualization.</p>

```python
apps_under_100 = popular_app_cats[popular_app_cats['Price'] < 100]

fig, ax = plt.subplots()
fig.set_size_inches(15, 8)

ax = sns.stripplot(x = 'Price', y = 'Category', data = apps_under_100, jitter = True, linewidth = 1)
ax.set_title('App pricing trend across categories after filtering for junk apps')
```
![output_15_1](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/70e46451-5df0-4267-b72e-1aa19c3eb19e)


## 9. Popularity of paid apps vs free apps

<p>For apps in the Play Store today, there are five types of pricing strategies: free, freemium, paid, paymium, and subscription. Let's focus on free and paid apps only. Some characteristics of free apps are:</p>
<ul>
<li>Free to download.</li>
<li>Main source of income often comes from advertisements.</li>
<li>Often created by companies that have other products and the app serves as an extension of those products.</li>
<li>Can serve as a tool for customer retention, communication, and customer service.</li>
</ul>
<p>Some characteristics of paid apps are:</p>
<ul>
<li>Users are asked to pay once for the app to download and use it.</li>
<li>The user can't really get a feel for the app before buying it.</li>
</ul>
<p>Are paid apps installed as much as free apps? It turns out that paid apps have a relatively lower number of installs than free apps, though the difference is not as stark as I would have expected!</p>

```python
trace0 = go.Box(
    y=apps[apps['Type'] == 'Paid']['Installs'],
    name='Paid'
)
trace1 = go.Box(
    y=apps[apps['Type'] == 'Free']['Installs'],
    name='Free'
)
layout = go.Layout(
    title="Number of downloads of paid apps vs. free apps",
    yaxis=dict(title="Log number of downloads",
               type='log',
               autorange=True)
)
data = [trace0, trace1]
plotly.offline.iplot({'data': data, 'layout': layout})
```
![9](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/a781bdee-2059-402b-8894-95b3d5fd77fd)

## 10. Sentiment analysis of user reviews
<p>Mining user review data to determine how people feel about your product, brand, or service can be done using a technique called sentiment analysis. User reviews for apps can be analyzed to identify if the mood is positive, negative or neutral about that app. For example, positive words in an app review might include words such as 'amazing', 'friendly', 'good', 'great', and 'love'. Negative words might be words like 'malware', 'hate', 'problem', 'refund', and 'incompetent'.</p>
<p>By plotting sentiment polarity scores of user reviews for paid and free apps, we observe that free apps receive a lot of harsh comments, as indicated by the outliers on the negative y-axis. Reviews for paid apps appear never to be extremely negative. This may indicate something about app quality, i.e., paid apps being of higher quality than free apps on average. The median polarity score for paid apps is a little higher than free apps, thereby syncing with our previous observation.</p>
<p>In this notebook, we analyzed over ten thousand apps from the Google Play Store. We can use our findings to inform our decisions should we ever wish to create an app ourselves.</p>

```python
reviews_df = pd.read_csv('datasets/user_reviews.csv')
merged_df = apps.merge(reviews_df, on='App')
merged_df = merged_df.dropna(subset=['Sentiment', 'Review'])

sns.set_style('ticks')
fig, ax = plt.subplots()
fig.set_size_inches(11, 8)

ax = sns.boxplot(x='Type', y='Sentiment_Polarity', data=merged_df)
ax.set_title('Sentiment Polarity Distribution')
```

![output_19_1](https://github.com/mashoodsyed66/Android-App-market-analysis/assets/65015378/0e8d6a03-e0f7-432a-8650-c165b696989e)

<br> </br>

# Conclusion

After conducting a comprehensive analysis of the Google Play Store apps and reviews, several key insights have emerged:

### 1. App Categories:
- The Android app market is diverse, with over 33 unique categories represented in our dataset.
- Categories such as Family, Game, and Tools dominate the market, indicating high user demand in these areas.

### 2. App Size and Price:
- App size and price influence user engagement and adoption.
- Most top-rated apps have sizes ranging from 2 MB to 20 MB, indicating a preference for moderately sized applications.
- While some categories, like Medical and Family, have higher-priced apps, games are typically priced below $20.

### 3. Paid vs. Free Apps:
- Paid apps have fewer installations compared to free apps, but they tend to receive less negative feedback in user reviews.
- Free apps, on the other hand, may attract harsh criticism due to their availability and widespread usage.

### 4. User Sentiment Analysis:
- Sentiment analysis of user reviews reveals that free apps often receive more negative comments compared to paid apps.
- This suggests a potential correlation between app quality and price, with paid apps generally being of higher quality.

In conclusion, our analysis underscores the importance of understanding user preferences, market trends, and pricing strategies in the competitive landscape of the Android app market. By leveraging these insights, developers and marketers can make informed decisions to optimize their apps for success on the Google Play Store.
