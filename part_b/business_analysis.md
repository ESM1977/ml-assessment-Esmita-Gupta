# B1. Problem Formulation

## (a) Machine Learning Formulation

### Target Variable
The **target variable** is **`items_sold`**, which represents the number of items sold in a store during a given month under a particular promotion.

### Candidate Input Features
Possible input features include:

- **promotion_type**  
  (Flat Discount, BOGO, Free Gift with Purchase, Category-Specific Offer, Loyalty Points Bonus)

- **store characteristics**
  - store size
  - store location type (urban, semi-urban, rural)

- **customer and market factors**
  - monthly footfall
  - local competition density
  - customer demographics

- **time-related features**
  - month
  - season
  - festive period indicator
  - year

- **historical performance features**
  - previous month sales
  - previous promotion performance
  - average basket size
  - repeat customer rate

### Type of Machine Learning Problem
This is a **supervised machine learning regression problem**.

### Justification
It is a **supervised** problem because we have historical data with known inputs (promotion type, store details, market conditions, time features, etc.) and a known output (`items_sold`).

It is a **regression** problem because the target variable is a **continuous numeric value**. The goal is to predict how many items will be sold, not to assign the store to a category or class.

So, the business problem can be framed as:

> **Predict the number of items sold for each store-month-promotion combination, and then choose the promotion expected to produce the highest sales.**

## (b) Why `items_sold` is a Better Target than Total Sales Revenue

Using **`items_sold`** is more reliable than using **total sales revenue** for this problem because the company wants to measure the **effectiveness of promotions in increasing sales volume**.

### Why sales volume is better
- **Revenue can be affected by price changes**  
  A promotion such as a flat discount may increase the number of items purchased but reduce the revenue earned per item. In that case, revenue may look lower even though the promotion was successful in driving customer purchases.

- **Revenue mixes price effect and promotion effect**  
  Total sales revenue depends on both:
  - how many items were sold, and
  - at what price they were sold.  
  This makes it harder to isolate the true impact of the promotion.

- **Items sold directly matches the business question**  
  The company wants to know which promotion should be deployed to **maximize the number of items sold**. So `items_sold` is more closely aligned with the decision they want to make.

### Broader principle illustrated
This illustrates an important principle in real-world ML projects:

> **The target variable should closely match the real business objective and should measure the outcome we actually want to optimize.**

If the wrong target is chosen, the model may learn patterns that are mathematically correct but not useful for decision-making. Therefore, target selection should always reflect the **true goal of the business problem**, not just the easiest or most commonly available metric.

## (c) Alternative Modelling Strategy

Instead of using one single global model for all 50 stores, a better approach would be a **segmented or hierarchical modelling strategy**.

### Proposed approach
The stores can first be grouped based on important differences such as:

- urban, semi-urban, and rural location
- store size
- customer profile
- competition density

Then, the company can either:
- build **separate models for each store segment**, or
- use a **hierarchical model / mixed-effects approach** that captures both overall patterns and location-specific differences.

### Justification
Stores in different locations may respond differently to the same promotion. For example:
- a **BOGO** offer may work better in urban high-footfall stores,
- while a **flat discount** may be more effective in rural or price-sensitive markets.

A single global model may average these differences and miss important local patterns. As a result, its recommendations may be too general and less accurate.

A segmented or hierarchical strategy is better because it:
- captures differences in promotion response across store types,
- gives more tailored predictions,
- and leads to better promotion decisions for each group of stores.

### Conclusion
So, instead of one global model, the retailer should use a **location-aware modelling strategy** that reflects how different store groups behave differently under promotions.

# B2. Data and EDA Strategy

## (a) Joining the Four Tables and Defining the Final Modelling Dataset

To build a useful modelling dataset, the four raw tables should be joined in a structured way so that each row contains all information needed to predict promotion performance.

### Step 1: Start with the transactions table
The **transactions** table is the main fact table because it contains the actual sales activity. It will usually include fields such as:

- transaction date
- store ID
- promotion ID or promotion type
- items sold
- revenue
- customer purchases

### Step 2: Join store attributes
Join the **store attributes** table to the transactions table using:

- **store_id**

This adds store-level information such as:

- location type
- store size
- footfall
- competition density
- customer demographic indicators

### Step 3: Join promotion details
Join the **promotion details** table using:

