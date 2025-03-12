# Power Outage Impact Analysis

This is a project for DSC 80 at UCSD.

By Turki Alrasheed

---

# Introduction

This project analyzes major power outages in the US from January 2000 to July 2016 from a dataset found on Purdue University’s Laboratory for Advancing Sustainable Critical Infrastructure, at https://engineering.purdue.edu/LASCI/research-data/outages.

The dataset includes information about the major outages, more specifically general information(time and state), climate conditions, cause, extent, land-use characteristics, Electricity consumption information, and economic outputs.

I will focus on answering my research question: **What are the signs indicating that a power outage has a big “impact” on people’s lives?** This is important because power outages are events that can have significant economic, social, and infrastructural impacts. Learning the signs of an impactful power outage can help communities and utility companies better prepare for it and reduce its impact.

The original dataFrame contains 1534 rows(1534 outages), and 57 columns. However, I will use some of the columns for my analysis.

Here are the columns(24) I am interested in along with its descriptions:

|Column                |Description|
|---                |---        |
|`'YEAR'`                |Year when the outage event occurred|
|`'MONTH'`                |Month when the outage event occurred|
|`'U.S._STATE'`                |State the outage occurred in|
|`'POSTAL.CODE'`                |Represents the postal code of the U.S. states|
|`'NERC.REGION'`                |North American Electric Reliability Corporation (NERC) regions involved in the outage event|
|`'CLIMATE.REGION'`                |U.S. Climate regions as specified by National Centers for Environmental Information (9 Regions)|
|`'ANOMALY.LEVEL'`                |Oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season|
|`'CLIMATE.CATEGORY'`                |Categories—“Warm”, “Cold” or “Normal” episodes of the climate based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)|
|`'OUTAGE.START.DATE'`                |Day of the year when the outage event started|
|`'OUTAGE.START.TIME'`                |Time of the day when the outage event started|
|`'OUTAGE.RESTORATION.DATE'`                |Day of the year when power was restored to all the customers|
|`'OUTAGE.RESTORATION.TIME'`                |Time of the day when power was restored to all the customers|
|`'CAUSE.CATEGORY'`                |Categories of all the events causing the major power outages|
|`'OUTAGE.DURATION'`                |Duration of outage events (in minutes)|
|`'CUSTOMERS.AFFECTED'`                |Number of customers affected by the power outage event|
|`'TOTAL.PRICE'`                |Average monthly electricity price in the U.S. state (cents/kilowatt-hour)|
|`'TOTAL.SALES'`                |Total electricity consumption in the U.S. state (megawatt-hour)|
|`'TOTAL.CUSTOMERS'`                |Annual number of total customers served in the U.S. state|
|`'PC.REALGSP.REL'`                |Relative per capita real GSP as compared to the total per capita real GDP of the U.S. (expressed as fraction of per capita State real GDP & per capita US real GDP)|
|`'TOTAL.REALGSP'`                |Real GSP contributed by all industries (total) (measured in 2009 chained U.S. dollars)|
|`'UTIL.REALGSP'`                |Real GSP contributed by Utility industry (measured in 2009 chained U.S. dollars)|
|`'UTIL.CONTRI'`                |Utility industry׳s contribution to the total GSP in the State (expressed as percent of the total real GDP that is contributed by the Utility industry) (in %)|
|`'POPULATION'`                |Population in the U.S. state in a year|
|`'POPDEN_URBAN'`                |Population density of the urban areas (persons per square mile)|

---

# Data Cleaning and Exploratory Data Analysis
Before doing exploratory data analysis, I should clean the data.
## Cleaning
- I selected the columns I am interested in, which are listed above.

