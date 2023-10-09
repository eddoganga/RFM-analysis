## Project RFM Analysis

 RFM analysis is a marketing technique used to quantitatively rank and group customers based on the recency, frequency and monetary total of their recent transactions to identify the best customers and perform targeted marketing campaigns.Here's a step-by-step guide to performing RFM analysis using a case study as an example:

## Step 1: Data Collection
Gather the necessary data from your customer database.
```
import pandas as pd
df = pd.read_csv('rfm_data.csv')
print(df.head())
```
## Step 2: Data Preparation
This includes cleaning and transforming the data to calculate the RFM scores.
```
import datetime as dt
# Convert PurchaseDate column to datetime
df['PurchaseDate'] = pd.to_datetime(df['PurchaseDate'])

# Calculate the current date for recency calculation
current_date = df['PurchaseDate'].max()


# Create a new column 'Recency' for days since the last purchase
df['Recency'] = (current_date - df['PurchaseDate']).dt.days

# Calculate RFM values for each customer
df = df.groupby('CustomerID').agg({
    'PurchaseDate': lambda x: (current_date - x.max()).days,  
    'OrderID': 'count',                                      
    'TransactionAmount': 'sum'                               
}).reset_index()


# Rename the columns
df.rename(columns={
    'PurchaseDate': 'Recency',
    'OrderID': 'Frequency',
    'TransactionAmount': 'MonetaryValue'
}, inplace=True)


# Display the RFM data
print(df.head())

```
Standardize the RFM values
```
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
rfm_scaled = scaler.fit_transform(df)
rfm_scaled = pd.DataFrame(rfm_scaled, index=df.index, columns=df.columns)
```
## Step 3 RFM Score Calculations
Assign a score to each customer for each of the RFM dimensions
```
# Create RFM quartiles
quantiles = df.quantile(q=[0.25, 0.5, 0.75])
quantiles = quantiles.to_dict()

# Define functions to assign RFM scores
def recency_score(x, column, quantile_dict):
    if x <= quantile_dict[column][0.25]:
        return 4
    elif x <= quantile_dict[column][0.5]:
        return 3
    elif x <= quantile_dict[column][0.75]:
        return 2
    else:
        return 1

def frequency_monetary_score(x, column, quantile_dict):
    if x <= quantile_dict[column][0.25]:
        return 1
    elif x <= quantile_dict[column][0.5]:
        return 2
    elif x <= quantile_dict[column][0.75]:
        return 3
    else:
        return 4

# Calculate RFM scores
df['R'] = df['Recency'].apply(recency_score, args=('Recency', quantiles))
df['F'] = df['Frequency'].apply(frequency_monetary_score, args=('Frequency', quantiles))
df['M'] = df['MonetaryValue'].apply(frequency_monetary_score, args=('MonetaryValue', quantiles))

# Calculate RFM Score by combining R, F, and M scores
df['RFM_Score'] = df['R'].map(str) + df['F'].map(str) + df['M'].map(str)
```

## Step 4 Segment Customers

Now, segment your customers based on their RFM scores. You can create a 5x5 grid (a 5x5 matrix) where each cell represents a combination of R, F, and M scores. For example:

- High Recency, High Frequency, High Monetary Value: Your most valuable and active customers.
- Low Recency, High Frequency, High Monetary Value: Customers who buy frequently but haven't bought recently.
- High Recency, Low Frequency, High Monetary Value: Customers who spend a lot but don't buy often.
```
# Define segments based on RFM scores (e.g., High-Value, At-Risk, etc.)
rfm_segments = {
    'Best Customers': ['444'],
    'Loyal Customers': ['344', '434', '443'],
    'Potential Loyalists': ['334', '343'],
    'At-Risk Customers': ['211', '122', '132', '213', '312', '321'],
    'Cant Lose Them': ['111'],
    'Lost Customers': ['233', '322', '231', '132'],
    'Promising': ['324', '423', '422', '421', '142', '241', '143']
}

# Create a function to assign segments based on RFM Score
def assign_segment(rfm_score, segments):
    for segment, score_list in segments.items():
        if rfm_score in score_list:
            return segment
    return 'Other'

# Assign segments to customers
df['Segment'] = df['RFM_Score'].apply(assign_segment, segments=rfm_segments)

# Display the RFM analysis results
print(df)
```

## Step 5 Analysis

Once you have segmented your customers, analyze each segment's characteristics and behaviors. Consider the following questions:

- What makes each segment unique?
- Are there common patterns or preferences within each segment?
- How can you tailor your marketing strategies to each segment?

- Offer exclusive discounts or promotions to the "High Recency, Low Frequency, High Monetary Value" segment to encourage them to buy more frequently.
- Send re-engagement emails to the "Low Recency, High Frequency, Low Monetary Value" segment to bring them back.
- Provide personalized product recommendations to different segments based on their purchase history.

## Step 6 Monitor and Iterate

Continuously monitor customer behavior and adjust your strategies accordingly. Periodically re-segment your customers to ensure your marketing efforts stay relevant.

## Step 7 Measure the Impact

Track the impact of your strategies on customer behavior and revenue.