- **promotion_id**  
  or, if no ID exists, using **promotion_type**

This adds information such as:

- promotion category
- discount structure
- campaign rules
- duration of the promotion

### Step 4: Join calendar table
Join the **calendar** table using:

- **transaction_date**

This adds time-related information such as:

- month
- weekday/weekend flag
- festival flag
- holiday season indicator
- other seasonal markers

## Grain of the Final Modelling Dataset

The final modelling dataset should have the grain:

> **One row = one store × one month × one promotion**

This grain is appropriate because the business decision is:

> which promotion should be deployed in each store each month to maximise items sold.

So each row should represent the outcome of running a specific promotion in a specific store during a specific month.

## Aggregations to Perform Before Modelling

Because transactions are usually recorded at a much lower level (for example, one row per customer purchase or one row per bill), they must be aggregated before modelling.

### Key aggregations
For each **store-month-promotion** combination, I would aggregate:

- **total items sold** → target variable
- **total sales revenue**
- **number of transactions**
- **average basket size**
- **average selling price per item**
- **total footfall** or average footfall for that month
- **promotion duration** within the month, if relevant
- **number of weekend days**
- **number of festival days**
- **share of sales from repeat customers**, if available

### Time-based derived features
I would also create features such as:

- month
- quarter
- season
- whether the month included a major festival
- previous month’s items sold
- previous promotion performance for that store

## Why this structure is useful

This approach ensures that all relevant information from stores, promotions, and the calendar is available in one dataset at the same decision level.

It also avoids mixing different levels of data. Instead of modelling raw transaction rows, we model the exact business unit that the company wants to optimise: **promotion performance by store and month**.
## (b) EDA to Perform Before Building the Model

Before building a model, I would perform exploratory data analysis (EDA) to understand the data, identify patterns, detect problems, and guide feature engineering.

Below are some important analyses and charts I would use.

### 1. Distribution of the target variable (`items_sold`)
**Chart:** Histogram or boxplot of `items_sold`

**What I would look for:**
- whether the target is highly skewed
- whether there are extreme outliers
- whether there are many zero or very low sales months

**How this would influence modelling:**
- If `items_sold` is highly skewed, I may consider a log transformation.
- If there are many outliers, I may use models that are more robust to extreme values.
- If there are many zero-sales rows, I would check whether those are valid business cases or data quality issues.

### 2. Promotion-wise sales comparison
**Chart:** Boxplot or bar chart of `items_sold` by `promotion_type`

**What I would look for:**
- which promotions tend to generate higher or lower sales
- whether the effect of promotions varies widely
- whether some promotions show high variability across stores

**How this would influence modelling:**
- If promotion type clearly affects sales, it confirms that this is an important feature.
- If some promotions behave differently across contexts, I may include interaction features such as:
  - promotion × location type
  - promotion × store size
- It may also support segmented modelling.

### 3. Sales by store type or location
**Chart:** Boxplot or grouped bar chart of `items_sold` by urban / semi-urban / rural location and by store size

**What I would look for:**
- whether certain store types consistently sell more items
- whether promotion response differs across location groups
- whether store size strongly affects outcomes

**How this would influence modelling:**
- If store characteristics strongly affect sales, they must be included as features.
- If response patterns differ a lot, I may build segment-specific models or hierarchical models instead of one global model.
- I may also engineer interaction terms such as:
  - promotion × location
  - promotion × store size

### 4. Time-series trend analysis
**Chart:** Line chart of monthly total `items_sold` over time

**What I would look for:**
- trend over time
- seasonality
- festival or holiday spikes
- sudden changes caused by external factors

**How this would influence modelling:**
- If there is seasonality, I would create time-based features such as:
  - month
  - quarter
  - festive season flag
  - weekend share
- It also confirms that the train-test split should be time-based, not random.

### 5. Correlation analysis for numerical features
**Chart:** Correlation heatmap

**What I would look for:**
- relationships between numerical variables and `items_sold`
- multicollinearity between predictors
- whether some features are highly redundant

**How this would influence modelling:**
- Strongly related features may be useful predictors.
- Highly correlated inputs may create problems for linear models, so I may remove or combine some variables.
- It helps decide whether simpler models are enough or whether more flexible models are needed.

