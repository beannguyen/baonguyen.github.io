---
title: "Volume Profile and Value Area in trading"
date: 2023-01-15T21:27:58+07:00
---

Volume profile is a technical analysis tool used in trading to help traders understand market activity and price action. It is essentially a way of visualizing the volume of trades that occur at different price levels for a specific stock, currency, or commodity. By analyzing volume profile, traders can gain insights into market sentiment, support and resistance levels, and the overall supply and demand for a given asset.

In this post, we will explore:

1. The implementation of volume profile
2. Utilizing volume profile to identify the value area.

<br />

{{< highlight python >}}
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

import datetime
from pandas_datareader import data as dreader
import yfinance as yf
{{< /highlight >}}


{{< highlight python >}}
symbol = 'MSFT'
start_time = datetime.datetime(2015, 9, 1).date().isoformat()
end_time = datetime.datetime.now().date().isoformat()
{{< /highlight >}}


{{< highlight python >}}
ticker = yf.Ticker(symbol)
df = ticker.history(start=start_time, end=end_time)
df.head()
{{< /highlight >}}

<br />
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Dividends</th>
      <th>Stock Splits</th>
    </tr>
    <tr>
      <th>Date</th>
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
      <th>2015-09-01 00:00:00-04:00</th>
      <td>37.554181</td>
      <td>37.928211</td>
      <td>37.100006</td>
      <td>37.242493</td>
      <td>49688900</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-09-02 00:00:00-04:00</th>
      <td>37.723402</td>
      <td>38.631756</td>
      <td>37.295942</td>
      <td>38.613945</td>
      <td>37671500</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-09-03 00:00:00-04:00</th>
      <td>38.658460</td>
      <td>39.166069</td>
      <td>38.542689</td>
      <td>38.738609</td>
      <td>28285200</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-09-04 00:00:00-04:00</th>
      <td>38.124143</td>
      <td>38.328967</td>
      <td>37.580911</td>
      <td>37.946033</td>
      <td>37138800</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-09-08 00:00:00-04:00</th>
      <td>38.560509</td>
      <td>39.183890</td>
      <td>38.471456</td>
      <td>39.085930</td>
      <td>32469800</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>
<br />

## Volume Profile

This code creates a market profile by dividing the price range of the market into equal intervals, calculating the number of trades that occur within each interval, and plotting the results as a histogram. The resulting chart provides a visual representation of the distribution of trading activity across different price levels, making it easier to understand market activity and make informed trading decisions.


{{< highlight python >}}

import pandas as pd
import matplotlib.pyplot as plt

def calculate_volume_profile(dataframe):
    # Calculate the range of prices
    price_range = dataframe['Close'].max() - dataframe['Close'].min()

    # Divide the price range into 100 equal intervals
    intervals = 100
    price_intervals = price_range / intervals

    # Create a list of price interval boundaries
    price_boundaries = [dataframe['Close'].min() + (price_intervals * i) for i in range(intervals)]

    # Create a dictionary to store the time and trade volume for each price interval
    volume_profile = {price: 0 for price in price_boundaries}

    # Iterate over each row in the DataFrame
    for index, row in dataframe.iterrows():
        # Determine which price interval the trade belongs to
        for i, price in enumerate(price_boundaries):
            if row['Close'] < price:
                volume_profile[price_boundaries[i - 1]] += row['Volume']
                break

    return volume_profile, price_intervals

# Call the volume_profile function
volume_profile, price_intervals = calculate_volume_profile(df)


plt.figure(figsize=(20, 20), dpi= 120, facecolor='w', edgecolor='k')

plt.subplot(321)
plt.plot(df.index, df.Close)
plt.xlabel('Date')
plt.ylabel('Close')

# Plot the market profile data as a histogram
plt.subplot(322)
plt.barh(list(volume_profile.keys()), list(volume_profile.values()), height=price_intervals)
plt.xlabel('Volume profile')
plt.ylabel('Close')

plt.show()

{{< /highlight >}}


{{< figure src="/volume-profile-post/output_6_0.png" title="Figure 1" >}}
    


The volume profile chart has been generated, but the example is not practically useful as it utilizes all historical data from 2015 to present.
Let's create a monthly volume profile instead.


