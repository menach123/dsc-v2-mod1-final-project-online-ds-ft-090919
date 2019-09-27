
## Final Project Submission

Please fill out:
* Student name: 
* Student pace: self paced / part time / full time
* Scheduled project review date/time: 
* Instructor name: 
* Blog post URL:



```python
import numpy as np
import pandas as pd

import scipy.stats as scs

import matplotlib.pyplot as plt
import seaborn as sns

import scipy.stats as scs
import statsmodels.api as sm

from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split, cross_val_score

import warnings
warnings.filterwarnings('ignore')

from sklearn.metrics import mean_squared_error
from sklearn.model_selection import cross_val_score

import folium
from folium.plugins import HeatMap

target= 'price'
```

### Custom Function


```python


def kfolds(data, k):
    #Force data as pandas dataframe
    data = pd.DataFrame(data)
    num_observations = len(data)
    fold_size = num_observations//k
    leftovers = num_observations%k
    folds = []
    start_obs = 0
    for fold_n in range(1,k+1):
        if fold_n <= leftovers:
            #Fold Size will be 1 larger to account for leftovers
            fold =  data.iloc[start_obs : start_obs+fold_size+1] 
            folds.append(fold)
            start_obs +=  fold_size + 1
        else:
            fold =  data.iloc[start_obs : start_obs+fold_size] 
            folds.append(fold)
            start_obs +=  fold_size
            
    return folds 

def resd_check(results, data, column):
    a=results.resid/results.resid.std()
    a = a.loc[a.abs() > 2]
    
    
    plt.figure(figsize= (13,8))
    sns.distplot(data[column].iloc[a.index])
    sns.distplot(data[column])
    plt.title('Checking Dist. Initial v. High,\n'+column)
    return

def binning_count(data_series, bins): # binning  numerical features
    data_series = data_series.astype(float)
    bins_rad = pd.cut(data_series, bins)
    print(bins_rad.isna().sum())
    bins_rad = bins_rad.cat.as_ordered()
    bins_rad = bins_rad.sort_values()
    return bins_rad.value_counts()

def binning(data_series, bins):
    data_series = data_series.astype(float)
    return pd.cut(data_series, bins)

def flip_diff(dataframe, column):
    dataout=[]
    id_ = None
    previous_index = None
    for value in dataframe.id.unique():
            for index in dataframe.loc[dataframe.id == value].index:
                
                if id_ == dataframe.id.loc[index]:
                    dataout.append(dataframe[column].loc[index] - dataframe[column].loc[previous_index])
                else:
                    dataout.append(np.nan)
                id_ = dataframe.id.loc[index]
                previous_index=index
    return dataout

#mean normalization
def mean_norm(dataseries):
    return (dataseries-dataseries.mean())/(dataseries.max()- dataseries.min())

#min-max scaling
def min_max(dataseries):
    return (dataseries-dataseries.min())/(dataseries.max()- dataseries.min())

def make_heatmap(data, columns=None, figsize=(20, 20)):
    if columns is None:
        corr = data.corr()
    else:
        corr = datadf[columns].corr()
    plt.figure(figsize=figsize)
    sns.heatmap(np.abs(corr), cmap=sns.color_palette('Blues'), annot=True, fmt='0.2g')
    plt.show()
    
def make_ols_model(df, target='price', columns_to_use=None, add_constant=True):
    
    # just build a model and see the output

    X = df[columns_to_use]
    y = df[target]

    # add a constant to my X
    if add_constant:
        X = sm.add_constant(X)
    
    ols = sm.OLS(y, X)
    results = ols.fit()
    print(results.summary())
    return ols, results

def scat_plot(data, column, target= 'price'):
    plt.figure(figsize= (13, 8))
    sns.scatterplot(x= data[column], y= data[target])

def bar_hist_plot(dataseries):
    plt.figure(figsize= (13, 8))
    a=dataseries.value_counts(sort=False);
    a.sort_index().plot(kind= 'barh')

def viol_plot(data, column, target= 'price'):
    plt.figure(figsize= (13, 8))
    sns.violinplot(x=column, y=target, data= data)

def equal_bin(dataseries):
    return [dataseries.min()-.01, dataseries.quantile(.25), dataseries.quantile(.5), 
            dataseries.quantile(.75), dataseries.max()]

def equal_bin10(dataseries, quantile_step=10):
    
    a = [dataseries.min()-.01]
    
    for quant in np.array(list(range(1,quantile_step)))/quantile_step:
        a.append(dataseries.quantile(quant))
    a.append(dataseries.max())  
    return a

def viol2(x, y):
    plt.figure(figsize= (13, 8))
    sns.violinplot(x=x, y=y)

from folium.plugins import HeatMap
```


```python

```

### Gather Data


```python
data = pd.read_csv('kc_house_data.csv')
data['ones'] = data.price* 0 + 1
```


```python
data.head()
```




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
      <th>id</th>
      <th>date</th>
      <th>price</th>
      <th>bedrooms</th>
      <th>bathrooms</th>
      <th>sqft_living</th>
      <th>sqft_lot</th>
      <th>floors</th>
      <th>waterfront</th>
      <th>view</th>
      <th>...</th>
      <th>sqft_above</th>
      <th>sqft_basement</th>
      <th>yr_built</th>
      <th>yr_renovated</th>
      <th>zipcode</th>
      <th>lat</th>
      <th>long</th>
      <th>sqft_living15</th>
      <th>sqft_lot15</th>
      <th>ones</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7129300520</td>
      <td>10/13/2014</td>
      <td>221900.0</td>
      <td>3</td>
      <td>1.00</td>
      <td>1180</td>
      <td>5650</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>...</td>
      <td>1180</td>
      <td>0.0</td>
      <td>1955</td>
      <td>0.0</td>
      <td>98178</td>
      <td>47.5112</td>
      <td>-122.257</td>
      <td>1340</td>
      <td>5650</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6414100192</td>
      <td>12/9/2014</td>
      <td>538000.0</td>
      <td>3</td>
      <td>2.25</td>
      <td>2570</td>
      <td>7242</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>2170</td>
      <td>400.0</td>
      <td>1951</td>
      <td>1991.0</td>
      <td>98125</td>
      <td>47.7210</td>
      <td>-122.319</td>
      <td>1690</td>
      <td>7639</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5631500400</td>
      <td>2/25/2015</td>
      <td>180000.0</td>
      <td>2</td>
      <td>1.00</td>
      <td>770</td>
      <td>10000</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>770</td>
      <td>0.0</td>
      <td>1933</td>
      <td>NaN</td>
      <td>98028</td>
      <td>47.7379</td>
      <td>-122.233</td>
      <td>2720</td>
      <td>8062</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2487200875</td>
      <td>12/9/2014</td>
      <td>604000.0</td>
      <td>4</td>
      <td>3.00</td>
      <td>1960</td>
      <td>5000</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1050</td>
      <td>910.0</td>
      <td>1965</td>
      <td>0.0</td>
      <td>98136</td>
      <td>47.5208</td>
      <td>-122.393</td>
      <td>1360</td>
      <td>5000</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1954400510</td>
      <td>2/18/2015</td>
      <td>510000.0</td>
      <td>3</td>
      <td>2.00</td>
      <td>1680</td>
      <td>8080</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1680</td>
      <td>0.0</td>
      <td>1987</td>
      <td>0.0</td>
      <td>98074</td>
      <td>47.6168</td>
      <td>-122.045</td>
      <td>1800</td>
      <td>7503</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>




```python
data.info()
#  date, sqft_basement contain string a n
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 21597 entries, 0 to 21596
    Data columns (total 22 columns):
    id               21597 non-null int64
    date             21597 non-null object
    price            21597 non-null float64
    bedrooms         21597 non-null int64
    bathrooms        21597 non-null float64
    sqft_living      21597 non-null int64
    sqft_lot         21597 non-null int64
    floors           21597 non-null float64
    waterfront       19221 non-null float64
    view             21534 non-null float64
    condition        21597 non-null int64
    grade            21597 non-null int64
    sqft_above       21597 non-null int64
    sqft_basement    21597 non-null object
    yr_built         21597 non-null int64
    yr_renovated     17755 non-null float64
    zipcode          21597 non-null int64
    lat              21597 non-null float64
    long             21597 non-null float64
    sqft_living15    21597 non-null int64
    sqft_lot15       21597 non-null int64
    ones             21597 non-null float64
    dtypes: float64(9), int64(11), object(2)
    memory usage: 3.6+ MB
    


```python
data.describe()
#'date' 'sqft_basement' not included.
```




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
      <th>id</th>
      <th>price</th>
      <th>bedrooms</th>
      <th>bathrooms</th>
      <th>sqft_living</th>
      <th>sqft_lot</th>
      <th>floors</th>
      <th>waterfront</th>
      <th>view</th>
      <th>condition</th>
      <th>grade</th>
      <th>sqft_above</th>
      <th>yr_built</th>
      <th>yr_renovated</th>
      <th>zipcode</th>
      <th>lat</th>
      <th>long</th>
      <th>sqft_living15</th>
      <th>sqft_lot15</th>
      <th>ones</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.159700e+04</td>
      <td>2.159700e+04</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>2.159700e+04</td>
      <td>21597.000000</td>
      <td>19221.000000</td>
      <td>21534.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>17755.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.000000</td>
      <td>21597.0</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>4.580474e+09</td>
      <td>5.402966e+05</td>
      <td>3.373200</td>
      <td>2.115826</td>
      <td>2080.321850</td>
      <td>1.509941e+04</td>
      <td>1.494096</td>
      <td>0.007596</td>
      <td>0.233863</td>
      <td>3.409825</td>
      <td>7.657915</td>
      <td>1788.596842</td>
      <td>1970.999676</td>
      <td>83.636778</td>
      <td>98077.951845</td>
      <td>47.560093</td>
      <td>-122.213982</td>
      <td>1986.620318</td>
      <td>12758.283512</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.876736e+09</td>
      <td>3.673681e+05</td>
      <td>0.926299</td>
      <td>0.768984</td>
      <td>918.106125</td>
      <td>4.141264e+04</td>
      <td>0.539683</td>
      <td>0.086825</td>
      <td>0.765686</td>
      <td>0.650546</td>
      <td>1.173200</td>
      <td>827.759761</td>
      <td>29.375234</td>
      <td>399.946414</td>
      <td>53.513072</td>
      <td>0.138552</td>
      <td>0.140724</td>
      <td>685.230472</td>
      <td>27274.441950</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000102e+06</td>
      <td>7.800000e+04</td>
      <td>1.000000</td>
      <td>0.500000</td>
      <td>370.000000</td>
      <td>5.200000e+02</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>370.000000</td>
      <td>1900.000000</td>
      <td>0.000000</td>
      <td>98001.000000</td>
      <td>47.155900</td>
      <td>-122.519000</td>
      <td>399.000000</td>
      <td>651.000000</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.123049e+09</td>
      <td>3.220000e+05</td>
      <td>3.000000</td>
      <td>1.750000</td>
      <td>1430.000000</td>
      <td>5.040000e+03</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>7.000000</td>
      <td>1190.000000</td>
      <td>1951.000000</td>
      <td>0.000000</td>
      <td>98033.000000</td>
      <td>47.471100</td>
      <td>-122.328000</td>
      <td>1490.000000</td>
      <td>5100.000000</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>3.904930e+09</td>
      <td>4.500000e+05</td>
      <td>3.000000</td>
      <td>2.250000</td>
      <td>1910.000000</td>
      <td>7.618000e+03</td>
      <td>1.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>7.000000</td>
      <td>1560.000000</td>
      <td>1975.000000</td>
      <td>0.000000</td>
      <td>98065.000000</td>
      <td>47.571800</td>
      <td>-122.231000</td>
      <td>1840.000000</td>
      <td>7620.000000</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.308900e+09</td>
      <td>6.450000e+05</td>
      <td>4.000000</td>
      <td>2.500000</td>
      <td>2550.000000</td>
      <td>1.068500e+04</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>4.000000</td>
      <td>8.000000</td>
      <td>2210.000000</td>
      <td>1997.000000</td>
      <td>0.000000</td>
      <td>98118.000000</td>
      <td>47.678000</td>
      <td>-122.125000</td>
      <td>2360.000000</td>
      <td>10083.000000</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>max</th>
      <td>9.900000e+09</td>
      <td>7.700000e+06</td>
      <td>33.000000</td>
      <td>8.000000</td>
      <td>13540.000000</td>
      <td>1.651359e+06</td>
      <td>3.500000</td>
      <td>1.000000</td>
      <td>4.000000</td>
      <td>5.000000</td>
      <td>13.000000</td>
      <td>9410.000000</td>
      <td>2015.000000</td>
      <td>2015.000000</td>
      <td>98199.000000</td>
      <td>47.777600</td>
      <td>-121.315000</td>
      <td>6210.000000</td>
      <td>871200.000000</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>



### Scrubbing the Data
Cleaning null values and placeholders

#### date
Convert string values to datetime64 format (yyyy-mm-dd)


```python
data.date=pd.to_datetime(data.date)
```

#### sqft_basement
Changing 0 and ? to NaN


```python
a=data.sqft_basement.replace(to_replace= '?',value= np.nan)
a= a.astype(float)
data.sqft_basement= a.replace(to_replace= 0, value= np.nan)
data.sqft_basement=data.sqft_basement.fillna(value=0)
data.sqft_basement.describe()
```




    count    21597.000000
    mean       285.716581
    std        439.819830
    min          0.000000
    25%          0.000000
    50%          0.000000
    75%        550.000000
    max       4820.000000
    Name: sqft_basement, dtype: float64



#### Used yr_built to make age


```python
data['age'] = 2015- data.yr_built

