---
title: "How to Create Custom Zipline Bundles From Binance Data Part 2"
date: 2019-06-13
tags: [Zipline, Binance]
excerpt: "Zipline custom data bundles from Binance"
header:
    image: /assets/images/crypto_trading_candlesticks.jpg
---

In [part 1](https://0xboz.github.io/blog/how-to-create-custom-zipline-bundles-from-binance-data-part-1/), we have covered how to create custom data bundles from Binance csv files. Today, let us create another module which will allow us to fetch [Binance API](https://github.com/binance-exchange/binance-official-api-docs) trading data and create Zipline bundles instantly. Since there are some similarities between part 1 and part 2 and some setup details will be omitted accordingly in this tutorial, I strongly suggest you do a quick review of part 1 before jumping onto this one. 

Fire up your favorite text editor and create a file named ```binance_api.py``` in the directory of our env. ```$HOME/anaconda3/envs/zipline/lib/python3.5/site-packages/zipline/data/bundles```.

As usual, we need to import some essentials.
```python
import bs4 as bs
from binance.client import Client
import csv
from datetime import datetime as dt
from datetime import timedelta
import numpy as np
from os import listdir, mkdir, remove
from os.path import exists, isfile, join
from pathlib import Path
import pandas as pd
import pickle
import requests
from trading_calendars import register_calendar
from trading_calendars.exchange_calendar_binance import BinanceExchangeCalendar
```
If you have followed my tutorials all along, you should have all packages installed already. BinanceExchangeCalendar is a custom trading calendar created in my tutorial [part 1](https://0xboz.github.io/blog/how-to-create-custom-zipline-bundles-from-binance-data-part-1/). Here is [our Zipline installation on Debian 9 Stretch](https://0xboz.github.io/blog/how-to-install-zipline-debian-stretch/). Once you have all the pre-requisites in place, you can move onto the next step.

## Get Binance Trading Pair Tickers
Now, let us set up some variables.
```python
user_home = str(Path.home())
custom_data_path = join(user_home, '.zipline/custom_data')
```
Create one function to collection all Binance trading ticker pairs and another as a ticker pair generator.
```python
def tickers():
    """
    Save Binance trading pair tickers to a pickle file
    Return a list of trading ticker pairs
    """
    cmc_binance_url = 'https://coinmarketcap.com/exchanges/binance/'
    response = requests.get(cmc_binance_url)
    if response.ok:
        soup = bs.BeautifulSoup(response.text, 'html.parser')
        table = soup.find('table', {'id': 'exchange-markets'})
        ticker_pairs = []

        for row in table.findAll('tr')[1:]:
            ticker_pair = row.findAll('td')[2].text
            ticker_pairs.append(ticker_pair.strip().replace('/', ''))

    if not exists(custom_data_path):
        mkdir(custom_data_path)

    with open(join(custom_data_path, 'binance_ticker_pairs.pickle'), 'wb') as f:
        pickle.dump(ticker_pairs, f)

    return ticker_pairs

def tickers_generator():
    """
    Return a tuple (sid, ticker_pair)
    """
    tickers_file = join(custom_data_path, 'binance_ticker_pairs.pickle')
    if not isfile(tickers_file):
        ticker_pairs = tickers()

    else:
        with open(tickers_file, 'rb') as f:
            ticker_pairs = pickle.load(f)[:]

    return (tuple((sid, ticker)) for sid, ticker in enumerate(ticker_pairs))
```
## Dataframes for Metadata & Ticker Data
Next, we are going to create two functions which can process the dataframe for ticker pair meta data and trading data, respectively.
```python
def metadata_df():
    """
    Return meta data dataframe
    """
    metadata_dtype = [
        ('symbol', 'object'),
        ('asset_name', 'object'),
        ('start_date', 'datetime64[ns]'),
        ('end_date', 'datetime64[ns]'),
        ('first_traded', 'datetime64[ns]'),
        ('auto_close_date', 'datetime64[ns]'),
        ('exchange', 'object'), ]
    metadata_df = pd.DataFrame(
        np.empty(len(tickers()), dtype=metadata_dtype))

    return metadata_df

def df_generator(interval):
    """
    Return a tuple of sid and trading dataframe, which later can used by daily_bar_writer or minute_bar_writer
    """
    client = Client("", "")
    start = '2017-7-14'  # Binance launch date
    end = dt.utcnow().strftime('%Y-%m-%d')  # Current day

    for item in tickers_generator():

        sid = item[0]
        ticker_pair = item[1]
        df = pd.DataFrame(
            columns=['date', 'open', 'high', 'low', 'close', 'volume'])

        symbol = ticker_pair
        asset_name = ticker_pair
        exchange = 'Binance'

        klines = client.get_historical_klines_generator(
            ticker_pair, interval, start, end)

        for kline in klines:
            line = kline[:]
            del line[6:]
            # Make a real copy of kline
            # Binance API forbids the change of open time
            line[0] = np.datetime64(line[0], 'ms')
            line[0] = pd.Timestamp(line[0], 'ms')
            df.loc[len(df)] = line

        df['date'] = pd.to_datetime(df['date'])
        df.set_index('date', inplace=True)
        df = df.astype({'open': 'float64', 'high': 'float64',
                        'low': 'float64', 'close': 'float64', 'volume': 'float64'})

        start_date = df.index[0]
        end_date = df.index[-1]
        first_traded = start_date
        auto_close_date = end_date + pd.Timedelta(days=1)

        # Check if there is no missing session; skip the ticker pair otherwise
        if interval == '1d' and len(df.index) - 1 != pd.Timedelta(end_date - start_date).days:            
            continue
        elif interval == '1m' and timedelta(minutes=(len(df.index) + 60)) != end_date - start_date:            
            continue

        yield (sid, df), symbol, asset_name, start_date, end_date, first_traded, auto_close_date, exchange
```
## Create Custom Ingest Function
Great! It is time for our custom ingest function.
```python
def api_to_bundle(interval='1m'):

    def ingest(environ,
               asset_db_writer,
               minute_bar_writer,
               daily_bar_writer,
               adjustment_writer,
               calendar,
               start_session,
               end_session,
               cache,
               show_progress,
               output_dir
               ):

        metadata = metadata_df()

        def minute_data_generator():
            return (sid_df for (sid_df, *metadata.iloc[sid_df[0]]) in df_generator(interval='1m'))

        def daily_data_generator():
            return (sid_df for (sid_df, *metadata.iloc[sid_df[0]]) in df_generator(interval='1d'))

        if interval == '1d':
            daily_bar_writer.write(
                daily_data_generator(), show_progress=True)
        elif interval == '1m':
            minute_bar_writer.write(
                minute_data_generator(), show_progress=True)

        # Drop the ticker rows which have missing sessions in their data sets
        metadata.dropna(inplace=True)

        asset_db_writer.write(equities=metadata)
        print(metadata)

    return ingest

# Register custom trading calendar
register_calendar('Binance_api', BinanceExchangeCalendar())
```
## Update Extension.py
Last but not the least, we need to update ```$HOME/.zipline/extension.py```
```python
from zipline.data.bundles import register
from zipline.data.bundles.binance_api import api_to_bundle

register(
    'binance_api',
    api_to_bundle(interval='1d'),
    calendar_name='Binance_api',
)
```

## Put-it-all-together

```python
import bs4 as bs
from binance.client import Client
import csv
from datetime import datetime as dt
from datetime import timedelta
import numpy as np
from os import listdir, mkdir, remove
from os.path import exists, isfile, join
from pathlib import Path
import pandas as pd
import pickle
import requests
from trading_calendars import register_calendar
from trading_calendars.exchange_calendar_binance import BinanceExchangeCalendar


user_home = str(Path.home())
custom_data_path = join(user_home, '.zipline/custom_data')

def tickers():
    """
    Save Binance trading pair tickers to a pickle file
    Return a list of trading ticker pairs
    """
    cmc_binance_url = 'https://coinmarketcap.com/exchanges/binance/'
    response = requests.get(cmc_binance_url)
    if response.ok:
        soup = bs.BeautifulSoup(response.text, 'html.parser')
        table = soup.find('table', {'id': 'exchange-markets'})
        ticker_pairs = []

        for row in table.findAll('tr')[1:]:
            ticker_pair = row.findAll('td')[2].text
            ticker_pairs.append(ticker_pair.strip().replace('/', ''))

    if not exists(custom_data_path):
        mkdir(custom_data_path)

    with open(join(custom_data_path, 'binance_ticker_pairs.pickle'), 'wb') as f:
        pickle.dump(ticker_pairs, f)

    return ticker_pairs

def tickers_generator():
    """
    Return a tuple (sid, ticker_pair)
    """
    tickers_file = join(custom_data_path, 'binance_ticker_pairs.pickle')
    if not isfile(tickers_file):
        ticker_pairs = tickers()

    else:
        with open(tickers_file, 'rb') as f:
            ticker_pairs = pickle.load(f)[:]

    return (tuple((sid, ticker)) for sid, ticker in enumerate(ticker_pairs))

def metadata_df():
    """
    Return meta data dataframe
    """
    metadata_dtype = [
        ('symbol', 'object'),
        ('asset_name', 'object'),
        ('start_date', 'datetime64[ns]'),
        ('end_date', 'datetime64[ns]'),
        ('first_traded', 'datetime64[ns]'),
        ('auto_close_date', 'datetime64[ns]'),
        ('exchange', 'object'), ]
    metadata_df = pd.DataFrame(
        np.empty(len(tickers()), dtype=metadata_dtype))

    return metadata_df

def df_generator(interval):
    """
    Return a tuple of sid and trading dataframe, which later can used by daily_bar_writer or minute_bar_writer
    """
    client = Client("", "")
    start = '2017-7-14'  # Binance launch date
    end = dt.utcnow().strftime('%Y-%m-%d')  # Current day

    for item in tickers_generator():

        sid = item[0]
        ticker_pair = item[1]
        df = pd.DataFrame(
            columns=['date', 'open', 'high', 'low', 'close', 'volume'])

        symbol = ticker_pair
        asset_name = ticker_pair
        exchange = 'Binance'

        klines = client.get_historical_klines_generator(
            ticker_pair, interval, start, end)

        for kline in klines:
            line = kline[:]
            del line[6:]
            # Make a real copy of kline
            # Binance API forbids the change of open time
            line[0] = np.datetime64(line[0], 'ms')
            line[0] = pd.Timestamp(line[0], 'ms')
            df.loc[len(df)] = line

        df['date'] = pd.to_datetime(df['date'])
        df.set_index('date', inplace=True)
        df = df.astype({'open': 'float64', 'high': 'float64',
                        'low': 'float64', 'close': 'float64', 'volume': 'float64'})

        start_date = df.index[0]
        end_date = df.index[-1]
        first_traded = start_date
        auto_close_date = end_date + pd.Timedelta(days=1)

        # Check if there is no missing session; skip the ticker pair otherwise
        if interval == '1d' and len(df.index) - 1 != pd.Timedelta(end_date - start_date).days:            
            continue
        elif interval == '1m' and timedelta(minutes=(len(df.index) + 60)) != end_date - start_date:            
            continue

        yield (sid, df), symbol, asset_name, start_date, end_date, first_traded, auto_close_date, exchange

def api_to_bundle(interval='1m'):

    def ingest(environ,
               asset_db_writer,
               minute_bar_writer,
               daily_bar_writer,
               adjustment_writer,
               calendar,
               start_session,
               end_session,
               cache,
               show_progress,
               output_dir
               ):

        metadata = metadata_df()

        def minute_data_generator():
            return (sid_df for (sid_df, *metadata.iloc[sid_df[0]]) in df_generator(interval='1m'))

        def daily_data_generator():
            return (sid_df for (sid_df, *metadata.iloc[sid_df[0]]) in df_generator(interval='1d'))

        if interval == '1d':
            daily_bar_writer.write(
                daily_data_generator(), show_progress=True)
        elif interval == '1m':
            minute_bar_writer.write(
                minute_data_generator(), show_progress=True)

        # Drop the ticker rows which have missing sessions in their data sets
        metadata.dropna(inplace=True)

        asset_db_writer.write(equities=metadata)
        print(metadata)

    return ingest

# Register custom trading calendar
register_calendar('Binance_api', BinanceExchangeCalendar())
```
Here is a sample output.
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/binance_api_zipline_bundle_results.jpg" alt="binance api zipline custom bundle">

Again, you can also find the source code on my [github](https://github.com/0xboz/zipline_bundle) as always. I hope you have enjoyed my tutorials so far. Shoot me a message if you have questions and comments. Ciao!