- I combined `'OUTAGE.START.DATE'` and `'OUTAGE.START.TIME'` columns into one pd.Timestamp column called `'OUTAGE.START'`. This would now include date and time. I did the same with `'OUTAGE.RESTORATION.DATE'` and `'OUTAGE.RESTORATION.TIME'` and named the new column `'OUTAGE.RESTORATION'`. Then, I  dropped `'OUTAGE.START.DATE'`,`'OUTAGE.START.TIME'`,`'OUTAGE.RESTORATION.DATE'`, and `'OUTAGE.RESTORATION.TIME'`.

- I converted `'ANOMALY.LEVEL'`,`'OUTAGE.DURATION'`,`'TOTAL.PRICE'`,`'TOTAL.SALES'`,
`'TOTAL.REALGSP'`, `'UTIL.REALGSP'`,`'POPDEN_URBAN'`,`'PC.REALGSP.REL'`, and `'UTIL.CONTRI'` to floats.

- Some `'OUTAGE.DURATION'` and `'CUSTOMERS.AFFECTED'` values are 0, which does not make sense. I replaced the 0's with np.nan since a 0 is likely indicative of missing values.

- I wanted to create a column called  `'IMPACT_SCORE'` by performing principal component analysis(PCA) on the standardized columns of `'OUTAGE.DURATION'` and `'CUSTOMERS.AFFECTED'`. However, this cannot be performed with np.nan values, so I decided to impute these columns. After performing permutations test for missingness of these columns, I found significant results showing missingness of both columns depending on `'CAUSE.CATEGORY'`(MAR). So, I performed multiple probabilistic imputation conditioned on `'CAUSE.CATEGORY'` for both columns. After that, I standardized the columns, performed PCA, and added the column to my dataframe. Lastly, I dropped `'OUTAGE.DURATION'` and `'CUSTOMERS.AFFECTED'`.

Here is the first 5 rows of my cleaned DataFrame:

| U.S._STATE   | POSTAL.CODE   | NERC.REGION   | CAUSE.CATEGORY     | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   |   TOTAL.CUSTOMERS |   TOTAL.PRICE |   TOTAL.SALES |   TOTAL.REALGSP |   UTIL.REALGSP |   UTIL.CONTRI |   PC.REALGSP.REL |   POPULATION |   POPDEN_URBAN |   MONTH |   YEAR | OUTAGE.START        | OUTAGE.RESTORATION   |   IMPACT_SCORE |
|:-------------|:--------------|:--------------|:-------------------|:-------------------|----------------:|:-------------------|------------------:|--------------:|--------------:|----------------:|---------------:|--------------:|-----------------:|-------------:|---------------:|--------:|-------:|:--------------------|:---------------------|---------------:|
| Minnesota    | MN            | MRO           | severe weather     | East North Central |            -0.3 | normal             |       2.5957e+06  |          9.28 |   6.56252e+06 |          274182 |           4802 |       1.75139 |          1.07738 |  5.34812e+06 |           2279 |       7 |   2011 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |      -0.113518 |
| Minnesota    | MN            | MRO           | intentional attack | East North Central |            -0.1 | normal             |       2.64074e+06 |          9.28 |   5.28423e+06 |          291955 |           5226 |       1.79    |          1.08979 |  5.45712e+06 |           2279 |       5 |   2014 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |      -0.638695 |
| Minnesota    | MN            | MRO           | severe weather     | East North Central |            -1.5 | cold               |       2.5869e+06  |          8.15 |   5.22212e+06 |          267895 |           4571 |       1.70627 |          1.06683 |  5.3109e+06  |           2279 |      10 |   2010 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |      -0.120676 |
| Minnesota    | MN            | MRO           | severe weather     | East North Central |            -0.1 | normal             |       2.60681e+06 |          9.19 |   5.78706e+06 |          277627 |           5364 |       1.93209 |          1.07148 |  5.38044e+06 |           2279 |       6 |   2012 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |      -0.179503 |
| Minnesota    | MN            | MRO           | severe weather     | East North Central |             1.2 | warm               |       2.67353e+06 |         10.43 |   5.97034e+06 |          292023 |           4873 |       1.6687  |          1.09203 |  5.48959e+06 |           2279 |       7 |   2015 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |       0.243088 |