### 6. Missing values and data quality checks
**Analysis:** Missing value table, invalid value checks, duplicate checks

**What I would look for:**
- missing values in key fields
- impossible values such as negative sales or negative footfall
- duplicate records
- inconsistent promotion labels

**How this would influence modelling:**
- Missing values may require imputation or removal.
- Bad data may need cleaning before modelling.
- Inconsistent categories would need standardisation before encoding.

## (c) Effect of Promotion Imbalance and How to Address It

If **80% of transactions happened without any promotion**, the dataset is imbalanced. This can affect the model because it may learn the pattern of **non-promotion periods** much better than the pattern of **promotion periods**.

### How this imbalance could affect the model
- The model may become biased toward the majority case, which is **no promotion**.
- It may underestimate the true impact of promotions because it has fewer examples of promotional behaviour to learn from.
- Predictions for promoted periods may be less accurate and less reliable.
- The model may recommend conservative decisions simply because it has seen many more non-promotion cases.

### Steps to address it
- **Include a promotion indicator feature**  
  Create a variable showing whether a promotion was running or not, so the model can clearly separate promotion and non-promotion periods.

- **Use balanced sampling for training analysis**  
  For some analyses, I would compare promoted and non-promoted periods more carefully, and possibly use undersampling of non-promotion rows or oversampling of promotion periods if appropriate.

- **Evaluate performance separately**  
  I would check model performance separately for:
  - promotion periods
  - non-promotion periods  
  This ensures the model is not performing well only on the majority group.

- **Create promotion-specific features**  
  Add features such as promotion type, promotion intensity, promotion duration, and interactions with store/location variables so the model can better learn promotional effects.

- **Consider causal or uplift-style analysis**  
  If the real goal is to measure the effect of promotions, I may also consider methods that compare outcomes with and without promotion more directly, rather than relying only on a standard prediction model.

### Conclusion
This imbalance can make the model less sensitive to promotional effects. So, I would handle it by improving feature design, checking subgroup performance, and ensuring the model learns enough from the smaller but important promotion cases.

# B3. Model Evaluation and Deployment

## (a) Train-Test Split, Why Random Split Is Inappropriate, and Evaluation Metrics

### Train-Test Split Strategy

Since the data is **monthly store-level data over three years**, I would use a **time-based split**, not a random split.

A good setup would be:

- **Training set:** first 2 to 2.5 years of data
- **Validation set:** next few months for tuning and model selection
- **Test set:** most recent months (for example, the last 6 months)

So the model is always trained on the **past** and tested on the **future**.

Because the data comes from **50 stores**, I would make sure that all stores are represented across the time periods, but the split should still respect time order.

A stronger approach would be **rolling or walk-forward validation**, for example:

- Train on Year 1 → validate on early Year 2
- Train on Year 1 + early Year 2 → validate on later Year 2
- Train on first 2 years → test on Year 3

This gives a more realistic view of how the model performs over time.

### Why a Random Split Is Inappropriate

A random split is not suitable because this is **time-ordered data**.

If we randomly split the rows:

- the training data may include future months,
- while the test data may include earlier months.

This creates **data leakage**, because the model indirectly learns patterns from the future that would not be available in real use.

It also ignores the fact that retail sales are affected by:

- seasonality,
- trends,
- changing customer behaviour,
- changing promotion strategies,
- and market conditions over time.

In the real business setting, the company will use past data to predict future promotion performance. So the evaluation setup should match that real decision process.

### Evaluation Metrics

Because the task is to predict **items sold**, this is a **regression problem**. I would mainly use the following metrics:

#### 1. MAE (Mean Absolute Error)
MAE shows the average absolute difference between predicted and actual items sold.

**Interpretation in business terms:**  
If MAE = 20, then on average the model is off by about **20 items sold per store-month**.

**Why useful:**  
It is easy for business stakeholders to understand and gives a direct sense of average prediction error.

#### 2. RMSE (Root Mean Squared Error)
RMSE measures prediction error like MAE, but it gives **more penalty to larger mistakes**.

**Interpretation in business terms:**  
A high RMSE means the model sometimes makes large errors, such as badly overestimating or underestimating sales for certain promotions or stores.

**Why useful:**  
This matters because large mistakes can lead to poor promotion decisions, stock issues, or lost sales opportunities.

