---
title: "How to Create Custom Zipline Bundles From Binance Data Part 1"
date: 2019-06-11
tags: [Zipline, Binance]
excerpt: "Zipline custom data bundles from Binance"
header:
    image: /assets/images/matrix_data.jpg
---

We have successfully [installed Zipline](https://0xboz.github.io/blog/how-to-install-zipline-debian-stretch/) and [downloaded all trading pairs](https://0xboz.github.io/blog/how-to-download-binance-trading-pair-tickers/) from Binance. Now it is time to create custom data bundles from those data sets. In tutorial part 1, I am going to show you how to create the data bundle from csv files. In part 2, we are going to skip downloading csv files and create Zipline data bundles directly from Binance public API.



First, create a file named ```binance_csv.py```, and import all the modules we need.
Note: Save this file into Zipline package folder in our env ```$HOME/anaconda3/envs/zipline/lib/python3.5/site-packages/zipline/data/bundles```
```python
import bs4 as bs
from binance.client import Client
import csv
from datetime import datetime as dt
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

If you have followed my previous tutorials, you should have all the packages in the env except ```binance.client```. Let us get that out of the way. 
```
(zipline) pip install python-binance
```

Don't worry about ```BinanceExchangeCalendar``` for now, we will create it later. 


## Binance Data in CSV
Create a script to download Binance trading data in csv.
```python
# Set up the directories where we are going to save those csv files
user_home = str(Path.home())
csv_data_path = join(user_home, '.zipline/custom_data/csv')
custom_data_path = join(user_home, '.zipline/custom_data')

def save_csv(reload_tickers=False, interval='1m'):
    """
    Save Zipline bundle ready csv for Binance trading ticker pair
    :param reload_tickers: True or False
    :type reload_tickers: boolean
    :param interval: Default 1m. Other available ones: 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d, 3d, 1w, 1M
    :type interval: str
    """

    if not exists(csv_data_path):
        mkdir(csv_data_path)

    if reload_tickers:
        ticker_pairs = tickers()  # tickers() will be included in put-it-all-together session
    else:
        ticker_pickle = join(
            custom_data_path, 'binance_ticker_pairs.pickle')
        with open(ticker_pickle, 'rb') as f:
            ticker_pairs = pickle.load(f)

    client = Client("", "")
    start = '2017-7-14'  # Binance launch date
    end = dt.utcnow().strftime('%Y-%m-%d')  # Current day
    csv_filenames = [csv_filename for csv_filename in listdir(
        csv_data_path) if isfile(join(csv_data_path, csv_filename))]

    for ticker_pair in ticker_pairs:
        filename = "Binance_{}_{}.csv".format(ticker_pair, interval)        

        if csv_filenames != [] and filename in csv_filenames:
            remove(join(csv_data_path, filename))

        output = join(csv_data_path, filename)
        klines = client.get_historical_klines_generator(
            ticker_pair, interval, start, end)
        for index, kline in enumerate(klines):
            with open(output, 'a+') as f:
                writer = csv.writer(f)
                if index == 0:
                    writer.writerow(
                        ['date', 'open', 'high', 'low', 'close', 'volume'])
                # Make a real copy of kline
                # Binance API forbids the change of open time
                line = kline[:]
                del line[6:]
                line[0] = np.datetime64(line[0], 'ms')
                line[0] = pd.Timestamp(line[0], 'ms')
                writer.writerow(line)

        print('{} saved.'.format(filename))

    return [file for file in listdir(csv_data_path) if isfile(join(csv_data_path, file))]
```
## Create Custom Ingest Function
Once all csv have been downloaded, we need a function to loop through those files and create custom bundles.

```python
def csv_to_bundle(interval='1m'):

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

        # Get all available csv filenames
        csv_filenames = save_csv(reload_tickers=True, interval=interval)
        # Loop through the filenames and create a dict to keep some temp meta data
        ticker_pairs = [{'exchange': pair.split('_')[0],
                         'symbol': pair.split('_')[1],
                         'interval':pair.split('_')[2].split('.')[0],
                         'file_path':join(csv_data_path, pair)}
                        for pair in csv_filenames]

        # Create an empty meta data dataframe
        metadata_dtype = [
            ('symbol', 'object'),
            ('asset_name', 'object'),
            ('start_date', 'datetime64[ns]'),
            ('end_date', 'datetime64[ns]'),
            ('first_traded', 'datetime64[ns]'),
            ('auto_close_date', 'datetime64[ns]'),
            ('exchange', 'object'), ]
        metadata = pd.DataFrame(
            np.empty(len(ticker_pairs), dtype=metadata_dtype))

        minute_data_sets = []
        daily_data_sets = []

        for sid, ticker_pair in enumerate(ticker_pairs):
            df = pd.read_csv(ticker_pair['file_path'],
                             index_col=['date'],
                             parse_dates=['date'])

            symbol = ticker_pair['symbol']
            asset_name = ticker_pair['symbol']
            start_date = df.index[0]
            end_date = df.index[-1]
            first_traded = start_date
            auto_close_date = end_date + pd.Timedelta(days=1)
            exchange = ticker_pair['exchange']

            # Update metadata
            metadata.iloc[sid] = symbol, asset_name, start_date, end_date, first_traded, auto_close_date, exchange

            if ticker_pair['interval'] == '1m':
                minute_data_sets.append((sid, df))

            if ticker_pair['interval'] == '1d':
                daily_data_sets.append((sid, df))

        if minute_data_sets != []:
            minute_bar_writer.write(minute_data_sets, show_progress=True)

        if daily_data_sets != []:
            daily_bar_writer.write(daily_data_sets, show_progress=True)
        
        asset_db_writer.write(equities=metadata)
        print(metadata)

    return ingest
```

## Create Custom Trading Calendar

Looking good! Next, we can create a custom trading calendar for Binance. Since the release of [Harrison's tutorial on this topic](https://www.youtube.com/watch?v=rYV102VfG7o), Quantopian has moved the trading calendar into a separate package. In our setup, you should be find them in this directory ```$HOME/anaconda3/envs/zipline/lib/python3.5/site-packages/trading_calendars```

Fortunately, we don't have to write up our own 24/7 calendar from scratch. We just need to copy ```always_open.py``` and paste it as ```exchange_calendar_binance.py```. Fire up your favorite text editor and change a few things.
```python
from datetime import time

from trading_calendars import TradingCalendar
from trading_calendars import register_calendar
from zipline.utils.memoize import lazyval


class BinanceExchangeCalendar(TradingCalendar):
    """A TradingCalendar for an exchange that's open every minute of every day.
    """
    name = 'Binance'
    tz = 'UTC'
    weekmask = '1111111'
    open_times = (
        (None, time(0)),
    )
    close_times = (
        (None, time(23, 59)),
    )

```
## Put-it-all-together
Let us put all together for ```binance_csv.py```

```python
import bs4 as bs
from binance.client import Client
import csv
from datetime import datetime as dt
import numpy as np
from os import listdir, mkdir, remove
from os.path import exists, isfile, join
from pathlib import Path
import pandas as pd
import pickle
import requests
from trading_calendars import register_calendar
from trading_calendars.exchange_calendar_binance import BinanceExchangeCalendar

# Set up the directories where we are going to save those csv files
user_home = str(Path.home())
csv_data_path = join(user_home, '.zipline/custom_data/csv')
custom_data_path = join(user_home, '.zipline/custom_data')

def tickers():
    """
    Save Binance trading pair tickers to a pickle file
    Return a pickle
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

def save_csv(reload_tickers=False, interval='1m'):
    """
    Save Zipline bundle ready csv for Binance trading ticker pair
    :param reload_tickers: True or False
    :type reload_tickers: boolean
    :param interval: Default 1m. Other available ones: 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d, 3d, 1w, 1M
    :type interval: str
    """

    if not exists(csv_data_path):
        mkdir(csv_data_path)

    if reload_tickers:
        ticker_pairs = tickers()
    else:
        ticker_pickle = join(
            custom_data_path, 'binance_ticker_pairs.pickle')
        with open(ticker_pickle, 'rb') as f:
            ticker_pairs = pickle.load(f)

    client = Client("", "")
    start = '2017-7-14'  # Binance launch date
    end = dt.utcnow().strftime('%Y-%m-%d')  # Current day
    csv_filenames = [csv_filename for csv_filename in listdir(
        csv_data_path) if isfile(join(csv_data_path, f))]

    for ticker_pair in ticker_pairs:
        filename = "Binance_{}_{}.csv".format(ticker_pair, interval)        

        if csv_filenames != [] and filename in csv_filenames:
            remove(join(csv_data_path, filename))

        output = join(csv_data_path, filename)
        klines = client.get_historical_klines_generator(
            ticker_pair, interval, start, end)
        for index, kline in enumerate(klines):
            with open(output, 'a+') as f:
                writer = csv.writer(f)
                if index == 0:
                    writer.writerow(
                        ['date', 'open', 'high', 'low', 'close', 'volume'])
                # Make a real copy of kline
                # Binance API forbids the change of open time
                line = kline[:]
                del line[6:]
                line[0] = np.datetime64(line[0], 'ms')
                line[0] = pd.Timestamp(line[0], 'ms')
                writer.writerow(line)

        print('{} saved.'.format(filename))

    return [file for file in listdir(csv_data_path) if isfile(join(csv_data_path, f))]

def csv_to_bundle(interval='1m'):

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

        # Get all available csv filenames
        csv_filenames = save_csv(reload_tickers=True, interval=interval)
        # Loop through the filenames and create a dict to keep some temp meta data
        ticker_pairs = [{'exchange': pair.split('_')[0],
                         'symbol': pair.split('_')[1],
                         'interval':pair.split('_')[2].split('.')[0],
                         'file_path':join(csv_data_path, pair)}
                        for pair in csv_filenames]

        # Create an empty meta data dataframe
        metadata_dtype = [
            ('symbol', 'object'),
            ('asset_name', 'object'),
            ('start_date', 'datetime64[ns]'),
            ('end_date', 'datetime64[ns]'),
            ('first_traded', 'datetime64[ns]'),
            ('auto_close_date', 'datetime64[ns]'),
            ('exchange', 'object'), ]
        metadata = pd.DataFrame(
            np.empty(len(ticker_pairs), dtype=metadata_dtype))

        minute_data_sets = []
        daily_data_sets = []

        for sid, ticker_pair in enumerate(ticker_pairs):
            df = pd.read_csv(ticker_pair['file_path'],
                             index_col=['date'],
                             parse_dates=['date'])

            symbol = ticker_pair['symbol']
            asset_name = ticker_pair['symbol']
            start_date = df.index[0]
            end_date = df.index[-1]
            first_traded = start_date
            auto_close_date = end_date + pd.Timedelta(days=1)
            exchange = ticker_pair['exchange']

            # Update metadata
            metadata.iloc[sid] = symbol, asset_name, start_date, end_date, first_traded, auto_close_date, exchange

            if ticker_pair['interval'] == '1m':
                minute_data_sets.append((sid, df))

            if ticker_pair['interval'] == '1d':
                daily_data_sets.append((sid, df))

        if minute_data_sets != []:
            minute_bar_writer.write(minute_data_sets, show_progress=True)

        if daily_data_sets != []:
            daily_bar_writer.write(daily_data_sets, show_progress=True)
        
        asset_db_writer.write(equities=metadata)
        print(metadata)

    return ingest
```
We are almost there! However, we still need add a few more lines in ```$HOME/.zipline/extension.py``` and tell Zipline to ingest the data as the way we wanted.
```python
from zipline.data.bundles import register
from zipline.data.bundles.binance_csv import csv_to_bundle

register(
    'binance_csv',
    csv_to_bundle(interval='1d'), # Daily ('1d') or Minute ('1m') Data
    calendar_name='Binance_csv',
)

```
## How to Run
This is the easy part. Activate the env and run the ingest command as usual.
```
conda activate zipline
(zipline) zipline ingest -b binance_csv
```

In part 2 of this tutorial, we are going to take a look how to create custom data bundles directly from Binance public API. Cheers!