```

#### yr_renovated 
Changing 0 to NaN  



```python
data.yr_renovated = data.yr_renovated.replace(to_replace= 0, value= np.nan)
```

#### Using yr_renovated to make since_ren


```python
data['since_ren'] = 2015- data.yr_renovated 
```

#### waterfront
Changing waterfront to NaN to 0.


```python
data.waterfront = data.waterfront.fillna(value = 0)

```

#### view
Changing Nan values to 0


```python
data.view = data.view.fillna(value= 0)
```

#### sqft_basement
Change NaN values to 0


```python
data.sqft_basement = data.sqft_basement.fillna(value= 0)
```

#### yr_renovated
Change NaN values to 0


```python
data.yr_renovated = data.yr_renovated.fillna(value= 0)
```

#### since_ren
Change NaN values to 0


```python
data.since_ren = data.since_ren.fillna(value= 0)
```

#### renovated
Binary 0 or 1 (yes)


```python
#data['renovated'] = data.yr_renovated
data['renovated']=data.yr_renovated.loc[data.yr_renovated != 0]/data.yr_renovated.loc[data.yr_renovated != 0]
data.renovated = data.renovated.fillna(value= 0)
```


```python
data.grade.unique()
```




    array([ 7,  6,  8, 11,  9,  5, 10, 12,  4,  3, 13], dtype=int64)




```python
data.columns
```




    Index(['id', 'date', 'price', 'bedrooms', 'bathrooms', 'sqft_living',
           'sqft_lot', 'floors', 'waterfront', 'view', 'condition', 'grade',
           'sqft_above', 'sqft_basement', 'yr_built', 'yr_renovated', 'zipcode',
           'lat', 'long', 'sqft_living15', 'sqft_lot15', 'ones', 'age',
           'since_ren', 'renovated'],
          dtype='object')



#### month


```python
data['month'] = data.date.dt.month
```

#### Creating a Possible Flipped Sale Database
We identified 177 resales of property that were sold in the period of study. We two meanful features:

change_date  The days between sales.

change_price The cost differential between the previous sale and current sale price.

Condition and grade were examined, but there was no change between any of the sales.


```python
multiple = data.id.value_counts()
len(multiple.loc[multiple > 1])
resales = len(multiple.loc[multiple > 1])
# 176, The number of time a home has been resold in the time period.
resales/ len(data.id)
# Far less then 1% of homes have been sold in the time period.
multiple = multiple.loc[multiple > 1]
multiple = multiple.to_dict()
data['num_sales'] = data.id.map(multiple)
data['num_sales'] = data['num_sales'].fillna(1)
flip = data.loc[data.num_sales > 1]
data = data.drop(columns= 'num_sales')
flip = flip.sort_values('date')

# adding new column examine. 
flip['change_date'] = flip_diff(flip, 'date')

#flip = flip.change_date.apply(lambda x: x.days)      

flip['change_price'] = flip_diff(flip, 'price')

# Removing initial sales
flip = flip.dropna()

# converting timedelta to days
flip.change_date = flip.change_date.apply(lambda x: x.days)

# 
flip['percent_change'] = flip.change_price/(flip.price- flip.change_price)
```

### Data Condition

binning values, playing with null values, normalizing data, standardizing data
manipulating data to fit OLS assumptions
Grouping data to create categories that might make sense for your data

#### id
Unique identified for a house. 
Although not valuable for predictive modeling, the feature id homes that have been sold multiple times.


```python

multiple = data.id.value_counts()
len(multiple.loc[multiple > 1])
resales = len(multiple.loc[multiple > 1])
# 176, The number of time a home has been resold in the time period.
print(resales)

```

    176
    


```python
# Far less then 1% of homes have been sold in the time period.
resales/ len(data.id)  
```




    0.008149279992591563



#### date
The date  the house was sold. The initial value was a string value. The information was convert to a date. See Data Scrubbing section for more information.




```python
print(data.date.min(),'-',data.date.max())

```

    2014-05-02 00:00:00 - 2015-05-27 00:00:00
    


```python
data.date.value_counts().sort_index()
```




    2014-05-02     67
    2014-05-03      4
    2014-05-04      5
    2014-05-05     84
    2014-05-06     83
    2014-05-07     93
    2014-05-08     81
    2014-05-09     81
    2014-05-10      5
    2014-05-11      2
    2014-05-12     80
    2014-05-13     86
    2014-05-14     81
    2014-05-15     82
    2014-05-16     73
    2014-05-17      1
    2014-05-18      7
    2014-05-19     83
    2014-05-20    116
    2014-05-21     94
    2014-05-22     91
    2014-05-23     84
    2014-05-24     11
    2014-05-25      5
    2014-05-26      8
    2014-05-27    104
    2014-05-28    111
    2014-05-29     75
    2014-05-30     65
    2014-05-31      6
                 ... 
    2015-04-18      5
    2015-04-19      6
    2015-04-20     77
    2015-04-21    119
    2015-04-22    121
    2015-04-23    110
    2015-04-24    103
    2015-04-25     16
    2015-04-26     13
    2015-04-27    126
    2015-04-28    121
    2015-04-29    113
    2015-04-30     83
    2015-05-01     77
    2015-05-02      6
    2015-05-03     10
    2015-05-04    102
    2015-05-05     94
    2015-05-06     88
    2015-05-07     76
    2015-05-08     54
    2015-05-09      3
    2015-05-10      2
    2015-05-11     40
    2015-05-12     49
    2015-05-13     31
    2015-05-14     11
    2015-05-15      1
    2015-05-24      1
    2015-05-27      1
    Name: date, Length: 372, dtype: int64



The date range is 5/2/2014 to 5/27/2015

#### bedrooms
Number of Bedrooms/House
The initial values ranged from 1 to 33 bedrooms, but there are very fews value over 6. 



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='bedrooms', y='price', data= data)
plt.title('Bedrooms v. Price Violin Plot');
```


![png](student_files/student_48_0.png)


#### Bedrooms v. Price Violin Plot
-skewed to data in all categories


```python
a=data.bedrooms.value_counts(sort=False);
a.sort_index().plot(kind= 'barh')
plt.title('Initial Bedroom Data');
plt.xlabel('Count');
plt.ylabel('Bedrooms');

```


![png](student_files/student_50_0.png)



```python

```

#### bathrooms
Number of bathrooms/bedrooms



```python
a=data.bathrooms.value_counts(sort=False);
plt.figure(figsize= (13, 8))
a.sort_index().plot(kind= 'barh')
plt.title('Initial Bathroom Data');
plt.xlabel('Count');
plt.ylabel('Bathrooms');
```


![png](student_files/student_53_0.png)



```python
bins = [0, 1.75, 3.75, 8.0]
data['bin_bathrooms'] = binning(data['bathrooms'], bins)
bar_hist_plot(data['bin_bathrooms'])
```


![png](student_files/student_54_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='bin_bathrooms', y='price', data= data.loc[data.price< data.price.quantile(.95)])
plt.title('bathrooms v. Price Violin Plot');
```


![png](student_files/student_55_0.png)



```python
plt.figure(figsize= (13, 8))

#sns.violinplot(x= binning(data['bathrooms'],5), y= data[target]);
sns.scatterplot(x= data.bathrooms, y= data[target]);
```


![png](student_files/student_56_0.png)


#### Bathroom v. Price Violin Plot
-skewed to data in all categories

-simliar to Bedroom

-High heteroscedasticity 

#### sqft_living
Square footage of the home 

The range is 370 sqft to 13540 sqft. 


```python
# binning values
bins = [370, 1000, 2000, 3000, 4000, 5000, 13540]


plt.figure(figsize= (13, 8))

sns.distplot(data.sqft_living)
plt.title('Initial Living Space Sqft Data');
plt.xlabel('Sqft');


```


![png](student_files/student_59_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x= binning(data['sqft_living'],10), y= data[target]);
```


![png](student_files/student_60_0.png)



```python
plt.figure(figsize= (13, 8));
scat_plot(data, 'sqft_living');
```


    <Figure size 936x576 with 0 Axes>



![png](student_files/student_61_1.png)


#### Square Footage for Living Space v. Price Violin Plot
-skewed to data in all categories

-simliar to Bedroom

-There is high heterodiacity

##### sqft_lot
Square footage of the lot
The range is 520 sqft to 1,651,359 sqft.



```python

plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_lot, hist=False)
plt.title('Initial Lot Space Sqft Data');
plt.xlabel('Sqft');
plt.ylabel('Count');

```


![png](student_files/student_64_0.png)



```python
plt.figure(figsize= (13, 8))
sns.scatterplot(x= data.sqft_lot, y= data.price)
plt.title('Initial Lot Space Sqft Data');
plt.xlabel('Sqft');
plt.ylabel('Count');
```


![png](student_files/student_65_0.png)


#### KDE plot for Lot Space
-Looks like it is a highly skewed.

#### floors
Total floors (levels) in house
The range is 1 to 3.5 floor. 


```python
a=data.floors.value_counts(sort=False);
plt.figure(figsize= (13, 8))
a.sort_index().plot(kind= 'barh')
plt.title('Initial Floor Data');
plt.xlabel('Count');
plt.ylabel('Floors');
```


![png](student_files/student_68_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='floors', y='price', data= data)
plt.title('Floors v. Price Violin Plot');
```


![png](student_files/student_69_0.png)



```python
scat_plot(data, 'floors')
```


![png](student_files/student_70_0.png)


#### bin_floors



```python
bins= [0, 1.5, 2.5, 3.5]
```


```python
a= binning(data.floors, bins).astype(str)

```


```python
count = 1
for i in a.unique():
    b = a.loc[a == i]
    a.iloc[b.index] = count
    count += 1
data['bin_floors'] = a
```


```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='bin_floors', y='price', data= data)
plt.title('bin_floors v. Price Violin Plot');
```


![png](student_files/student_75_0.png)



```python
scat_plot(data, 'bin_floors')
```


![png](student_files/student_76_0.png)


#### waterfront
House which has a view to a waterfront




```python
data.waterfront.value_counts()
```




    0.0    21451
    1.0      146
    Name: waterfront, dtype: int64




```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='waterfront', y='price', data= data)
plt.title('Waterfront v. Price Violin Plot');
```


![png](student_files/student_79_0.png)


#### view
Has been viewed
The range is 0 to 4.


```python
a = data.view.value_counts()
a.sort_index().plot(kind= 'barh')
plt.title('Initial View Data');
plt.xlabel('Count');
plt.ylabel('View');
```


![png](student_files/student_81_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='view', y='price', data= data)
plt.title('view v. Price Violin Plot');
```


![png](student_files/student_82_0.png)


#### view Analysis
High skewedness

#### condition
How good the condition is ( Overall 
The range is 1 to 5.


```python
a = data.condition.value_counts()
a.sort_index().plot(kind= 'barh')
plt.title('Initial Condition Data');
plt.xlabel('Count');
plt.ylabel('Condition');
```


![png](student_files/student_85_0.png)



```python
viol_plot(data, 'condition')
```


![png](student_files/student_86_0.png)


#### Analysis of Condition
High skewness in the high count values.

#### grade
overall grade given to the housing unit, based on King County grading system


```python
a = data.grade.value_counts()
a.sort_index().plot(kind= 'bar')
plt.title('Initial Grade Data');
plt.xlabel('Count');
plt.ylabel('Grade');
```


![png](student_files/student_89_0.png)



```python
viol_plot(data, 'grade')
```


![png](student_files/student_90_0.png)



```python
scat_plot(data, 'grade')
```


![png](student_files/student_91_0.png)


#### Analysis of Grade.
skewness increase as the grade increases
High Heteroscedasticity 


```python

```

#### Analysis seem to increase as the grade goes up.

#### sqft_above
Square footage of house apart from basement


```python

plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_above, bins=10)
plt.title('Initial Sqft Except from Basement Data');
plt.xlabel('Sqft');
plt.ylabel('Count');
```


![png](student_files/student_96_0.png)



```python
scat_plot(data, 'sqft_above')
```


![png](student_files/student_97_0.png)



```python
data['binning_sqft_above'] = binning(data.sqft_above, 10)

bar_hist_plot(data.binning_sqft_above)
data = data.drop(columns= 'binning_sqft_above')
```


![png](student_files/student_98_0.png)


#### Analysis of sqft_above
-High Heteroscedasticity 

#### sqft_basement
Square footage of the basement.  ? is the place hold value and the feature is formated as a string. See Data Scrubbing 


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_basement, bins=30)
plt.title('Initial Basement Sqft Data');
plt.xlabel('Sqft');
plt.ylabel('Count');
```


![png](student_files/student_101_0.png)



```python
scat_plot(data, 'sqft_basement')
```


![png](student_files/student_102_0.png)


#### Analysis of sqft basement
-High Heteroscedasticity


#### yr_built
Built Year
The range is 1900 to 2015


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.yr_built, bins=50)
plt.title('Initial Built Year Data');
plt.xlabel('Year');

```


