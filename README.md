# Cryptonite
A look at correlation between cryptocurrencies

## FINC-780

### Project C1 – Is there a pattern between cryptocurrencies?

#### Overview

Cryptocurrency is a digital or virtual currency, powered by blockchain technology and act as a medium of exchange using cryptography. It is hard to counterfeit due to its cryptographic security and not controlled by any central authority. This feature makes it immune to government interference or manipulation. Currently there is a lot of self-proclaimed experts on the trends and prediction of price movement. Even so, most of these analysis lacks a strong foundation of data and statistics to back up the claims.

The purpose of this project is to determine whether there is any pattern on prices between different cryptocurrencies using correlation in python. We will collect data, analyze and visualize data on several cryptocurrencies. If there is a pattern we will then look further, using Bitcoin as a control variable and investigate other similar altcoins for correlation.
This document highlights the code used within the Jupyter Notebook. For the results and graphs please refer to the Jupyter Notebook [viewer](http://nbviewer.jupyter.org/github/apcalypso/Cryptonite/blob/25b90adb460b52f527372064eaf15869f98f76ce/PROJECT%20C.ipynb).

### Coding explanation

**Class CryptoCompare**

To collect data for the cryptocurrency prices we created a class called **CryptoCompare** in python. Within **CryptoCompare** we created three main functions: extract raw data, time period specification and data depository. 
Cryptocompare API is used within function **Cryptocompare_histo** to collect and retrieve data from the several exchanges to create an aggregate of the prices. Notice the url has the term **CCCAGG** which stands for *CryptoCompare Current Aggregate*. 



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

While retrieving the raw data we want to specify the time unit of the data in minutes, hour or day. Since we want to research on the price pattern we need to compare across different time periods. The two function below **time_period** and **check_time_interval** within **CryptoCompare** would do the trick.

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

After specifying the time period and extracting the data, we want to save the data into a csv file. This is to prevent extracting data repetitively (i.e. wasting time and space) to analyze the data and also to prevent the download of redundant data that has already existed in the **FinalFantasy** folder.

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

**merge_dfs_on_column** is a for loop that merges N number of data frames so that we can do analysis on N number of crypto-assets, which provides flexibility moving forward.

```python
def merge_dfs_on_column(dataframes, labels):
    '''Merge a single column of each dataframe into a new combined dataframe'''
    series_dict = {}
    for index in range(len(dataframes)):
        series_dict[labels[index]] = dataframes[index][labels[index]]
    return pd.DataFrame(series_dict)
```
**Scatter Plot**

To make the scatter plot more interactive, trace called **trace_arr** is added within the function **df_scatter**. By hovering the cursor over the graph, all the cryptocurrency prices can be viewed for a specific point of time. 

```python
def df_scatter(df, title, seperate_y_axis=False, y_axis_label='', scale='linear', initial_hide=False):
    '''Generate a scatter plot of the entire dataframe'''
    label_arr = list(df)
    series_arr = list(map(lambda col: df[col], label_arr))
    
    layout = go.Layout(
        title=title,
        legend=dict(orientation="h"),
        xaxis=dict(type='date'),
        yaxis=dict(
            title=y_axis_label,
            showticklabels= not seperate_y_axis,
            type=scale))
    
    y_axis_config = dict(
        overlaying='y',
        showticklabels=False,
        type=scale )
    
    visibility = 'visible'
    if initial_hide:
        visibility = 'legendonly'
        
    # Form Trace For Each Series
    trace_arr = []
    for index, series in enumerate(series_arr):
        trace = go.Scatter(
            x=series.index, 
            y=series, 
            name=label_arr[index],
            visible=visibility
        )        
        # Add seperate axis for the series
        if seperate_y_axis:
            trace['yaxis'] = 'y{}'.format(index + 1)
            layout['yaxis{}'.format(index + 1)] = y_axis_config    
        trace_arr.append(trace)

    fig = go.Figure(data=trace_arr, layout=layout)
    py.iplot(fig)
```

**Correlation Heatmap**

**correlation_heatmap** is created to visually show the calculated correlation across different cryptocurrency prices. **Colorscale** ‘Jet’ is used to create obvious contrast colors to review the results.

```python
def correlation_heatmap(df, title, colorscale, absolute_bounds=True):
    heatmap = go.Heatmap(
        z=df.corr(method='pearson').as_matrix(),
        x=df.columns,
        y=df.columns,
        colorscale=colorscale,  
        reversescale=True,
        colorbar=dict(title='Pearson Coefficient'),)
    layout = go.Layout(title=title)
    
    if absolute_bounds:
        heatmap['zmax'] = 1.0
        heatmap['zmin'] = -1.0
        
    fig = go.Figure(data=[heatmap], layout=layout)
    py.iplot(fig)
```

**Resizing Plot Output**

Sometime the graphs are too small to view, the code below is used to configure the size of the graph. **Figsiz**e is used to customize the base and length of the graph, **dpi** shows the dots per inch to change the resolution of the graph line.

plt.figure(figsize=(15, 10), dpi=130)

### Source Citations
Cryptocompare API. Cryptocompare.com.
    https://www.cryptocompare.com/api/#

Triest, Patrick. Analyzing Cryptocurrency markets Using Python. Blog.patricktriest.com. 2017 Aug 20.
	https://blog.patricktriest.com/analyzing-cryptocurrencies-python/

Murray, Nate. 100 cryptocurrencies described in four words or less. Techcrunch.com. 2017 Nov 19.
    https://techcrunch.com/2017/11/19/100-cryptocurrencies-described-in-4-words-or-less/

Block Geeks. What is Cryptocurrency: Everything You Need To Know [Ultimate Guide]. 2016. Blockgeeks.com. 
	https://blockgeeks.com/guides/what-is-cryptocurrency/

Heatmaps, contours and 2d histograms in Python. 
	https://plot.ly/python/heatmaps-contours-and-2dhistograms-tutorial/
