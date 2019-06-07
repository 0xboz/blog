---
title: "How to Download Binance Ticker Pairs"
date: 2019-06-07
tags: [binance, ]
excerpt: "A simple python script to download all Binance trading pairs"
---
I am planning to create custom data bundles for [Zipline](http://www.zipline.io/). [Binance](https://www.binance.com/) is the leading crypto exchange as of this writing and it provides massive trading data via its [public API](https://github.com/binance-exchange/binance-official-api-docs). A series of upcoming tutorials will dive deep into crypto data bundles for Zipline.

I suppose your OS has Python installed already. If not, take your time and check out [this tutorial](https://www.beginnersbook.in/how-to-install-python-windows-linux-mac/).

For the best practice, let us create a virtual environment first. (A quick note: my desktop is running on Debian 9 Stretch)

```
mkdir get_binance_tickers
cd get_binance_tickers
python3 -m venv venv
```
Activate the venv

```
source venv/bin/activate
```
Update pip
```
(venv) pip install --upgrade pip
```
Install requests and BeautifulSoup4
```
(venv) pip install requests BeautifulSoup4
```

We are going to scrape Binance page on [CoinMarketCap](https://coinmarketcap.com), and save all trading pairs into a pickle file for later use.

Create get_binance_tickers.py and fire up your favorite editor.
```
(venv) touch get_binance_tickers.py
```

Here is the python code, which also can be found on my github page.
```python
#! /usr/bin/env python
import bs4 as bs
import requests
import pickle


def save_ticker_pairs():
    cmc_binance_url = 'https://coinmarketcap.com/exchanges/binance/'
    response = requests.get(cmc_binance_url)
    if response.ok:
        soup = bs.BeautifulSoup(response.text, 'html.parser')
        table = soup.find('table', {'id': 'exchange-markets'})
        ticker_pairs = []

        for row in table.findAll('tr')[1:]:
            ticker_pair = row.findAll('td')[2].text
            ticker_pairs.append(ticker_pair.strip().replace('/', ''))

    with open('binance_ticker_pairs.pickle', 'wb') as f:
        pickle.dump(ticker_pairs, f)


if __name__ == '__main__':
    save_ticker_pairs()

```

Make the file excutable and run the command
```
(venv) chmod +x get_binance_tickers.py
(venv) python get_binance_tickers.py
```

Before we are going to use 'binance_ticker_pairs.pickle' and retrieve trading data through Binance public API, I will discuss how to install Zipline in the next tutorial.