## Exploratory Data Analysis

### Univariate Analysis
In my first step of exploratory data analysis, I wanted examine distributions of single variables.

#### Distribution of Impact Scores of Power Outages
From the histogram and box plot below, we can see that impact scores are highly right skewed, containing many outliers. This makes sense since the dataset contains 1534 rows. We can see that the most common value falls between -0.4 and -0.6 and the median is -0.3, so more values have less impact than the mean, which also makes sense since the outliers push the mean to the right and the values themselves are standard deviations away from the mean.

<iframe
  src="assets/impact_uni.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Distribution of Causes of Power Outages
From the bar plot below, we can see that the most common cause of power outages is severe weather, which makes sense from our intuition. It is followed by intentional attack. We see other causes after that, but they are more rare.

<iframe
  src="assets/causes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



### Bivariate Analysis
#### Box Plots of Impact Scores by Cause Category
From the plot below, we can see that severe weather has the most outliers, meaning had many impactful power outages. The most impactful power outage observed was caused by a fuel supply emergency. Islanding and public appeal appear to be the least impactful causes.
<iframe
  src="assets/impact_cause.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Trend of Impact Score over the Years
From the line plot below, we can see that the most impactful year was 2002 and the least impactful year was 2013. We see a trend from 2002 that the average impact year is going down over time generally.
<iframe
  src="assets/impact_year.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Average Impact Score by U.S. State
From this map visualization, we can see which states had the most impactful power outages on average and which states had the least impactful power outages on average. For instance, we can see that Florida, West Virginia, and New York had the most impactful power outages on average and Montanta, South Dakota, and Vermont had the least impactful power outages on average.
<iframe
  src="assets/impact_state.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Grouping and Aggregates

#### Impact Score Average by NERC Region and Cause Category
Each slot in this pivot table represents the average impact score conditioned on the cause category and NERC region. From this, we can see the combinations of NERC region and cause that resulted in high impact power outages from the past. This could help us find powerful patterns, so we can know where impactful power outages occur. We can see that equipment failure in RFC had more powerful power outages than other NERC regions. Also, in RFC, the fuel supply emergency is unusually high due to the outlier we saw in our box plot of impact scores by cause category. Lastly, for severe weather causes, FRCC had most impact power outages on average.

| NERC.REGION   |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:--------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| ASCC          |           -0.398372 |              nan        |           nan        |  nan        |      nan        |      nan         |                     nan         |
| ECAR          |           -0.230388 |              nan        |            -0.418185 |  nan        |      nan        |        0.518388  |                       2.91534   |
| FRCC          |           -0.160836 |              nan        |            -0.625005 |  nan        |       -0.116905 |        1.44744   |                      -0.49631   |
| FRCC, SERC    |          nan        |              nan        |           nan        |  nan        |      nan        |      nan         |                       0.0508044 |
| HECO          |          nan        |              nan        |           nan        |  nan        |      nan        |       -0.031349  |                      -0.566547  |
| HI            |          nan        |              nan        |           nan        |  nan        |      nan        |        0.324253  |                     nan         |
| MRO           |          nan        |                0.775508 |            -0.376839 |   -0.647669 |       -0.552291 |        0.0497565 |                     nan         |
| NPCC          |           -0.305414 |                1.0337   |            -0.592205 |   -0.545928 |       -0.310625 |        0.207407  |                       0.783275  |
| PR            |          nan        |              nan        |           nan        |  nan        |      nan        |       -0.480671  |                     nan         |
| RFC           |            0.732515 |                3.27263  |            -0.562378 |   -0.643695 |       -0.524463 |        0.282686  |                      -0.0424385 |
| SERC          |           -0.356173 |                1.08776  |            -0.577594 |   -0.639618 |       -0.48263  |       -0.0402071 |                      -0.277758  |
| SPP           |           -0.217272 |               -0.669433 |            -0.578619 |   -0.5869   |       -0.511717 |        0.389341  |                       2.33478   |
| TRE           |           -0.372642 |                0.80409  |            -0.599756 |  nan        |       -0.492202 |        0.549893  |                      -0.0298481 |
| WECC          |           -0.253889 |                0.456716 |            -0.567782 |   -0.635749 |       -0.412674 |        0.598164  |                      -0.131216  |

