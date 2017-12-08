# Cryptonite
A look at correlation between cryptocurrencies

## FINC-780

### Project C1 – Is there a pattern between cryptocurrencies?

#### Overview

Cryptocurrency is a digital or virtual currency, powered by blockchain technology and act as a medium of exchange using cryptography. It is hard to counterfeit due to its cryptographic security and not controlled by any central authority. This feature makes it immune to government interference or manipulation. Currently there is a lot of self-proclaimed experts on the trends and prediction of price movement. Even so, most of these analysis lacks a strong foundation of data and statistics to back up the claims.

The purpose of this project is to determine whether there is any pattern on prices between different cryptocurrencies using correlation in python. We will collect data, analyze and visualize data on several cryptocurrencies. If there is a pattern we will then look further, using Bitcoin as a control variable and investigate other similar altcoins for correlation.
This document highlights the code used within the Jupyter Notebook. For the results and graphs please refer to the Jupyter Notebook.

### Coding explanation

**Class CryptoCompare**

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

While retrieving the raw data we want to specify the time unit of the data in minutes, hour or day. Since we want to research on the price pattern we need to compare across different time periods. The two function below time_period and check_time_interval within CryptoCompare would do the trick.

```python
    def time_period(self, name):
        self.period = self.check_time_interval(name)
        return self

    def check_time_interval(self, value):
        return {
            'day': 'histoday',
            'hour': 'histohour',
            'minute': 'histominute'
        }[value]
```

After specifying the time period and extracting the data, we want to save the data in a csv file. This is to prevent extracting data repetitively (i.e. wasting time and space) to analyze the data and also to prevent the download of redundant data that has already existed in the FinalFantasy folder.

```python
    def done(self):
        directory = "FinalFantasy"
        # Create a loop to get multiple cryptocurrencies in USD Value
        if self.is_save == True:
            print("Saving dataframe to CSV")
            
            try:
                os.makedirs(directory)
            except OSError as e:
                print(e.errno)
                if e.errno != errno.EEXIST:
                    print("Folder exist")
                else:
                    print("Creating Directory " + directory)
        for crypto in self.crypto_list:
             #print(crypto)
            kryptonite = self.cryptocompare_histo(crypto)
            if self.is_save == True:
                starttime = kryptonite.iloc[0].time
                endtime = kryptonite.iloc[-1].time
                period = self.period
                name = crypto
                
                # Folder/starttime-endtime-period-cryptoname - CREATING FOLDER AND LABELING
                file_name = "{}/{}-{}-{}-{}".format(directory, starttime, endtime, period, name)
                if os.path.isfile(file_name) == False:
                    kryptonite.to_csv(file_name, sep='\t')
                else:
                    print("File already exist")
            self.crypto_dict[crypto] = kryptonite
        return self
```

**Merging Data**

merge_dfs_on_column is a for loop that merges N number of data frames so that we can do analysis on N number of crypto-assets, which provides flexibility moving forward.

```python
def merge_dfs_on_column(dataframes, labels):
    '''Merge a single column of each dataframe into a new combined dataframe'''
    series_dict = {}
    for index in range(len(dataframes)):
        series_dict[labels[index]] = dataframes[index][labels[index]]
    return pd.DataFrame(series_dict)
```





