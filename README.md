# RFM Analysis Documentation

## Introduction
The following Python script performs RFM (Recency, Frequency, Monetary) Analysis using the Plotly library to visualize and analyze customer segmentation based on their purchasing behavior. RFM analysis involves evaluating three key metrics - recency, frequency, and monetary value.


# Import necessary libraries
```python
import pandas as pd
import plotly.express as px
import plotly.io as pio
import plotly.graph_objects as go
from datetime import datetime
```

# Set Plotly template
```python
pio.templates.default = "plotly_white"
```

# Load the RFM data from a CSV file
```python
data = pd.read_csv("rfm_data.csv")
print(data.head())
```

# Convert 'PurchaseDate' to datetime
```python
data['PurchaseDate'] = pd.to_datetime(data['PurchaseDate'])
```

# Calculate Recency
```python
data['Recency'] = (datetime.now().date() - data['PurchaseDate'].dt.date).dt.days
```

# Calculate Frequency
```python
frequency_data = data.groupby('CustomerID')['OrderID'].count().reset_index()
frequency_data.rename(columns={'OrderID': 'Frequency'}, inplace=True)
data = data.merge(frequency_data, on='CustomerID', how='left')
```

# RFM Segment Distribution
```python
segment_counts = data['Value Segment'].value_counts().reset_index()
segment_counts.columns = ['Value Segment', 'Count']

pastel_colors = px.colors.qualitative.Pastel

# Create the bar chart
fig_segment_dist = px.bar(segment_counts, x='Value Segment', y='Count', 
                          color='Value Segment', color_discrete_sequence=pastel_colors,
                          title='RFM Value Segment Distribution')

# Update the layout
fig_segment_dist.update_layout(xaxis_title='RFM Value Segment',
                              yaxis_title='Count',
                              showlegend=False)

# Show the figure
fig_segment_dist.show()
```


# Calculate Monetary Value
```python
monetary_data = data.groupby('CustomerID')['TransactionAmount'].sum().reset_index()
monetary_data.rename(columns={'TransactionAmount': 'MonetaryValue'}, inplace=True)
data = data.merge(monetary_data, on='CustomerID', how='left')

print(data.head())
```
# Define scoring criteria for each RFM value
```python
recency_scores = [5, 4, 3, 2, 1]  # Higher score for lower recency (more recent)
frequency_scores = [1, 2, 3, 4, 5]  # Higher score for higher frequency
monetary_scores = [1, 2, 3, 4, 5]  # Higher score for higher monetary value

# Calculate RFM scores
data['RecencyScore'] = pd.cut(data['Recency'], bins=5, labels=recency_scores)
data['FrequencyScore'] = pd.cut(data['Frequency'], bins=5, labels=frequency_scores)
data['MonetaryScore'] = pd.cut(data['MonetaryValue'], bins=5, labels=monetary_scores)

# Convert RFM scores to numeric type
data['RecencyScore'] = data['RecencyScore'].astype(int)
data['FrequencyScore'] = data['FrequencyScore'].astype(int)
data['MonetaryScore'] = data['MonetaryScore'].astype(int)


# Calculate RFM score by combining the individual scores
data['RFM_Score'] = data['RecencyScore'] + data['FrequencyScore'] + data['MonetaryScore']

print(data.head())
```

