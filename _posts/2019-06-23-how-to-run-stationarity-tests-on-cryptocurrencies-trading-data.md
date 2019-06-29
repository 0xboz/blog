---
title: "How to Run Stationarity Tests on Cryptocurrencies Trading Data"
date: 2019-06-23
tags: [Catalyst, Stationarity, Augmented Dickey-Fuller (ADF) Test, Phillips-Perron (PP) Test, Kwiatkowski-Phillips-Schmidt-Shin (KPSS) Test]
excerpt: "Identify Stationary Data for Crypto Trading"
header:
    image: /assets/images/skyline.jpg
---

Before you start creating models and testing stratgies, you need to identify the stationarity in any time-series analysis. 

During my studies, I got somewhat confused with stationarity and homoscedasticity. If you feel like the same, here is [a quick way](https://www.quora.com/What-is-the-difference-between-stationarity-and-homoscedasticity) to differentiate those two.

> Stationarity means stability in any aspects of a variable. However, homoscedasticity means stability in the variance (and in the mean). Therefore, a stationary process is homoscedastic, but a homoscedastic process is not necessarily stationary.

In crypto trading, you should determine wether working with levels of the price or the differences of the prices (price returns). Generally speaking, the crypto price is following an up-trend, meaning the time-series price data is non-stationary. To avoid spurious regression, a few commonly used Unit Root tests are pretty handy to check those series.

In this tutorial, we are going to write a short python script to run those tests on BTC/USDT minute data ingested from Catalyst. If you have not installed Catalyst yet, take a quick look at [this post](https://0xboz.github.io/blog/how-to-install-catalyst-debian-stretch/) and come back later.

## Install arch
Activate the env and install arch
```
source venv/bin/activate
pip install arch==4.3.1
```
Since there is a compability issue among Catalyst, pandas and arch at the time of writing, I would recommend install *arch==4.3.1* instead of *arch==4.8.0*. My env has *pandas=0.19.2* and *enigma-catalyst=0.5.21*, in case you wonder.

## Ingest Data
Open your terminal and ingest the data we need for the tests.

```
(venv) catalyst ingest-exchange -x poloniex -f minute -i btc_usdt
```

Start your text editor and create a file named ```stationarity_test.py```.
```python
from arch.unitroot import ADF, PhillipsPerron, KPSS
import pandas as pd


class StationarityTests:
    """
    Stationarity Testing
    Also often called Unit Root tests
    Three commonly used tests to check stationarity of the data
    """

    def __init__(self, significance=.05):
        self.SignificanceLevel = significance
        self.pValue = None
        self.isStationary = None
```
Here we have defined a class called ```StationarityTests```, and hard-coded the significance as 0.05.  It also contains an isStationary variable that will hold the results of each tests. If the time series is stationary, isStationary will be True, otherwise it will be False. 

We will define each type of tests as a function. For instance, let us take a look at ```ADF_Test()```. If the p-Value is less than the significance defined above-mentioned, we reject the Null Hypothesis that the time series contains a unit root. In other words, by rejecting the Null hypothesis, we can conclude that the time series is stationary.

```python
    def ADF_Test(self, timeseries, printResults=True):
        """
        Augmented Dickey-Fuller (ADF) Test
        Null Hypothesis is Unit Root
        Reject Null Hypothesis >> Series is stationary >> Use price levels
        Fail to Reject >> Series has a unit root >> Use price returns
        """

        adfTest = ADF(timeseries)

        self.pValue = adfTest.pvalue

        if (self.pValue < self.SignificanceLevel):
            self.isStationary = True
        else:
            self.isStationary = False

        if printResults:

            print('Augmented Dickey-Fuller (ADF) Test Results: {}'.format(
                'Stationary' if self.isStationary else 'Not Stationary'))
```
Similarly, we can create functions for Phillips-Perron (PP) and Kwiatkowski-Phillips-Schmidt-Shin (KPSS) tests. Put it all together, our script contains the following.
```python
from arch.unitroot import ADF, PhillipsPerron, KPSS
import pandas as pd


class StationarityTests:
    """
    Stationarity Testing
    Also often called Unit Root tests
    Three commonly used tests to check stationarity of the data
    """

    def __init__(self, significance=.05):
        self.SignificanceLevel = significance
        self.pValue = None
        self.isStationary = None

    def ADF_Test(self, timeseries, printResults=True):
        """
        Augmented Dickey-Fuller (ADF) Test
        Null Hypothesis is Unit Root
        Reject Null Hypothesis >> Series is stationary >> Use price levels
        Fail to Reject >> Series has a unit root >> Use price returns
        """

        adfTest = ADF(timeseries)

        self.pValue = adfTest.pvalue

        if (self.pValue < self.SignificanceLevel):
            self.isStationary = True
        else:
            self.isStationary = False

        if printResults:

            print('Augmented Dickey-Fuller (ADF) Test Results: {}'.format(
                'Stationary' if self.isStationary else 'Not Stationary'))

    def PP_Test(self, timeseries, printResults=True):
        """
        Phillips-Perron (PP) Test
        Null Hypothesis is Unit Root
        Reject Null Hypothesis >> Series is stationary >> Use price levels
        Fail to Reject >> Series has a unit root >> Use price returns
        """

        ppTest = PhillipsPerron(timeseries)

        self.pValue = ppTest.pvalue

        if (self.pValue < self.SignificanceLevel):
            self.isStationary = True
        else:
            self.isStationary = False

        if printResults:
            print('Phillips-Perron (PP) Test Results: {}'.format(
                'Stationary' if self.isStationary else 'Not Stationary'))

    def KPSS_Test(self, timeseries, printResults=True):
        """
        Kwiatkowski-Phillips-Schmidt-Shin (KPSS) Test
        Null Hypothesis is Unit Root
        Reject Null Hypothesis >> Series has a unit root >> Use price returns
        Fail to Reject >> Series is stationary >> Use price levels
        """
        kpssTest = KPSS(timeseries)

        self.pValue = kpssTest.pvalue

        if (self.pValue < self.SignificanceLevel):
            self.isStationary = False
        else:
            self.isStationary = True

        if printResults:
            print('Kwiatkowski-Phillips-Schmidt-Shin (KPSS) Test Results: {}'.format(
                'Stationary' if self.isStationary else 'Not Stationary'))

```
## Stationarity Testing
In the same directory, let us create another file named ```catalyst_test.py``` to run those tests.
```python
from catalyst.api import symbol, record
from catalyst import run_algorithm
import numpy as np
import pandas as pd
import stationarity_test  # Importing the script we created earlier


def initialize(context):
    context.asset = symbol('btc_usdt')


def handle_data(context, data):
    # The last known prices of current date and the day before
    yesterday_price, current_price = data.history(
        context.asset, 'price', 2, '1T')
    # Calculate return
    simple_return = current_price / yesterday_price
    # Calculate log return
    log_return = np.log(current_price) - np.log(yesterday_price)
    record(price=current_price, simple_return=simple_return, log_return=log_return)


def analyze(context, perf):
    sTest = stationarity_test.StationarityTests()    

    print('# Price Stationarity Testing')
    sTest.ADF_Test(perf.price)
    sTest.PP_Test(perf.price)
    sTest.KPSS_Test(perf.price)
    print('# Simple Return Stationarity Testing')
    sTest.ADF_Test(perf.simple_return)
    sTest.PP_Test(perf.simple_return)
    sTest.KPSS_Test(perf.simple_return)
    print('# Log Return Stationarity Testing')
    sTest.ADF_Test(perf.log_return)
    sTest.PP_Test(perf.log_return)
    sTest.KPSS_Test(perf.log_return)


if __name__ == '__main__':
    run_algorithm(capital_base=1000,
                  data_frequency='minute',
                  initialize=initialize,
                  handle_data=handle_data,
                  analyze=analyze,
                  exchange_name='poloniex',
                  quote_currency='usdt',
                  start=pd.to_datetime('2018-9-1', utc=True),
                  end=pd.to_datetime('2018-9-3', utc=True))

```
Finally, run this script in the terminal
```
(venv) python catalyst_test.py
```

I hope you enjoyed this tutorial. As always, the source code can be found on my [github](https://github.com/0xboz/stationarity_on_crypto_trading_data) as well. Feel free to shoot me a message if you have any questions/comments. I also created a ```QUANT CHANNEL``` on one of the most popular crypto servers (Ripple XRP). Feel like stopping by and say hi? We have tons of down-to-earth folks who have been in crypto-verse for a long time. Let us talk more about crypto and quantitative trading over there. [Here is the discord invite link](https://discord.gg/jchMcc2).

Stay safe and happy trading. 