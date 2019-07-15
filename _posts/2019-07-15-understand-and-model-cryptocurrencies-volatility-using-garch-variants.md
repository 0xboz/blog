---
title: "Understand and Model Cryptocurrencies Volatility Using GARCH Variants"
date: 2019-07-15
tags: [ARCH, GARCH, EGARCH, Cryptocurrencies, Volatility]
excerpt: "Forecast cryptocurrencies volatility using GARCH and its variants"
header:
    image: /assets/images/architecture_night_buildings.jpg
---

I had a difficult time to understand GARCH and its variants. In this post, I am going to show you what I have come across while learning and experimenting on this topic. If you are well-versed in this area, please do keep reading and point out the mistakes in this piece. Hopefully, it will help someone in the future.

We are going to use BTCUSD as an example for the rest of this article. Feel free to try the following techniques on other cryptocurrencies pairs. Shoot me a message ([Patreon](https://www.patreon.com/0xboz), [Discord](https://discord.gg/jchMcc2) or [Twitter](https://twitter.com/0xboz)) and let me know what you have found.


## Data

First of all, let us get BTCUSD minute data. Due to the sheer size of the data, you can scale down the time window if the code takes too long.

```python
from catalyst.api import symbol, record
from catalyst import run_algorithm

import pandas as pd

trading_pair = 'btc_usd'
frequency = 'minute'
exchange = 'bitfinex'
start = '2017-4-1'
end = '2019-4-1'
capital_base = 1000
quote_currency = trading_pair.split('_')[1]

def initialize(context):
    context.asset = symbol(trading_pair)

def handle_data(context, data):
    
    # The last known price and volume of current date/minute and the day/minute before
    if frequency == 'daily':        
        price = data.current(context.asset, 'price')
        volume = data.current(context.asset, 'volume')
    elif frequency == 'minute':        
        price = data.current(context.asset, 'price')
        volume = data.current(context.asset, 'volume')

    record(price=price, volume=volume)
  
if __name__ == '__main__':   
    perf = run_algorithm(capital_base=capital_base,
                         data_frequency=frequency,
                         initialize=initialize,
                         handle_data=handle_data,                      
                         exchange_name=exchange,
                         quote_currency=quote_currency,
                         start=pd.to_datetime(start, utc=True),
                         end=pd.to_datetime(end, utc=True))
```
Once it is done, we will have 2-year of BTCUSD minute data from [Bitfinex](https://www.bitfinex.com/).

Now, it is time to resample our data and perform a few simple calculations.

```python
import numpy as np
# Resample to minute data
m_df = pd.DataFrame(perf.loc[:, ['price', 'volume']], index=perf.index).resample('T', closed='left', label='left').mean().copy()
m_df.index.name = 'timestamp'
m_df['log_price'] = np.log(m_df.price)
m_df['return'] = m_df.price.pct_change().dropna()
m_df['log_return'] = m_df['log_price'] - m_df['log_price'].shift(1)

# Calculate squared log return
m_df['squared_log_return'] = np.power(m_df['log_return'], 2)

# Scale up 100x
m_df['return_100x'] = np.multiply(m_df['return'], 100)
m_df['log_return_100x'] = np.multiply(m_df['log_return'], 100)

m_df.head()
```
```
	                        price	volume	    log_price	return	  log_return   squared_log_return	return_100x	log_return_100x
timestamp								
2017-04-01 00:01:00+00:00	1081.5	6.752945	6.986104	NaN	        NaN	        NaN	            NaN	        NaN
2017-04-01 00:02:00+00:00	1081.4	3.928597	6.986012	-0.000092	-0.000092	8.550413e-09	-0.009246	-0.009247
2017-04-01 00:03:00+00:00	1081.7	2.540370	6.986289	0.000277	0.000277	7.693949e-08	0.027742	0.027738
2017-04-01 00:04:00+00:00	1081.6	16.532780	6.986197	-0.000092	-0.000092	8.547253e-09	-0.009245	-0.009245
2017-04-01 00:05:00+00:00	1082.2	15.992445	6.986751	0.000555	0.000555	3.075589e-07	0.055473	0.055458
```

You may wonder why we need scale up 100 times for simple returns and log returns. This is because ```ARCH``` Python package might run into convergence issues. Do not worry, and I will show you another way around without scaling up.

## Volatility

Since there is a general consense that the volatility is more sticky than the price in financial market, much of the effort has been allocated to study the models of volatility. Before we can jump into any fancy models and math, let us define what is volatility and how to caculate it.

The following is cited from [Wikipedia](https://en.wikipedia.org/wiki/Volatility_(finance)).
> In finance, volatility (symbol σ) is the degree of variation of a trading price series over time as measured by the standard deviation of logarithmic returns.

We can arrive at the mathematical definiton with ease.
\begin{equation*}
\sigma = \sqrt{\frac{1}{n-1}\sum_{i=1}^n(\bar{r} - r_i)^2}
\end{equation*}

Where,   
$\sigma$ is volatility,      
$\bar{r}$ is mean log return,  
$r_i$ is the log return at time i,  
$n$ is the number of log return observations.

## Daily Volatility & Variance

A simple volatility calculated from a range of the sample time series is not that useful when we need to implement more advanced models. We are more inclined to estimate the volatility/variance for a certain time interval.

> Equally as important as the forecast methodology is the proxy of variance that one measures performance against.

One common way is to use daily squared log return $r_d^2$ as a proxy of eastimating the true variance. Let us resample the data by day and calculate daily squared log returns.

```python
# Resample to daily data
d_df = pd.DataFrame(m_df.loc[:, ['price', 'volume']], index=m_df.index).resample('D', closed='left', label='left').mean().copy()
d_df['log_price'] = np.log(d_df.price)
d_df['return'] = d_df.price.pct_change().dropna()
d_df['log_return'] = d_df['log_price'] - d_df['log_price'].shift(1)
d_df['squared_log_return'] = np.power(d_df['log_return'], 2)

# Scale up 100x
d_df['return_100x'] = np.multiply(d_df['return'], 100)
d_df['log_return_100x'] = np.multiply(d_df['log_return'], 100)

d_df.head()
```
```
                                price	volume	log_price	return	log_return	squared_log_return	return_100x	log_return_100x
timestamp								
2017-04-01 00:00:00+00:00	1086.633148	11.808373	6.990839	NaN	        NaN	        NaN	        NaN	        NaN
2017-04-02 00:00:00+00:00	1095.521806	14.183194	6.998986	0.008180	0.008147	0.000066	0.818000	0.814672
2017-04-03 00:00:00+00:00	1138.100972	21.106648	7.037116	0.038867	0.038130	0.001454	3.886656	3.813028
2017-04-04 00:00:00+00:00	1143.858542	14.609319	7.042163	0.005059	0.005046	0.000025	0.505893	0.504617
2017-04-05 00:00:00+00:00	1130.486875	15.630853	7.030404	-0.011690	-0.011759	0.000138	-1.168997	-1.175883
```
However, the downside of this method is incredibly noisy. 

Another commonly accepted approach is called *realized volatility (RV)*, coined by Bollershev. Basically, the summation of squared intraday returns aggregated to the daily level can be used to measure that day's variance. For instance, if we have BTCUSD minute data, its daily variance can be estimated as follows:

\begin{equation*}
RV_{d_{j+1}} = \sum_{i = 1}^{1440} r_{t_{i + 1440(j+1)}}^2 
\end{equation*}

Here is how we implement it in Python.

```python
d_df['realized_variance_1min'] = pd.Series(m_df.loc[:, 'squared_log_return'], index=perf.index).resample('D', closed='left', label='left').sum().copy()
d_df['realized_volatility_1min'] = np.sqrt(d_df['realized_variance_1min'])
d_df.head()
```

```
	                        price	volume	log_price	return	log_return	squared_log_return	return_100x	log_return_100x	realized_variance_1min	realized_volatility_1min
timestamp										
2017-04-01 00:00:00+00:00	1086.633148	11.808373	6.990839	NaN	        NaN	        NaN	        NaN	        NaN	        0.000645	0.025388
2017-04-02 00:00:00+00:00	1095.521806	14.183194	6.998986	0.008180	0.008147	0.000066	0.818000	0.814672	0.000757	0.027519
2017-04-03 00:00:00+00:00	1138.100972	21.106648	7.037116	0.038867	0.038130	0.001454	3.886656	3.813028	0.000751	0.027406
2017-04-04 00:00:00+00:00	1143.858542	14.609319	7.042163	0.005059	0.005046	0.000025	0.505893	0.504617	0.000510	0.022578
2017-04-05 00:00:00+00:00	1130.486875	15.630853	7.030404	-0.011690	-0.011759	0.000138	-1.168997	-1.175883	0.000493	0.022203
```

Another common interval to calculate RV is 5-min data interval. Of course, the formula mentioned above will be re-written like this.

\begin{equation*}
RV_{d_{j+1}} = \sum_{i = 1}^{288} r_{t_{i + 288(j+1)}}^2 
\end{equation*}

```python
# Resample to 5-min data
five_min_df = pd.DataFrame(m_df.loc[:, ['price', 'volume']], index=m_df.index).resample('5T', closed='left', label='left').mean().copy()
five_min_df['log_price'] = np.log(five_min_df.price)
five_min_df['log_return'] = five_min_df['log_price'] - five_min_df['log_price'].shift(1)
five_min_df['squared_log_return'] = np.power(five_min_df['log_return'], 2)
five_min_df.head()
```
```
	                         price	volume	log_price	log_return	squared_log_return
timestamp					
2017-04-01 00:00:00+00:00	1081.55	7.438673	6.986150	NaN	        NaN
2017-04-01 00:05:00+00:00	1081.98	19.154190	6.986548	0.000397	1.580051e-07
2017-04-01 00:10:00+00:00	1082.96	2.362552	6.987453	0.000905	8.196350e-07
2017-04-01 00:15:00+00:00	1083.78	2.617299	6.988210	0.000757	5.728938e-07
2017-04-01 00:20:00+00:00	1083.86	2.761562	6.988284	0.000074	5.448358e-09
```
```python
d_df['realized_variance_5min'] = pd.Series(five_min_df.loc[:, 'squared_log_return'], index=perf.index).resample('D', closed='left', label='left').sum().copy()
d_df['realized_volatility_5min'] = np.sqrt(d_df['realized_variance_5min'])
d_df.head()
```
```
	                        price	volume	log_price	return	log_return	squared_log_return	return_100x	log_return_100x	realized_variance_5min	realized_volatility_5min
timestamp										
2017-04-01 00:00:00+00:00	1086.633148	11.808373	6.990839	NaN	        NaN	        NaN	        NaN	        NaN	        0.000435	0.020851
2017-04-02 00:00:00+00:00	1095.521806	14.183194	6.998986	0.008180	0.008147	0.000066	0.818000	0.814672	0.000434	0.020839
2017-04-03 00:00:00+00:00	1138.100972	21.106648	7.037116	0.038867	0.038130	0.001454	3.886656	3.813028	0.000704	0.026524
2017-04-04 00:00:00+00:00	1143.858542	14.609319	7.042163	0.005059	0.005046	0.000025	0.505893	0.504617	0.000458	0.021391
2017-04-05 00:00:00+00:00	1130.486875	15.630853	7.030404	-0.011690	-0.011759	0.000138	-1.168997	-1.175883	0.000380	0.019500
```

### Updating...


<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>