{{< highlight python >}}
def plot_volume_profile(df, volume_profile, price_intervals):
    plt.figure(figsize=(20, 20), dpi= 120, facecolor='w', edgecolor='k')

    plt.subplot(321)
    plt.plot(df.index, df.Close)
    plt.xlabel('Date')
    plt.ylabel('Close')

    # Plot the market profile data as a histogram
    plt.subplot(322)
    plt.barh(list(volume_profile.keys()), list(volume_profile.values()), height=price_intervals)
    plt.xlabel('Volume profile')
    plt.ylabel('Close')

    plt.show()


month_df = df[(df.index >= '2022-01-01') & (df.index <= '2022-01-31')].copy()
volume_profile, price_intervals = calculate_volume_profile(month_df)
plot_volume_profile(month_df, volume_profile, price_intervals)

{{< /highlight >}}



{{< figure src="/volume-profile-post/output_8_0.png" title="Figure 2" >}}
    


As we can observe, there is the most trading activity between the prices of 235 and 250. When the price drops or rises below or above these levels, it tends to return to this range. This can be referred to as the value area.

## Value Area

The value area is calculated as the range of prices where 70% of the total trading volume took place. This information is used to identify key levels of support and resistance, as well as potential areas for buying and selling. The value area is a commonly used strategy in futures markets, but it can also be applied to other markets, such as the stock and forex markets.


{{< highlight python >}}
# Calculate the value area as the range of prices with the highest trading volume
max_volume = max(volume_profile.values()) * 0.7
value_area = [price for price in volume_profile if volume_profile[price] >= max_volume]

print("The value area is between", max(value_area), "and", max(value_area))

{{< /highlight >}}

    The value area is between 293.6030807495117 and 293.6030807495117


Let's plot the value area to see if we can gain any useful insights.


{{< highlight python >}}
plt.figure(figsize=(20, 20), dpi= 120, facecolor='w', edgecolor='k')

plt.subplot(321)
plt.plot(month_df.index, month_df.Close)
plt.axhline(max(value_area), color='red', linestyle='dashed')
plt.xlabel('Date')
plt.ylabel('Close')

# Plot the market profile data as a histogram
plt.subplot(322)
plt.barh(list(volume_profile.keys()), list(volume_profile.values()), height=price_intervals)
plt.xlabel('Volume profile')
plt.ylabel('Close')

plt.show()
{{< /highlight >}}



{{< figure src="/volume-profile-post/output_13_0.png" title="Figure 3" >}}
    


In January 2022, we noticed that the price dropped below the value area before rebounding. This suggests that the market recognizes the value area as a strong support level.

#### Using Value Area to Generate Trading Signals

The value area can be used to generate trading signals in several ways. Here are some of the most common strategies:

1. Buying at Support: 
  - If the price falls to the bottom of the value area, it may indicate that demand is stronger at that level and that the price may bounce back up. 
  - Traders can use this as an opportunity to buy.
  
2. Selling at Resistance: 
  - If the price rises to the top of the value area, it may indicate that supply is stronger at that level and that the price may pull back down. 
  - Traders can use this as an opportunity to sell.
  
3. Breakouts: 
  - If the price breaks through the top or bottom of the value area, it may indicate a change in market sentiment and that the price is likely to continue moving in the direction of the breakout. 
  - Traders can use this as a signal to enter a trade in the direction of the breakout.
  
4. Reversals: 
  - If the price returns to the value area after breaking out, it may indicate that the initial move was a false breakout and that the price is likely to reverse. 
  - Traders can use this as a signal to enter a trade in the opposite direction.

_It is important to note that while the value area can provide valuable information about market behavior, it should not be used in isolation to generate trading signals. It should be combined with other technical indicators and market analysis to form a complete trading strategy._


## Conclustion

Volume profile and value area are important tools for traders to understand market behavior and identify potential opportunities. Volume profile provides a visual representation of trading activity, while value area is determined by calculating the range of prices where 70% of the total trading volume occurred. By combining the information from both volume profile and value area, traders can identify key levels of support and resistance, as well as potential areas for buying and selling.

There are several strategies that traders can use to generate trading signals based on the value area, including buying at support, selling at resistance, breakouts, and reversals. However, it is important to note that the value area should not be used in isolation to generate trading signals. Instead, it should be combined with other technical indicators and market analysis to form a complete trading strategy.

In conclusion, volume profile and value area provide traders with valuable insights into market behavior, and can be used to create a comprehensive trading strategy. To be successful, traders need to understand the dynamics of these tools and incorporate them into their analysis and decision-making process.