---

# Assessment of Missingness

## NMAR Analysis

## Missingness Dependency


---

# Hypothesis Testing
I will perform a permutation test to see if there's a significant difference in average impact score when the climate is normal and when the climate is abnormal(warm or cold). The relevant columns for this analysis are `'CLIMATE.CATEGORY'` and `'IMPACT_SCORE'`. Before performing this test, I filtered the rows to only include causes of severe weather because the other causes like intentional attack have no relation to weather, so if I included it, it would've made the results unreliable. Secondly, I classified warm or cold `'CLIMATE.CATEGORY'` as abnormal.

**Null Hypothesis:** The average impact score of severe weather when the climate is normal the same as the average impact score of severe weather when climate is warm or cold.

**Alternate Hypothesis:** The average impact score of severe weather when climate is warm or cold is higher than when climate is normal.

**Test Statistic:** Mean of impact score of severe weather in abnormal(hot or cold) climate - Mean of impact score of severe weather in normal climate.

The significance level is set to the standard value of 0.05.

Below is the histogram of 5000 simulated test statistics under the null hypothesis.

<iframe
  src="assets/hyp_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We get a very high p-value: **0.8266**. This p-value exceeds the significance level, so we fail to reject the null hypothesis. We do not have sufficient statistical evidence to claim that the average impact score of severe weather when climate is abnormal is higher than when climate is normal.

**Note:** I performed another test with the alternative hypothesis being that there is a difference in means and with the test statistic being absolute difference of means, resulting in **0.3586** p-value, still failing to reject null hypothesis. These results suggest that there is no strong statistical evidence that average severe weather impact scores differ based on whether the climate is normal or abnormal (warm/cold).

---

# Framing a Prediction Problem

My prediction model will try to predict the impact score of a power outage. This will be a regression problem because we are predicting continuous numerical variables. The response variable will be the `'IMPACT_SCORE'` column. I chose this because it is the variable I engineered to represent the impact of a power outage using the number of customers affected and the duration of the power outage. 

### Metrics
**Root Mean Squared Error (RMSE):** I chose this metric because it is sensitive to outliers, which is needed because we want weigh big impact power outages more heavily since it would affect people more.

**R² Score:** This is an additional metric to assess how much of the variability in impact scores my model explains.

### Available Data

It's important to note that the features I would know at the start of the power outage and the possible features I can use are:
- `'U.S._STATE'`  
- `'NERC.REGION'`  
- `'CAUSE.CATEGORY'`  
- `'ANOMALY.LEVEL'`  
- `'CLIMATE.REGION'`  
- `'TOTAL.CUSTOMERS'`  
- `'TOTAL.PRICE'`  
- `'TOTAL.SALES'`  
- `'TOTAL.REALGSP'`  
- `'UTIL.REALGSP'`  
- `'UTIL.CONTRI'`  
- `'PC.REALGSP.REL'`  
- `'POPULATION'`  
- `'POPDEN_URBAN'`  
- `'MONTH'`  
- `'YEAR'`  
- `'HOUR'`  
- `'DAY'`  

All of these information are available at the start of a power outage. Note that `'MONTH'`,`'YEAR'`,`'HOUR'`, and `'DAY'` refer to the start of the power outage, which is available. I created the `'HOUR'`, and `'DAY'` feature columns from the `'OUTAGE.START'` column. I removed `'OUTAGE.RESTORATION`' because we would not know when an outage stopped at the time it started.	


---

# Baseline Model

---

# Final Model

---

# Fairness Analysis

---