![png](student_files/student_105_0.png)



```python
data.yr_built.describe()
[1899, 1951, 1975, 1997, 2015]
pd.cut(data.yr_built, [1899, 1912, 1929, 1939, 1951, 1975, 1997, 2016])

bar_hist_plot(pd.cut(data.yr_built, [1899, 1912, 1929, 1939, 1951, 1975, 1997, 2016]))

```


![png](student_files/student_106_0.png)



```python
plt.figure(figsize=(13,8))
sns.violinplot(x= pd.cut(data.yr_built, [1899, 1912, 1929, 1939, 1951, 1975, 1997, 2016]), y= data.price);


```


![png](student_files/student_107_0.png)



```python

```

#### yr_renovated
Year when house was renovated. The 744 have renovations.


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.yr_renovated, bins=50)
plt.title('Initial Renovated Year Data');
plt.xlabel('Year');
```


![png](student_files/student_110_0.png)


#### zipcode


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.zipcode, bins=70)
plt.title('Initial Zipcode Data');
plt.xlabel('Year');
```


![png](student_files/student_112_0.png)



```python
a=data.groupby(['zipcode'])['price'].mean()
data['zip_mean'] = data.zipcode.apply(lambda x: a[x])
scat_plot(data, 'zip_mean')
data['zip_mean'].describe()
```




    count    2.159700e+04
    mean     5.402966e+05
    std      2.344425e+05
    min      2.342840e+05
    25%      3.594963e+05
    50%      4.936253e+05
    75%      6.452442e+05
    max      2.161300e+06
    Name: zip_mean, dtype: float64




![png](student_files/student_113_1.png)



```python
a=data.groupby(['zipcode'])['price'].median()
data['zip_median'] = data.zipcode.apply(lambda x: a[x])
scat_plot(data, 'zip_median')
data['zip_median'].describe()
```




    count    2.159700e+04
    mean     4.859554e+05
    std      1.966655e+05
    min      2.350000e+05
    25%      3.350000e+05
    50%      4.459500e+05
    75%      5.720000e+05
    max      1.895000e+06
    Name: zip_median, dtype: float64




![png](student_files/student_114_1.png)



```python
a= binning(data.zip_median, 6)
viol2(x=a, y=data.price)
```


![png](student_files/student_115_0.png)



```python
a=data.groupby(['zipcode'])['price'].quantile(.75)
data['zip_75q'] = data.zipcode.apply(lambda x: a[x])
scat_plot(data, 'zip_75q')
data['zip_75q'].describe()
```




    count    2.159700e+04
    mean     6.180254e+05
    std      2.811627e+05
    min      2.685000e+05
    25%      4.052125e+05
    50%      5.517500e+05
    75%      7.190000e+05
    max      2.560000e+06
    Name: zip_75q, dtype: float64




![png](student_files/student_116_1.png)



```python
a=data.groupby(['zipcode'])['price'].count()
data['zip_count'] = data.zipcode.apply(lambda x: a[x])
```

#### lat
Latitude coordinate. The range is 47.1559 to 47.7776


```python
print(max(data.lat), min(data.lat))
```

    47.7776 47.1559
    


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.lat, bins=10)
plt.title('Initial Latitude Data');
plt.xlabel('Year');
```


![png](student_files/student_120_0.png)



```python
a= data.groupby(['zipcode'])['lat'].median()
data['zip_median_lat'] = data.zipcode.apply(lambda x: a[x])
data['zip_EW'] = (data['zip_median_lat'] >= data.lat)*1


a= data.groupby(['zipcode', 'zip_EW'])['price'].median()
b=[]
for index in range(0,len(data)):
    b.append(a[data.zipcode.iloc[index]][data.zip_EW.iloc[index]])
data['sub_zip_median'] = b    



a= data.groupby(['zipcode', 'zip_EW'])['price'].quantile(.75)
b=[]
for index in range(0,len(data)):
    b.append(a[data.zipcode.iloc[index]][data.zip_EW.iloc[index]])
data['sub_zip_75q'] = b 

```


```python
plt.figure(figsize= (13, 8))
sns.distplot(data['sub_zip_median'], bins=10);

```


![png](student_files/student_122_0.png)


#### long
Longitude coordinate


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.long, bins=10)
plt.title('Initial Longitude Data');
plt.xlabel('Year');
```


![png](student_files/student_124_0.png)



```python
data['bin_lat'] = binning(data.lat, equal_bin10(data.lat, quantile_step=25))
data['bin_lat']  = data['bin_lat'].astype(str)
data['bin_long'] = binning(data.long, equal_bin10(data.long, quantile_step=25))
data['bin_long'] = data['bin_long'].astype(str)
column = 'bin_lat'
a = []
count = 1
for index, value in enumerate(data[column]):
    for i in data[column].unique():
        if i == value:
            a.append(count)
        count += 1
    count = 1
data[column] = a  
column = 'bin_long'
a = []
count = 1
for index, value in enumerate(data[column]):
    for i in data[column].unique():
        if i == value:
            a.append(count)
        count += 1
    count = 1
data[column] = a
data['zone'] = data['bin_lat']*10 + data['bin_long']
```


```python
data['bin_long'].nunique()
```




    25




```python

a= data.groupby(['zone'])['price'].mean()
data['zone_mean'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_mean')
data['log_zone_mean'] = np.log(data['zone_mean'])
```


![png](student_files/student_127_0.png)



```python
a= data.groupby(['zone'])['price'].quantile(.95)
data['zone_95q'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_95q')
```


![png](student_files/student_128_0.png)



```python
data['log_zone_95q'] = np.log(data['zone_95q'])
```


```python
a= data.groupby(['zone'])['price'].max()
data['zone_max'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_max')


```


![png](student_files/student_130_0.png)



```python
a= data.groupby(['zone'])['price'].count()
data['zone_count'] = data.zone.apply(lambda x: a[x])
```


```python

```

#### sqft_living15
The square footage of interior housing living space for the nearest 15 neighbors. The range is 399 to 6210 sq ft


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_living15, bins=100)
plt.title('Initial Living Space of Neighbors Data');
plt.xlabel('Year');
```


![png](student_files/student_134_0.png)



```python
scat_plot(data, 'sqft_living15')
```


![png](student_files/student_135_0.png)



```python
data.sqft_living15.describe()
sns.violinplot(x= binning(data.sqft_living15, [398, 1490, 1840, 2360, 6210]), y= data.price);
```


![png](student_files/student_136_0.png)


#### sqft_lot15
The square footage of the land lots of the nearest 15 neighbors. The range is 651 to 871,200 sq ft.


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_lot15, bins=100)
plt.title('Initial Living Space of Neighbors Data');
plt.xlabel('Year');
```


![png](student_files/student_138_0.png)



```python

viol2(binning(data.sqft_lot15, equal_bin(data.sqft_lot15)), data.price)


```


![png](student_files/student_139_0.png)


#### change_date  (flip)



```python
plt.figure(figsize= (13, 8))
sns.distplot(flip.change_date, bins=30);


```


![png](student_files/student_141_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x= data.month, y= data.price);
```


![png](student_files/student_142_0.png)



```python
plt.figure(figsize= (13, 8))
a = data.month.value_counts()
a.sort_index().plot(kind= 'barh');
plt.ylabel('Count');
```


![png](student_files/student_143_0.png)



```python
data['log_grade'] = np.log(data["grade"])

```

#### Creating a Possible Flipped Sale Database
We identified 177 resales of property that were sold in the period of study. We two meanful features:

change_date  The days between sales.

change_price The cost differential between the previous sale and current sale price.

Condition and grade were examined, but there was no change between any of the sales. 



```python
multiple = data.id.value_counts()
len(multiple.loc[multiple > 1])
resales = len(multiple.loc[multiple > 1])
# 176, The number of time a home has been resold in the time period.
resales/ len(data.id)
# Far less then 1% of homes have been sold in the time period.
multiple = multiple.loc[multiple > 1]
multiple = multiple.to_dict()
data['num_sales'] = data.id.map(multiple)
data['num_sales'] = data['num_sales'].fillna(1)
flip = data.loc[data.num_sales > 1]
data = data.drop(columns= 'num_sales')
flip = flip.sort_values('date')

# adding new column examine. 
flip['change_date'] = flip_diff(flip, 'date')

#flip = flip.change_date.apply(lambda x: x.days)      

flip['change_price'] = flip_diff(flip, 'price')

# Removing initial sales
flip = flip.dropna()

# converting timedelta to days
flip.change_date = flip.change_date.apply(lambda x: x.days)

# 
flip['percent_change'] = flip.change_price/(flip.price- flip.change_price)

```


```python
scat_plot(flip, 'percent_change')
```


![png](student_files/student_147_0.png)



```python
data['scaled_log_grade'] = (data['log_grade']-np.mean(data['log_grade']))/np.sqrt(np.var(data['log_grade']))

```


```python
scat_plot(data, 'scaled_log_grade')
```


![png](student_files/student_149_0.png)



```python
sns.violinplot(x = (1 *(data.price > data.zone_95q)), y = data.price)

a = 1 *(data.price > data.zone_95q)
a.sum()/len(data)

```




    0.054498309950456084




![png](student_files/student_150_1.png)



```python

```

### Exploration (Analysis)

#### Confusion Matrix


```python
make_heatmap(data=data, figsize=(20, 20))
```


![png](student_files/student_154_0.png)



```python
data.corr().price.abs().sort_values()

```




    bin_lat             0.001708
    zone                0.004323
    month               0.009928
    id                  0.016772
    zip_EW              0.017120
    long                0.022036
    bin_long            0.026289
    condition           0.036056
    zip_count           0.050969
    zipcode             0.053402
    yr_built            0.053953
    age                 0.053953
    since_ren           0.064950
    sqft_lot15          0.082845
    zone_count          0.087549
    sqft_lot            0.089876
    renovated           0.117543
    yr_renovated        0.117855
    bin_floors          0.237264
    floors              0.256804
    waterfront          0.264306
    lat                 0.306692
    bedrooms            0.308787
    zip_median_lat      0.310784
    sqft_basement       0.321108
    zone_max            0.390955
    view                0.393497
    log_zone_95q        0.481499
    zone_95q            0.485036
    bathrooms           0.525906
    log_zone_mean       0.532312
    zone_mean           0.547987
    sqft_living15       0.585241
    sqft_above          0.605368
    zip_median          0.632561
    zip_75q             0.634993
    log_grade           0.635153
    scaled_log_grade    0.635153
    zip_mean            0.638168
    sub_zip_median      0.647361
    sub_zip_75q         0.651066
    grade               0.667951
    sqft_living         0.701917
    price               1.000000
    ones                     NaN
    Name: price, dtype: float64



#### Scatter Matrix
Going to start OLS with sqft_living and add to the OLS going down the regression 


```python
columns= ['sqft_living ']
```


```python
make_heatmap(data=flip, figsize=(20, 20))
```


![png](student_files/student_158_0.png)


#### Modeling / Cross Validation


```python
test= []
```


```python
make_ols_model(df=data, target='price', columns_to_use='sqft_living' , add_constant=True)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.493
    Model:                            OLS   Adj. R-squared:                  0.493
    Method:                 Least Squares   F-statistic:                 2.097e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0006e+05
    No. Observations:               21597   AIC:                         6.001e+05
    Df Residuals:                   21595   BIC:                         6.001e+05
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ===============================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
    -------------------------------------------------------------------------------
    const       -4.399e+04   4410.023     -9.975      0.000   -5.26e+04   -3.53e+04
    sqft_living   280.8630      1.939    144.819      0.000     277.062     284.664
    ==============================================================================
    Omnibus:                    14801.942   Durbin-Watson:                   1.982
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):           542662.604
    Skew:                           2.820   Prob(JB):                         0.00
    Kurtosis:                      26.901   Cond. No.                     5.63e+03
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 5.63e+03. This might indicate that there are
    strong multicollinearity or other numerical problems.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5ebb6fba8>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ededc5c0>)



#### Data OLS Test 1

R squared is low

cond no. is high. I will remove constant to see if the improves.


