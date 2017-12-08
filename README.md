# Cryptonite
A look at correlation between cryptocurrencies

## FINC-780

### Project C1 – Is there a pattern between cryptocurrencies?

#### Overview

Cryptocurrency is a digital or virtual currency, powered by blockchain technology and act as a medium of exchange using cryptography. It is hard to counterfeit due to its cryptographic security and not controlled by any central authority. This feature makes it immune to government interference or manipulation. Currently there is a lot of self-proclaimed experts on the trends and prediction of price movement. Even so, most of these analysis lacks a strong foundation of data and statistics to back up the claims.

The purpose of this project is to determine whether there is any pattern on prices between different cryptocurrencies using correlation in python. We will collect data, analyze and visualize data on several cryptocurrencies. If there is a pattern we will then look further, using Bitcoin as a control variable and investigate other similar altcoins for correlation.
This document highlights the code used within the Jupyter Notebook. For the results and graphs please refer to the Jupyter Notebook.

### Coding explanation

*Class CryptoCompare*

To collect data for the cryptocurrency prices we created a class called CryptoCompare in python. Within CryptoCompare we created three main functions: extract raw data, time period specification and data depository. 
Cryptocompare API is used within function Cryptocompare_histo to collect and retrieve data from the several exchanges to create an aggregate of the prices. Notice the url has the term CCCAGG which stands for CryptoCompare Current Aggregate. 



```python
    def cryptocompare_histo(self, pair):    
        url = 'https://min-api.cryptocompare.com/data/{1}?fsym={0}&tsym=USD&limit=2000&aggregate=3&e=CCCAGG'.format(pair, self.period)
        # changing the limit number allows us to control the number of observation that we want to analyze.
        # Here we download data for 2000 observations which is the maximum limit.
        # This API allows us to extract data for every 3 time unit.
        resp = requests.get(url=url)
        data = json.loads(resp.text)
        cols = data['Data'][0].keys()
        btc_frame = pd.DataFrame(data['Data'], columns = cols)
        btc_frame['dtime'] = (pd.to_datetime(btc_frame['time'],unit='s'))
        return btc_frame
```
