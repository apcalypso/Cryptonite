# Cryptonite
A look at correlation between cryptocurrencies


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