```python
make_ols_model(df=data, target='price', columns_to_use='sqft_living' , add_constant=False)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.839
    Model:                            OLS   Adj. R-squared:                  0.839
    Method:                 Least Squares   F-statistic:                 1.124e+05
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0011e+05
    No. Observations:               21597   AIC:                         6.002e+05
    Df Residuals:                   21596   BIC:                         6.002e+05
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ===============================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
    -------------------------------------------------------------------------------
    sqft_living   263.1647      0.785    335.319      0.000     261.626     264.703
    ==============================================================================
    Omnibus:                    16021.993   Durbin-Watson:                   1.980
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):           689028.909
    Skew:                           3.128   Prob(JB):                         0.00
    Kurtosis:                      29.955   Cond. No.                         1.00
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5ea512c88>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ea512ba8>)



#### Data OLS Test 2
R squared improved

Linearity good (only variable).
F - statistic is high.
Skewing is a problem. -Will try log transform.



```python
data['log_sqft_living'] = np.log(data["sqft_living"])
```


```python
make_ols_model(df=data, target='price', columns_to_use='log_sqft_living' , add_constant=False)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.714
    Model:                            OLS   Adj. R-squared:                  0.714
    Method:                 Least Squares   F-statistic:                 5.390e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0631e+05
    No. Observations:               21597   AIC:                         6.126e+05
    Df Residuals:                   21596   BIC:                         6.126e+05
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living    7.3e+04    314.426    232.161      0.000    7.24e+04    7.36e+04
    ==============================================================================
    Omnibus:                    19995.129   Durbin-Watson:                   1.967
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1400805.293
    Skew:                           4.276   Prob(JB):                         0.00
    Kurtosis:                      41.517   Cond. No.                         1.00
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5ea512e48>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ededcd68>)



#### Data OLS Test 3
R squared worst

Linearity good (only variable).

F - statistic is high.

Skewing is less of a problem. -Will try log transform.


```python
columns= ['log_sqft_living', 'grade']
make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.805
    Model:                            OLS   Adj. R-squared:                  0.805
    Method:                 Least Squares   F-statistic:                 4.457e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0217e+05
    No. Observations:               21597   AIC:                         6.044e+05
    Df Residuals:                   21595   BIC:                         6.044e+05
    Df Model:                           2                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.498e+05   2233.777    -67.044      0.000   -1.54e+05   -1.45e+05
    grade            2.189e+05   2180.541    100.404      0.000    2.15e+05    2.23e+05
    ==============================================================================
    Omnibus:                    19949.996   Durbin-Watson:                   1.973
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1905587.098
    Skew:                           4.135   Prob(JB):                         0.00
    Kurtosis:                      48.268   Cond. No.                         17.2
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5efc149b0>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ea512978>)



#### Data OLS Test 4
R squared better

Linearity good.

Normality is better, but not good.

F - statistic is high but better

Skewing got worst but only by a little. -Will try log transform.


```python
data['log_grade'] = np.log(data["grade"])
columns= ['log_sqft_living', 'log_grade']
make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.735
    Model:                            OLS   Adj. R-squared:                  0.735
    Method:                 Least Squares   F-statistic:                 2.988e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0550e+05
    No. Observations:               21597   AIC:                         6.110e+05
    Df Residuals:                   21595   BIC:                         6.110e+05
    Df Model:                           2                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.792e+05   6163.081    -29.080      0.000   -1.91e+05   -1.67e+05
    log_grade        9.408e+05    2.3e+04     40.974      0.000    8.96e+05    9.86e+05
    ==============================================================================
    Omnibus:                    20219.415   Durbin-Watson:                   1.977
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1612143.698
    Skew:                           4.305   Prob(JB):                         0.00
    Kurtosis:                      44.441   Cond. No.                         81.3
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5ea5006a0>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ea5005f8>)



#### Data OLS Test 5
R squared worse

Linearity good.

Normality is better, but not good.

F - statistic is passing

Skewing got better. -Adding basementsqft


```python

columns= ['log_sqft_living', 'log_grade', 'sqft_basement']
make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.771
    Model:                            OLS   Adj. R-squared:                  0.771
    Method:                 Least Squares   F-statistic:                 2.420e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0392e+05
    No. Observations:               21597   AIC:                         6.078e+05
    Df Residuals:                   21594   BIC:                         6.079e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -2.695e+05   5932.474    -45.421      0.000   -2.81e+05   -2.58e+05
    log_grade        1.234e+06   2.19e+04     56.307      0.000    1.19e+06    1.28e+06
    sqft_basement     294.7408      5.048     58.389      0.000     284.846     304.635
    ==============================================================================
    Omnibus:                    18969.591   Durbin-Watson:                   1.973
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1290940.634
    Skew:                           3.920   Prob(JB):                         0.00
    Kurtosis:                      40.055   Cond. No.                     5.60e+03
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 5.6e+03. This might indicate that there are
    strong multicollinearity or other numerical problems.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5ea5002b0>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ea5004a8>)



#### Data OLS Test 8
R squared worse

Linearity got worse.

Normality is better, but not good.

F - statistic got worse. Not passing

Skewing got better. -Will make basement a categorical feature


```python
data['basement'] = data.sqft_basement.replace(to_replace= 0, value= np.nan)
data.basement = data.basement/data.basement
data.basement = data.basement.fillna(0)
columns= ['log_sqft_living', 'log_grade', 'basement']
make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.745
    Model:                            OLS   Adj. R-squared:                  0.745
    Method:                 Least Squares   F-statistic:                 2.106e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0506e+05
    No. Observations:               21597   AIC:                         6.101e+05
    Df Residuals:                   21594   BIC:                         6.101e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -2.191e+05   6180.225    -35.444      0.000   -2.31e+05   -2.07e+05
    log_grade        1.062e+06   2.28e+04     46.475      0.000    1.02e+06    1.11e+06
    basement         1.426e+05   4728.937     30.154      0.000    1.33e+05    1.52e+05
    ==============================================================================
    Omnibus:                    20182.453   Durbin-Watson:                   1.976
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1661367.010
    Skew:                           4.278   Prob(JB):                         0.00
    Kurtosis:                      45.107   Cond. No.                         82.7
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    




    (<statsmodels.regression.linear_model.OLS at 0x2a5ea4f3b38>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2a5ea4f3fd0>)



#### Data OLS Test 9
R squared better

Normality is worse. 


Skewing got worst but only by a little. -Taking out basement and add lat next


```python

columns= ['log_sqft_living', 'grade', 'lat']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)


```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.832
    Model:                            OLS   Adj. R-squared:                  0.831
    Method:                 Least Squares   F-statistic:                 3.553e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:04   Log-Likelihood:            -3.0059e+05
    No. Observations:               21597   AIC:                         6.012e+05
    Df Residuals:                   21594   BIC:                         6.012e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living   2.03e+05   6394.031     31.743      0.000     1.9e+05    2.15e+05
    grade            1.537e+05   2314.914     66.395      0.000    1.49e+05    1.58e+05
    lat             -4.561e+04    781.899    -58.326      0.000   -4.71e+04   -4.41e+04
    ==============================================================================
    Omnibus:                    19984.635   Durbin-Watson:                   1.971
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          2118529.368
    Skew:                           4.107   Prob(JB):                         0.00
    Kurtosis:                      50.820   Cond. No.                         178.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


```python
current= [results.rsquared, results.condition_number]
current
```




    [0.831521778529237, 178.25252875884587]



#### Data OLS Test 10
R squared better

Normality same 

Skewing got worst but only by a little. -Try log transform


```python
data['log_lat'] = np.log(data.lat)
columns= ['log_sqft_living', 'log_grade', 'log_lat']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
print('\nR2', current[0], results.rsquared, '\nCond. No.',current[1], results.condition_number) 
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.824
    Model:                            OLS   Adj. R-squared:                  0.824
    Method:                 Least Squares   F-statistic:                 3.369e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:05   Log-Likelihood:            -3.0107e+05
    No. Observations:               21597   AIC:                         6.021e+05
    Df Residuals:                   21594   BIC:                         6.022e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living  2.665e+05   6581.381     40.496      0.000    2.54e+05    2.79e+05
    log_grade        9.945e+05   1.87e+04     53.164      0.000    9.58e+05    1.03e+06
    log_lat         -9.025e+05   8618.843   -104.711      0.000   -9.19e+05   -8.86e+05
    ==============================================================================
    Omnibus:                    20429.977   Durbin-Watson:                   1.972
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          2226615.964
    Skew:                           4.258   Prob(JB):                         0.00
    Kurtosis:                      52.009   Cond. No.                         90.6
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    
    R2 0.831521778529237 0.8239606798202903 
    Cond. No. 178.25252875884587 90.64089969897728
    

#### Data OLS Test 11
R squared very bad

Normality same 


I going to see if we can create better location feature. Going to replace lat with zip_mean.


```python
columns= ['log_sqft_living', 'grade', 'zip_mean']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]
print('\nR2', current[0], results.rsquared, '\nCond. No.',current[1], results.condition_number) 
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.861
    Model:                            OLS   Adj. R-squared:                  0.861
    Method:                 Least Squares   F-statistic:                 4.457e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:05   Log-Likelihood:            -2.9852e+05
    No. Observations:               21597   AIC:                         5.970e+05
    Df Residuals:                   21594   BIC:                         5.971e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.459e+05   1886.642    -77.333      0.000    -1.5e+05   -1.42e+05
    grade             1.65e+05   1929.936     85.500      0.000    1.61e+05    1.69e+05
    zip_mean            0.7104      0.008     93.239      0.000       0.695       0.725
    ==============================================================================
    Omnibus:                    23252.829   Durbin-Watson:                   1.976
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          4402774.802
    Skew:                           5.133   Prob(JB):                         0.00
    Kurtosis:                      72.190   Cond. No.                     9.48e+05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 9.48e+05. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    R2 0.831521778529237 0.8609567453531688 
    Cond. No. 178.25252875884587 947678.552096688
    


![png](student_files/student_181_1.png)


#### OLS Test 12
worst than test 9.

Cond. No. is out of control.


```python

columns= ['log_sqft_living', 'grade', 'zip_median']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]
print('\nR2', current[0]-results.rsquared, '\nCond. No.',current[1]-results.condition_number) 
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.857
    Model:                            OLS   Adj. R-squared:                  0.857
    Method:                 Least Squares   F-statistic:                 4.326e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:05   Log-Likelihood:            -2.9880e+05
    No. Observations:               21597   AIC:                         5.976e+05
    Df Residuals:                   21594   BIC:                         5.976e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.484e+05   1910.590    -77.664      0.000   -1.52e+05   -1.45e+05
    grade            1.653e+05   1959.721     84.372      0.000    1.62e+05    1.69e+05
    zip_median          0.8232      0.009     89.032      0.000       0.805       0.841
    ==============================================================================
    Omnibus:                    23269.273   Durbin-Watson:                   1.973
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          4298977.267
    Skew:                           5.150   Prob(JB):                         0.00
    Kurtosis:                      71.346   Cond. No.                     8.43e+05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 8.43e+05. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    R2 -0.025823245675149264 
    Cond. No. -842994.6984241798
    

#### Test 13
Not much better than test 14. - tray higher quantile


```python
columns= ['log_sqft_living', 'grade', 'zip_75q']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]
print('\nR2', current[0]-results.rsquared, '\nCond. No.',current[1]-results.condition_number) 
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.861
    Model:                            OLS   Adj. R-squared:                  0.861
    Method:                 Least Squares   F-statistic:                 4.468e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:05   Log-Likelihood:            -2.9850e+05
    No. Observations:               21597   AIC:                         5.970e+05
    Df Residuals:                   21594   BIC:                         5.970e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.442e+05   1885.103    -76.487      0.000   -1.48e+05    -1.4e+05
    grade            1.656e+05   1925.442     86.020      0.000    1.62e+05    1.69e+05
    zip_75q             0.5924      0.006     93.588      0.000       0.580       0.605
    ==============================================================================
    Omnibus:                    23249.600   Durbin-Watson:                   1.975
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          4426611.769
    Skew:                           5.130   Prob(JB):                         0.00
    Kurtosis:                      72.382   Cond. No.                     1.09e+06
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 1.09e+06. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    R2 -0.029733522942698265 
    Cond. No. -1092799.5073060167
    

#### Test 14 
Better than 13. 

Linearity is still problem


```python
columns= ['log_sqft_living', 'grade', 'sub_zip_median' ]
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]
print('\nR2', current[0]-results.rsquared, '\nCond. No.',current[1]-results.condition_number) 
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.860
    Model:                            OLS   Adj. R-squared:                  0.860
    Method:                 Least Squares   F-statistic:                 4.411e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:05   Log-Likelihood:            -2.9862e+05
    No. Observations:               21597   AIC:                         5.972e+05
    Df Residuals:                   21594   BIC:                         5.973e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.427e+05   1896.087    -75.287      0.000   -1.46e+05   -1.39e+05
    grade            1.598e+05   1958.203     81.630      0.000    1.56e+05    1.64e+05
    sub_zip_median      0.8146      0.009     91.797      0.000       0.797       0.832
    ==============================================================================
    Omnibus:                    23596.770   Durbin-Watson:                   1.975
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          4618311.007
    Skew:                           5.262   Prob(JB):                         0.00
    Kurtosis:                      73.862   Cond. No.                     8.59e+05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 8.59e+05. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    R2 -0.028199673126099167 
    Cond. No. -859051.7347879737
    

#### OLS Test 16
Not good. -Will go back to Test 10, and examine residuals


```python