#### 3. MAPE (Mean Absolute Percentage Error) or WAPE
This metric measures error as a percentage of actual sales.

**Interpretation in business terms:**  
It tells us how large the prediction error is relative to the actual number of items sold.

**Why useful:**  
It helps compare model performance across stores of different sizes.  
For example, an error of 20 items is much more serious for a small store than for a large store.

**Note:**  
MAPE can be problematic if actual sales are zero or very small, so **WAPE** may sometimes be more stable.

#### 4. R-squared (optional supporting metric)
R-squared shows how much of the variation in `items_sold` is explained by the model.

**Interpretation in business terms:**  
A higher R-squared means the model captures more of the sales pattern across stores, months, and promotions.

**Why useful:**  
It is helpful as a summary measure, but it should not be used alone because it does not directly show business error size.

### How I Would Use the Metrics Together

I would not rely on only one metric.

- **MAE** tells me the average size of the error in units the business understands.
- **RMSE** tells me whether the model makes some very large mistakes.
- **MAPE/WAPE** tells me how serious the errors are relative to store sales volume.
- **R-squared** gives a general view of model fit.

For this business problem, **MAE and RMSE would be the most important**, because the retailer needs reliable sales predictions to choose the best promotion for each store and month.

### Conclusion

The correct evaluation approach is a **time-based split with future data held out for testing**, ideally supported by rolling validation. A random split is inappropriate because it causes leakage and gives an unrealistically optimistic estimate of performance.

The most useful metrics are **MAE, RMSE, and percentage-based error metrics**, because they show how prediction errors translate into real business impact when deciding which promotion to deploy.

## (b) Using Feature Importance to Explain Different Recommendations for the Same Store

If the model recommends **Loyalty Points Bonus for Store 12 in December** and **Flat Discount for Store 12 in March**, I would explain that the model is not only looking at the store itself. It is also using **time-related, promotional, and contextual features** that change from month to month.

### How I would investigate this

#### 1. Compare the input features for Store 12 in December vs March
I would place the two store-month records side by side and compare features such as:

- month / season
- festival or holiday flag
- weekend intensity
- expected footfall
- past sales trend
- customer mix
- local competition conditions
- promotion type effects from historical data

This helps identify what changed between December and March.

---

#### 2. Check the most important features in the model
Using **feature importance** from the trained model, I would identify which variables most strongly influence predictions overall.

For example, the model may show that important features are:

- promotion type
- month
- festival flag
- store footfall
- store location type
- previous month sales

If **month** and **festival-related variables** are highly important, it suggests that the best promotion depends not only on the store, but also on the seasonal context.

---

#### 3. Use local explanation for this specific prediction
Global feature importance tells us what matters overall, but for one specific recommendation I would also use a **local explanation method** such as:

- SHAP values, or
- a feature contribution breakdown

This would show why the model preferred one promotion in December and another in March for Store 12.

For example:
- In **December**, the model may see holiday shopping behaviour, higher gift-buying demand, and stronger response from loyal customers, making **Loyalty Points Bonus** more effective.
- In **March**, the model may detect lower seasonal demand or more price-sensitive behaviour, making **Flat Discount** a better way to increase items sold.

---

### How I would communicate this to the marketing team

I would explain it in simple business language:

> “The model is giving different recommendations because customer behaviour changes across months. Even for the same store, December and March are different business situations. December may have festive demand, repeat shoppers, and gift-oriented buying, while March may have more price-sensitive shopping. The model uses these changing factors to estimate which promotion is likely to sell the most items in each month.”

I would also support this with a small table like this:

| Feature | Store 12 - December | Store 12 - March | Business meaning |
|--------|----------------------|------------------|------------------|
| Month | December | March | Seasonal demand differs |
| Festival flag | Yes | No | Holiday effect present only in December |
| Expected footfall | Higher | Lower | More shoppers in December |
| Best predicted promotion | Loyalty Points Bonus | Flat Discount | Different customer response expected |

### Why this is useful

This explanation helps the marketing team understand that:

- the model is **not inconsistent**
- it is adapting recommendations based on **context**
- the best promotion depends on both **store characteristics** and **monthly conditions**

### Conclusion

Feature importance helps us show that the recommendation changes because the model considers more than just store identity. It also uses time, seasonality, demand conditions, and promotion response patterns. So, the same store can correctly receive different promotion recommendations in different months.