columns= ['log_sqft_living', 'grade']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.805
    Model:                            OLS   Adj. R-squared:                  0.805
    Method:                 Least Squares   F-statistic:                 4.457e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:05   Log-Likelihood:            -3.0217e+05
    No. Observations:               21597   AIC:                         6.044e+05
    Df Residuals:                   21595   BIC:                         6.044e+05
    Df Model:                           2                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.498e+05   2233.777    -67.044      0.000   -1.54e+05   -1.45e+05
    grade            2.189e+05   2180.541    100.404      0.000    2.15e+05    2.23e+05
    ==============================================================================
    Omnibus:                    19949.996   Durbin-Watson:                   1.973
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1905587.098
    Skew:                           4.135   Prob(JB):                         0.00
    Kurtosis:                      48.268   Cond. No.                         17.2
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_189_1.png)


Residual is very skewed to the lower end. 


```python
len(a.loc[a.abs() > 2])/len(a)
```




    0.0



#### OLS Test 17
-will add zone_mean- Binned values of lat and long (quantities in each bin) with the mean of the sales price in the area

##### Residual Check 1
Around 3% of the residuals are outside of 2 standard deviations . I will check the model to see if the regression improves.


```python
columns= ['log_sqft_living', 'grade', 'zone_mean']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.840
    Model:                            OLS   Adj. R-squared:                  0.840
    Method:                 Least Squares   F-statistic:                 3.773e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:06   Log-Likelihood:            -3.0005e+05
    No. Observations:               21597   AIC:                         6.001e+05
    Df Residuals:                   21594   BIC:                         6.001e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.592e+05   2029.334    -78.460      0.000   -1.63e+05   -1.55e+05
    grade            1.827e+05   2046.028     89.283      0.000    1.79e+05    1.87e+05
    zone_mean           0.6462      0.009     68.506      0.000       0.628       0.665
    ==============================================================================
    Omnibus:                    21318.100   Durbin-Watson:                   1.985
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          2913417.858
    Skew:                           4.500   Prob(JB):                         0.00
    Kurtosis:                      59.183   Cond. No.                     9.19e+05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 9.19e+05. This might indicate that there are
    strong multicollinearity or other numerical problems.
    


![png](student_files/student_193_1.png)



```python
a=results.resid/results.resid.std()
a = a.loc[a.abs() > 2]
len(a)/len(results.resid)
```




    0.031208038153447238




```python
 
for col in ['price',
    'log_zone_mean',
    'zone_mean', 
    'log_sqft_living',
    'sqft_living',
    'grade',
    'log_grade',
    'bedrooms', 
    'bathrooms',
    'sqft_lot',
    'floors',
    'waterfront',
    'view',
    'condition',
    'sqft_above',
    'sqft_basement',
    'yr_built',
    'yr_renovated',
    'sqft_living15',
    'sqft_lot15',
    'age',
    'since_ren',
    'renovated',
    'month',
    'basement',
    'log_lat',
    'zip_mean',
    'zip_median',
    'zip_75q',
    'sub_zip_median',
    'sub_zip_75q']:
    resd_check(results, data, col)
```


![png](student_files/student_195_0.png)



![png](student_files/student_195_1.png)



![png](student_files/student_195_2.png)



![png](student_files/student_195_3.png)



![png](student_files/student_195_4.png)



![png](student_files/student_195_5.png)



![png](student_files/student_195_6.png)



![png](student_files/student_195_7.png)



![png](student_files/student_195_8.png)



![png](student_files/student_195_9.png)



![png](student_files/student_195_10.png)



![png](student_files/student_195_11.png)



![png](student_files/student_195_12.png)



![png](student_files/student_195_13.png)



![png](student_files/student_195_14.png)



![png](student_files/student_195_15.png)



![png](student_files/student_195_16.png)



![png](student_files/student_195_17.png)



![png](student_files/student_195_18.png)



![png](student_files/student_195_19.png)



![png](student_files/student_195_20.png)



![png](student_files/student_195_21.png)



![png](student_files/student_195_22.png)



![png](student_files/student_195_23.png)



![png](student_files/student_195_24.png)



![png](student_files/student_195_25.png)



![png](student_files/student_195_26.png)



![png](student_files/student_195_27.png)



![png](student_files/student_195_28.png)



![png](student_files/student_195_29.png)



![png](student_files/student_195_30.png)


#### OLS Test 18
-will log transform  zone_mean

##### Residual Check 1
Around 3% of the residuals are outside of 2 standard deviations . I will check the model to see if the regression improves.


```python
columns= ['log_sqft_living', 'grade', 'log_zone_mean']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.810
    Model:                            OLS   Adj. R-squared:                  0.810
    Method:                 Least Squares   F-statistic:                 3.060e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:18   Log-Likelihood:            -3.0192e+05
    No. Observations:               21597   AIC:                         6.038e+05
    Df Residuals:                   21594   BIC:                         6.039e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.825e+04   6167.426     -2.959      0.003   -3.03e+04   -6161.448
    grade            2.007e+05   2298.065     87.330      0.000    1.96e+05    2.05e+05
    log_zone_mean   -6.508e+04   2849.994    -22.836      0.000   -7.07e+04   -5.95e+04
    ==============================================================================
    Omnibus:                    19715.870   Durbin-Watson:                   1.970
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1843166.706
    Skew:                           4.059   Prob(JB):                         0.00
    Kurtosis:                      47.523   Cond. No.                         60.3
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_197_1.png)


#### Test 19
Linearity good. R squared acceptable. Take out outilier (3%) shouold get better.

Going to min_max scale the log_zone_mean


```python
for col in ['price',
'zone_mean', 
'sqft_living',
'grade',
'bedrooms', 
'bathrooms',
'sqft_lot',
'floors',
'waterfront',
'view',
'condition',
'sqft_above',
'sqft_basement',
'yr_built',
'yr_renovated',
'sqft_living15',
'sqft_lot15',
'age',
'since_ren',
'renovated',
'month',
'basement',
'sub_zip_median',
'sub_zip_75q']:
    resd_check(results, data, col)
```


![png](student_files/student_199_0.png)



![png](student_files/student_199_1.png)



![png](student_files/student_199_2.png)



![png](student_files/student_199_3.png)



![png](student_files/student_199_4.png)



![png](student_files/student_199_5.png)



![png](student_files/student_199_6.png)



![png](student_files/student_199_7.png)



![png](student_files/student_199_8.png)



![png](student_files/student_199_9.png)



![png](student_files/student_199_10.png)



![png](student_files/student_199_11.png)



![png](student_files/student_199_12.png)



![png](student_files/student_199_13.png)



![png](student_files/student_199_14.png)



![png](student_files/student_199_15.png)



![png](student_files/student_199_16.png)



![png](student_files/student_199_17.png)



![png](student_files/student_199_18.png)



![png](student_files/student_199_19.png)



![png](student_files/student_199_20.png)



![png](student_files/student_199_21.png)



![png](student_files/student_199_22.png)



![png](student_files/student_199_23.png)



```python
sns.distplot(min_max(data.log_zone_mean))
data['min_max_log_zone_mean'] =min_max(data.log_zone_mean)
data.min_max_log_zone_mean.describe()
```




    count    21597.000000
    mean         0.434736
    std          0.189408
    min          0.000000
    25%          0.285129
    50%          0.428124
    75%          0.544492
    max          1.000000
    Name: min_max_log_zone_mean, dtype: float64




![png](student_files/student_200_1.png)



```python
sns.distplot((data.log_zone_mean))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2a5d6dc9240>




![png](student_files/student_201_1.png)



```python
columns= ['log_sqft_living', 'grade', 'min_max_log_zone_mean']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.838
    Model:                            OLS   Adj. R-squared:                  0.838
    Method:                 Least Squares   F-statistic:                 3.722e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:28   Log-Likelihood:            -3.0017e+05
    No. Observations:               21597   AIC:                         6.004e+05
    Df Residuals:                   21594   BIC:                         6.004e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living       -1.513e+05   2036.344    -74.318      0.000   -1.55e+05   -1.47e+05
    grade                  1.826e+05   2061.778     88.574      0.000    1.79e+05    1.87e+05
    min_max_log_zone_mean   6.67e+05   1.01e+04     66.295      0.000    6.47e+05    6.87e+05
    ==============================================================================
    Omnibus:                    21804.054   Durbin-Watson:                   1.979
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          3137357.661
    Skew:                           4.666   Prob(JB):                         0.00
    Kurtosis:                      61.304   Cond. No.                         60.9
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_202_1.png)


#### Test 20
Going to min_max scale the grade


```python
data['min_max_grade'] = min_max(data.grade)
```


```python
sns.distplot(min_max(data.grade))

```




    <matplotlib.axes._subplots.AxesSubplot at 0x2a5eba08940>




![png](student_files/student_205_1.png)



```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.850
    Model:                            OLS   Adj. R-squared:                  0.850
    Method:                 Least Squares   F-statistic:                 4.071e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:29   Log-Likelihood:            -2.9936e+05
    No. Observations:               21597   AIC:                         5.987e+05
    Df Residuals:                   21594   BIC:                         5.987e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living       -7.693e+04   1132.921    -67.906      0.000   -7.92e+04   -7.47e+04
    min_max_grade          1.812e+06    1.8e+04    100.771      0.000    1.78e+06    1.85e+06
    min_max_log_zone_mean  6.464e+05   9682.277     66.764      0.000    6.27e+05    6.65e+05
    ==============================================================================
    Omnibus:                    22149.579   Durbin-Watson:                   1.977
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          3498439.884
    Skew:                           4.763   Prob(JB):                         0.00
    Kurtosis:                      64.619   Cond. No.                         80.4
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_206_1.png)



```python
for col in ['price',
'zone_mean', 
'sqft_living',
'grade',
'bedrooms', 
'bathrooms',
'sqft_lot',
'floors',
'waterfront',
'view',
'condition',
'sqft_above',
'sqft_basement',
'yr_built',
'yr_renovated',
'sqft_living15',
'sqft_lot15',
'age',
'since_ren',
'renovated',
'month',
'basement',
'sub_zip_median',
'sub_zip_75q']:
    resd_check(results, data, col)
```


![png](student_files/student_207_0.png)



![png](student_files/student_207_1.png)



![png](student_files/student_207_2.png)



![png](student_files/student_207_3.png)



![png](student_files/student_207_4.png)



![png](student_files/student_207_5.png)



![png](student_files/student_207_6.png)



![png](student_files/student_207_7.png)



![png](student_files/student_207_8.png)



![png](student_files/student_207_9.png)



![png](student_files/student_207_10.png)



![png](student_files/student_207_11.png)



![png](student_files/student_207_12.png)



![png](student_files/student_207_13.png)



![png](student_files/student_207_14.png)



![png](student_files/student_207_15.png)



![png](student_files/student_207_16.png)



![png](student_files/student_207_17.png)



![png](student_files/student_207_18.png)



![png](student_files/student_207_19.png)



![png](student_files/student_207_20.png)



![png](student_files/student_207_21.png)



![png](student_files/student_207_22.png)



![png](student_files/student_207_23.png)


#### Test 21
Adding waterfront


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.864
    Model:                            OLS   Adj. R-squared:                  0.864
    Method:                 Least Squares   F-statistic:                 3.434e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:39   Log-Likelihood:            -2.9827e+05
    No. Observations:               21597   AIC:                         5.965e+05
    Df Residuals:                   21593   BIC:                         5.966e+05
    Df Model:                           4                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living        -7.42e+04   1078.796    -68.781      0.000   -7.63e+04   -7.21e+04
    min_max_grade          1.753e+06   1.71e+04    102.279      0.000    1.72e+06    1.79e+06
    min_max_log_zone_mean  6.467e+05   9206.794     70.238      0.000    6.29e+05    6.65e+05
    waterfront             9.598e+05   2.01e+04     47.844      0.000    9.21e+05    9.99e+05
    ==============================================================================
    Omnibus:                    20740.126   Durbin-Watson:                   1.970
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          3093809.677
    Skew:                           4.251   Prob(JB):                         0.00
    Kurtosis:                      61.015   Cond. No.                         93.6
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_209_1.png)



```python
data.corr().loc[columns]
```




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
      <th>id</th>
      <th>price</th>
      <th>bedrooms</th>
      <th>bathrooms</th>
      <th>sqft_living</th>
      <th>sqft_lot</th>
      <th>floors</th>
      <th>waterfront</th>
      <th>view</th>
      <th>condition</th>
      <th>...</th>
      <th>zone_max</th>
      <th>zone_count</th>
      <th>log_grade</th>
      <th>scaled_log_grade</th>
      <th>log_sqft_living</th>
      <th>basement</th>
      <th>log_lat</th>
      <th>min_max_log_zone_mean</th>
      <th>min_max_grade</th>
      <th>residual</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>log_sqft_living</th>
      <td>-0.001864</td>
      <td>0.611839</td>
      <td>0.621820</td>
      <td>0.762135</td>
      <td>0.954607</td>
      <td>0.150102</td>
      <td>0.367564</td>
      <td>0.078900</td>
      <td>0.244804</td>
      <td>-0.049620</td>
      <td>...</td>
      <td>0.128795</td>
      <td>-0.108029</td>
      <td>0.744356</td>
      <td>0.744356</td>
      <td>1.000000</td>
      <td>0.233548</td>
      <td>0.038881</td>
      <td>0.261217</td>
      <td>0.743038</td>
      <td>0.270836</td>
    </tr>
    <tr>
      <th>min_max_grade</th>
      <td>0.008188</td>
      <td>0.667951</td>
      <td>0.356563</td>
      <td>0.665838</td>
      <td>0.762779</td>
      <td>0.114731</td>
      <td>0.458794</td>
      <td>0.082818</td>
      <td>0.249082</td>
      <td>-0.146896</td>
      <td>...</td>
      <td>0.189847</td>
      <td>-0.105987</td>
      <td>0.992855</td>
      <td>0.992855</td>
      <td>0.743038</td>
      <td>0.050701</td>
      <td>0.113687</td>
      <td>0.343140</td>
      <td>1.000000</td>
      <td>0.060409</td>
    </tr>
    <tr>
      <th>min_max_log_zone_mean</th>
      <td>0.013413</td>
      <td>0.532312</td>
      <td>0.115965</td>
      <td>0.214993</td>
      <td>0.281800</td>
      <td>-0.014861</td>
      <td>0.131150</td>
      <td>0.029324</td>
      <td>0.131915</td>
      <td>0.038159</td>
      <td>...</td>
      <td>0.691135</td>
      <td>-0.134829</td>
      <td>0.335044</td>
      <td>0.335044</td>
      <td>0.261217</td>
      <td>0.121546</td>
      <td>0.410482</td>
      <td>1.000000</td>
      <td>0.343140</td>
      <td>0.034923</td>
    </tr>
    <tr>
      <th>waterfront</th>
      <td>-0.003599</td>
      <td>0.264306</td>
      <td>-0.002127</td>
      <td>0.063629</td>
      <td>0.104637</td>
      <td>0.021459</td>
      <td>0.020797</td>
      <td>1.000000</td>
      <td>0.380543</td>
      <td>0.016648</td>
      <td>...</td>
      <td>0.047573</td>
      <td>-0.025526</td>
      <td>0.073448</td>
      <td>0.073448</td>
      <td>0.078900</td>
      <td>0.039220</td>
      <td>-0.012111</td>
      <td>0.029324</td>
      <td>0.082818</td>
      <td>0.001255</td>
    </tr>
  </tbody>
</table>
<p>4 rows × 51 columns</p>
</div>




```python
for col in ['price',
'zone_mean', 
'sqft_living',
'grade',
'bedrooms', 
'bathrooms',
'sqft_lot',
'floors',
'waterfront',
'view',
'condition',
'sqft_above',
'sqft_basement',
'yr_built',
'yr_renovated',
'sqft_living15',
'sqft_lot15',
'age',
'since_ren',
'renovated',
'month',
'basement',
'sub_zip_median',
'sub_zip_75q']:
    resd_check(results, data, col)
```


![png](student_files/student_211_0.png)



![png](student_files/student_211_1.png)



![png](student_files/student_211_2.png)



![png](student_files/student_211_3.png)



![png](student_files/student_211_4.png)



![png](student_files/student_211_5.png)



![png](student_files/student_211_6.png)



![png](student_files/student_211_7.png)



![png](student_files/student_211_8.png)



![png](student_files/student_211_9.png)



![png](student_files/student_211_10.png)



![png](student_files/student_211_11.png)



![png](student_files/student_211_12.png)



![png](student_files/student_211_13.png)



![png](student_files/student_211_14.png)



![png](student_files/student_211_15.png)



![png](student_files/student_211_16.png)



![png](student_files/student_211_17.png)



![png](student_files/student_211_18.png)



![png](student_files/student_211_19.png)



![png](student_files/student_211_20.png)



![png](student_files/student_211_21.png)



![png](student_files/student_211_22.png)



![png](student_files/student_211_23.png)



```python

```

#### Separating  residual


```python
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


```python
a= data.grade.sort_values()


for i in a.unique():
    print(i)
    print(len(keep_data.grade.loc[data.grade == i])/len(data.grade.loc[data.grade == i]))
```

    3
    0.0
    4
    1.0
    5
    0.9669421487603306
    6
    0.9916584887144259
    7
    0.9946512146200134
    8
    0.9798845836768343
    9
    0.9391969407265774
    10
    0.8597883597883598
    11
    0.7192982456140351
    12
    0.48314606741573035
    13
    0.07692307692307693
    

#### New Variable high_grade
Binary variable for grades with that have removed residual percentages in the last model (3 and grades over 9) other grades recieved a one.


```python
data['high_grade'] = data.grade
data.high_grade.loc[(data.high_grade <12) & (data.high_grade > 3)] = 1
data.high_grade.loc[(data.high_grade >= 12) | (data.high_grade == 3)] = 0
```


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.887
    Model:                            OLS   Adj. R-squared:                  0.887
    Method:                 Least Squares   F-statistic:                 3.394e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:49   Log-Likelihood:            -2.9627e+05
    No. Observations:               21597   AIC:                         5.925e+05
    Df Residuals:                   21592   BIC:                         5.926e+05
    Df Model:                           5                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living        1.279e+05   3202.965     39.935      0.000    1.22e+05    1.34e+05
    min_max_grade          1.105e+06   1.84e+04     59.916      0.000    1.07e+06    1.14e+06
    min_max_log_zone_mean  6.571e+05   8393.723     78.290      0.000    6.41e+05    6.74e+05
    waterfront             8.766e+05   1.83e+04     47.826      0.000    8.41e+05    9.13e+05
    high_grade            -1.236e+06   1.86e+04    -66.303      0.000   -1.27e+06    -1.2e+06
    ==============================================================================
    Omnibus:                    17069.397   Durbin-Watson:                   1.969
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1610754.709
    Skew:                           3.174   Prob(JB):                         0.00
    Kurtosis:                      44.829   Cond. No.                         119.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_218_1.png)



```python
for col in ['price',
'zone_mean', 
'sqft_living',
'grade',
'bedrooms', 
'bathrooms',
'sqft_lot',
'floors',
'waterfront',
'view',
'condition',
'sqft_above',
'sqft_basement',
'yr_built',
'yr_renovated',
'sqft_living15',
'sqft_lot15',
'age',
'since_ren',
'renovated',
'month',
'basement',
'sub_zip_median',
'sub_zip_75q']:
    resd_check(results, data, col)
```


![png](student_files/student_219_0.png)



![png](student_files/student_219_1.png)



![png](student_files/student_219_2.png)



![png](student_files/student_219_3.png)



![png](student_files/student_219_4.png)



![png](student_files/student_219_5.png)



![png](student_files/student_219_6.png)



![png](student_files/student_219_7.png)



![png](student_files/student_219_8.png)



![png](student_files/student_219_9.png)



![png](student_files/student_219_10.png)



![png](student_files/student_219_11.png)



![png](student_files/student_219_12.png)



![png](student_files/student_219_13.png)



![png](student_files/student_219_14.png)



![png](student_files/student_219_15.png)



![png](student_files/student_219_16.png)



![png](student_files/student_219_17.png)



![png](student_files/student_219_18.png)



![png](student_files/student_219_19.png)



![png](student_files/student_219_20.png)



![png](student_files/student_219_21.png)



![png](student_files/student_219_22.png)



![png](student_files/student_219_23.png)



```python
a= data.bin_floors.sort_values()
for i in a.unique():
    print(i)
    print(len(keep_data.bin_floors.loc[data.bin_floors == i])/len(data.bin_floors.loc[data.bin_floors == i]))
```

    1
    0.9798140348088691
    2
    0.9436636493568366
    3
    0.9579288025889967
    


```python
a= data.grade.sort_values()
for i in a.unique():
    print(i)
    print(len(keep_data.grade.loc[data.grade == i])/len(data.grade.loc[data.grade == i]))
```

    3
    0.0
    4
    0.8888888888888888
    5
    0.9669421487603306
    6
    0.9906771344455348
    7
    0.9950969467350123
    8
    0.9782357790601813
    9
    0.9334608030592734
    10
    0.845679012345679
    11
    0.706766917293233
    12
    0.1797752808988764
    13
    0.46153846153846156
    


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.888
    Model:                            OLS   Adj. R-squared:                  0.888
    Method:                 Least Squares   F-statistic:                 2.855e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:54:59   Log-Likelihood:            -2.9618e+05
    No. Observations:               21597   AIC:                         5.924e+05
    Df Residuals:                   21591   BIC:                         5.924e+05
    Df Model:                           6                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living        1.278e+05   3189.532     40.074      0.000    1.22e+05    1.34e+05
    min_max_grade          1.203e+06   1.98e+04     60.910      0.000    1.16e+06    1.24e+06
    min_max_log_zone_mean  6.505e+05   8372.934     77.690      0.000    6.34e+05    6.67e+05
    waterfront             8.717e+05   1.83e+04     47.745      0.000    8.36e+05    9.07e+05
    high_grade            -1.218e+06   1.86e+04    -65.457      0.000   -1.25e+06   -1.18e+06
    bin_floors            -4.164e+04   3074.975    -13.541      0.000   -4.77e+04   -3.56e+04
    ==============================================================================
    Omnibus:                    17055.879   Durbin-Watson:                   1.971
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1633007.792
    Skew:                           3.165   Prob(JB):                         0.00
    Kurtosis:                      45.127   Cond. No.                         125.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_222_1.png)



```python
for col in ['price',
'zone_mean', 
'sqft_living',
'grade',
'bedrooms', 
'bathrooms',
'sqft_lot',
'floors',
'waterfront',
'view',
'condition',
'sqft_above',
'sqft_basement',
'yr_built',
'yr_renovated',
'sqft_living15',
'sqft_lot15',
'age',
'since_ren',
'renovated',
'month',
'basement',
'sub_zip_median',
'sub_zip_75q']:
    resd_check(results, data, col)
```


![png](student_files/student_223_0.png)



![png](student_files/student_223_1.png)



![png](student_files/student_223_2.png)



![png](student_files/student_223_3.png)



![png](student_files/student_223_4.png)



![png](student_files/student_223_5.png)



![png](student_files/student_223_6.png)



![png](student_files/student_223_7.png)



![png](student_files/student_223_8.png)



![png](student_files/student_223_9.png)



![png](student_files/student_223_10.png)



![png](student_files/student_223_11.png)



![png](student_files/student_223_12.png)



![png](student_files/student_223_13.png)



![png](student_files/student_223_14.png)



![png](student_files/student_223_15.png)



![png](student_files/student_223_16.png)



![png](student_files/student_223_17.png)



![png](student_files/student_223_18.png)



![png](student_files/student_223_19.png)



![png](student_files/student_223_20.png)



![png](student_files/student_223_21.png)



![png](student_files/student_223_22.png)



![png](student_files/student_223_23.png)



```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors', 'condition']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.889
    Model:                            OLS   Adj. R-squared:                  0.889
    Method:                 Least Squares   F-statistic:                 2.471e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:55:08   Log-Likelihood:            -2.9609e+05
    No. Observations:               21597   AIC:                         5.922e+05
    Df Residuals:                   21590   BIC:                         5.922e+05
    Df Model:                           7                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living        1.125e+05   3370.985     33.372      0.000    1.06e+05    1.19e+05
    min_max_grade          1.245e+06   1.99e+04     62.520      0.000    1.21e+06    1.28e+06
    min_max_log_zone_mean  6.407e+05   8368.624     76.564      0.000    6.24e+05    6.57e+05
    waterfront             8.664e+05   1.82e+04     47.645      0.000    8.31e+05    9.02e+05
    high_grade            -1.243e+06   1.86e+04    -66.752      0.000   -1.28e+06   -1.21e+06
    bin_floors            -3.065e+04   3167.317     -9.678      0.000   -3.69e+04   -2.44e+04
    condition              3.222e+04   2375.123     13.565      0.000    2.76e+04    3.69e+04
    ==============================================================================
    Omnibus:                    17075.033   Durbin-Watson:                   1.971
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1672176.357
    Skew:                           3.163   Prob(JB):                         0.00
    Kurtosis:                      45.641   Cond. No.                         137.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_224_1.png)



```python
data['lot_living'] = data.sqft_lot15- data.sqft_living15
data.lot_living.loc[data.lot_living <= 0] = 0