## (c) End-to-End Deployment Process

The model should be deployed as a **monthly decision-support system** that generates promotion recommendations for all 50 stores at the start of each month.

### 1. Save the trained model
After training, I would save the **entire pipeline**, not just the model.

This means saving:
- the preprocessing steps
- feature transformations
- the trained model itself

This can be done using tools such as:
- `joblib`
- `pickle`

Saving the full pipeline is important because new data must go through the exact same preprocessing steps as the training data.

**Example idea:**  
Save one packaged object such as:

- `promotion_recommendation_pipeline.pkl`

This ensures consistency between training and production use.

---

### 2. Prepare new monthly input data
At the start of every month, the system would collect the latest input data for all 50 stores, such as:

- store attributes
- planned month
- calendar features
- expected footfall
- competition indicators
- customer demographic information
- the candidate promotions to evaluate

To generate recommendations, I would create a **store-month-promotion scoring table**.

That means for each store, I would create one row for each possible promotion.

So if there are:
- **50 stores**
- **5 promotions**

then the monthly scoring dataset would contain:

- **50 × 5 = 250 rows**

Each row would represent:

> one store × one month × one candidate promotion

The same feature engineering steps used in training would then be applied:
- join store, calendar, and promotion tables
- create time features
- encode categories
- scale numerical variables through the saved pipeline

---

### 3. Feed the new data into the saved model
The saved pipeline would be loaded and used to predict expected `items_sold` for each row in the monthly scoring table.

So for each store, the model would produce predicted sales for all five promotion options.

Then the system would choose:

> the promotion with the highest predicted `items_sold`

for that store in that month.

The final output to the marketing team could be a table like:

| Store | Month | Promotion | Predicted Items Sold |
|------|------|-----------|----------------------|
| Store 1 | December | BOGO | 1240 |
| Store 2 | December | Loyalty Points Bonus | 980 |
| Store 3 | December | Flat Discount | 1115 |

This becomes the monthly recommendation file.

---

### 4. Operational workflow
A practical monthly deployment flow would be:

1. collect latest monthly data
2. validate data quality
3. create the store-month-promotion scoring dataset
4. load the saved model pipeline
5. generate predictions for all promotion options
6. select the best promotion for each store
7. send recommendations to the marketing team
8. store predictions for later evaluation

This process can be automated using a scheduled job or workflow tool.

---

### 5. Monitoring model performance
Even if the model is not retrained every month, it must still be monitored regularly.

Once actual monthly sales become available, I would compare:

- predicted `items_sold`
- actual `items_sold`

### Metrics to monitor
I would track:

- **MAE**
- **RMSE**
- **MAPE or WAPE**
- performance by:
  - store type
  - region
  - promotion type
  - month/season

This is important because the model may perform well overall but poorly for certain stores or promotions.

---

### 6. Drift and degradation monitoring
I would monitor two main kinds of change:

#### (a) Performance drift
This means prediction accuracy gets worse over time.

Signs:
- MAE or RMSE increasing month after month
- poor recommendation outcomes compared to past performance
- model no longer selecting promotions effectively

#### (b) Data drift
This means the input data distribution has changed from what the model saw during training.

Examples:
- footfall patterns shift
- customer behaviour changes
- competition density changes
- promotion response changes
- new seasonal patterns appear

I would monitor:
- summary statistics of key features
- category frequency changes
- feature distributions over time

---

### 7. When retraining is needed
Retraining should be triggered when:

- prediction error crosses a defined threshold
- drift is consistently detected
- new promotion strategies are introduced
- major business conditions change
- enough new data has accumulated

For example, retraining might happen:
- every quarter,
- every 6 months,
- or earlier if monitoring shows strong degradation.

---

### 8. Communication and governance
To make deployment reliable, I would also keep:

- model version number
- training data period
- feature list
- evaluation results
- retraining date
- business owner approval

This helps with transparency and reproducibility.

---

## Conclusion

The deployment process should use a **saved end-to-end pipeline**, monthly prepared scoring data for all store-promotion combinations, and an automated recommendation step that selects the promotion with the highest predicted sales for each store.

To ensure the model remains useful, I would monitor both **prediction performance** and **data drift**, and retrain the model when errors increase or business conditions change significantly.