```


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors', 'condition']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.889
    Model:                            OLS   Adj. R-squared:                  0.889
    Method:                 Least Squares   F-statistic:                 2.471e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:55:09   Log-Likelihood:            -2.9609e+05
    No. Observations:               21597   AIC:                         5.922e+05
    Df Residuals:                   21590   BIC:                         5.922e+05
    Df Model:                           7                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living        1.125e+05   3370.985     33.372      0.000    1.06e+05    1.19e+05
    min_max_grade          1.245e+06   1.99e+04     62.520      0.000    1.21e+06    1.28e+06
    min_max_log_zone_mean  6.407e+05   8368.624     76.564      0.000    6.24e+05    6.57e+05
    waterfront             8.664e+05   1.82e+04     47.645      0.000    8.31e+05    9.02e+05
    high_grade            -1.243e+06   1.86e+04    -66.752      0.000   -1.28e+06   -1.21e+06
    bin_floors            -3.065e+04   3167.317     -9.678      0.000   -3.69e+04   -2.44e+04
    condition              3.222e+04   2375.123     13.565      0.000    2.76e+04    3.69e+04
    ==============================================================================
    Omnibus:                    17075.033   Durbin-Watson:                   1.971
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1672176.357
    Skew:                           3.163   Prob(JB):                         0.00
    Kurtosis:                      45.641   Cond. No.                         137.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_226_1.png)



```python
a= data.bin_bathrooms.sort_values()
for i in a.unique():
    print(i)
    print(len(keep_data.bin_bathrooms.loc[data.bin_bathrooms == i])/len(data.bin_bathrooms.loc[data.bin_bathrooms == i]))
```

    (0.0, 1.75]
    0.9901518747033697
    (1.75, 3.75]
    0.9592699929505757
    (3.75, 8.0]
    0.6169154228855721
    

#### Using the bin_bathrooms
us


```python
data.bin_bathrooms = data.bin_bathrooms.astype(str)

data.bin_bathrooms.loc[data.bin_bathrooms == '(3.75, 8.0]'] = '0'
data.bin_bathrooms.loc[data.bin_bathrooms == '(0.0, 1.75]'] = '1'
data.bin_bathrooms.loc[data.bin_bathrooms == '(1.75, 3.75]'] = '2'
data.bin_bathrooms = data.bin_bathrooms.astype(int)
```


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors', 'condition', 'bin_bathrooms']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.891
    Model:                            OLS   Adj. R-squared:                  0.891
    Method:                 Least Squares   F-statistic:                 2.213e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:55:10   Log-Likelihood:            -2.9586e+05
    No. Observations:               21597   AIC:                         5.917e+05
    Df Residuals:                   21589   BIC:                         5.918e+05
    Df Model:                           8                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living        1.214e+05   3362.457     36.103      0.000    1.15e+05    1.28e+05
    min_max_grade          1.274e+06   1.98e+04     64.502      0.000    1.24e+06    1.31e+06
    min_max_log_zone_mean  6.328e+05   8290.969     76.322      0.000    6.17e+05    6.49e+05
    waterfront             8.494e+05    1.8e+04     47.150      0.000    8.14e+05    8.85e+05
    high_grade            -1.236e+06   1.84e+04    -67.055      0.000   -1.27e+06    -1.2e+06
    bin_floors            -1.193e+04   3256.084     -3.663      0.000   -1.83e+04   -5544.462
    condition              3.031e+04   2352.409     12.883      0.000    2.57e+04    3.49e+04
    bin_bathrooms         -6.649e+04   3126.821    -21.264      0.000   -7.26e+04   -6.04e+04
    ==============================================================================
    Omnibus:                    16189.264   Durbin-Watson:                   1.975
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1460990.144
    Skew:                           2.920   Prob(JB):                         0.00
    Kurtosis:                      42.868   Cond. No.                         139.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_230_1.png)



```python


ols, results = make_ols_model(df=keep_data, target='price', columns_to_use=columns, add_constant=True)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.944
    Model:                            OLS   Adj. R-squared:                  0.944
    Method:                 Least Squares   F-statistic:                 4.352e+04
    Date:                Fri, 27 Sep 2019   Prob (F-statistic):               0.00
    Time:                        10:55:11   Log-Likelihood:            -2.7552e+05
    No. Observations:               20835   AIC:                         5.510e+05
    Df Residuals:                   20827   BIC:                         5.511e+05
    Df Model:                           8                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living         1.71e+05   2799.350     61.068      0.000    1.65e+05    1.76e+05
    min_max_grade          9.031e+05    1.3e+04     69.240      0.000    8.78e+05    9.29e+05
    min_max_log_zone_mean  5.525e+05   5356.026    103.164      0.000    5.42e+05    5.63e+05
    waterfront             7.999e+05   1.64e+04     48.663      0.000    7.68e+05    8.32e+05
    high_grade            -1.481e+06   1.71e+04    -86.503      0.000   -1.51e+06   -1.45e+06
    bin_floors            -5102.8392   2074.898     -2.459      0.014   -9169.800   -1035.878
    condition              3.239e+04   1505.598     21.513      0.000    2.94e+04    3.53e+04
    bin_bathrooms         -4.017e+04   2094.861    -19.174      0.000   -4.43e+04   -3.61e+04
    ==============================================================================
    Omnibus:                     1244.503   Durbin-Watson:                   1.992
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):             1777.209
    Skew:                           0.529   Prob(JB):                         0.00
    Kurtosis:                       3.962   Cond. No.                         177.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_231_1.png)



```python

```


```python

```


```python
linreg = LinearRegression(fit_intercept= False)

X = keep_data[columns]
Y = keep_data.price


cv_5_results = cross_val_score(linreg, X, Y, cv=5, scoring="r2", )
```


```python
print(cv_5_results)
print(Y.std(), Y.mean())
l
```

    [0.72062535 0.73258769 0.734121   0.73924078 0.7308158 ]
    258796.8683912203 500636.1260859131
    


    ---------------------------------------------------------------------------

    NotFittedError                            Traceback (most recent call last)

    <ipython-input-450-b13ce6700eea> in <module>()
          1 print(cv_5_results)
          2 print(Y.std(), Y.mean())
    ----> 3 linreg.score(X,Y)
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\base.py in score(self, X, y, sample_weight)
        406         from .metrics import r2_score
        407         from .metrics.regression import _check_reg_targets
    --> 408         y_pred = self.predict(X)
        409         # XXX: Remove the check in 0.23
        410         y_type, _, _, _ = _check_reg_targets(y, y_pred, None)
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\linear_model\base.py in predict(self, X)
        219             Returns predicted values.
        220         """
    --> 221         return self._decision_function(X)
        222 
        223     _preprocess_data = staticmethod(_preprocess_data)
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\linear_model\base.py in _decision_function(self, X)
        200 
        201     def _decision_function(self, X):
    --> 202         check_is_fitted(self, "coef_")
        203 
        204         X = check_array(X, accept_sparse=['csr', 'csc', 'coo'])
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\utils\validation.py in check_is_fitted(estimator, attributes, msg, all_or_any)
        912 
        913     if not all_or_any([hasattr(estimator, attr) for attr in attributes]):
    --> 914         raise NotFittedError(msg % {'name': type(estimator).__name__})
        915 
        916 
    

    NotFittedError: This LinearRegression instance is not fitted yet. Call 'fit' with appropriate arguments before using this method.



```python
linreg.score(X, Y)
```


    ---------------------------------------------------------------------------

    NotFittedError                            Traceback (most recent call last)

    <ipython-input-431-a5431be71de5> in <module>()
    ----> 1 linreg.score(X, Y)
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\base.py in score(self, X, y, sample_weight)
        406         from .metrics import r2_score
        407         from .metrics.regression import _check_reg_targets
    --> 408         y_pred = self.predict(X)
        409         # XXX: Remove the check in 0.23
        410         y_type, _, _, _ = _check_reg_targets(y, y_pred, None)
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\linear_model\base.py in predict(self, X)
        219             Returns predicted values.
        220         """
    --> 221         return self._decision_function(X)
        222 
        223     _preprocess_data = staticmethod(_preprocess_data)
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\linear_model\base.py in _decision_function(self, X)
        200 
        201     def _decision_function(self, X):
    --> 202         check_is_fitted(self, "coef_")
        203 
        204         X = check_array(X, accept_sparse=['csr', 'csc', 'coo'])
    

    ~\.conda\envs\learn-env\lib\site-packages\sklearn\utils\validation.py in check_is_fitted(estimator, attributes, msg, all_or_any)
        912 
        913     if not all_or_any([hasattr(estimator, attr) for attr in attributes]):
    --> 914         raise NotFittedError(msg % {'name': type(estimator).__name__})
        915 
        916 
    

    NotFittedError: This LinearRegression instance is not fitted yet. Call 'fit' with appropriate arguments before using this method.



```python
flip.plot.hexbin(figsize= (13,8), x= 'long', y= 'lat', C='change_price')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2a5ee4d2198>




![png](student_files/student_237_1.png)



```python
data.loc[data.zone_mean > data.zone_mean.quantile(.95)].plot.hexbin(figsize= (13,8), x= 'long', y= 'lat', C='zone_count', add)

```




    <matplotlib.axes._subplots.AxesSubplot at 0x2a5ebea37f0>




![png](student_files/student_238_1.png)


### Interpret


```python
def generateBaseMap(default_location=[40.693943, -73.985880], default_zoom_start=12):
    base_map = folium.Map(location=default_location, control_scale=True, zoom_start=default_zoom_start, zoom_control= True)
    return base_map 
def feature_heatmap(data, feature_column= 'price', radius= 8):
    base_map = generateBaseMap(default_location=[data.lat.mean()-data.lat.std(), data.long.median()], default_zoom_start=9.0)
    HeatMap(data=data[['lat', 'long', feature_column]].groupby(['lat', 'long']).sum().reset_index().values.tolist(), radius= radius, max_zoom=13).add_to(base_map)
    return base_map
```


```python
feature_heatmap(data, radius= 5)
```








```python


base_map = generateBaseMap(default_location=[data.lat.mean()-data.lat.std(), data.long.median()], default_zoom_start=9.0)
HeatMap(data=flip[['lat', 'long', 'price']].groupby(['lat', 'long']).sum().reset_index().values.tolist(), radius=7, max_zoom=13).add_to(base_map)
base_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF9lNzIzOWUzNzI4NGQ0NDMzOWM0ZmI0M2RhZjBkMzcwNSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9sZWFmbGV0LmdpdGh1Yi5pby9MZWFmbGV0LmhlYXQvZGlzdC9sZWFmbGV0LWhlYXQuanMiPjwvc2NyaXB0Pgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2U3MjM5ZTM3Mjg0ZDQ0MzM5YzRmYjQzZGFmMGQzNzA1IiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF9lNzIzOWUzNzI4NGQ0NDMzOWM0ZmI0M2RhZjBkMzcwNSA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF9lNzIzOWUzNzI4NGQ0NDMzOWM0ZmI0M2RhZjBkMzcwNSIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDcuNDIxNTQxMjI2MjI0MywgLTEyMi4yMzEwMDAwMDAwMDAwMV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA5LjAsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKICAgICAgICAgICAgTC5jb250cm9sLnNjYWxlKCkuYWRkVG8obWFwX2U3MjM5ZTM3Mjg0ZDQ0MzM5YzRmYjQzZGFmMGQzNzA1KTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl81MjVjNThlNDc3MjQ0YTMzOGVmYmVlN2VhZTZhZmU2MSA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZTcyMzllMzcyODRkNDQzMzljNGZiNDNkYWYwZDM3MDUpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBoZWF0X21hcF82NGVlNThjZDdhNmU0MDg3YmI2MTRiM2NiZmMxZDkwYyA9IEwuaGVhdExheWVyKAogICAgICAgICAgICAgICAgW1s0Ny4yNzI5LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxNjAwMDAuMF0sIFs0Ny4yNzQxLCAtMTIyLjMzNywgMzEwMDAwLjBdLCBbNDcuMjg5MiwgLTEyMi4yMiwgMTI1MDAwLjBdLCBbNDcuMjkzMywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMTk1MDAwLjBdLCBbNDcuMjk1MSwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMTE1MDAwLjBdLCBbNDcuMjk3OSwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMTgyNzAwLjBdLCBbNDcuMzA3NiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMTM2NTAwLjBdLCBbNDcuMzA5NSwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMjk5MDAwLjBdLCBbNDcuMzIwNiwgLTEyMi4yNjg5OTk5OTk5OTk5OSwgNDE5MTAwLjBdLCBbNDcuMzI4MSwgLTEyMi4zNDI5OTk5OTk5OTk5OSwgOTY1MDAuMF0sIFs0Ny4zMzI3LCAtMTIyLjMwNiwgMjUwMDAwLjBdLCBbNDcuMzQ2NywgLTEyMi4zMDcsIDE1MjUwMC4wXSwgWzQ3LjM0NzMsIC0xMjIuMzE0LCAyMjUwMDAuMF0sIFs0Ny4zNjQzLCAtMTIyLjE4NSwgMzI1MDAwLjBdLCBbNDcuMzY5OSwgLTEyMi4zMDksIDM3NTAwMC4wXSwgWzQ3LjM3MzEsIC0xMjIuMjg2LCAyNzk5NTAuMF0sIFs0Ny4zNzYyLCAtMTIyLjIxOSwgMTQ1MDAwLjBdLCBbNDcuMzc3NywgLTEyMi4wMjMsIDMzNTAwMC4wXSwgWzQ3LjM4MTMsIC0xMjIuMjQzLCAyMzUwMDAuMF0sIFs0Ny4zODIyLCAtMTIyLjMxNiwgMzA5OTUwLjBdLCBbNDcuMzg2NSwgLTEyMi4yMTcwMDAwMDAwMDAwMSwgMTc1MDAwLjBdLCBbNDcuMzk0MiwgLTEyMi4xODksIDU3NDg1MC4wXSwgWzQ3LjQxMDUsIC0xMjIuMjk1LCAxNzAwMDAuMF0sIFs0Ny40MTM2LCAtMTIyLjMzMywgNjg1MDAwLjBdLCBbNDcuNDE0NSwgLTEyMi40NjMsIDIyMDUwMC4wXSwgWzQ3LjQyMTEsIC0xMjIuMjksIDQwMzUwMC4wXSwgWzQ3LjQyODgsIC0xMjIuMTk4LCAxODUwMDAuMF0sIFs0Ny40Mzc1LCAtMTIyLjE3NiwgNjY2MDAwLjBdLCBbNDcuNDM3OSwgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMjkwMDAwLjBdLCBbNDcuNDQxMywgLTEyMi4zNDg5OTk5OTk5OTk5OSwgNjg1Mjc1LjBdLCBbNDcuNDQ3NiwgLTEyMS43NzEsIDMyMjAwMC4wXSwgWzQ3LjQ0OTcsIC0xMjIuMjg5LCA0MTU1MDAuMF0sIFs0Ny40NDk5LCAtMTIyLjE4OSwgMTY4MDAwLjBdLCBbNDcuNDU1LCAtMTIyLjM1LCA1MjUwMDAuMF0sIFs0Ny40NTcyLCAtMTIyLjE2NywgNTQ1MzI1LjBdLCBbNDcuNDYwOCwgLTEyMi4zNCwgMzcxMDAwLjBdLCBbNDcuNDYxMSwgLTEyMi4zMjQsIDQ0NTAwMC4wXSwgWzQ3LjQ2NjMsIC0xMjIuMzU5LCAzOTAwMDAuMF0sIFs0Ny40NzEyLCAtMTIyLjEsIDY1NTUwMC4wXSwgWzQ3LjQ3MywgLTEyMi4xNDksIDMxNDk1MC4wXSwgWzQ3LjQ3NTYsIC0xMjIuMzAxLCAyMjQwMDAuMF0sIFs0Ny40NzU4LCAtMTIyLjI4OCwgMTYxMDAwLjBdLCBbNDcuNDc1OSwgLTEyMS43MzQsIDUwMjAwMC4wXSwgWzQ3LjQ3ODcsIC0xMjIuMjMsIDU3NzAwMC4wXSwgWzQ3LjQ4Mzk5OTk5OTk5OTk5NSwgLTEyMi4yMTEsIDE3NTAwMC4wXSwgWzQ3LjQ4NTIsIC0xMjIuMjUxLCAzMTAwMDAuMF0sIFs0Ny40ODY5LCAtMTIyLjMyNCwgMzMwMTI1LjBdLCBbNDcuNDg4NywgLTEyMi4yMjMsIDYzNTAwMC4wXSwgWzQ3LjQ4OTcsIC0xMjIuMjQsIDE2NTAwMC4wXSwgWzQ3LjQ4OTksIC0xMjIuMzM3LCAyODQ3MDAuMF0sIFs0Ny40OTA4LCAtMTIyLjIyMywgMTc1MDAwLjBdLCBbNDcuNDkyNCwgLTEyMi4yMzcwMDAwMDAwMDAwMSwgMTY1MDAwLjBdLCBbNDcuNDk2LCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxNTExMDAuMF0sIFs0Ny41MDA4LCAtMTIyLjM2NiwgNDkwMDAwLjBdLCBbNDcuNTAxNCwgLTEyMi4xNzIwMDAwMDAwMDAwMSwgMjcyMDAwLjBdLCBbNDcuNTA0NSwgLTEyMi4zMywgMTI0MDAwLjBdLCBbNDcuNTEzOCwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMzkyOTAwLjBdLCBbNDcuNTE2OCwgLTEyMS44ODMsIDYxMDAwMC4wXSwgWzQ3LjUxODQsIC0xMjEuODg2MDAwMDAwMDAwMDEsIDEyMTk4NjYuMF0sIFs0Ny41MTkyLCAtMTIyLjI2NiwgNTczMDAwLjBdLCBbNDcuNTIwOCwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMzQwMDAwLjBdLCBbNDcuNTI0OSwgLTEyMi4zNywgMjE2MDAwLjBdLCBbNDcuNTI1NiwgLTEyMi4zOCwgMjYwMDAwLjBdLCBbNDcuNTI2LCAtMTIxLjgxLCA1NDU1MDAuMF0sIFs0Ny41Mjc2LCAtMTIyLjM1OSwgNTQ2OTQwLjBdLCBbNDcuNTQxOSwgLTEyMi4yNzEsIDE2MzgwMC4wXSwgWzQ3LjU0MzQsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDMyMDAwMC4wXSwgWzQ3LjU0NDMsIC0xMjIuMTcyMDAwMDAwMDAwMDEsIDU4NTAwMC4wXSwgWzQ3LjU1MDIsIC0xMjIuMjc0LCAzMTAwMDAuMF0sIFs0Ny41NTAzLCAtMTIyLjEwMiwgMjc4MDAwMC4wXSwgWzQ3LjU1MjEsIC0xMjIuMTE1LCAxMjEwMDAwLjBdLCBbNDcuNTUyNywgLTEyMi4xMiwgMTQ5MTUwMC4wXSwgWzQ3LjU1MzQsIC0xMjIuMDAyMDAwMDAwMDAwMDEsIDE1MjIwMDAuMF0sIFs0Ny41NTU0LCAtMTIyLjI3LCA3NzMwMDAuMF0sIFs0Ny41NTc2LCAtMTIyLjI3NywgODIwMDAwLjBdLCBbNDcuNTU4NywgLTEyMS45MDQsIDY0NDUwMC4wXSwgWzQ3LjU2MDUsIC0xMjIuMTU3OTk5OTk5OTk5OTksIDE2NTAwMDAuMF0sIFs0Ny41NjE3LCAtMTIyLjE1Nzk5OTk5OTk5OTk5LCAxNDEwMDAwLjBdLCBbNDcuNTY0NCwgLTEyMi4wOTI5OTk5OTk5OTk5OSwgNTU1MDAwLjBdLCBbNDcuNTY2LCAtMTIyLjE0LCAzNzUwMDAuMF0sIFs0Ny41NjgxLCAtMTIyLjI4NSwgMTA3MDAwMC4wXSwgWzQ3LjU3MTEsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDIwMTUwMC4wXSwgWzQ3LjU3MTk5OTk5OTk5OTk5NiwgLTEyMi4yOSwgNTUwMDAwLjBdLCBbNDcuNTc1LCAtMTIyLjI4OCwgMzUwMDAwLjBdLCBbNDcuNTg2OTk5OTk5OTk5OTk2LCAtMTIyLjEzNCwgNTIyNTAwLjBdLCBbNDcuNTk3MSwgLTEyMi4yOTUsIDkxNTUwMC4wXSwgWzQ3LjYxNDcsIC0xMjIuMywgNDMwMDAwLjBdLCBbNDcuNjE3LCAtMTIyLjA1MSwgNTAwMDAwLjBdLCBbNDcuNjE5MSwgLTEyMi4wNDI5OTk5OTk5OTk5OSwgMzY4MjUwLjBdLCBbNDcuNjIzMywgLTEyMi4wNDYsIDM0NTAwMC4wXSwgWzQ3LjYzMTUsIC0xMjIuMTAxLCA2NjEwMDAuMF0sIFs0Ny42MzI4LCAtMTIyLjIzNiwgMTgxNTAwMC4wXSwgWzQ3LjYzNTksIC0xMjIuMDUyLCA3NDUwMDAuMF0sIFs0Ny42NDU4LCAtMTIyLjIxOSwgMTk0MDAwMC4wXSwgWzQ3LjY0NTgsIC0xMjEuOTA0LCA0NDM1MDAuMF0sIFs0Ny42NDkxLCAtMTIyLjA2MSwgMzEwMDAwLjBdLCBbNDcuNjQ5OSwgLTEyMi4wODgsIDU1MDAwMC4wXSwgWzQ3LjY1MDMsIC0xMjIuNDEsIDU1NDcyOS4wXSwgWzQ3LjY1OTgsIC0xMjIuMzU1LCA3MTUwMDAuMF0sIFs0Ny42NjI1LCAtMTIyLjA1OSwgMzg1MDAwLjBdLCBbNDcuNjY1MiwgLTEyMi4zMzgsIDExMTUwMDAuMF0sIFs0Ny42NjcxLCAtMTIyLjA1MSwgMzQ5MDAwMC4wXSwgWzQ3LjY3MTksIC0xMjIuMzgyLCA5ODkwMDAuMF0sIFs0Ny42NzM2LCAtMTIyLjMyMywgODUwMDAwLjBdLCBbNDcuNjc0LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCA1MjUwMDAuMF0sIFs0Ny42Nzc0LCAtMTIyLjE2NCwgMjcwMDAwLjBdLCBbNDcuNjgxMDAwMDAwMDAwMDA0LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCA2MjUwMDAuMF0sIFs0Ny42ODQxLCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCA3NDAwMDAuMF0sIFs0Ny42ODQyLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCA0MTAwMDAuMF0sIFs0Ny42ODQ1LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxMzY1MDAwLjBdLCBbNDcuNjg2NSwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMzQwMDAwLjBdLCBbNDcuNjkwMiwgLTEyMi4zODcsIDg3NDk1MC4wXSwgWzQ3LjY5NDIsIC0xMjIuMzA0LCA4NTEwMDAuMF0sIFs0Ny43MDE5LCAtMTIyLjMxMSwgMzUwMDAwLjBdLCBbNDcuNzAzNSwgLTEyMi4xNjQsIDU4OTk1MC4wXSwgWzQ3LjcwMzcsIC0xMjIuMjk2LCA5NTQ5NTAuMF0sIFs0Ny43MDc2LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAyNDAwMDAuMF0sIFs0Ny43MDc2LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCA0ODk5NTAuMF0sIFs0Ny43MDg5LCAtMTIyLjI0MSwgMTAwNDkwMC4wXSwgWzQ3LjcwODksIC0xMjIuMjAxMDAwMDAwMDAwMDEsIDM2MDAwMC4wXSwgWzQ3LjcxMTIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDUzOTAwMC4wXSwgWzQ3LjcxMTQsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDIwNzAwMC4wXSwgWzQ3LjcxMywgLTEyMi4wNzIsIDE3MzIwMDAuMF0sIFs0Ny43MTQyLCAtMTIyLjI4NiwgNDUyMDAwLjBdLCBbNDcuNzE1NSwgLTEyMi4zMTUsIDQ0OTAwMC4wXSwgWzQ3LjcxNzYsIC0xMjIuMDMyOTk5OTk5OTk5OTksIDQzMDAwMC4wXSwgWzQ3LjcxODYsIC0xMjIuMzU0LCA1NzAwMDAuMF0sIFs0Ny43MzUxLCAtMTIyLjI5NSwgMTAzMjUwMC4wXSwgWzQ3LjczNzIsIC0xMjIuMzE2LCA5NjI1MDAuMF0sIFs0Ny43MzcyLCAtMTIyLjMwNywgNDMwMDAwLjBdLCBbNDcuNzQ3OSwgLTEyMi4zMTgsIDM5OTAwMC4wXSwgWzQ3Ljc0OTMsIC0xMjIuMzUxLCA3MTg1MDAuMF0sIFs0Ny43NjA1LCAtMTIyLjA4NSwgMTExMzUwMC4wXSwgWzQ3Ljc3MTEsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDUzMDAwMC4wXSwgWzQ3Ljc3MjEsIC0xMjIuMjA2LCA0NzAwMDAuMF0sIFs0Ny43NzI0LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCA4OTAwMDAuMF1dLAogICAgICAgICAgICAgICAgeyJibHVyIjogMTUsICJtYXgiOiAxLjAsICJtYXhab29tIjogMTMsICJtaW5PcGFjaXR5IjogMC41LCAicmFkaXVzIjogN30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZTcyMzllMzcyODRkNDQzMzljNGZiNDNkYWYwZDM3MDUpOwogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
base_map = generateBaseMap(default_location=[data.lat.mean()-data.lat.std(), data.long.median()], default_zoom_start=9.0)
HeatMap(data=data[['lat', 'long', 'ones']].loc[data.zone_95q> data.zone_95q.quantile(.99)].groupby(['lat',
                                                                                                        'long']).sum().reset_index().values.tolist(), radius=7, max_zoom=13).add_to(base_map)
base_map
```