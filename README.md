
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
import random

from sklearn.metrics import r2_score

import folium.plugins as HeatMapWithTime



target= 'price'
```

### Custom Function


```python
def generateBaseMap(default_location=[40.693943, -73.985880], default_zoom_start=12):
    '''Create base map for the folium plot'''
    base_map = folium.Map(location=default_location, control_scale=True, titles= 'Stamen Toner')
    return base_map 
def feature_heatmap(data, feature_column= 'price', radius= 8):
    '''Create heatmap for given groupby dataframe with latitude and longitude data'''
    base_map = generateBaseMap(default_location=[data.lat.mean()-data.lat.std(), data.long.median()], default_zoom_start=9.0)
    HeatMap(data=data[['lat', 'long', feature_column]].groupby(['lat', 'long']).sum().reset_index().values.tolist(), radius= radius, max_zoom=13, min_opacity=.1).add_to(base_map)
    return base_map
```

See <a href="https://towardsdatascience.com/data-101s-spatial-visualizations-and-analysis-in-python-with-folium-39730da2adf">Spatial Visualizations and Analysis in Python with Folium</a>, for the source of functions.



```python
def convert_cat_str_to_num(dataseries):
    '''Converting categorical feature in string from into arbituary numerical features'''
    count = 1
   
    for i in dataseries.unique():
        dataseries.loc[dataseries == i] = count
        count += 1
    return dataseries

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
    """Adding new column to flip dataframe with the 
    difference of proceeding sales for given feature"""
    dataout=[]
    id_ = None
    previous_index = None
    for value in dataframe.id.unique():
            for index in dataframe.loc[dataframe.id == value].index:
                
                if id_ == dataframe.id.loc[index]:
                    dataout.append(dataframe[column].loc[index] - 
                                   dataframe[column].loc[previous_index])
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
    
    '''Just build a model and see the output'''

    X = df[columns_to_use]
    y = df[target]

    # add a constant to my X
    if add_constant:
        X = sm.add_constant(X)
    
    ols = sm.OLS(y, X)
    results = ols.fit()
    print(results.summary())
    return ols, results

def make_ols_model1(df, target='price', columns_to_use=None, add_constant=True):
    
    '''Just build a model with summary'''

    X = df[columns_to_use]
    y = df[target]

    # add a constant to my X
    if add_constant:
        X = sm.add_constant(X)
    
    ols = sm.OLS(y, X)
    results = ols.fit()
    # print(results.summary())
    return ols, results


def scat_plot(data, column, target= 'price'):
    '''Provide scatterpolt for a given dataframe for a given column'''
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
Changing 0 and ? to NaN, then back to zero.


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
We identified 177 resales of property that were sold in the period of study. We had two meanful features:

<li>change_date ---  The days between sales.<li>change_price --- The cost differential between the previous sale and current sale price.</li>
    
Condition and grade were examined, but there was no change between any of the sales.

Step of creating resale dataframe.
<ol>
    <li>Identify the property ids that are in the dataset more than once.</li>
    <li>Mark those id rows in the dataframe.</li>
    <li>Pulled marked rows into separate dataframe.</li>
    <li>Calculate difference in important features from one sale to another.</li>
    <li>Remove intial sales</li>
    
        


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

Methods for manipulating data to fit OLS assumptions.
<ul>
    <li>Binning values</li>
    <li>Playing with null values</li>
    <li>Normalizing data</li>
    <li>Standardizing data</li>
</ul>

Grouping data to create categories that might make sense for your data

#### id
Unique identified for a house. 
Although not valuable for predictive modeling, the feature id does identify homes that have been sold multiple times.


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
The date  the house was sold. The initial value was a string value. The information was convert to a date. See Data Scrubbing section for more information. The date is 5/2/2014 to 5/27/2015. 




```python
print(data.date.min(),'-',data.date.max())

```

    2014-05-02 00:00:00 - 2015-05-27 00:00:00
    

The date range is 5/2/2014 to 5/27/2015

#### bedrooms
Number of Bedrooms/House
The initial values ranged from 1 to 33 bedrooms, but there are very fews value over 6. 



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='bedrooms', y='price', data= data)
plt.title('Bedrooms v. Price Violin Plot');
```


![png](student_files/student_49_0.png)


#### Bedrooms v. Price Violin Plot
-skewed to data in all categories


```python
a=data.bedrooms.value_counts(sort=False);
a.sort_index().plot(kind= 'barh')
plt.title('Initial Bedroom Data');
plt.xlabel('Count');
plt.ylabel('Bedrooms');

```


![png](student_files/student_51_0.png)


The majority of the entries have 3 and 4 bedrooms. 

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


![png](student_files/student_54_0.png)


#### bin_bathroom
Create to see if it limited skewness. It did not help


```python
bins = [0, 1.75, 3.75, 8.0]
data['bin_bathrooms'] = binning(data['bathrooms'], bins)
bar_hist_plot(data['bin_bathrooms'])
plt.title('Binned Bathroom Counts')
plt.xlabel('Count')
plt.ylabel('Number of Bathrooms')
```




    Text(0, 0.5, 'Number of Bathrooms')




![png](student_files/student_56_1.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='bin_bathrooms', y='price', data= data.loc[data.price< data.price.quantile(.95)])
plt.title('Bathrooms v. Price Violin Plot');
```


![png](student_files/student_57_0.png)



```python
data.bin_bathrooms = data.bin_bathrooms.astype(str)


data.bin_bathrooms.loc[data.bin_bathrooms == '(3.75, 8.0]'] = '1'
data.bin_bathrooms.loc[data.bin_bathrooms == '(0.0, 1.75]'] = '2'
data.bin_bathrooms.loc[data.bin_bathrooms == '(1.75, 3.75]'] = '3'
data.bin_bathrooms = data.bin_bathrooms.astype(int)
```


```python

```


```python
plt.figure(figsize= (13, 8))

#sns.violinplot(x= binning(data['bathrooms'],5), y= data[target]);
sns.scatterplot(x= data.bathrooms, y= data[target]);
plt.title('Bathrooms v. Price');
```


![png](student_files/student_60_0.png)


#### Bathroom 
<ul>
    <li>Skewed to data in all categories</li>
    <li>Simliar to bedroom</li>
    <li>High heteroscedasticity </li>
    <li>Seems to be a linearity relationship </li>
</ul>

#### sqft_living
Square footage of the home 

The range is 370 sqft to 13540 sqft. 


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_living)
plt.title('Initial Living Space Sqft Data');
plt.xlabel('Sqft');

```


![png](student_files/student_63_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x= binning(data['sqft_living'],10), y= data[target]);
plt.title('Living Space Sqft v Price');
```


![png](student_files/student_64_0.png)



```python
plt.figure(figsize= (13, 8));
scat_plot(data, 'sqft_living');
plt.title('Living Space Sqft v Price');
```


    <Figure size 936x576 with 0 Axes>



![png](student_files/student_65_1.png)


#### Square Footage for Living Space v. Price Violin Plot
<ul>
    <li>Skewed to data in all categories</li>
    <li>Seems nonlinear, may need log tranform (</li>
    <li>High heteroscedasticity </li>
</ul>

Creating data log_sqft_living and price_per_sqft_living (not used)


```python
data['log_sqft_living'] = np.log(data["sqft_living"])
```


```python
data['price_per_sqft_living'] = data.price/ data.sqft_living
```

##### sqft_lot
Square footage of the lot
The range is 520 sqft to 1,651,359 sqft.
<ul>
    <li>Highly skewed</li>
    <li>Not used</li>
</ul>


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_lot, hist=False)
plt.title('Initial Lot Space Sqft Data');
plt.xlabel('Sqft');
plt.ylabel('Count');
```


![png](student_files/student_70_0.png)



```python
plt.figure(figsize= (13, 8))
sns.scatterplot(x= data.sqft_lot, y= data.price)
plt.title('Initial Lot Space Sqft Data');
plt.xlabel('Sqft');
plt.ylabel('Count');
```


![png](student_files/student_71_0.png)


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


![png](student_files/student_73_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='floors', y='price', data= data)
plt.title('Floors v. Price Violin Plot');
```


![png](student_files/student_74_0.png)



```python
scat_plot(data, 'floors')
plt.title('Floors v. Price');
```


![png](student_files/student_75_0.png)


#### bin_floors
Rounded half sizes down.


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


![png](student_files/student_80_0.png)


#### waterfront
House which has a view to a waterfront. Both values are skewed, but with waterfront is less skewed.




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


![png](student_files/student_83_0.png)


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


![png](student_files/student_85_0.png)



```python
plt.figure(figsize= (13, 8))
sns.violinplot(x='view', y='price', data= data)
plt.title('view v. Price Violin Plot');
```


![png](student_files/student_86_0.png)


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


![png](student_files/student_89_0.png)



```python
viol_plot(data, 'condition')
```


![png](student_files/student_90_0.png)


#### Analysis of Condition
High skewness in the high count values.

#### grade
overall grade given to the housing unit, based on King County grading system


```python
a = data.grade.value_counts()
a.sort_index().plot(kind= 'bar')
plt.title('Initial Grade Data');

plt.xlabel('Grade');
plt.ylabel('Count');
```


![png](student_files/student_93_0.png)



```python
viol_plot(data, 'grade')
```


![png](student_files/student_94_0.png)



```python
scat_plot(data, 'grade')
```


![png](student_files/student_95_0.png)


#### Analysis of Grade.
Skewness increase as the grade increases
High Heteroscedasticity, variability to increase as the grade goes up.

#### sqft_above
Square footage of house apart from basement


```python

plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_above, bins=10)
plt.title('Initial Sqft Except from Basement Data');
plt.xlabel('Sqft');
plt.ylabel('Count');
```


![png](student_files/student_98_0.png)



```python
scat_plot(data, 'sqft_above')
```


![png](student_files/student_99_0.png)



```python
data['binning_sqft_above'] = binning(data.sqft_above, 10)

bar_hist_plot(data.binning_sqft_above)
data = data.drop(columns= 'binning_sqft_above')
```


![png](student_files/student_100_0.png)


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


![png](student_files/student_103_0.png)



```python
scat_plot(data, 'sqft_basement')
```


![png](student_files/student_104_0.png)


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


![png](student_files/student_107_0.png)



```python
data.yr_built.describe()
[1899, 1951, 1975, 1997, 2015]
pd.cut(data.yr_built, [1899, 1912, 1929, 1939, 1951, 1975, 1997, 2016])

bar_hist_plot(pd.cut(data.yr_built, [1899, 1912, 1929, 1939, 1951, 1975, 1997, 2016]))

```


![png](student_files/student_108_0.png)



```python
plt.figure(figsize=(13,8))
sns.violinplot(x= pd.cut(data.yr_built, [1899, 1912, 1929, 1939, 1951, 1975, 1997, 2016]), y= data.price);


```


![png](student_files/student_109_0.png)



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


![png](student_files/student_112_0.png)


#### zipcode


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.zipcode, bins=70)
plt.title('Initial Zipcode Data');
plt.xlabel('Year');
```


![png](student_files/student_114_0.png)


Zipcode is a categorical feature and does not lend it self to linear regression. The data will into a more usable form.It is an geographic area that could be associated with price using a pandas groupby method and statistic method like mean. 


```python
a=data.groupby(['zipcode'])['price'].mean()
data['zip_mean'] = data.zipcode.apply(lambda x: a[x])
scat_plot(data, 'zip_mean')
plt.title('Average Price by Zipcode v. Price')

```




    Text(0.5, 1.0, 'Average Price by Zipcode v. Price')




![png](student_files/student_116_1.png)



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




![png](student_files/student_117_1.png)



```python
a= binning(data.zip_median, 6)
viol2(x=a, y=data.price)
```


![png](student_files/student_118_0.png)



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




![png](student_files/student_119_1.png)



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


![png](student_files/student_123_0.png)



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


![png](student_files/student_125_0.png)


#### long
Longitude coordinate


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.long, bins=10)
plt.title('Initial Longitude Data');
plt.xlabel('Year');
```


![png](student_files/student_127_0.png)


#### zone
Creates equal spaced zone boxes accross the latitude and longitude ranges of the dataset.


```python
a = np.array(range(1,50+1))* (data.lat.max()-data.lat.min())/ 50+ data.lat.min()
a[0]= a[0]-1
data['bin_lat'] = binning(data.lat, a)
data['bin_lat']  = data['bin_lat'].astype(str)
data['bin_lat'] = data['bin_lat'].fillna('1')
print ( a[1],',',  a[2],'The length of box latitude', a[2]-a[1])
# Each bin is 0.86 miles long.


a = np.array(range(1,50+1))* (data.long.max()-data.long.min())/ 50+ data.long.min()
a[0]= a[0]-1
data['bin_long'] = binning(data.long, a)
data['bin_long'] = data['bin_long'].astype(str)
data['bin_long'] = data['bin_long'].fillna('1')
print ( a[1],',', a[2],',The length of box longitude', a[2]-a[1])
# Each bins is 1.131 miles long.
print(1.131* .86,' sq miles bins')
a = convert_cat_str_to_num(data['bin_lat'])
b = convert_cat_str_to_num(data['bin_long'])
data['zone'] = b*100 + a
#multipling longitude bin number by 100 and add to latitude bin 
#to create unique bin number for zone
```

    47.180768 , 47.193202 The length of box latitude 0.012433999999998946
    -122.47084 , -122.44676 ,The length of box longitude 0.02407999999999788
    0.97266  sq miles bins
    


```python

data.plot.hexbin(x='long', y='lat', C='zone', figsize=(20, 20))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2518c6350f0>




![png](student_files/student_130_1.png)



```python
a= data.groupby(['zone'])['price'].mean()
data['zone_mean'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_mean')
data['log_zone_mean'] = np.log(data['zone_mean'])

data['min_max_zone_mean'] = min_max(dataseries=data.zone_mean)

```


![png](student_files/student_131_0.png)



```python
a= data.groupby(['zone'])['price'].quantile(.95)
data['zone_95q'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_95q')
```


![png](student_files/student_132_0.png)



```python
data['log_zone_95q'] = np.log(data['zone_95q'])
```


```python
a= data.groupby(['zone'])['price'].max()
data['zone_max'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_max')


```


![png](student_files/student_134_0.png)



```python
a= data.groupby(['zone'])['price'].count()
data['zone_count'] = data.zone.apply(lambda x: a[x])
```


```python

a= data.groupby(['zone'])['price_per_sqft_living'].mean()
data['zone_price_sqft_living'] = data.zone.apply(lambda x: a[x])
scat_plot(data, 'zone_price_sqft_living')

```


![png](student_files/student_136_0.png)


#### sqft_living15
The square footage of interior housing living space for the nearest 15 neighbors. The range is 399 to 6210 sq ft


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_living15, bins=100)
plt.title('Initial Living Space of Neighbors Data');
plt.xlabel('Year');
```


![png](student_files/student_138_0.png)



```python
scat_plot(data, 'sqft_living15')
```


![png](student_files/student_139_0.png)



```python
data.sqft_living15.describe()
sns.violinplot(x= binning(data.sqft_living15, [398, 1490, 1840, 2360, 6210]), y= data.price);
```


![png](student_files/student_140_0.png)


#### sqft_lot15
The square footage of the land lots of the nearest 15 neighbors. The range is 651 to 871,200 sq ft.


```python
plt.figure(figsize= (13, 8))
sns.distplot(data.sqft_lot15, bins=100)
plt.title('Initial Living Space of Neighbors Data');
plt.xlabel('Year');
```


![png](student_files/student_142_0.png)



```python

viol2(binning(data.sqft_lot15, equal_bin(data.sqft_lot15)), data.price)


```


![png](student_files/student_143_0.png)


#### change_date  (flip)



```python
plt.figure(figsize= (13, 8))
sns.distplot(flip.change_date, bins=30);


```


![png](student_files/student_145_0.png)


#### month

January = 1, February = 2, ... 


```python
plt.figure(figsize= (13, 8))
sns.violinplot(x= data.month, y= data.price);
```


![png](student_files/student_147_0.png)



```python
plt.figure(figsize= (13, 8))
data[['zone', 'price', 'month']].groupby('month').quantile(.95).mean()
```




    0.95
    price    1.146417e+06
    zone     2.201550e+03
    dtype: float64




    <Figure size 936x576 with 0 Axes>



```python

```


```python

```


```python
plt.figure(figsize= (13, 8))
a = data.month.value_counts()
a.sort_index().plot(kind= 'barh');
plt.ylabel('Count');
```


![png](student_files/student_151_0.png)



```python
data['log_grade'] = np.log(data["grade"])

```

#### Creating a Possible Flipped Sale Database
We identified 177 resales of property that were sold in the period of study. We two meanful features:
<ul>
    <li>change_date  The days between sales.</li>
    <li>change_price The cost differential between the previous sale and current sale pric</li>
</ul>

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


![png](student_files/student_155_0.png)



```python
data['scaled_log_grade'] = (data['log_grade']-np.mean(data['log_grade']))/np.sqrt(np.var(data['log_grade']))

```


```python
scat_plot(data, 'scaled_log_grade')
```


![png](student_files/student_157_0.png)



```python
sns.violinplot(x = (1 *(data.price > data.zone_95q)), y = data.price)

a = 1 *(data.price > data.zone_95q)
a.sum()/len(data)

```




    0.0682039172107237




![png](student_files/student_158_1.png)



```python

```

### Exploration (Analysis)

#### Confusion Matrix


```python
make_heatmap(data=data, figsize=(20, 20))
```


![png](student_files/student_162_0.png)



```python
data.corr().price.abs().sort_values(ascending= False)

```




    price                     1.000000
    min_max_zone_mean         0.739220
    zone_mean                 0.739220
    sqft_living               0.701917
    log_zone_mean             0.697515
    zone_95q                  0.696062
    log_zone_95q              0.668947
    grade                     0.667951
    sub_zip_75q               0.651066
    sub_zip_median            0.647361
    zip_mean                  0.638168
    scaled_log_grade          0.635153
    log_grade                 0.635153
    zip_75q                   0.634993
    zip_median                0.632561
    log_sqft_living           0.611839
    sqft_above                0.605368
    sqft_living15             0.585241
    zone_price_sqft_living    0.575843
    zone_max                  0.568236
    price_per_sqft_living     0.556056
    bathrooms                 0.525906
    view                      0.393497
    sqft_basement             0.321108
    zip_median_lat            0.310784
    bedrooms                  0.308787
    lat                       0.306692
    waterfront                0.264306
    floors                    0.256804
    bin_floors                0.237264
    yr_renovated              0.117855
    renovated                 0.117543
    bin_lat                   0.102327
    bin_bathrooms             0.097678
    sqft_lot                  0.089876
    sqft_lot15                0.082845
    since_ren                 0.064950
    age                       0.053953
    yr_built                  0.053953
    zipcode                   0.053402
    zip_count                 0.050969
    bin_long                  0.043968
    zone                      0.042110
    condition                 0.036056
    long                      0.022036
    zip_EW                    0.017120
    id                        0.016772
    month                     0.009928
    zone_count                0.007547
    ones                           NaN
    Name: price, dtype: float64



#### Scatter Matrix
Because living space is the most correlated variable, we going to start OLS with sqft_living and add to the OLS going down the correlation list above.  

#### Modeling / Cross Validation


```python
test= pd.DataFrame()
```


```python
make_ols_model1(df=data, target='price', columns_to_use='sqft_living' , add_constant=True);
```

#### Data OLS Test 1
<ul>
    <li>R squared is low</li>
    <li>cond no. is high. I will remove constant to see if the improves.</li>
<ul>


```python
make_ols_model1(df=data, target='price', columns_to_use='sqft_living' , add_constant=False);
```

#### Data OLS Test 2
<ul>
    <li>R squared improved</li>
    <li>Linearity good (only variable).</li>
    <li>Skewness is a problem. -Will try log transform.</li>
<ul>



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
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:26:55   Log-Likelihood:            -3.0631e+05
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
    




    (<statsmodels.regression.linear_model.OLS at 0x2518cc0f438>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2518cc0f8d0>)



#### Data OLS Test 3

<ul>
    <li>R squared better</li>
    <li>Linearity good (only variable).</li>
    <li>Adding grade</li>
<ul>




```python
columns= ['log_sqft_living', 'grade']
make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
```




    (<statsmodels.regression.linear_model.OLS at 0x2518ca7d550>,
     <statsmodels.regression.linear_model.RegressionResultsWrapper at 0x2518ca7d710>)



#### Data OLS Test 4
<ul>
    <li>R squared better</li>
    <li>Linearity good.</li>
    <li>Normality is better, but not good.</li>
    <li>Skewing got worst but only by a little. Will try log transform.</li>
</ul>



```python
data['log_grade'] = np.log(data["grade"])
columns= ['log_sqft_living', 'log_grade']
make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False);
```

#### Data OLS Test 5

<ul>
    <li>R squared worse</li>
    <li>Linearity good</li>
    <li>Skewing got better. -Adding basementsqft</li>
</ul>



```python

columns= ['log_sqft_living', 'log_grade', 'sqft_basement']
make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False);
```

#### Data OLS Test 8

<ul>
    <li>R squared worse</li>
    <li>Linearity got worse</li>
    <li>Skewing got better. -Will make basement a categorical feature</li>
</ul>



```python
data['basement'] = data.sqft_basement.replace(to_replace= 0, value= np.nan)
data.basement = data.basement/data.basement
data.basement = data.basement.fillna(0)
columns= ['log_sqft_living', 'log_grade', 'basement']
make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False);
```

#### Data OLS Test 9
<ul>
    <li>R squared better</li>
    <li>Linearity better</li>
    <li>Normality is worse.</li>
    <li>Skewing got worst but only by a little. -Taking out basement and add lat next</li>
</ul>



```python

columns= ['log_sqft_living', 'grade', 'lat']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)


```

#### Data OLS Test 10
<ul>
    <li>R squared better</li>
    <li>Skewing got worst but only by a little. -Try log transform</li>
</ul>



```python
data['log_lat'] = np.log(data.lat)
columns= ['log_sqft_living', 'log_grade', 'log_lat']
ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
```

#### Data OLS Test 11
<ul>
    <li>R squared worse</li>
    <li>I going to see if we can create better location feature. Going to replace lat with zip_mean.</li>
<ul>


```python
columns= ['log_sqft_living', 'grade', 'zip_mean']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.861
    Model:                            OLS   Adj. R-squared:                  0.861
    Method:                 Least Squares   F-statistic:                 4.457e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:31:50   Log-Likelihood:            -2.9852e+05
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
    


![png](student_files/student_186_1.png)


#### OLS Test 12
<ul>
    <li>R squared worst than test 9</li>
    <li>Cond. No. is out of control.</li>
<ul>


```python

columns= ['log_sqft_living', 'grade', 'zip_median']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.857
    Model:                            OLS   Adj. R-squared:                  0.857
    Method:                 Least Squares   F-statistic:                 4.326e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:31:51   Log-Likelihood:            -2.9880e+05
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
    

#### Test 13
Not much better than test 14. - tray higher quantile


```python
columns= ['log_sqft_living', 'grade', 'zip_75q']
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]

```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.861
    Model:                            OLS   Adj. R-squared:                  0.861
    Method:                 Least Squares   F-statistic:                 4.468e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:31:51   Log-Likelihood:            -2.9850e+05
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
    

#### Test 14 
Better than 13. 

Linearity is still problem


```python
columns= ['log_sqft_living', 'grade', 'sub_zip_median' ]
ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
tests= [results.rsquared, results.condition_number]

```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.860
    Model:                            OLS   Adj. R-squared:                  0.860
    Method:                 Least Squares   F-statistic:                 4.411e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:31:51   Log-Likelihood:            -2.9862e+05
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
    

#### OLS Test 15
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
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:31:51   Log-Likelihood:            -3.0217e+05
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
    


![png](student_files/student_194_1.png)


Residual is very skewed to the lower end. 


```python
len(a.loc[a.abs() > 2])/len(a)
```




    0.0



#### OLS Test 16
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
    Dep. Variable:                  price   R-squared:                       0.884
    Model:                            OLS   Adj. R-squared:                  0.884
    Method:                 Least Squares   F-statistic:                 5.480e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:31:51   Log-Likelihood:            -2.9657e+05
    No. Observations:               21597   AIC:                         5.932e+05
    Df Residuals:                   21594   BIC:                         5.932e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.139e+05   1748.673    -65.161      0.000   -1.17e+05   -1.11e+05
    grade            1.295e+05   1837.322     70.467      0.000    1.26e+05    1.33e+05
    zone_mean           0.7659      0.006    121.165      0.000       0.754       0.778
    ==============================================================================
    Omnibus:                    23764.487   Durbin-Watson:                   1.986
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          5839291.087
    Skew:                           5.241   Prob(JB):                         0.00
    Kurtosis:                      82.870   Cond. No.                     1.00e+06
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large,  1e+06. This might indicate that there are
    strong multicollinearity or other numerical problems.
    


![png](student_files/student_198_1.png)



```python
a=results.resid/results.resid.std()
a = a.loc[a.abs() > 2]
len(a)/len(results.resid)
```




    0.03273602815205816




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


![png](student_files/student_200_0.png)



![png](student_files/student_200_1.png)



![png](student_files/student_200_2.png)



![png](student_files/student_200_3.png)



![png](student_files/student_200_4.png)



![png](student_files/student_200_5.png)



![png](student_files/student_200_6.png)



![png](student_files/student_200_7.png)



![png](student_files/student_200_8.png)



![png](student_files/student_200_9.png)



![png](student_files/student_200_10.png)



![png](student_files/student_200_11.png)



![png](student_files/student_200_12.png)



![png](student_files/student_200_13.png)



![png](student_files/student_200_14.png)



![png](student_files/student_200_15.png)



![png](student_files/student_200_16.png)



![png](student_files/student_200_17.png)



![png](student_files/student_200_18.png)



![png](student_files/student_200_19.png)



![png](student_files/student_200_20.png)



![png](student_files/student_200_21.png)



![png](student_files/student_200_22.png)



![png](student_files/student_200_23.png)



![png](student_files/student_200_24.png)



![png](student_files/student_200_25.png)



![png](student_files/student_200_26.png)



![png](student_files/student_200_27.png)



![png](student_files/student_200_28.png)



![png](student_files/student_200_29.png)



![png](student_files/student_200_30.png)


#### OLS Test 17
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
    Dep. Variable:                  price   R-squared:                       0.805
    Model:                            OLS   Adj. R-squared:                  0.805
    Method:                 Least Squares   F-statistic:                 2.972e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:32:04   Log-Likelihood:            -3.0217e+05
    No. Observations:               21597   AIC:                         6.043e+05
    Df Residuals:                   21594   BIC:                         6.044e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    ===================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------
    log_sqft_living -1.351e+05   5937.652    -22.754      0.000   -1.47e+05   -1.23e+05
    grade            2.176e+05   2240.300     97.113      0.000    2.13e+05    2.22e+05
    log_zone_mean   -7662.4648   2876.198     -2.664      0.008   -1.33e+04   -2024.904
    ==============================================================================
    Omnibus:                    19902.190   Durbin-Watson:                   1.973
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          1886354.554
    Skew:                           4.121   Prob(JB):                         0.00
    Kurtosis:                      48.037   Cond. No.                         57.4
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_202_1.png)


#### Test 18
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


![png](student_files/student_204_0.png)



![png](student_files/student_204_1.png)



![png](student_files/student_204_2.png)



![png](student_files/student_204_3.png)



![png](student_files/student_204_4.png)



![png](student_files/student_204_5.png)



![png](student_files/student_204_6.png)



![png](student_files/student_204_7.png)



![png](student_files/student_204_8.png)



![png](student_files/student_204_9.png)



![png](student_files/student_204_10.png)



![png](student_files/student_204_11.png)



![png](student_files/student_204_12.png)



![png](student_files/student_204_13.png)



![png](student_files/student_204_14.png)



![png](student_files/student_204_15.png)



![png](student_files/student_204_16.png)



![png](student_files/student_204_17.png)



![png](student_files/student_204_18.png)



![png](student_files/student_204_19.png)



![png](student_files/student_204_20.png)



![png](student_files/student_204_21.png)



![png](student_files/student_204_22.png)



![png](student_files/student_204_23.png)



```python
sns.distplot(min_max(data.log_zone_mean))
data['min_max_log_zone_mean'] =min_max(data.log_zone_mean)

```


![png](student_files/student_205_0.png)



```python
columns= ['log_sqft_living', 'grade', 'min_max_log_zone_mean']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.866
    Model:                            OLS   Adj. R-squared:                  0.866
    Method:                 Least Squares   F-statistic:                 4.651e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:32:14   Log-Likelihood:            -2.9812e+05
    No. Observations:               21597   AIC:                         5.963e+05
    Df Residuals:                   21594   BIC:                         5.963e+05
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    =========================================================================================
                                coef    std err          t      P>|t|      [0.025      0.975]
    -----------------------------------------------------------------------------------------
    log_sqft_living       -1.336e+05   1858.892    -71.869      0.000   -1.37e+05    -1.3e+05
    grade                  1.408e+05   1971.837     71.414      0.000    1.37e+05    1.45e+05
    min_max_log_zone_mean  1.463e+06   1.48e+04     99.152      0.000    1.43e+06    1.49e+06
    ==============================================================================
    Omnibus:                    24413.261   Durbin-Watson:                   1.984
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          5730590.268
    Skew:                           5.533   Prob(JB):                         0.00
    Kurtosis:                      82.030   Cond. No.                         98.2
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_206_1.png)


#### Test 19
Going to min_max scale the grade


```python
data['min_max_grade'] = min_max(data.grade)
```


```python
sns.distplot(min_max(data.grade))

```




    <matplotlib.axes._subplots.AxesSubplot at 0x25192d4d8d0>




![png](student_files/student_209_1.png)



```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


![png](student_files/student_210_0.png)



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


#### Test 20
Adding waterfront


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


![png](student_files/student_213_0.png)



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


![png](student_files/student_214_0.png)



![png](student_files/student_214_1.png)



![png](student_files/student_214_2.png)



![png](student_files/student_214_3.png)



![png](student_files/student_214_4.png)



![png](student_files/student_214_5.png)



![png](student_files/student_214_6.png)



![png](student_files/student_214_7.png)



![png](student_files/student_214_8.png)



![png](student_files/student_214_9.png)



![png](student_files/student_214_10.png)



![png](student_files/student_214_11.png)



![png](student_files/student_214_12.png)



![png](student_files/student_214_13.png)



![png](student_files/student_214_14.png)



![png](student_files/student_214_15.png)



![png](student_files/student_214_16.png)



![png](student_files/student_214_17.png)



![png](student_files/student_214_18.png)



![png](student_files/student_214_19.png)



![png](student_files/student_214_20.png)



![png](student_files/student_214_21.png)



![png](student_files/student_214_22.png)



![png](student_files/student_214_23.png)


#### OLS Test 21

#### Separating  residual


```python
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

#### New Variable high_grade
Binary variable for grades with that have removed residual percentages in the last model (3 and grades over 11) other grades recieved a one.


```python
data['high_grade'] = data.grade
data.high_grade.loc[(data.high_grade <12) & (data.high_grade > 3)] = 1
data.high_grade.loc[(data.high_grade >= 12) | (data.high_grade == 3)] = 0
```


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


![png](student_files/student_220_0.png)



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


![png](student_files/student_221_0.png)



![png](student_files/student_221_1.png)



![png](student_files/student_221_2.png)



![png](student_files/student_221_3.png)



![png](student_files/student_221_4.png)



![png](student_files/student_221_5.png)



![png](student_files/student_221_6.png)



![png](student_files/student_221_7.png)



![png](student_files/student_221_8.png)



![png](student_files/student_221_9.png)



![png](student_files/student_221_10.png)



![png](student_files/student_221_11.png)



![png](student_files/student_221_12.png)



![png](student_files/student_221_13.png)



![png](student_files/student_221_14.png)



![png](student_files/student_221_15.png)



![png](student_files/student_221_16.png)



![png](student_files/student_221_17.png)



![png](student_files/student_221_18.png)



![png](student_files/student_221_19.png)



![png](student_files/student_221_20.png)



![png](student_files/student_221_21.png)



![png](student_files/student_221_22.png)



![png](student_files/student_221_23.png)


#### OLS Test 22


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


![png](student_files/student_223_0.png)



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


![png](student_files/student_224_0.png)



![png](student_files/student_224_1.png)



![png](student_files/student_224_2.png)



![png](student_files/student_224_3.png)



![png](student_files/student_224_4.png)



![png](student_files/student_224_5.png)



![png](student_files/student_224_6.png)



![png](student_files/student_224_7.png)



![png](student_files/student_224_8.png)



![png](student_files/student_224_9.png)



![png](student_files/student_224_10.png)



![png](student_files/student_224_11.png)



![png](student_files/student_224_12.png)



![png](student_files/student_224_13.png)



![png](student_files/student_224_14.png)



![png](student_files/student_224_15.png)



![png](student_files/student_224_16.png)



![png](student_files/student_224_17.png)



![png](student_files/student_224_18.png)



![png](student_files/student_224_19.png)



![png](student_files/student_224_20.png)



![png](student_files/student_224_21.png)



![png](student_files/student_224_22.png)



![png](student_files/student_224_23.png)


#### OLS Test 23


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors', 'condition']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


![png](student_files/student_226_0.png)


#### OLS Test 24


```python
data['lot_living'] = data.sqft_lot15- data.sqft_living15
data.lot_living.loc[data.lot_living <= 0] = 0


```


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'bin_floors', 'condition']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['residual'] = results.resid
data['pred'] = results.predict
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```


![png](student_files/student_229_0.png)


#### OLS Test 25
Using bin_bathrooms


```python
data.bin_bathrooms = data.bin_bathrooms.astype(str)
data['min_max_zone_mean'] = min_max(dataseries=data.zone_mean)

data.bin_bathrooms.loc[data.bin_bathrooms == '(3.75, 8.0]'] = '0'
data.bin_bathrooms.loc[data.bin_bathrooms == '(0.0, 1.75]'] = '1'
data.bin_bathrooms.loc[data.bin_bathrooms == '(1.75, 3.75]'] = '2'
data.bin_bathrooms = data.bin_bathrooms.astype(int)
```


```python
columns= ['log_sqft_living', 'min_max_grade', 'min_max_log_zone_mean', 'waterfront', 'high_grade', 'condition', 'bathrooms']

ols, results = make_ols_model1(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['pred'] = results.predict()
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  

```


![png](student_files/student_232_0.png)


#### OLS Test 26


```python
columns= ['log_sqft_living','min_max_grade', 'min_max_zone_mean', 'high_grade', 'condition', 'bedrooms', 'bathrooms']

ols, results = make_ols_model(df=data, target='price', columns_to_use=columns, add_constant=False)
plt.figure(figsize= (13,8))
sns.distplot(results.resid, bins=100)
plt.title('Model Residual Dis');
data['pred'] = results.predict()
data['residual'] = results.resid
keep_data = data.loc[data.residual.abs() <= data.residual.std()*2] # removed anything above 3 std deviation in the residuals. 
removed_data = data.loc[data.residual.abs() > data.residual.std()*2]  
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:                  price   R-squared:                       0.913
    Model:                            OLS   Adj. R-squared:                  0.913
    Method:                 Least Squares   F-statistic:                 3.233e+04
    Date:                Mon, 30 Sep 2019   Prob (F-statistic):               0.00
    Time:                        03:32:56   Log-Likelihood:            -2.9347e+05
    No. Observations:               21597   AIC:                         5.870e+05
    Df Residuals:                   21590   BIC:                         5.870e+05
    Df Model:                           7                                         
    Covariance Type:            nonrobust                                         
    =====================================================================================
                            coef    std err          t      P>|t|      [0.025      0.975]
    -------------------------------------------------------------------------------------
    log_sqft_living    9.244e+04   3386.791     27.295      0.000    8.58e+04    9.91e+04
    min_max_grade      7.018e+05   1.84e+04     38.224      0.000    6.66e+05    7.38e+05
    min_max_zone_mean  3.496e+06   2.75e+04    127.101      0.000    3.44e+06    3.55e+06
    high_grade        -9.856e+05   1.76e+04    -55.966      0.000   -1.02e+06   -9.51e+05
    condition          3.431e+04   2064.013     16.625      0.000    3.03e+04    3.84e+04
    bedrooms           2336.2600   1753.643      1.332      0.183   -1101.010    5773.530
    bathrooms          4.816e+04   2578.420     18.678      0.000    4.31e+04    5.32e+04
    ==============================================================================
    Omnibus:                    21950.180   Durbin-Watson:                   1.972
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):          4557987.930
    Skew:                           4.581   Prob(JB):                         0.00
    Kurtosis:                      73.578   Cond. No.                         202.
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    


![png](student_files/student_234_1.png)


#### OLS Test 27 Final

Below is the parameters of the linear regression.


```python
results.params
```




    log_sqft_living      9.244104e+04
    min_max_grade        7.017882e+05
    min_max_zone_mean    3.496142e+06
    high_grade          -9.855810e+05
    condition            3.431438e+04
    bedrooms             2.336260e+03
    bathrooms            4.815888e+04
    dtype: float64




```python
a= results.params
print('Sqft Living does not have a easily translatable coefficient because of log transformation.')
print('Grade-- $', int(a[1]/(data.grade.max()-data.grade.min())), 'per change in grade', )
print('Mean Price of the Zone Area-- $', round(a[2]/
                (data.zone_mean.max()-data.zone_mean.min()), 2), 'per change in avg price')
print('Non Standard Grade Penalty-- -$', int(abs(a[3])), 'if grade 3 or grades over 11 ')
print('Condition-- $', int(a[4]), 'per change in condition', )
print('Bedrooms-- $', int(a[5]), 'per change in bedrooms', )
print('Bathrooms-- $', int(a[6]), 'per change in bathrooms', )

```

    Sqft Living does not have a easily translatable coefficient because of log transformation.
    Grade-- $ 70178 per change in grade
    Mean Price of the Zone Area-- $ 0.71 per change in avg price
    Non Standard Grade Penalty-- -$ 985580 if grade 3 or grades over 11 
    Condition-- $ 34314 per change in condition
    Bedrooms-- $ 2336 per change in bedrooms
    Bathrooms-- $ 48158 per change in bathrooms
    

#### Cross Validation


```python
linreg = LinearRegression(fit_intercept= True)

X = keep_data[columns]
Y = keep_data.price


cvs = cross_val_score(linreg, X, Y, cv=20, scoring="r2", )

print('The cross validation R squared is ',round(cvs.mean(), 3),', and the standard deviation is ', round(cvs.std(), 3))


```

    The cross validation R squared is  0.828 , and the standard deviation is  0.017
    

### Interpret

#### High Density Sales Areas

Below is the areas with more that 75 sales. The scale is from 75 to the max zone average, 188.


```python
base_map = folium.Map(
    location=[data.lat.mean()-data.lat.std(), data.long.median()],
    
    zoom_start=9)

HeatMap(data= data[['lat', 'long', 'ones']].loc[(data.zone_count>= 75) 
    ].groupby(['lat', 'long']).
        sum().reset_index().values.tolist(), radius=8, max_zoom=13).add_to(base_map)
base_map

```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF8xMzk2MzZmZGE1ZTU0YjQ3OGMwN2JmYTg5ZWJjYWM5YiB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9sZWFmbGV0LmdpdGh1Yi5pby9MZWFmbGV0LmhlYXQvZGlzdC9sZWFmbGV0LWhlYXQuanMiPjwvc2NyaXB0Pgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzEzOTYzNmZkYTVlNTRiNDc4YzA3YmZhODllYmNhYzliIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF8xMzk2MzZmZGE1ZTU0YjQ3OGMwN2JmYTg5ZWJjYWM5YiA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF8xMzk2MzZmZGE1ZTU0YjQ3OGMwN2JmYTg5ZWJjYWM5YiIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDcuNDIxNTQxMjI2MjI0MywgLTEyMi4yMzEwMDAwMDAwMDAwMV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA5LAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfNzBjYmFmMWMzOTBhNDBkMTgzZDhkNzg0OGYyZGFhOTkgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIkRhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzOTYzNmZkYTVlNTRiNDc4YzA3YmZhODllYmNhYzliKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgaGVhdF9tYXBfNzNhODM0NTllOTQ0NGRiM2FiMWQzNGYwYzZjMTM5MDYgPSBMLmhlYXRMYXllcigKICAgICAgICAgICAgICAgIFtbNDcuMzQyNSwgLTEyMi4wMywgMS4wXSwgWzQ3LjM0MjksIC0xMjIuMDM2LCAxLjBdLCBbNDcuMzQzMSwgLTEyMi4wMywgMS4wXSwgWzQ3LjM0MzIsIC0xMjIuMDI1LCAxLjBdLCBbNDcuMzQzMywgLTEyMi4wMzcsIDEuMF0sIFs0Ny4zNDM4LCAtMTIyLjAzNywgMS4wXSwgWzQ3LjM0NDgsIC0xMjIuMDI0LCAxLjBdLCBbNDcuMzQ1MiwgLTEyMi4wMjIsIDEuMF0sIFs0Ny4zNDU1LCAtMTIyLjAyMywgMS4wXSwgWzQ3LjM0NzMsIC0xMjIuMDM3LCAxLjBdLCBbNDcuMzQ3MywgLTEyMi4wMzEsIDEuMF0sIFs0Ny4zNDczLCAtMTIyLjAzLCAyLjBdLCBbNDcuMzQ3NCwgLTEyMi4wMzcsIDEuMF0sIFs0Ny4zNDc0LCAtMTIyLjAyNSwgMS4wXSwgWzQ3LjM0NzcsIC0xMjIuMDI0LCAxLjBdLCBbNDcuMzQ3OSwgLTEyMi4wMjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM0OCwgLTEyMi4wMzUsIDEuMF0sIFs0Ny4zNDgyLCAtMTIyLjAzNywgMS4wXSwgWzQ3LjM0ODQsIC0xMjIuMDM3LCAxLjBdLCBbNDcuMzQ4NCwgLTEyMi4wMzYsIDEuMF0sIFs0Ny4zNDg0LCAtMTIyLjAzMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzQ4NywgLTEyMi4wMzYsIDEuMF0sIFs0Ny4zNDg3LCAtMTIyLjAyMSwgMS4wXSwgWzQ3LjM0ODgsIC0xMjIuMDMyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNDg5LCAtMTIyLjAyMiwgMS4wXSwgWzQ3LjM0OSwgLTEyMi4wMzYsIDEuMF0sIFs0Ny4zNDksIC0xMjIuMDMxLCAxLjBdLCBbNDcuMzQ5LCAtMTIyLjAyNSwgMS4wXSwgWzQ3LjM0OSwgLTEyMi4wMjQsIDEuMF0sIFs0Ny4zNDksIC0xMjIuMDIxLCAxLjBdLCBbNDcuMzQ5MSwgLTEyMi4wMjksIDEuMF0sIFs0Ny4zNDkxLCAtMTIyLjAyLCAxLjBdLCBbNDcuMzQ5MiwgLTEyMi4wMzM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM0OTIsIC0xMjIuMDMsIDEuMF0sIFs0Ny4zNDkyLCAtMTIyLjAyNSwgMS4wXSwgWzQ3LjM0OTMsIC0xMjIuMDMzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNDkzLCAtMTIyLjAyNSwgMS4wXSwgWzQ3LjM0OTMsIC0xMjIuMDIzLCAxLjBdLCBbNDcuMzQ5NCwgLTEyMi4wMjIsIDEuMF0sIFs0Ny4zNDk1LCAtMTIyLjAzNywgMS4wXSwgWzQ3LjM0OTYsIC0xMjIuMDIyLCAxLjBdLCBbNDcuMzQ5NywgLTEyMi4wMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM0OTcsIC0xMjIuMDI1LCAxLjBdLCBbNDcuMzQ5OCwgLTEyMi4wMzcsIDEuMF0sIFs0Ny4zNDk5LCAtMTIyLjAyNywgMS4wXSwgWzQ3LjM1LCAtMTIyLjAzNywgMS4wXSwgWzQ3LjM1MDEsIC0xMjIuMDIsIDEuMF0sIFs0Ny4zNTAyLCAtMTIyLjAyMiwgMS4wXSwgWzQ3LjM1MDIsIC0xMjIuMDIsIDEuMF0sIFs0Ny4zNTA0LCAtMTIyLjAzNSwgMi4wXSwgWzQ3LjM1MDQsIC0xMjIuMDI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTA1LCAtMTIyLjAzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzUwNSwgLTEyMi4wMjcsIDIuMF0sIFs0Ny4zNTMxLCAtMTIyLjAyMSwgMS4wXSwgWzQ3LjM1MzEsIC0xMjIuMDE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTMzLCAtMTIyLjAyNywgMS4wXSwgWzQ3LjM1MzMsIC0xMjIuMDIzLCAxLjBdLCBbNDcuMzUzMywgLTEyMi4wMTYsIDEuMF0sIFs0Ny4zNTMzLCAtMTIyLjAxNSwgMS4wXSwgWzQ3LjM1MzUsIC0xMjIuMDI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTM1LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzUzNywgLTEyMi4wMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM1MzgsIC0xMjIuMDI1LCAxLjBdLCBbNDcuMzUzOCwgLTEyMi4wMjEsIDEuMF0sIFs0Ny4zNTM5LCAtMTIyLjAyNSwgMi4wXSwgWzQ3LjM1MzksIC0xMjIuMDI0LCAxLjBdLCBbNDcuMzUzOSwgLTEyMi4wMjEsIDEuMF0sIFs0Ny4zNTQxLCAtMTIyLjAyNCwgMS4wXSwgWzQ3LjM1NDEsIC0xMjIuMDE1LCAxLjBdLCBbNDcuMzU0MiwgLTEyMi4wMjcsIDEuMF0sIFs0Ny4zNTQyLCAtMTIyLjAxNCwgMS4wXSwgWzQ3LjM1NDMsIC0xMjIuMDIyLCAxLjBdLCBbNDcuMzU0NiwgLTEyMi4wMTUsIDEuMF0sIFs0Ny4zNTUsIC0xMjIuMDU5LCAxLjBdLCBbNDcuMzU1LCAtMTIyLjA1NywgMS4wXSwgWzQ3LjM1NTEsIC0xMjIuMDYxLCAxLjBdLCBbNDcuMzU1MSwgLTEyMi4wNTQsIDEuMF0sIFs0Ny4zNTUyLCAtMTIyLjAyNjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuMzU1MiwgLTEyMi4wMjMsIDEuMF0sIFs0Ny4zNTUyLCAtMTIyLjAxNSwgMS4wXSwgWzQ3LjM1NTIsIC0xMjIuMDE0LCAxLjBdLCBbNDcuMzU1NSwgLTEyMi4wNjEsIDEuMF0sIFs0Ny4zNTU1LCAtMTIyLjA1NCwgMS4wXSwgWzQ3LjM1NTYsIC0xMjIuMDI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTU3LCAtMTIyLjA1OSwgMS4wXSwgWzQ3LjM1NTcsIC0xMjIuMDI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTU3LCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjM1NTksIC0xMjIuMDIzLCAxLjBdLCBbNDcuMzU1OSwgLTEyMi4wMjIsIDEuMF0sIFs0Ny4zNTU5LCAtMTIyLjAxNCwgMS4wXSwgWzQ3LjM1NiwgLTEyMi4wNTcsIDIuMF0sIFs0Ny4zNTYsIC0xMjIuMDE0LCAxLjBdLCBbNDcuMzU2MSwgLTEyMi4wNTYsIDEuMF0sIFs0Ny4zNTYxLCAtMTIyLjAxNSwgMS4wXSwgWzQ3LjM1NjIsIC0xMjIuMDIyLCAxLjBdLCBbNDcuMzU2MywgLTEyMi4wMzksIDEuMF0sIFs0Ny4zNTY0LCAtMTIyLjA0LCAxLjBdLCBbNDcuMzU2NCwgLTEyMi4wMTYsIDEuMF0sIFs0Ny4zNTY1LCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjM1NjUsIC0xMjIuMDE1LCAxLjBdLCBbNDcuMzU2NiwgLTEyMi4wMzksIDEuMF0sIFs0Ny4zNTY4LCAtMTIyLjA1NiwgMS4wXSwgWzQ3LjM1NjgsIC0xMjIuMDU1LCAxLjBdLCBbNDcuMzU2OCwgLTEyMi4wMzgsIDEuMF0sIFs0Ny4zNTcsIC0xMjIuMDQsIDEuMF0sIFs0Ny4zNTcyLCAtMTIyLjAxNSwgMS4wXSwgWzQ3LjM1NzMsIC0xMjIuMDE2LCAxLjBdLCBbNDcuMzU3NSwgLTEyMi4wMTYsIDEuMF0sIFs0Ny4zNTc2LCAtMTIyLjA1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzU3NiwgLTEyMi4wMzksIDIuMF0sIFs0Ny4zNTc2LCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjM1NzcsIC0xMjIuMDM4LCAxLjBdLCBbNDcuMzU4MywgLTEyMi4wNTYsIDEuMF0sIFs0Ny4zNTg2LCAtMTIyLjA1NiwgMS4wXSwgWzQ3LjM1ODYsIC0xMjIuMDM2LCAxLjBdLCBbNDcuMzU4OCwgLTEyMi4wNDQsIDEuMF0sIFs0Ny4zNTg4LCAtMTIyLjAzOCwgMS4wXSwgWzQ3LjM1ODk5OTk5OTk5OTk5NSwgLTEyMi4wNSwgMS4wXSwgWzQ3LjM1ODk5OTk5OTk5OTk5NSwgLTEyMi4wNDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM1OTEsIC0xMjIuMDM4LCAxLjBdLCBbNDcuMzU5MSwgLTEyMi4wMzYsIDEuMF0sIFs0Ny4zNTkyLCAtMTIyLjA0NiwgMS4wXSwgWzQ3LjM1OTMsIC0xMjIuMDUxLCAxLjBdLCBbNDcuMzU5MywgLTEyMi4wMzcsIDEuMF0sIFs0Ny4zNTk0LCAtMTIyLjA1NiwgMS4wXSwgWzQ3LjM1OTQsIC0xMjIuMDU0LCAxLjBdLCBbNDcuMzU5NCwgLTEyMi4wNCwgMS4wXSwgWzQ3LjM1OTQsIC0xMjIuMDM2LCAxLjBdLCBbNDcuMzU5NSwgLTEyMi4wNDUsIDEuMF0sIFs0Ny4zNTk1LCAtMTIyLjA0MiwgMS4wXSwgWzQ3LjM1OTUsIC0xMjIuMDM4LCAxLjBdLCBbNDcuMzU5NiwgLTEyMi4wNCwgMS4wXSwgWzQ3LjM1OTYsIC0xMjIuMDM2LCAxLjBdLCBbNDcuMzU5NywgLTEyMi4wNTEsIDEuMF0sIFs0Ny4zNTk3LCAtMTIyLjA0NSwgMS4wXSwgWzQ3LjM1OTcsIC0xMjIuMDQsIDEuMF0sIFs0Ny4zNiwgLTEyMi4wMzksIDEuMF0sIFs0Ny4zNjAxLCAtMTIyLjA0NiwgMS4wXSwgWzQ3LjM2MDEsIC0xMjIuMDM1LCAxLjBdLCBbNDcuMzYwMSwgLTEyMi4wMzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM2MDIsIC0xMjIuMDUyLCAxLjBdLCBbNDcuMzYwMywgLTEyMi4wMzcsIDEuMF0sIFs0Ny4zNjA1LCAtMTIyLjA0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzYwNSwgLTEyMi4wNCwgMS4wXSwgWzQ3LjM2MDYsIC0xMjIuMDM5LCAxLjBdLCBbNDcuMzYwNywgLTEyMi4wMzgsIDIuMF0sIFs0Ny4zNjA3LCAtMTIyLjAzMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzYwOCwgLTEyMi4wNTYsIDEuMF0sIFs0Ny4zNjA4LCAtMTIyLjA0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzYwOCwgLTEyMi4wMzYsIDIuMF0sIFs0Ny4zNjA5LCAtMTIyLjA1MywgMS4wXSwgWzQ3LjM2MDksIC0xMjIuMDUsIDEuMF0sIFs0Ny4zNjA5LCAtMTIyLjA0Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzYwOSwgLTEyMi4wMjksIDEuMF0sIFs0Ny4zNjA5LCAtMTIyLjAyNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzYxMSwgLTEyMi4wNDcwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjM2MTMsIC0xMjIuMDUzLCAxLjBdLCBbNDcuMzYxMywgLTEyMi4wNDEsIDEuMF0sIFs0Ny4zNjEzLCAtMTIyLjAzOCwgMS4wXSwgWzQ3LjM2MTQsIC0xMjIuMDQ1LCAxLjBdLCBbNDcuMzYxNCwgLTEyMi4wMzUsIDEuMF0sIFs0Ny4zNjE1LCAtMTIyLjA1NywgMS4wXSwgWzQ3LjM2MTcsIC0xMjIuMDYxLCAxLjBdLCBbNDcuMzYxNywgLTEyMi4wNTQsIDEuMF0sIFs0Ny4zNjE3LCAtMTIyLjA1MiwgMS4wXSwgWzQ3LjM2MTcsIC0xMjIuMDUsIDEuMF0sIFs0Ny4zNjE3LCAtMTIyLjA0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzYxNywgLTEyMi4wNDYsIDEuMF0sIFs0Ny4zNjE3LCAtMTIyLjAzLCAxLjBdLCBbNDcuMzYxOCwgLTEyMi4wMzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM2MTk5OTk5OTk5OTk5NSwgLTEyMi4wMjksIDEuMF0sIFs0Ny4zNjIxLCAtMTIyLjAzMSwgMS4wXSwgWzQ3LjM2MjIsIC0xMjIuMDU5LCAxLjBdLCBbNDcuMzYyMiwgLTEyMi4wNDEsIDEuMF0sIFs0Ny4zNjI0LCAtMTIyLjA0NSwgMS4wXSwgWzQ3LjM2MjQsIC0xMjIuMDMxLCAxLjBdLCBbNDcuMzYyNSwgLTEyMi4wMjUsIDEuMF0sIFs0Ny4zNjI2LCAtMTIyLjA0LCAxLjBdLCBbNDcuMzYyNywgLTEyMi4wNCwgMS4wXSwgWzQ3LjM2MjcsIC0xMjIuMDM3LCAxLjBdLCBbNDcuMzYyOCwgLTEyMi4wNSwgMS4wXSwgWzQ3LjM2MjgsIC0xMjIuMDQ1LCAxLjBdLCBbNDcuMzYyOSwgLTEyMi4wMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM2MywgLTEyMi4wMzUsIDEuMF0sIFs0Ny4zNjMsIC0xMjIuMDMzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjMxLCAtMTIyLjA1LCAxLjBdLCBbNDcuMzYzMSwgLTEyMi4wMjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM2MzIsIC0xMjIuMDYxLCAxLjBdLCBbNDcuMzYzNCwgLTEyMi4wNTIsIDEuMF0sIFs0Ny4zNjM0LCAtMTIyLjA0NiwgMS4wXSwgWzQ3LjM2MzQsIC0xMjIuMDMxLCAxLjBdLCBbNDcuMzYzNCwgLTEyMi4wMjcsIDEuMF0sIFs0Ny4zNjM1LCAtMTIyLjA1NCwgMS4wXSwgWzQ3LjM2MzUsIC0xMjIuMDUyLCAxLjBdLCBbNDcuMzYzNSwgLTEyMi4wNDUsIDEuMF0sIFs0Ny4zNjM3LCAtMTIyLjA1NywgMS4wXSwgWzQ3LjM2MzgsIC0xMjIuMDUzLCAyLjBdLCBbNDcuMzYzOSwgLTEyMi4wNTUsIDEuMF0sIFs0Ny4zNjM5LCAtMTIyLjA0Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzY0MiwgLTEyMi4wNTIsIDEuMF0sIFs0Ny4zNjQzLCAtMTIyLjA0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzY0MywgLTEyMi4wNDEsIDEuMF0sIFs0Ny4zNjQ0LCAtMTIyLjA0LCAxLjBdLCBbNDcuMzY0NSwgLTEyMi4wMzksIDEuMF0sIFs0Ny4zNjQ2LCAtMTIyLjAzNywgMS4wXSwgWzQ3LjM2NDYsIC0xMjIuMDM2LCAxLjBdLCBbNDcuMzY0NywgLTEyMi4wNSwgMS4wXSwgWzQ3LjM2NDgsIC0xMjIuMDQ2LCAxLjBdLCBbNDcuMzY1LCAtMTIyLjAyNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzY1MSwgLTEyMi4wNTEsIDEuMF0sIFs0Ny4zNjUxLCAtMTIyLjA0NCwgMS4wXSwgWzQ3LjM2NTQsIC0xMjIuMDIxLCAxLjBdLCBbNDcuMzY1NSwgLTEyMi4wNDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM2NTUsIC0xMjIuMDI3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjU1LCAtMTIyLjAyNywgMS4wXSwgWzQ3LjM2NTUsIC0xMjIuMDE4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjU1LCAtMTIyLjAxNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzY1NSwgLTEyMi4wMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM2NTcsIC0xMjIuMDQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNjU3LCAtMTIyLjAzMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzY1NywgLTEyMi4wMjEsIDEuMF0sIFs0Ny4zNjU4LCAtMTIyLjA0LCAxLjBdLCBbNDcuMzY1OCwgLTEyMi4wMzcsIDEuMF0sIFs0Ny4zNjU4LCAtMTIyLjAyMSwgMS4wXSwgWzQ3LjM2NTgsIC0xMjIuMDE4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjU4LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzY1OSwgLTEyMi4wMjksIDEuMF0sIFs0Ny4zNjU5LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzY2MDAwMDAwMDAwMDEsIC0xMjIuMDE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNjYxLCAtMTIyLjAyMSwgMS4wXSwgWzQ3LjM2NjEsIC0xMjIuMDIsIDEuMF0sIFs0Ny4zNjYyLCAtMTIyLjAxODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzY2NSwgLTEyMi4wMjIsIDEuMF0sIFs0Ny4zNjY1LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzY2NiwgLTEyMi4wMywgMS4wXSwgWzQ3LjM2NjYsIC0xMjIuMDE4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjY3LCAtMTIyLjA0MSwgMS4wXSwgWzQ3LjM2NjgsIC0xMjIuMDMxLCAxLjBdLCBbNDcuMzY2OSwgLTEyMi4wMiwgMS4wXSwgWzQ3LjM2NywgLTEyMi4wMzEsIDEuMF0sIFs0Ny4zNjcsIC0xMjIuMDE4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjcxLCAtMTIyLjAyOSwgMS4wXSwgWzQ3LjM2NzIsIC0xMjIuMDE4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40Nzk4LCAtMTIyLjEzNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDgsIC0xMjIuMTM3LCAxLjBdLCBbNDcuNDgwNCwgLTEyMi4xNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ4MDQsIC0xMjIuMTUxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40ODA2LCAtMTIyLjE1MiwgMS4wXSwgWzQ3LjQ4MDcsIC0xMjIuMTU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40ODA5LCAtMTIyLjE1Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDgxMSwgLTEyMi4xNDksIDEuMF0sIFs0Ny40ODE1LCAtMTIyLjE1Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDgxNSwgLTEyMi4xNTIsIDIuMF0sIFs0Ny40ODE5LCAtMTIyLjE0LCAxLjBdLCBbNDcuNDgyMSwgLTEyMi4xNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ4MjQsIC0xMjIuMTUyLCAxLjBdLCBbNDcuNDgyNiwgLTEyMi4xNDksIDEuMF0sIFs0Ny40ODI3LCAtMTIyLjEzNSwgMS4wXSwgWzQ3LjQ4MjgsIC0xMjIuMTM5LCAxLjBdLCBbNDcuNDgyOCwgLTEyMi4xMzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ4MjksIC0xMjIuMTU2LCAxLjBdLCBbNDcuNDgzMSwgLTEyMi4xMzUsIDEuMF0sIFs0Ny40ODMyLCAtMTIyLjE0NSwgMS4wXSwgWzQ3LjQ4MzMsIC0xMjIuMTU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40ODMzLCAtMTIyLjEzOSwgMS4wXSwgWzQ3LjQ4MzUsIC0xMjIuMTM2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40ODM2LCAtMTIyLjEzNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDgzNywgLTEyMi4xNDgsIDEuMF0sIFs0Ny40ODM3LCAtMTIyLjEzNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDg1MywgLTEyMi4xNTQsIDEuMF0sIFs0Ny40ODU1LCAtMTIyLjE0OSwgMS4wXSwgWzQ3LjQ4NTYsIC0xMjIuMTU2LCAxLjBdLCBbNDcuNDg1NiwgLTEyMi4xNTQsIDEuMF0sIFs0Ny40ODU3LCAtMTIyLjE1Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDg2MywgLTEyMi4xNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ4NjMsIC0xMjIuMTQsIDEuMF0sIFs0Ny40ODY1LCAtMTIyLjE0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDg2NSwgLTEyMi4xNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ4NjYsIC0xMjIuMTU1LCAxLjBdLCBbNDcuNDg2NiwgLTEyMi4xNDcsIDEuMF0sIFs0Ny40ODY3LCAtMTIyLjE1MiwgMS4wXSwgWzQ3LjQ4NjgsIC0xMjIuMTQxLCAxLjBdLCBbNDcuNDg3MiwgLTEyMi4xNDM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ4NzMsIC0xMjIuMTQzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODc0LCAtMTIyLjE1MiwgMS4wXSwgWzQ3LjQ4NzYsIC0xMjIuMTUyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODc2LCAtMTIyLjE0NiwgMS4wXSwgWzQ3LjQ4NzcsIC0xMjIuMTQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODc3LCAtMTIyLjEzOSwgMS4wXSwgWzQ3LjQ4NzksIC0xMjIuMTUyLCAxLjBdLCBbNDcuNDg4MSwgLTEyMi4xNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ4ODEsIC0xMjIuMTQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40ODg3LCAtMTIyLjE1LCAyLjBdLCBbNDcuNDg4NywgLTEyMi4xNDUsIDEuMF0sIFs0Ny40ODg4LCAtMTIyLjE0LCAxLjBdLCBbNDcuNDg5LCAtMTIyLjE0LCAxLjBdLCBbNDcuNDg5MSwgLTEyMi4xNDksIDEuMF0sIFs0Ny40ODkxLCAtMTIyLjE0MSwgMS4wXSwgWzQ3LjQ4OTMsIC0xMjIuMTQ3LCAxLjBdLCBbNDcuNDg5MywgLTEyMi4xNDYsIDEuMF0sIFs0Ny40ODkzLCAtMTIyLjEzNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDg5NCwgLTEyMi4xMzUsIDEuMF0sIFs0Ny40ODk2LCAtMTIyLjE0LCAxLjBdLCBbNDcuNDg5NywgLTEyMi4xNDcsIDEuMF0sIFs0Ny40ODk5LCAtMTIyLjEzNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDksIC0xMjIuMTUsIDEuMF0sIFs0Ny40OTAyLCAtMTIyLjE1NCwgMS4wXSwgWzQ3LjQ5MDUsIC0xMjIuMTQsIDEuMF0sIFs0Ny40OTA3LCAtMTIyLjE1MiwgMS4wXSwgWzQ3LjQ5MDgsIC0xMjIuMTU0LCAxLjBdLCBbNDcuNDkwOSwgLTEyMi4xNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ5MDksIC0xMjIuMTU0LCAxLjBdLCBbNDcuNDkwOSwgLTEyMi4xNDM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ5MTAwMDAwMDAwMDAxLCAtMTIyLjE1Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDkxMDAwMDAwMDAwMDEsIC0xMjIuMTUsIDEuMF0sIFs0Ny40OTExLCAtMTIyLjE1NCwgMi4wXSwgWzQ3LjQ5MTEsIC0xMjIuMTM3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTEyLCAtMTIyLjE0NSwgMS4wXSwgWzQ3LjQ5MTMsIC0xMjIuMTU0LCAxLjBdLCBbNDcuNDkxMywgLTEyMi4xNTIsIDEuMF0sIFs0Ny40OTEzLCAtMTIyLjE0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDkxNCwgLTEyMi4xNTUsIDEuMF0sIFs0Ny40OTE1LCAtMTIyLjE1NSwgMS4wXSwgWzQ3LjQ5MTUsIC0xMjIuMTUxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40OTE1LCAtMTIyLjE0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDkxNiwgLTEyMi4xNTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ5MTYsIC0xMjIuMTQzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTE3LCAtMTIyLjE3LCAxLjBdLCBbNDcuNDkxNywgLTEyMi4xNjcsIDEuMF0sIFs0Ny40OTE4LCAtMTIyLjE3OCwgMS4wXSwgWzQ3LjQ5MTksIC0xMjIuMTgxLCAxLjBdLCBbNDcuNDkyLCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDkyLCAtMTIyLjE3MSwgMS4wXSwgWzQ3LjQ5MiwgLTEyMi4xNjYsIDEuMF0sIFs0Ny40OTIyLCAtMTIyLjE3OCwgMS4wXSwgWzQ3LjQ5MjMsIC0xMjIuMTc4LCAxLjBdLCBbNDcuNDkyMywgLTEyMi4xNjUsIDEuMF0sIFs0Ny40OTI0LCAtMTIyLjE3NSwgMS4wXSwgWzQ3LjQ5MjQsIC0xMjIuMTczOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTI3LCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDkyNywgLTEyMi4xNjksIDEuMF0sIFs0Ny40OTM0LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDkzNCwgLTEyMi4xNjYsIDEuMF0sIFs0Ny40OTM2LCAtMTIyLjE2NSwgMS4wXSwgWzQ3LjQ5MzgsIC0xMjIuMTcxLCAxLjBdLCBbNDcuNDkzOCwgLTEyMi4xNjEsIDEuMF0sIFs0Ny40OTM5LCAtMTIyLjE2NywgMS4wXSwgWzQ3LjQ5NCwgLTEyMi4xNjUsIDEuMF0sIFs0Ny40OTQyLCAtMTIyLjE2NiwgMS4wXSwgWzQ3LjQ5NDMsIC0xMjIuMTc4LCAxLjBdLCBbNDcuNDk0NCwgLTEyMi4xNjUsIDEuMF0sIFs0Ny40OTQ0LCAtMTIyLjE2MiwgMS4wXSwgWzQ3LjQ5NDUsIC0xMjIuMTY2LCAxLjBdLCBbNDcuNDk0NywgLTEyMi4xNzM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ5NDcsIC0xMjIuMTcxLCAxLjBdLCBbNDcuNDk1MiwgLTEyMi4xNzgsIDEuMF0sIFs0Ny40OTUyLCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDk1MywgLTEyMi4xNjksIDEuMF0sIFs0Ny40OTU0LCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDk1NCwgLTEyMi4xNjQsIDEuMF0sIFs0Ny40OTU0LCAtMTIyLjE2MywgMS4wXSwgWzQ3LjQ5NTUsIC0xMjIuMTYsIDEuMF0sIFs0Ny40OTU2LCAtMTIyLjE4MSwgMS4wXSwgWzQ3LjQ5NTYsIC0xMjIuMTczOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTU2LCAtMTIyLjE2MywgMS4wXSwgWzQ3LjQ5NTYsIC0xMjIuMTYyLCAxLjBdLCBbNDcuNDk1NiwgLTEyMi4xNjEsIDEuMF0sIFs0Ny40OTU3LCAtMTIyLjE2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDk1OSwgLTEyMi4xNzM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ5NTksIC0xMjIuMTcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40OTYsIC0xMjIuMTY5LCAxLjBdLCBbNDcuNDk2NCwgLTEyMi4xNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ5NjUsIC0xMjIuMTc2LCAxLjBdLCBbNDcuNDk2OSwgLTEyMi4xNzgsIDEuMF0sIFs0Ny40OTcsIC0xMjIuMTgsIDEuMF0sIFs0Ny40OTcsIC0xMjIuMTcxLCAxLjBdLCBbNDcuNDk3MSwgLTEyMi4xNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ5NzIsIC0xMjIuMTc2LCAxLjBdLCBbNDcuNDk3MywgLTEyMi4xNzcsIDEuMF0sIFs0Ny40OTc0LCAtMTIyLjE2OSwgMS4wXSwgWzQ3LjQ5NzcsIC0xMjIuMTc1LCAxLjBdLCBbNDcuNDk3NywgLTEyMi4xNjMsIDEuMF0sIFs0Ny40OTgwMDAwMDAwMDAwMSwgLTEyMi4xNywgMS4wXSwgWzQ3LjQ5ODEsIC0xMjIuMTYyLCAxLjBdLCBbNDcuNDk4MywgLTEyMi4xNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ5ODMsIC0xMjIuMTY3LCAxLjBdLCBbNDcuNDk4NCwgLTEyMi4xNzYsIDEuMF0sIFs0Ny40OTg3LCAtMTIyLjE2NiwgMS4wXSwgWzQ3LjQ5ODgsIC0xMjIuMTY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTg5LCAtMTIyLjE3MSwgMS4wXSwgWzQ3LjQ5OTEsIC0xMjIuMTc2LCAxLjBdLCBbNDcuNDk5MSwgLTEyMi4xNjQsIDEuMF0sIFs0Ny40OTkzLCAtMTIyLjE3NiwgMS4wXSwgWzQ3LjQ5OTYsIC0xMjIuMTYzLCAxLjBdLCBbNDcuNDk5OSwgLTEyMi4xNjUsIDEuMF0sIFs0Ny41LCAtMTIyLjE2OSwgMS4wXSwgWzQ3LjUwMDQsIC0xMjIuMTYyLCAyLjBdLCBbNDcuNTAwOCwgLTEyMi4xNjIsIDIuMF0sIFs0Ny41MDA5LCAtMTIyLjE2NCwgMS4wXSwgWzQ3LjUwMTQsIC0xMjIuMTcyMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny41MDE1LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTAxNSwgLTEyMi4xNjYsIDEuMF0sIFs0Ny41MDE2LCAtMTIyLjE2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTAxNywgLTEyMi4xNzM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUwMTcsIC0xMjIuMTY1LCAxLjBdLCBbNDcuNTAyNCwgLTEyMi4xNjksIDEuMF0sIFs0Ny41MDI2LCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTAyOCwgLTEyMi4xNywgMS4wXSwgWzQ3LjUxNjUsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTE2NSwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUxNjYsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNTE2OCwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUxNjgsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MTY4LCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjUxNjksIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MTcyLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjUxNzMsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTE3NSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUxNzUsIC0xMjIuMzYsIDEuMF0sIFs0Ny41MTc2LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUxNzcsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTE3NywgLTEyMi4zNTMsIDEuMF0sIFs0Ny41MTc4LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjUxNzksIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTE4MSwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUxODMsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTE4NCwgLTEyMi4zNjYsIDEuMF0sIFs0Ny41MTg1LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTE4NSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41MTg1LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjUxODUsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNTE4NiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41MTg2LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjUxODYsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MTg3LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTE4NywgLTEyMi4zNjMsIDEuMF0sIFs0Ny41MTg4LCAtMTIyLjM3NiwgMi4wXSwgWzQ3LjUxODgsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MTg4LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTE4OCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny41MTksIC0xMjIuMzUyLCAxLjBdLCBbNDcuNTE5NCwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41MTk0LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjUxOTUsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTE5NSwgLTEyMi4zNzQsIDEuMF0sIFs0Ny41MTk2LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUxOTYsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MTk4LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjUxOTgsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MTk4LCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjUxOTksIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MiwgLTEyMi4zNzQsIDEuMF0sIFs0Ny41MiwgLTEyMi4zNjUsIDEuMF0sIFs0Ny41MiwgLTEyMi4zNTQsIDEuMF0sIFs0Ny41MjAxLCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTIwMSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny41MjAyLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTIwMiwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMDIsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MjAyLCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTIwMiwgLTEyMi4zNTEsIDEuMF0sIFs0Ny41MjAzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTIwNiwgLTEyMi4zNjMsIDEuMF0sIFs0Ny41MjA3LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTIwOCwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMDgsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjA4LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTIwOSwgLTEyMi4zNzQsIDEuMF0sIFs0Ny41MjEyLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTIxMiwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMTIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny41MjEzLCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTIxNSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41MjE2LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjUyMTgsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MjE4LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTIxOSwgLTEyMi4zOCwgMS4wXSwgWzQ3LjUyMTksIC0xMjIuMzYxLCAxLjBdLCBbNDcuNTIyLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTIyLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjUyMiwgLTEyMi4zNTQsIDEuMF0sIFs0Ny41MjIxLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTIyMiwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMjIsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTIyMiwgLTEyMi4zNywgMS4wXSwgWzQ3LjUyMjIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjI0LCAtMTIyLjM5LCAxLjBdLCBbNDcuNTIyNCwgLTEyMi4zNiwgMS4wXSwgWzQ3LjUyMjQsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjI1LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTIyNSwgLTEyMi4zNTEsIDEuMF0sIFs0Ny41MjI2LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjUyMjYsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTIyNywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyMywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyMzIsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTIzMiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41MjMyLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTIzMiwgLTEyMi4zNTQsIDEuMF0sIFs0Ny41MjMzLCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTIzMywgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMzQsIC0xMjIuMzgsIDEuMF0sIFs0Ny41MjM1LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjUyMzcsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTIzNywgLTEyMi4zODMsIDEuMF0sIFs0Ny41MjM3LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUyMzcsIC0xMjIuMzUzLCAyLjBdLCBbNDcuNTIzOCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjUyMzgsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNTIzOSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMzksIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTIzOTk5OTk5OTk5OTk0LCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjUyNDIsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjQyLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI0MywgLTEyMi4zOSwgMS4wXSwgWzQ3LjUyNDMsIC0xMjIuMzYsIDEuMF0sIFs0Ny41MjQzLCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI0NCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNDUsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTI0NSwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNDUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTI0NywgLTEyMi4zOTEsIDEuMF0sIFs0Ny41MjQ4LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjUyNDksIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MjQ5LCAtMTIyLjM3LCAyLjBdLCBbNDcuNTI1LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI1LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjUyNSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyNSwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny41MjUsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjUxLCAtMTIyLjM3LCAxLjBdLCBbNDcuNTI1MiwgLTEyMi4zODIsIDEuMF0sIFs0Ny41MjUyLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI1MiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MjU0LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI1NSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41MjU1LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjUyNTYsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjU2LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjUyNTYsIC0xMjIuMzgsIDIuMF0sIFs0Ny41MjU2LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI1NiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41MjU2LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI1NywgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNTcsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTI1NywgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MjU3LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI1OCwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyNTgsIC0xMjIuMzc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjU5LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTI1OSwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyNiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MjYxLCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUyNjEsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNTI2MSwgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MjYyLCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI2MiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MjYzLCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI2NCwgLTEyMi4zODQsIDEuMF0sIFs0Ny41MjY0LCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjUyNjUsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MjY2LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI2NiwgLTEyMi4zODcsIDEuMF0sIFs0Ny41MjY2LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI2NywgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MjY3LCAtMTIyLjM2LCAxLjBdLCBbNDcuNTI2OCwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyNjksIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTI3LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjUyNywgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNywgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNzEsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjcxLCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjUyNzEsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTI3MSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41MjcxLCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjUyNzIsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNTI3NCwgLTEyMi4zODQsIDIuMF0sIFs0Ny41Mjc0LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI3NCwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNzUsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTI3NSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41Mjc1LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjUyNzUsIC0xMjIuMzgsIDEuMF0sIFs0Ny41Mjc2LCAtMTIyLjM1OSwgMi4wXSwgWzQ3LjUyNzcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNTI3NywgLTEyMi4zNTEsIDEuMF0sIFs0Ny41Mjc5LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjUyOCwgLTEyMi4zNTksIDEuMF0sIFs0Ny41MjgsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNTI4MSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyODEsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTI4MSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyODIsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTI4MiwgLTEyMi4zODQsIDEuMF0sIFs0Ny41MjgzLCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjUyODQsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTI4NCwgLTEyMi4zODcsIDEuMF0sIFs0Ny41Mjg1LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI4NSwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyODUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTI4NiwgLTEyMi4zODMsIDEuMF0sIFs0Ny41Mjg2LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUyODYsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNTI4NywgLTEyMi4zODUsIDEuMF0sIFs0Ny41Mjg3LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjUyODksIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTI5MSwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTEsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTI5MSwgLTEyMi4yNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyOTIsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTI5MiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTIsIC0xMjIuMjY4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MjkzLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI5MywgLTEyMi4zNjMsIDEuMF0sIFs0Ny41MjkzLCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjUyOTMsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTI5MywgLTEyMi4yNzQsIDEuMF0sIFs0Ny41MjkzLCAtMTIyLjI3MiwgMi4wXSwgWzQ3LjUyOTMsIC0xMjIuMjcxLCAxLjBdLCBbNDcuNTI5NCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTQsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNTI5NCwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTQsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTI5NCwgLTEyMi4yNzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTYsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mjk2LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjUyOTYsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTI5NywgLTEyMi4zODEsIDEuMF0sIFs0Ny41Mjk3LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjUyOTcsIC0xMjIuMjc3LCAxLjBdLCBbNDcuNTI5NywgLTEyMi4yNjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTgsIC0xMjIuMjc0LCAxLjBdLCBbNDcuNTI5OCwgLTEyMi4yNzMsIDEuMF0sIFs0Ny41Mjk4LCAtMTIyLjI3MSwgMS4wXSwgWzQ3LjUyOTgsIC0xMjIuMjY4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mjk4LCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTI5OCwgLTEyMS44ODcsIDEuMF0sIFs0Ny41Mjk4LCAtMTIxLjg4LCAxLjBdLCBbNDcuNTI5OCwgLTEyMS44Nzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyOTksIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTI5OSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTksIC0xMjIuMjY2LCAxLjBdLCBbNDcuNTI5OSwgLTEyMS44Nzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTMsIC0xMjIuMjY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzAxLCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjUzMDEsIC0xMjIuMjcxLCAxLjBdLCBbNDcuNTMwMiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzMDMsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzAzLCAtMTIyLjI3LCAxLjBdLCBbNDcuNTMwMywgLTEyMS44Nzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMDQsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTMwNCwgLTEyMi4zOCwgMS4wXSwgWzQ3LjUzMDQsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNTMwNCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzMDQsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNTMwNSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41MzA1LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjUzMDUsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzA1LCAtMTIyLjM1MywgMS4wXSwgWzQ3LjUzMDUsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTMwNSwgLTEyMi4yNzcsIDEuMF0sIFs0Ny41MzA2LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjUzMDcsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNTMwNywgLTEyMi4yNzcsIDEuMF0sIFs0Ny41MzA3LCAtMTIxLjg3NSwgMS4wXSwgWzQ3LjUzMDgsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTMwOCwgLTEyMi4zNjEsIDEuMF0sIFs0Ny41MzA5LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjUzMDksIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTMwOSwgLTEyMS44NzYsIDEuMF0sIFs0Ny41MzEwMDAwMDAwMDAwMSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjUzMTAwMDAwMDAwMDAxLCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMxMDAwMDAwMDAwMDEsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzEwMDAwMDAwMDAwMSwgLTEyMi4zNTEsIDEuMF0sIFs0Ny41MzExLCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjUzMTEsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTMxMSwgLTEyMi4zODEsIDEuMF0sIFs0Ny41MzExLCAtMTIyLjM4LCAxLjBdLCBbNDcuNTMxMSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMTEsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNTMxMiwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzMTIsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTMxMiwgLTEyMi4zNTQsIDEuMF0sIFs0Ny41MzEzLCAtMTIyLjM2LCAxLjBdLCBbNDcuNTMxMywgLTEyMi4zNTEsIDEuMF0sIFs0Ny41MzE0LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMxNCwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjUzMTQsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzE1LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTMxNiwgLTEyMi4yNjI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMTcsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTMxNywgLTEyMi4zODcsIDEuMF0sIFs0Ny41MzE3LCAtMTIyLjM3LCAxLjBdLCBbNDcuNTMxNywgLTEyMi4zNTEsIDEuMF0sIFs0Ny41MzE3LCAtMTIyLjI3NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMxOCwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMTgsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNTMxOSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41MzE5LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMxOSwgLTEyMi4zODIsIDEuMF0sIFs0Ny41MzIsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTMyLCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjUzMiwgLTEyMS44OCwgMS4wXSwgWzQ3LjUzMjEsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTMyMiwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzMjIsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTMyMiwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMjIsIC0xMjEuODgsIDEuMF0sIFs0Ny41MzIyLCAtMTIxLjg3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMyMywgLTEyMi4zODMsIDIuMF0sIFs0Ny41MzIzLCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjUzMjMsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNTMyNCwgLTEyMi4zODcsIDEuMF0sIFs0Ny41MzI0LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTMyNCwgLTEyMi4yNzUsIDEuMF0sIFs0Ny41MzI0LCAtMTIyLjI3MiwgMS4wXSwgWzQ3LjUzMjUsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzI2LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUzMjYsIC0xMjEuODc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzI3LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMyNywgLTEyMi4yNzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzMjcsIC0xMjEuODcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzI4LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMyOCwgLTEyMi4yNzUsIDEuMF0sIFs0Ny41MzI4LCAtMTIyLjI3NCwgMS4wXSwgWzQ3LjUzMjgsIC0xMjIuMjcyLCAxLjBdLCBbNDcuNTMyOCwgLTEyMS44ODEsIDEuMF0sIFs0Ny41MzI4LCAtMTIxLjg3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMyOCwgLTEyMS44NzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzMjksIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTMyOSwgLTEyMi4yNzEsIDEuMF0sIFs0Ny41MzI5LCAtMTIxLjg5LCAxLjBdLCBbNDcuNTMyOSwgLTEyMS44Nzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMywgLTEyMi4zNTQsIDEuMF0sIFs0Ny41MzMxLCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjUzMzEsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTMzMSwgLTEyMi4zNTMsIDEuMF0sIFs0Ny41MzMxLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjUzMzEsIC0xMjIuMjc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzMyLCAtMTIyLjI2NSwgMS4wXSwgWzQ3LjUzMzMsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzMzLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMzMywgLTEyMi4zODEsIDEuMF0sIFs0Ny41MzMzLCAtMTIxLjg3LCAxLjBdLCBbNDcuNTMzNCwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzMzQsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTMzNSwgLTEyMi4yNzIsIDEuMF0sIFs0Ny41MzM1LCAtMTIxLjg2OSwgMS4wXSwgWzQ3LjUzMzYsIC0xMjIuMjc3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41MzM3LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMzOCwgLTEyMi4zODQsIDEuMF0sIFs0Ny41MzM4LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjUzMzgsIC0xMjIuMzYsIDEuMF0sIFs0Ny41MzM5LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMzOSwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNCwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41MzQsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzQsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTM0LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM0MSwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNDIsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzQyLCAtMTIyLjM2LCAxLjBdLCBbNDcuNTM0MiwgLTEyMS44ODMsIDEuMF0sIFs0Ny41MzQyLCAtMTIxLjg3NiwgMS4wXSwgWzQ3LjUzNDMsIC0xMjIuMzcsIDEuMF0sIFs0Ny41MzQzLCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjUzNDQsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTM0NCwgLTEyMS44NzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNDQsIC0xMjEuODc0LCAxLjBdLCBbNDcuNTM0NSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41MzQ1LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM0NSwgLTEyMi4yNzQsIDEuMF0sIFs0Ny41MzQ2LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjUzNDYsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTM0NiwgLTEyMi4zNTMsIDEuMF0sIFs0Ny41MzQ2LCAtMTIxLjg4MSwgMS4wXSwgWzQ3LjUzNDYsIC0xMjEuODc1LCAyLjBdLCBbNDcuNTM0NywgLTEyMi4zOCwgMS4wXSwgWzQ3LjUzNDcsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTM0OCwgLTEyMi4yNjg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNDgsIC0xMjIuMjY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzQ4LCAtMTIxLjg3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM0OSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41MzUsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNTM1MSwgLTEyMi4zODEsIDEuMF0sIFs0Ny41MzUxLCAtMTIyLjM1MywgMS4wXSwgWzQ3LjUzNTIsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzUyLCAtMTIyLjM4LCAxLjBdLCBbNDcuNTM1MiwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNTIsIC0xMjIuMzcsIDEuMF0sIFs0Ny41MzUyLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjUzNTIsIC0xMjEuODgzLCAxLjBdLCBbNDcuNTM1MiwgLTEyMS44OCwgMS4wXSwgWzQ3LjUzNTMsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNTM1MywgLTEyMS44Nzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNTQsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzU0LCAtMTIyLjI3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM1NCwgLTEyMi4yNzMsIDEuMF0sIFs0Ny41MzU0LCAtMTIxLjg2OSwgMS4wXSwgWzQ3LjUzNTUsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzU1LCAtMTIyLjM5LCAxLjBdLCBbNDcuNTM1NSwgLTEyMS44OCwgMS4wXSwgWzQ3LjUzNTUsIC0xMjEuODc0LCAxLjBdLCBbNDcuNTM1NiwgLTEyMi4zODQsIDEuMF0sIFs0Ny41MzU2LCAtMTIxLjg4MSwgMS4wXSwgWzQ3LjUzNTYsIC0xMjEuODc1LCAxLjBdLCBbNDcuNTM1NywgLTEyMi4zNjgsIDEuMF0sIFs0Ny41MzU3LCAtMTIyLjM2NSwgMi4wXSwgWzQ3LjUzNTcsIC0xMjIuMjczLCAxLjBdLCBbNDcuNTM1OSwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNTksIC0xMjIuMzY3LCAxLjBdLCBbNDcuNTM1OSwgLTEyMi4yNzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNTksIC0xMjEuODc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzYsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzYsIC0xMjEuODc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzYxLCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjUzNjEsIC0xMjEuODgsIDEuMF0sIFs0Ny41MzYxLCAtMTIxLjg3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM2MiwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNjIsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTM2MiwgLTEyMi4yNzEsIDEuMF0sIFs0Ny41MzYyLCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM2MiwgLTEyMi4yNjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNjIsIC0xMjEuODc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzYyLCAtMTIxLjg3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM2MiwgLTEyMS44NzQsIDEuMF0sIFs0Ny41MzYzLCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjUzNjMsIC0xMjIuMzc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzYzLCAtMTIxLjg3LCAxLjBdLCBbNDcuNTM2NCwgLTEyMi4zNjUsIDEuMF0sIFs0Ny41MzY1LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM2NSwgLTEyMi4zNywgMS4wXSwgWzQ3LjUzNjUsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNTM2NiwgLTEyMi4zNTksIDEuMF0sIFs0Ny41MzY2LCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM2NywgLTEyMS44OCwgMS4wXSwgWzQ3LjUzNjgsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNTM2OCwgLTEyMi4yNjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNjgsIC0xMjIuMjY1LCAxLjBdLCBbNDcuNTM2OSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNjksIC0xMjEuODg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzY5LCAtMTIxLjg4NywgMS4wXSwgWzQ3LjUzNjksIC0xMjEuODc2LCAxLjBdLCBbNDcuNTM3LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjUzNywgLTEyMi4zNzUsIDEuMF0sIFs0Ny41MzcsIC0xMjEuODcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzcxLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM3MSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNzEsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNTM3MSwgLTEyMS44NywgMS4wXSwgWzQ3LjUzNzIsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTM3MiwgLTEyMi4yNjg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNzIsIC0xMjEuODc2LCAxLjBdLCBbNDcuNTM3MywgLTEyMi4zOTUsIDEuMF0sIFs0Ny41MzczLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTM3MywgLTEyMS44NzQsIDEuMF0sIFs0Ny41MzczLCAtMTIxLjg3LCAxLjBdLCBbNDcuNTM3NCwgLTEyMi4zOSwgMS4wXSwgWzQ3LjUzNzQsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzc0LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM3NCwgLTEyMi4yNywgMS4wXSwgWzQ3LjUzNzQsIC0xMjEuODc1LCAxLjBdLCBbNDcuNTM3NSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41Mzc1LCAtMTIyLjM3NCwgMi4wXSwgWzQ3LjUzNzYsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41Mzc2LCAtMTIxLjg3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM3NywgLTEyMi4zNzYsIDEuMF0sIFs0Ny41Mzc3LCAtMTIyLjI2NiwgMS4wXSwgWzQ3LjUzNzgsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTM3OCwgLTEyMi4zNjksIDEuMF0sIFs0Ny41Mzc5LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM3OSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzNzksIC0xMjIuMjY0LCAxLjBdLCBbNDcuNTM4MDAwMDAwMDAwMDA0LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTM4MDAwMDAwMDAwMDA0LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM4MDAwMDAwMDAwMDA0LCAtMTIyLjM1MywgMS4wXSwgWzQ3LjUzODAwMDAwMDAwMDAwNCwgLTEyMS44NywgMS4wXSwgWzQ3LjUzODEsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNTM4MSwgLTEyMi4yNjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzODEsIC0xMjEuODksIDEuMF0sIFs0Ny41MzgxLCAtMTIxLjg4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM4MSwgLTEyMS44NzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzODIsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTM4MiwgLTEyMS44NzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzODIsIC0xMjEuODcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzgzLCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM4MywgLTEyMS44Nzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzODMsIC0xMjEuODc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzg0LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjUzODQsIC0xMjIuMzcsIDEuMF0sIFs0Ny41Mzg0LCAtMTIyLjI3NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM4NCwgLTEyMS44ODksIDEuMF0sIFs0Ny41Mzg0LCAtMTIxLjg4LCAxLjBdLCBbNDcuNTM4NSwgLTEyMi4zNTUsIDEuMF0sIFs0Ny41Mzg1LCAtMTIxLjg4LCAxLjBdLCBbNDcuNTM4NSwgLTEyMS44NywgMS4wXSwgWzQ3LjUzODYsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzg3LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjUzODcsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41Mzg3LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjUzODcsIC0xMjIuMjc3LCAxLjBdLCBbNDcuNTM4NywgLTEyMi4yNjYsIDEuMF0sIFs0Ny41Mzg3LCAtMTIyLjI2NSwgMS4wXSwgWzQ3LjUzODcsIC0xMjEuODc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzg4LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjUzODgsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNTM4OCwgLTEyMi4yNzUsIDEuMF0sIFs0Ny41Mzg4LCAtMTIxLjg5LCAxLjBdLCBbNDcuNTM4OCwgLTEyMS44NywgMS4wXSwgWzQ3LjUzODksIC0xMjIuMzY3LCAxLjBdLCBbNDcuNTM4OSwgLTEyMi4zNTMsIDEuMF0sIFs0Ny41Mzg5LCAtMTIxLjg5LCAxLjBdLCBbNDcuNTM4OSwgLTEyMS44ODEsIDEuMF0sIFs0Ny41Mzg5LCAtMTIxLjg3NiwgMS4wXSwgWzQ3LjUzODk5OTk5OTk5OTk5NCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzODk5OTk5OTk5OTk5NCwgLTEyMS44Nzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzOTEsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTM5MSwgLTEyMi4yNzQsIDEuMF0sIFs0Ny41MzkyLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTM5MiwgLTEyMi4yNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzOTMsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNTM5MywgLTEyMi4zNTEsIDEuMF0sIFs0Ny41Mzk0LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjUzOTQsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzk0LCAtMTIyLjM2OCwgMi4wXSwgWzQ3LjUzOTQsIC0xMjEuODc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzk0LCAtMTIxLjg3NSwgMS4wXSwgWzQ3LjUzOTUsIC0xMjIuMzksIDEuMF0sIFs0Ny41Mzk1LCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjUzOTYsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTM5NiwgLTEyMi4zNjgsIDEuMF0sIFs0Ny41Mzk2LCAtMTIyLjI3NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM5NiwgLTEyMi4yNzQsIDEuMF0sIFs0Ny41Mzk2LCAtMTIyLjI2ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM5NiwgLTEyMS44ODksIDEuMF0sIFs0Ny41Mzk2LCAtMTIxLjg4MiwgMS4wXSwgWzQ3LjUzOTcsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTM5NywgLTEyMi4zODUsIDEuMF0sIFs0Ny41Mzk4LCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjUzOTgsIC0xMjEuODY5LCAxLjBdLCBbNDcuNTM5OSwgLTEyMi4zODUsIDEuMF0sIFs0Ny41Mzk5LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjU0LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjU0LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjU0LCAtMTIyLjI3NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQsIC0xMjIuMjc1LCAxLjBdLCBbNDcuNTQsIC0xMjIuMjcsIDEuMF0sIFs0Ny41NCwgLTEyMi4yNjg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0LCAtMTIyLjI2NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQsIC0xMjEuODc2LCAxLjBdLCBbNDcuNTQwMSwgLTEyMS44Nzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MDIsIC0xMjIuMzg3LCAzLjBdLCBbNDcuNTQwMiwgLTEyMi4zOCwgMS4wXSwgWzQ3LjU0MDIsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTQwMiwgLTEyMS44ODcsIDEuMF0sIFs0Ny41NDAyLCAtMTIxLjg3NiwgMS4wXSwgWzQ3LjU0MDIsIC0xMjEuODc1LCAxLjBdLCBbNDcuNTQwMywgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NDAzLCAtMTIxLjg4OSwgMS4wXSwgWzQ3LjU0MDQsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTQwNCwgLTEyMi4zNTIsIDEuMF0sIFs0Ny41NDA0LCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTQwNSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MDUsIC0xMjIuMjcsIDEuMF0sIFs0Ny41NDA1LCAtMTIyLjI2Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQwNSwgLTEyMS44ODIsIDEuMF0sIFs0Ny41NDA2LCAtMTIxLjg4OSwgMS4wXSwgWzQ3LjU0MDYsIC0xMjEuODg3LCAxLjBdLCBbNDcuNTQwNywgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MDcsIC0xMjEuODksIDEuMF0sIFs0Ny41NDA5LCAtMTIyLjM5LCAxLjBdLCBbNDcuNTQwOSwgLTEyMi4yNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MDksIC0xMjIuMjY3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDExLCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQxMSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NDEyLCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQxMiwgLTEyMi4yNywgMS4wXSwgWzQ3LjU0MTIsIC0xMjEuODc2LCAxLjBdLCBbNDcuNTQxMywgLTEyMS44NzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MTQsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNTQxNCwgLTEyMi4yODEsIDEuMF0sIFs0Ny41NDE0LCAtMTIxLjk5NCwgMS4wXSwgWzQ3LjU0MTYsIC0xMjIuMzgsIDEuMF0sIFs0Ny41NDE2LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQxNiwgLTEyMS45OTQsIDEuMF0sIFs0Ny41NDE3LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQxNywgLTEyMi4zNTUsIDEuMF0sIFs0Ny41NDE3LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjU0MTgsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTQxOCwgLTEyMi4yODksIDEuMF0sIFs0Ny41NDE4LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU0MTgsIC0xMjIuMjc1LCAxLjBdLCBbNDcuNTQxOCwgLTEyMi4wMSwgMS4wXSwgWzQ3LjU0MTksIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDE5LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU0MTksIC0xMjIuMjg5LCAxLjBdLCBbNDcuNTQxOSwgLTEyMi4yODgsIDEuMF0sIFs0Ny41NDE5LCAtMTIyLjI3MSwgMi4wXSwgWzQ3LjU0MiwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MiwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MiwgLTEyMi4yNjUsIDEuMF0sIFs0Ny41NDIxLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTQyMSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41NDIxLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQyMSwgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MjIsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNTQyMywgLTEyMi4zODUsIDEuMF0sIFs0Ny41NDIzLCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjU0MjMsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNTQyNCwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NDI0LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjU0MjUsIC0xMjIuMzksIDEuMF0sIFs0Ny41NDI1LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU0MjYsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDI2LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU0MjYsIC0xMjIuMjYyLCAxLjBdLCBbNDcuNTQyNiwgLTEyMS45OTUsIDEuMF0sIFs0Ny41NDI3LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQyNywgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MjcsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDI3LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjU0MjcsIC0xMjIuMjg4LCAzLjBdLCBbNDcuNTQyNywgLTEyMi4yNzUsIDEuMF0sIFs0Ny41NDI3LCAtMTIyLjI3MiwgMS4wXSwgWzQ3LjU0MjcsIC0xMjIuMjY1LCAxLjBdLCBbNDcuNTQyNywgLTEyMS45OTUsIDEuMF0sIFs0Ny41NDI4LCAtMTIyLjI3LCAxLjBdLCBbNDcuNTQyOSwgLTEyMi4zODcsIDEuMF0sIFs0Ny41NDI5LCAtMTIyLjAxMiwgMS4wXSwgWzQ3LjU0MjksIC0xMjEuOTk1LCAxLjBdLCBbNDcuNTQzLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU0MywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MywgLTEyMi4zNjgsIDEuMF0sIFs0Ny41NDMsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDMsIC0xMjIuMjksIDEuMF0sIFs0Ny41NDMsIC0xMjIuMjgsIDEuMF0sIFs0Ny41NDMsIC0xMjIuMjY2LCAxLjBdLCBbNDcuNTQzLCAtMTIyLjAxLCAxLjBdLCBbNDcuNTQzMSwgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MzEsIC0xMjEuOTk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDMyLCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU0MzIsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDMyLCAtMTIyLjI3NywgMS4wXSwgWzQ3LjU0MzIsIC0xMjIuMjcyLCAxLjBdLCBbNDcuNTQzMywgLTEyMi4zODQsIDEuMF0sIFs0Ny41NDMzLCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU0MzMsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTQzMywgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MzMsIC0xMjIuMjY0LCAxLjBdLCBbNDcuNTQzNCwgLTEyMi4zOTUsIDEuMF0sIFs0Ny41NDM0LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQzNCwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41NDM0LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjU0MzQsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDM0LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAzLjBdLCBbNDcuNTQzNCwgLTEyMi4yNywgMS4wXSwgWzQ3LjU0MzUsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNTQzNSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MzUsIC0xMjIuMjYsIDEuMF0sIFs0Ny41NDM2LCAtMTIyLjM5LCAxLjBdLCBbNDcuNTQzNiwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MzYsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNTQzNiwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MzcsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTQzNywgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MzcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNTQzNywgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MzcsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNTQzNywgLTEyMi4yOCwgMS4wXSwgWzQ3LjU0MzgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTQzOSwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MzksIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDM5LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU0MzksIC0xMjIuMDExMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDM5LCAtMTIxLjk5NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU0NCwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41NDQxLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTQ0MSwgLTEyMi4zNjksIDEuMF0sIFs0Ny41NDQxLCAtMTIyLjI3MywgMS4wXSwgWzQ3LjU0NDEsIC0xMjIuMDEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDQyLCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjU0NDIsIC0xMjIuMzgsIDEuMF0sIFs0Ny41NDQyLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTQ0MiwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NDIsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDQzLCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU0NDMsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTQ0MywgLTEyMi4zNzUsIDEuMF0sIFs0Ny41NDQzLCAtMTIxLjk5NCwgMS4wXSwgWzQ3LjU0NDQsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTQ0NCwgLTEyMi4zNjksIDEuMF0sIFs0Ny41NDQ1LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU0NDUsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTQ0NSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NDQ1LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU0NDUsIC0xMjIuMjczLCAxLjBdLCBbNDcuNTQ0NSwgLTEyMi4yNjYsIDEuMF0sIFs0Ny41NDQ1LCAtMTIyLjI2Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ0NSwgLTEyMi4wMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NDUsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDQ2LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU0NDYsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTQ0NiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41NDQ3LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ0NywgLTEyMi4zODksIDEuMF0sIFs0Ny41NDQ3LCAtMTIyLjI3LCAxLjBdLCBbNDcuNTQ0OCwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41NDQ4LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU0NDksIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDQ5LCAtMTIyLjI3MiwgMS4wXSwgWzQ3LjU0NDksIC0xMjIuMjYyLCAxLjBdLCBbNDcuNTQ0OSwgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NDksIC0xMjEuOTk1LCAxLjBdLCBbNDcuNTQ1LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjU0NSwgLTEyMi4zNiwgMS4wXSwgWzQ3LjU0NSwgLTEyMi4yNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NTEsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDUyLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ1MiwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NTIsIC0xMjIuMjcsIDEuMF0sIFs0Ny41NDUzLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjU0NTMsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNTQ1MywgLTEyMi4yNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NTMsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDUzLCAtMTIxLjk5NSwgMi4wXSwgWzQ3LjU0NTQsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTQ1NCwgLTEyMS45OTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NTUsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTQ1NSwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NTUsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNTQ1NiwgLTEyMi4zNTksIDEuMF0sIFs0Ny41NDU2LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ1NywgLTEyMi4zODMsIDEuMF0sIFs0Ny41NDU3LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjU0NTcsIC0xMjEuOTkxLCAxLjBdLCBbNDcuNTQ1OCwgLTEyMi4zNjksIDMuMF0sIFs0Ny41NDU4LCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ1OCwgLTEyMS45OTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NTksIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDU5LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ1OSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NDU5LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjU0NTksIC0xMjIuMjcsIDEuMF0sIFs0Ny41NDU5LCAtMTIyLjI2NSwgMS4wXSwgWzQ3LjU0NTksIC0xMjEuOTk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDYwMDAwMDAwMDAwMSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NjAwMDAwMDAwMDAxLCAtMTIyLjAxMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ2MDAwMDAwMDAwMDEsIC0xMjIuMDA5LCAxLjBdLCBbNDcuNTQ2MSwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NjEsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDYxLCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ2MiwgLTEyMi4zNjksIDEuMF0sIFs0Ny41NDYyLCAtMTIyLjI3NywgMS4wXSwgWzQ3LjU0NjIsIC0xMjIuMjY1LCAxLjBdLCBbNDcuNTQ2MywgLTEyMi4zOTcsIDEuMF0sIFs0Ny41NDYzLCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjU0NjMsIC0xMjIuMjcyLCAxLjBdLCBbNDcuNTQ2MywgLTEyMS45OTUsIDEuMF0sIFs0Ny41NDY0LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU0NjUsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNTQ2NSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41NDY1LCAtMTIyLjAxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ2NywgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NjcsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTQ2NywgLTEyMi4zNTksIDEuMF0sIFs0Ny41NDY4LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ2OCwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NjgsIC0xMjIuMjc0LCAxLjBdLCBbNDcuNTQ2OCwgLTEyMi4yNjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NjgsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NDY4LCAtMTIxLjk5NCwgMS4wXSwgWzQ3LjU0NjksIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTQ2OSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41NDY5LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjU0NjksIC0xMjIuMjc3LCAxLjBdLCBbNDcuNTQ2OSwgLTEyMS45OTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NywgLTEyMi4zMDIsIDEuMF0sIFs0Ny41NDcsIC0xMjIuMDEsIDEuMF0sIFs0Ny41NDcsIC0xMjEuOTk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDcxLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ3MSwgLTEyMS45OTYwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjU0NzIsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTQ3MiwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NzIsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDcyLCAtMTIyLjMsIDEuMF0sIFs0Ny41NDcyLCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ3MiwgLTEyMS45OTQsIDEuMF0sIFs0Ny41NDczLCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjU0NzMsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDc0LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ3NCwgLTEyMi4yOTUsIDEuMF0sIFs0Ny41NDc0LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjU0NzUsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTQ3NiwgLTEyMi4zOTcsIDEuMF0sIFs0Ny41NDc2LCAtMTIyLjM3LCAxLjBdLCBbNDcuNTQ3NiwgLTEyMi4zNiwgMS4wXSwgWzQ3LjU0NzYsIC0xMjIuMDEyLCAxLjBdLCBbNDcuNTQ3NiwgLTEyMi4wMDUsIDIuMF0sIFs0Ny41NDc3LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU0NzcsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NDc3LCAtMTIyLjAwMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ3NywgLTEyMi4wMDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NzgsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTQ3OCwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NzgsIC0xMjIuMDA1LCAxLjBdLCBbNDcuNTQ3OCwgLTEyMi4wLCAxLjBdLCBbNDcuNTQ3OCwgLTEyMS45OTksIDIuMF0sIFs0Ny41NDc5LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjU0NzksIC0xMjIuMDAyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDc5LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU0NzksIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDgsIC0xMjIuMzc1LCAyLjBdLCBbNDcuNTQ4LCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ4LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ4LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ4MSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41NDgxLCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjU0ODEsIC0xMjEuOTk1LCAxLjBdLCBbNDcuNTQ4MiwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0ODIsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDgyLCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjU0ODIsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNTQ4MiwgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0ODIsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDgzLCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ4MywgLTEyMi4zNiwgMS4wXSwgWzQ3LjU0ODMsIC0xMjIuMjc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDgzLCAtMTIyLjI3NywgMS4wXSwgWzQ3LjU0ODMsIC0xMjIuMjYxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDgzLCAtMTIyLjAwMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ4NCwgLTEyMi4yODcsIDEuMF0sIFs0Ny41NDg0LCAtMTIyLjI3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ4NCwgLTEyMi4yNjg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0ODUsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNTQ4NSwgLTEyMS45OTksIDEuMF0sIFs0Ny41NDg2LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTQ4NiwgLTEyMi4yNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0ODcsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTQ4NywgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NDg3LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ4NywgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0ODcsIC0xMjIuMjc3LCAxLjBdLCBbNDcuNTQ4NywgLTEyMi4yNzIsIDEuMF0sIFs0Ny41NDg4LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ4OCwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0ODgsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDg4LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU0ODksIC0xMjIuMzk4LCAxLjBdLCBbNDcuNTQ4OSwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0ODksIC0xMjIuMzYzLCAxLjBdLCBbNDcuNTQ4OSwgLTEyMi4yNzEsIDEuMF0sIFs0Ny41NDg5LCAtMTIyLjAwNywgMS4wXSwgWzQ3LjU0ODksIC0xMjIuMDA2LCAxLjBdLCBbNDcuNTQ5LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU0OSwgLTEyMi4zNzQsIDEuMF0sIFs0Ny41NDksIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny41NDkxLCAtMTIyLjM4NywgMy4wXSwgWzQ3LjU0OTEsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDkxLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU0OTEsIC0xMjIuMjc2MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny41NDkxLCAtMTIyLjI3MiwgMS4wXSwgWzQ3LjU0OTEsIC0xMjIuMjY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDkxLCAtMTIyLjAwNiwgMS4wXSwgWzQ3LjU0OTEsIC0xMjIuMDA1LCAxLjBdLCBbNDcuNTQ5MiwgLTEyMi4zODksIDEuMF0sIFs0Ny41NDkyLCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU0OTIsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDkyLCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjU0OTIsIC0xMjIuMywgMS4wXSwgWzQ3LjU0OTIsIC0xMjIuMjc2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDkyLCAtMTIyLjI3NCwgMS4wXSwgWzQ3LjU0OTIsIC0xMjIuMDA4LCAxLjBdLCBbNDcuNTQ5MywgLTEyMi4zODcsIDEuMF0sIFs0Ny41NDkzLCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ5MywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0OTMsIC0xMjIuMDAzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDkzLCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU0OTMsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDkzLCAtMTIxLjk5MywgMS4wXSwgWzQ3LjU0OTQsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDk0LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ5NCwgLTEyMi4yNzQsIDEuMF0sIFs0Ny41NDk0LCAtMTIxLjk5NCwgMS4wXSwgWzQ3LjU0OTUsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNTQ5NSwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0OTUsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDk1LCAtMTIxLjk5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ5NiwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NDk2LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTQ5NiwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NDk2LCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ5NiwgLTEyMi4yNzksIDEuMF0sIFs0Ny41NDk3LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjU0OTcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNTQ5NywgLTEyMi4yOTUsIDEuMF0sIFs0Ny41NDk4LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU0OTgsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTQ5OCwgLTEyMi4zNTUsIDEuMF0sIFs0Ny41NDk4LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ5OCwgLTEyMS45OTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0OTgsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDk5LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ5OSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0OTksIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTQ5OSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41NDk5LCAtMTIyLjM2LCAxLjBdLCBbNDcuNTQ5OSwgLTEyMi4yNzQsIDEuMF0sIFs0Ny41NDk5LCAtMTIyLjI2ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ5OSwgLTEyMi4yNjQsIDEuMF0sIFs0Ny41NDk5LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NSwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUsIC0xMjIuMjcyLCAxLjBdLCBbNDcuNTUwMSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MDEsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNTUwMSwgLTEyMi4yNjEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MDIsIC0xMjIuMjg2LCAxLjBdLCBbNDcuNTUwMiwgLTEyMi4yNzQsIDIuMF0sIFs0Ny41NTAyLCAtMTIyLjI2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUwMywgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MDMsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNTUwMywgLTEyMi4yODUsIDEuMF0sIFs0Ny41NTAzLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUwMywgLTEyMi4yOCwgMS4wXSwgWzQ3LjU1MDMsIC0xMjEuOTk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTA1LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUwNiwgLTEyMi4zOTUsIDEuMF0sIFs0Ny41NTA2LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU1MDYsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTUwNiwgLTEyMi4yOSwgMS4wXSwgWzQ3LjU1MDYsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNTUwNiwgLTEyMi4yNzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MDYsIC0xMjIuMjY4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTA2LCAtMTIxLjk5MywgMS4wXSwgWzQ3LjU1MDcsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTUwNywgLTEyMi4zODEsIDEuMF0sIFs0Ny41NTA3LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUwNywgLTEyMi4yNjEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MDgsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTA4LCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjU1MDgsIC0xMjIuMywgMS4wXSwgWzQ3LjU1MDgsIC0xMjIuMjY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTA5LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUwOSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny41NTA5LCAtMTIyLjI2MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUxLCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU1MSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NTEsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNTUxLCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUxLCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjU1MSwgLTEyMi4yNzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MSwgLTEyMi4yNzIsIDEuMF0sIFs0Ny41NTExLCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU1MTEsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTExLCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUxMiwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MTMsIC0xMjIuMzkzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTEzLCAtMTIyLjI3NSwgMS4wXSwgWzQ3LjU1MTMsIC0xMjIuMjczLCAxLjBdLCBbNDcuNTUxMywgLTEyMi4yNzIsIDEuMF0sIFs0Ny41NTEzLCAtMTIyLjI2NCwgMS4wXSwgWzQ3LjU1MTMsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTEzLCAtMTIxLjk5NCwgMS4wXSwgWzQ3LjU1MTQsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTUxNCwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MTQsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTE0LCAtMTIyLjI3MSwgMS4wXSwgWzQ3LjU1MTQsIC0xMjIuMjY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTE0LCAtMTIyLjI2NiwgMS4wXSwgWzQ3LjU1MTQsIC0xMjIuMjYyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTE0LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUxNSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MTUsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTE1LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUxNSwgLTEyMi4yNjIsIDEuMF0sIFs0Ny41NTE1LCAtMTIyLjAsIDEuMF0sIFs0Ny41NTE2LCAtMTIyLjM5OCwgMS4wXSwgWzQ3LjU1MTYsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTUxNiwgLTEyMi4yNjI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MTcsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTUxNywgLTEyMi4zODEsIDEuMF0sIFs0Ny41NTE3LCAtMTIyLjM2LCAxLjBdLCBbNDcuNTUxNywgLTEyMi4yNzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MTcsIC0xMjIuMjcyLCAxLjBdLCBbNDcuNTUxNywgLTEyMi4yNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MTcsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTE4LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU1MTgsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTUxOCwgLTEyMi4zNywgMS4wXSwgWzQ3LjU1MTgsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTE4LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjU1MTgsIC0xMjIuMjc3LCAyLjBdLCBbNDcuNTUxOCwgLTEyMi4yNjYsIDEuMF0sIFs0Ny41NTE4LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTUxOSwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NTE5LCAtMTIyLjAwMSwgMS4wXSwgWzQ3LjU1MTksIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTUyLCAtMTIyLjI5LCAxLjBdLCBbNDcuNTUyMSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjEsIC0xMjIuMjcxLCAxLjBdLCBbNDcuNTUyMSwgLTEyMS45OTksIDEuMF0sIFs0Ny41NTIxLCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUyMSwgLTEyMS45OTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MjEsIC0xMjEuOTk1LCAxLjBdLCBbNDcuNTUyMiwgLTEyMi4zODIsIDEuMF0sIFs0Ny41NTIyLCAtMTIyLjAwMSwgMS4wXSwgWzQ3LjU1MjIsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTUyMywgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjMsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTIzLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjU1MjMsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTI0LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjU1MjQsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTI0LCAtMTIyLjI3LCAxLjBdLCBbNDcuNTUyNCwgLTEyMi4yNjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MjQsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTUyNCwgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjQsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTI0LCAtMTIxLjk5MiwgMS4wXSwgWzQ3LjU1MjQsIC0xMjEuOTkxLCAxLjBdLCBbNDcuNTUyNSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny41NTI1LCAtMTIyLjMsIDEuMF0sIFs0Ny41NTI1LCAtMTIyLjI3NywgMS4wXSwgWzQ3LjU1MjUsIC0xMjIuMjc0LCAxLjBdLCBbNDcuNTUyNSwgLTEyMi4yNzMsIDEuMF0sIFs0Ny41NTI1LCAtMTIyLjI2ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUyNSwgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjUsIC0xMjEuOTksIDEuMF0sIFs0Ny41NTI2LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU1MjYsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNTUyNiwgLTEyMS45OTksIDEuMF0sIFs0Ny41NTI2LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUyNiwgLTEyMS45OTQsIDEuMF0sIFs0Ny41NTI3LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUyNywgLTEyMS45OTIsIDEuMF0sIFs0Ny41NTI3LCAtMTIxLjk5LCAxLjBdLCBbNDcuNTUyOCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MjgsIC0xMjIuMjcyLCAxLjBdLCBbNDcuNTUyOCwgLTEyMS45OTksIDIuMF0sIFs0Ny41NTI5LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUyOSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjksIC0xMjIuMjgxLCAxLjBdLCBbNDcuNTUyOSwgLTEyMi4yNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjksIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTI5LCAtMTIxLjk5LCAxLjBdLCBbNDcuNTUzMDAwMDAwMDAwMDA0LCAtMTIyLjM3LCAxLjBdLCBbNDcuNTUzMDAwMDAwMDAwMDA0LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjU1MzAwMDAwMDAwMDAwNCwgLTEyMi4zNTQsIDEuMF0sIFs0Ny41NTMxLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUzMSwgLTEyMS45OTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MzIsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTUzMiwgLTEyMi4zODEsIDEuMF0sIFs0Ny41NTMyLCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzMiwgLTEyMi4zNjUsIDEuMF0sIFs0Ny41NTMyLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzMiwgLTEyMi4yOCwgMi4wXSwgWzQ3LjU1MzIsIC0xMjIuMjc3LCAyLjBdLCBbNDcuNTUzMiwgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MzMsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTUzMywgLTEyMi4zODEsIDEuMF0sIFs0Ny41NTMzLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjU1MzMsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNTUzMywgLTEyMS45OTUsIDEuMF0sIFs0Ny41NTMzLCAtMTIxLjk5MywgMS4wXSwgWzQ3LjU1MzMsIC0xMjEuOTkyLCAxLjBdLCBbNDcuNTUzNCwgLTEyMi4zODIsIDEuMF0sIFs0Ny41NTM0LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU1MzQsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTUzNCwgLTEyMi4wMDIwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjU1MzUsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTUzNSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MzUsIC0xMjIuMzksIDEuMF0sIFs0Ny41NTM1LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzNSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MzUsIC0xMjIuMjY4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTM1LCAtMTIxLjk5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzNiwgLTEyMi4zODMsIDEuMF0sIFs0Ny41NTM2LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzNiwgLTEyMi4zNTksIDEuMF0sIFs0Ny41NTM2LCAtMTIyLjI4NiwgMi4wXSwgWzQ3LjU1MzYsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTM2LCAtMTIyLjI2NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzNywgLTEyMi4zOTgsIDEuMF0sIFs0Ny41NTM3LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUzNywgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MzcsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTM3LCAtMTIxLjk5NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUzOCwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MzksIC0xMjIuMywgMS4wXSwgWzQ3LjU1Mzk5OTk5OTk5OTk5NSwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NTQxLCAtMTIyLjI5LCAxLjBdLCBbNDcuNTU0MywgLTEyMi4zOTgsIDEuMF0sIFs0Ny41NTQzLCAtMTIyLjI5LCAxLjBdLCBbNDcuNTU0NSwgLTEyMi4yODgsIDEuMF0sIFs0Ny41NTQ2LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTU0NywgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1NDcsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTQ5LCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjU1NDksIC0xMjIuMjg3LCAxLjBdLCBbNDcuNTU1LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU1MSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny41NTUxLCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjU1NTEsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNTU1MiwgLTEyMi4zODEsIDIuMF0sIFs0Ny41NTUyLCAtMTIyLjMsIDEuMF0sIFs0Ny41NTUyLCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjU1NTMsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNTU1MywgLTEyMi4yODYsIDEuMF0sIFs0Ny41NTU0LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjU1NTQsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNTU1NSwgLTEyMi4zOTcsIDEuMF0sIFs0Ny41NTU2LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjU1NTcsIC0xMjIuMjgsIDEuMF0sIFs0Ny41NTU5LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjU1NTksIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTYwMDAwMDAwMDAwMDQsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTU2MDAwMDAwMDAwMDA0LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU1NjAwMDAwMDAwMDAwNCwgLTEyMi4yOTEsIDEuMF0sIFs0Ny41NTYxLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU2MSwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1NjMsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTYzLCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTU2MywgLTEyMi4yODEsIDEuMF0sIFs0Ny41NTYzLCAtMTIyLjI4LCAxLjBdLCBbNDcuNTU2NSwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1NjYsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTY3LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjU1NjgsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTU2OSwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1NjksIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTY5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU1Njk5OTk5OTk5OTk5NSwgLTEyMi4zOTUsIDEuMF0sIFs0Ny41NTY5OTk5OTk5OTk5OTUsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTU3MSwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NTcxLCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU3MiwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1NzIsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNTU3MywgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1NzMsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNTU3MywgLTEyMi4yODcsIDEuMF0sIFs0Ny41NTczLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTU3NCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1NzUsIC0xMjIuMywgMS4wXSwgWzQ3LjU1NzUsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNTU3NiwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1NzYsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTU3NiwgLTEyMi4yODEsIDEuMF0sIFs0Ny41NTc3LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjU1NzcsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTU3NywgLTEyMi4zODksIDEuMF0sIFs0Ny41NTc3LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU3NywgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1NzksIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTU3OSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41NTgsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTgsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTU4MSwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NTgyLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU4MywgLTEyMi4zOCwgMS4wXSwgWzQ3LjU1ODQsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTg1LCAtMTIyLjI5LCAxLjBdLCBbNDcuNTU4NiwgLTEyMi4zODMsIDEuMF0sIFs0Ny41NTg3LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU4OCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1ODgsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNTU4OSwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1ODksIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTg5LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU1ODksIC0xMjIuMjksIDEuMF0sIFs0Ny41NTksIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTU5LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU5MSwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NTkxLCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjU1OTEsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTU5MiwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NTkyLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU5MywgLTEyMi4zLCAxLjBdLCBbNDcuNTU5NCwgLTEyMi4yODUsIDEuMF0sIFs0Ny41NTk5LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTYwMSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MDIsIC0xMjIuMzksIDEuMF0sIFs0Ny41NjAyLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTYwMiwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NjAyLCAtMTIyLjI4LCAxLjBdLCBbNDcuNTYwNSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny41NjA3LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU2MDcsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NjA5LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjU2MTAwMDAwMDAwMDAxLCAtMTIyLjM5OCwgMS4wXSwgWzQ3LjU2MTEsIC0xMjIuMjgsIDIuMF0sIFs0Ny41NjE1LCAtMTIyLjI4LCAxLjBdLCBbNDcuNTYxNSwgLTEyMi4yNzksIDEuMF0sIFs0Ny41NjE2LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTYxNywgLTEyMi4zODUsIDEuMF0sIFs0Ny41NjE4LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjU2MTksIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTYyLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTYyLCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjU2MiwgLTEyMi4yODksIDEuMF0sIFs0Ny41NjIsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjIsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNTYyMSwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MjEsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTYyMSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny41NjIxLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTYyMiwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2MjIsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTYyMiwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NjIyLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjU2MjMsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTYyNCwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MjQsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNTYyNSwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MjYsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTYyNiwgLTEyMi4zODIsIDEuMF0sIFs0Ny41NjI3LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU2MjgsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTYyOSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2MywgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MywgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2MzEsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTYzMSwgLTEyMi4zODEsIDEuMF0sIFs0Ny41NjMxLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTYzMSwgLTEyMi4yODUsIDEuMF0sIFs0Ny41NjMyLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTYzMywgLTEyMi4zODUsIDEuMF0sIFs0Ny41NjMzLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTYzMywgLTEyMi4yODksIDEuMF0sIFs0Ny41NjMzLCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU2MzQsIC0xMjIuMzkzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjM0LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU2MzQsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NjM1LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTYzNSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MzUsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NjM2LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTYzNiwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2MzcsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTYzNywgLTEyMi4zNzUsIDEuMF0sIFs0Ny41NjM3LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU2MzgsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NjM4LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTYzOCwgLTEyMi4yOTUsIDEuMF0sIFs0Ny41NjM5LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTYzOSwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NjM5OTk5OTk5OTk5OSwgLTEyMi4zOTgsIDEuMF0sIFs0Ny41NjM5OTk5OTk5OTk5OSwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NDEsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjQxLCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU2NDIsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjQyLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY0MiwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NjQzLCAtMTIyLjM4LCAxLjBdLCBbNDcuNTY0MywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NDQsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjQ0LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU2NDQsIC0xMjIuMjksIDEuMF0sIFs0Ny41NjQ0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjU2NDUsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTY0NiwgLTEyMi4yOTUsIDEuMF0sIFs0Ny41NjQ3LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY0OCwgLTEyMi4yOTQsIDIuMF0sIFs0Ny41NjQ5LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTY1LCAtMTIyLjM5OCwgMS4wXSwgWzQ3LjU2NSwgLTEyMi4zODEsIDEuMF0sIFs0Ny41NjUsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNTY1LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY1LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjU2NTEsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTY1MiwgLTEyMi4yODEsIDEuMF0sIFs0Ny41NjU0LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjU2NTUsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTY1NSwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NTUsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NjU3LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU2NTcsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTY1NywgLTEyMi4yODUsIDEuMF0sIFs0Ny41NjU4LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjU2NTgsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNTY1OCwgLTEyMi4yOTEsIDEuMF0sIFs0Ny41NjU5LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjU2NTksIC0xMjIuMzgsIDEuMF0sIFs0Ny41NjU5LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU2NTksIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTY1OSwgLTEyMi4yODYsIDEuMF0sIFs0Ny41NjU5LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY2LCAtMTIyLjM4LCAyLjBdLCBbNDcuNTY2LCAtMTIyLjI5LCAxLjBdLCBbNDcuNTY2MSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41NjYxLCAtMTIyLjI5LCAyLjBdLCBbNDcuNTY2MiwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NjMsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjYzLCAtMTIyLjI5LCAxLjBdLCBbNDcuNTY2MywgLTEyMi4yODksIDEuMF0sIFs0Ny41NjYzLCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjU2NjQsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTY2NCwgLTEyMS45OTksIDEuMF0sIFs0Ny41NjY2LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU2NjYsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjY2LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU2NjcsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NjY4LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjU2NjgsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTY2OCwgLTEyMi4wMDUsIDEuMF0sIFs0Ny41NjY5LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTY2OSwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NjksIC0xMjIuMDEyLCAxLjBdLCBbNDcuNTY3LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU2NywgLTEyMi4zOCwgMS4wXSwgWzQ3LjU2NywgLTEyMi4wMTIsIDEuMF0sIFs0Ny41NjcxLCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY3MSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NjcxLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY3MSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NzEsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjcyLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTY3MiwgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2NzIsIC0xMjIuMDA1LCAxLjBdLCBbNDcuNTY3MiwgLTEyMi4wMDM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NzMsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTY3NCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2NzQsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTY3NCwgLTEyMi4yOSwgMS4wXSwgWzQ3LjU2NzUsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNTY3NiwgLTEyMi4zOTgsIDEuMF0sIFs0Ny41Njc3LCAtMTIyLjM4LCAxLjBdLCBbNDcuNTY3NywgLTEyMi4zNzYsIDEuMF0sIFs0Ny41Njc3LCAtMTIyLjI5LCAxLjBdLCBbNDcuNTY3NywgLTEyMi4yODUsIDEuMF0sIFs0Ny41Njc3LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjU2NzcsIC0xMjIuMDEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njc3LCAtMTIyLjAwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY3OCwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NzgsIC0xMjIuMDAzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njc5LCAtMTIyLjAxMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTY4MSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41NjgxLCAtMTIyLjI4NSwgMi4wXSwgWzQ3LjU2ODIsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjgyLCAtMTIyLjAwNSwgMS4wXSwgWzQ3LjU2ODMsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNTY4MywgLTEyMi4yODUsIDEuMF0sIFs0Ny41NjgzLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY4MywgLTEyMi4wMDUsIDEuMF0sIFs0Ny41Njg0LCAtMTIyLjMsIDEuMF0sIFs0Ny41Njg1LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjU2ODUsIC0xMjIuMDEyLCAxLjBdLCBbNDcuNTY4NiwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2ODYsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTY4NiwgLTEyMi4zODUsIDEuMF0sIFs0Ny41Njg2LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTY4NiwgLTEyMi4yODYsIDEuMF0sIFs0Ny41Njg3LCAtMTIyLjM5NywgMS4wXSwgWzQ3LjU2ODcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNTY4OCwgLTEyMi4zOTcsIDEuMF0sIFs0Ny41Njg4LCAtMTIyLjAxMiwgMS4wXSwgWzQ3LjU2ODksIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njg5LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTY4OSwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2ODksIC0xMjIuMDA2LCAxLjBdLCBbNDcuNTY4OSwgLTEyMS45OTUsIDEuMF0sIFs0Ny41Njg5OTk5OTk5OTk5OTYsIC0xMjIuMDA3LCAyLjBdLCBbNDcuNTY5MSwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2OTEsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTY5MSwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NjkxLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY5MSwgLTEyMi4wMDcsIDEuMF0sIFs0Ny41NjkxLCAtMTIyLjAwMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY5MSwgLTEyMS45OTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2OTIsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTY5MiwgLTEyMi4wMDYsIDEuMF0sIFs0Ny41NjkzLCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU2OTMsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNTY5MywgLTEyMi4yOTYsIDEuMF0sIFs0Ny41NjkzLCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjU2OTMsIC0xMjIuMDAyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41Njk0LCAtMTIyLjM5NywgMS4wXSwgWzQ3LjU2OTQsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTY5NCwgLTEyMi4zODcsIDEuMF0sIFs0Ny41Njk0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU2OTQsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njk0LCAtMTIyLjAwNSwgMS4wXSwgWzQ3LjU2OTQsIC0xMjIuMDAxLCAxLjBdLCBbNDcuNTY5NSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41Njk1LCAtMTIyLjAwNiwgMS4wXSwgWzQ3LjU2OTUsIC0xMjEuOTkyLCAxLjBdLCBbNDcuNTY5NiwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2OTYsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njk2LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU2OTYsIC0xMjIuMDAzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njk3LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU2OTcsIC0xMjEuOTk0LCAxLjBdLCBbNDcuNTY5OCwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2OTgsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTY5OCwgLTEyMS45OTksIDEuMF0sIFs0Ny41Njk5LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjU2OTksIC0xMjIuMjk2LCAxLjBdLCBbNDcuNTY5OSwgLTEyMi4yODgsIDEuMF0sIFs0Ny41Njk5LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU2OTksIC0xMjIuMDA2LCAxLjBdLCBbNDcuNTY5OSwgLTEyMS45OTMsIDEuMF0sIFs0Ny41NywgLTEyMi4zOCwgMS4wXSwgWzQ3LjU3LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU3LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjU3MDEsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNTcwMSwgLTEyMi4yODgsIDEuMF0sIFs0Ny41NzAyLCAtMTIyLjM4LCAxLjBdLCBbNDcuNTcwMiwgLTEyMi4yODcsIDEuMF0sIFs0Ny41NzAyLCAtMTIxLjk5NSwgMS4wXSwgWzQ3LjU3MDMsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTcwMywgLTEyMi4yODgsIDEuMF0sIFs0Ny41NzAzLCAtMTIyLjI4LCAxLjBdLCBbNDcuNTcwMywgLTEyMi4wMDM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3MDMsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzAzLCAtMTIxLjk5NjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTcwNCwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NzA0LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjU3MDQsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTcwNSwgLTEyMi4zOTgsIDEuMF0sIFs0Ny41NzA1LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTcwNSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41NzA1LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjU3MDUsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNTcwNiwgLTEyMi4wMDYsIDEuMF0sIFs0Ny41NzA2LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU3MDcsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNTcwNywgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3MDcsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTcwNywgLTEyMS45OTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU3MDgsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNTcwOSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3MTAwMDAwMDAwMDAxLCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTcxMDAwMDAwMDAwMDEsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNTcxMSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NzExLCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTcxMSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41NzExLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjU3MTEsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzEyLCAtMTIxLjk5NSwgMS4wXSwgWzQ3LjU3MTMsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTcxMywgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3MTMsIC0xMjEuOTk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NzE0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU3MTQsIC0xMjIuMzgsIDEuMF0sIFs0Ny41NzE0LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU3MTQsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTcxNCwgLTEyMi4wMDUsIDEuMF0sIFs0Ny41NzE0LCAtMTIyLjAwMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTcxNCwgLTEyMS45OTEsIDEuMF0sIFs0Ny41NzE1LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTcxNywgLTEyMi4zODIsIDEuMF0sIFs0Ny41NzE3LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU3MTcsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNTcxNywgLTEyMS45OTEsIDEuMF0sIFs0Ny41NzE4LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU3MTgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTcxOCwgLTEyMS45OTksIDEuMF0sIFs0Ny41NzE4LCAtMTIxLjk5NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTcxOSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjU3MTksIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTcxOSwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU3MTksIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzE5LCAtMTIyLjAwNywgMS4wXSwgWzQ3LjU3MTksIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTcxOTk5OTk5OTk5OTk2LCAtMTIyLjI5LCAzLjBdLCBbNDcuNTcxOTk5OTk5OTk5OTk2LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjU3MjEsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTcyMiwgLTEyMi4zODcsIDEuMF0sIFs0Ny41NzIyLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjU3MjMsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTcyMywgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NzIzLCAtMTIyLjAwNywgMS4wXSwgWzQ3LjU3MjQsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTcyNCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny41NzI1LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU3MjYsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTcyNywgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NzI4LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTcyOSwgLTEyMi4zMDIsIDEuMF0sIFs0Ny41NzMsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNTczLCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU3MywgLTEyMi4zOCwgMS4wXSwgWzQ3LjU3MzEsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzMyLCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU3MzMsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTczMywgLTEyMi4yODcsIDEuMF0sIFs0Ny41NzM0LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU3MzQsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTczNiwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3MzcsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTczOCwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3MzgsIC0xMjEuOTkyLCAxLjBdLCBbNDcuNTc0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU3NCwgLTEyMi4yODksIDEuMF0sIFs0Ny41NzQsIC0xMjIuMDEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzQyLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc0MywgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3NDQsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NzQ0LCAtMTIxLjk5MywgMS4wXSwgWzQ3LjU3NDUsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNTc0NSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny41NzQ2LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc0NiwgLTEyMi4yODksIDEuMF0sIFs0Ny41NzQ2LCAtMTIyLjAwOSwgMS4wXSwgWzQ3LjU3NDcsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTc0NywgLTEyMi4yODcsIDEuMF0sIFs0Ny41NzQ3LCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc0NywgLTEyMi4wMDgsIDEuMF0sIFs0Ny41NzQ4LCAtMTIyLjAxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc0OSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny41NzUsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzUsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNTc1LCAtMTIyLjI4OCwgMi4wXSwgWzQ3LjU3NTEsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzUxLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc1MiwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3NTIsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNTc1MiwgLTEyMi4wMDksIDEuMF0sIFs0Ny41NzUyLCAtMTIxLjk5NSwgMS4wXSwgWzQ3LjU3NTMsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNTc1NCwgLTEyMi4yODgsIDEuMF0sIFs0Ny41NzU0LCAtMTIyLjAxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc1NSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny41NzU1LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc1NSwgLTEyMi4yODksIDEuMF0sIFs0Ny41NzU2LCAtMTIyLjAxLCAxLjBdLCBbNDcuNTc1OCwgLTEyMi4zODIsIDEuMF0sIFs0Ny41NzU4LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU3NTksIC0xMjIuMjg4LCAxLjBdLCBbNDcuNTc1OSwgLTEyMi4wMTIsIDEuMF0sIFs0Ny41NzU5LCAtMTIxLjk5NSwgMS4wXSwgWzQ3LjU3NTksIC0xMjEuOTk0LCAxLjBdLCBbNDcuNTc2MiwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU3NjMsIC0xMjIuMCwgMS4wXSwgWzQ3LjU3NjQsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNTc2NCwgLTEyMi4yODcsIDEuMF0sIFs0Ny41NzY0LCAtMTIxLjk5NjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTc2NSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3NjYsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzY3LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU3NjcsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzY5LCAtMTIyLjM5NywgMS4wXSwgWzQ3LjU3NywgLTEyMi4zOTUsIDEuMF0sIFs0Ny41NzcsIC0xMjIuMzgsIDEuMF0sIFs0Ny41NzcsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNTc3MSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41NzcxLCAtMTIyLjM4LCAxLjBdLCBbNDcuNTc3MSwgLTEyMS45OTQsIDEuMF0sIFs0Ny41NzcyLCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjU3NzIsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNTc3MiwgLTEyMi4wMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3NzIsIC0xMjIuMDAxLCAxLjBdLCBbNDcuNTc3MiwgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3NzMsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzczLCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU3NzQsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNTc3NiwgLTEyMi4yODksIDEuMF0sIFs0Ny41Nzc2LCAtMTIxLjk5NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTc3OCwgLTEyMi4zODksIDEuMF0sIFs0Ny41Nzc4LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjU3NzgsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNTc3OSwgLTEyMi4yOTQsIDEuMF0sIFs0Ny41NzgxLCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc4MiwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NzgzLCAtMTIyLjAwMSwgMS4wXSwgWzQ3LjU3ODMsIC0xMjEuOTk0LCAxLjBdLCBbNDcuNTc4NCwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3ODQsIC0xMjIuMzc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41Nzg1LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTc4NSwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41Nzg2LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU3ODYsIC0xMjEuOTk1LCAxLjBdLCBbNDcuNTc4NywgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU3ODcsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Nzg4LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjU3ODksIC0xMjIuMzk2LCAxLjBdLCBbNDcuNTc4OSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3ODk5OTk5OTk5OTk5LCAtMTIyLjM5LCAxLjBdLCBbNDcuNTc4OTk5OTk5OTk5OTksIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTc5MSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU3OTEsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Nzk3LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU3OTgsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Nzk4LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU4LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTgwMywgLTEyMi4zOTgsIDEuMF0sIFs0Ny41ODAzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTgwNiwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU4MDcsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTgwNywgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU4MDcsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTgwOCwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU4MDksIC0xMjIuMzksIDEuMF0sIFs0Ny41ODExLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjU4MTUsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNTgxOCwgLTEyMi4zODcsIDEuMF0sIFs0Ny41ODIxLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTgyNCwgLTEyMi4zODQsIDEuMF0sIFs0Ny41ODI2LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU4MjYsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41ODI3LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU4MjgsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNTgyOCwgLTEyMi4zOTUsIDEuMF0sIFs0Ny41ODI4LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjU4MjksIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTgzLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTgzMSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU4MzYsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTgzNywgLTEyMi4zODIsIDEuMF0sIFs0Ny41ODM4LCAtMTIyLjM5LCAxLjBdLCBbNDcuNTgzOCwgLTEyMi4zODIsIDEuMF0sIFs0Ny41ODM5LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU4Mzk5OTk5OTk5OTk5NiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny41ODQxLCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU4NDIsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41ODQzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTg0MywgLTEyMi4zODUsIDEuMF0sIFs0Ny41ODQ1LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjU4NDYsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTg0NiwgLTEyMi4zODMsIDEuMF0sIFs0Ny41ODQ4LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU4NDgsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTg0OSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41ODUyLCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjU4NTIsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41ODUzLCAtMTIyLjM5LCAxLjBdLCBbNDcuNTg1NSwgLTEyMi4zODMsIDEuMF0sIFs0Ny41ODU4LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTg1OCwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU4NTksIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTg2MDAwMDAwMDAwMDEsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNTg2MSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41ODYyLCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU4NjUsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTg2NiwgLTEyMi4zODksIDEuMF0sIFs0Ny41ODY5OTk5OTk5OTk5OTYsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTg3MiwgLTEyMi4zOSwgMS4wXSwgWzQ3LjU4NzQsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNTg3NCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU4NzUsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTg3NiwgLTEyMi4zOTEsIDEuMF0sIFs0Ny41ODc4LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjU4ODMsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNTg4OCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU4ODksIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTg5LCAtMTIyLjM4NywgMi4wXSwgWzQ3LjU4OTEsIC0xMjIuMzg3LCAzLjBdLCBbNDcuNTg5MiwgLTEyMi4zODcsIDEuMF0sIFs0Ny41ODkyLCAtMTIyLjM4MywgMS4wXSwgWzQ3LjU4OTUsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNTg5NiwgLTEyMi4zODksIDEuMF0sIFs0Ny41OTAxLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkwMiwgLTEyMi4zODcsIDEuMF0sIFs0Ny41OTAzLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkwMywgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU5MDQsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNTkwOSwgLTEyMi4zODQsIDEuMF0sIFs0Ny41OTE0LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5MTUsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTkxNiwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5MTcsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41OTE3LCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjU5MTcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTkxNywgLTEyMi4yOSwgMS4wXSwgWzQ3LjU5MTgsIC0xMjIuMjk1LCAyLjBdLCBbNDcuNTkxOSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU5MiwgLTEyMi4yOTUsIDEuMF0sIFs0Ny41OTIsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTIyLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTkyMywgLTEyMi4zMDEsIDEuMF0sIFs0Ny41OTIzLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkyMywgLTEyMi4yOTYsIDEuMF0sIFs0Ny41OTI0LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjU5MjQsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTI0LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5MjQsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNTkyNCwgLTEyMi4yODcsIDEuMF0sIFs0Ny41OTI1LCAtMTIyLjMwMiwgMi4wXSwgWzQ3LjU5MjUsIC0xMjIuMjk1LCAzLjBdLCBbNDcuNTkyNSwgLTEyMi4yODcsIDEuMF0sIFs0Ny41OTI2LCAtMTIyLjMsIDEuMF0sIFs0Ny41OTI2LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkyNiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41OTI2LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5MjYsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTI4LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5MjgsIC0xMjIuMjksIDEuMF0sIFs0Ny41OTI5LCAtMTIyLjI5LCAxLjBdLCBbNDcuNTkzLCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5MywgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5MywgLTEyMi4yOTEsIDEuMF0sIFs0Ny41OTMsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNTkzMywgLTEyMi4yOTIsIDEuMF0sIFs0Ny41OTM0LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkzNiwgLTEyMi4zMDEsIDEuMF0sIFs0Ny41OTM3LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjU5MzksIC0xMjIuMzAyLCAxLjBdLCBbNDcuNTkzOSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5NDEsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTQyLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTk0MiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41OTQzLCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjU5NDMsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTQzLCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjU5NDQsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTQ1LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjU5NDcsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNTk0OSwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41OTQ5LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjU5NTEsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNTk1MywgLTEyMi4yOTQsIDEuMF0sIFs0Ny41OTU1LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjU5NTgsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTU5LCAtMTIyLjMsIDEuMF0sIFs0Ny41OTU5LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTk1OSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5NTksIC0xMjIuMjkxLCAxLjBdLCBbNDcuNTk2MDAwMDAwMDAwMDA0LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTk2MiwgLTEyMi4zMDEsIDEuMF0sIFs0Ny41OTYyLCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU5NjMsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNTk2NCwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5NjgsIC0xMjIuMjksIDEuMF0sIFs0Ny41OTY5LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTk3LCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjU5NzEsIC0xMjIuMjk1LCAyLjBdLCBbNDcuNTk3MiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41OTcyLCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5NzIsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNTk3MywgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5NzYsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTc2LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU5NzgsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTc4LCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjU5NzksIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41OTgxLCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU5ODIsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41OTg1LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNTk4NiwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5OTQsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNTk5NiwgLTEyMi4yODgsIDEuMF0sIFs0Ny41OTk4LCAtMTIyLjMsIDEuMF0sIFs0Ny41OTk5LCAtMTIyLjMsIDIuMF0sIFs0Ny41OTk5LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MDAxLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjAwNCwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwMDQsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjAwNSwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42MDEwMDAwMDAwMDAwMDYsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNjAxMiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42MDEyLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYwMTMsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MDE4LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjAxOCwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYwMTgsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjAxOSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYwMjEsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjAyMSwgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MDI0LCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjYwMjQsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjAyNSwgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MDI1LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYwMjcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjAyOCwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwMjksIC0xMjIuMzAyLCAxLjBdLCBbNDcuNjAzLCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYwMzEsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNjAzMiwgLTEyMi4yODUsIDEuMF0sIFs0Ny42MDMzLCAtMTIyLjMsIDEuMF0sIFs0Ny42MDMzLCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjYwMzMsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjAzNCwgLTEyMi4yODksIDEuMF0sIFs0Ny42MDM1LCAtMTIyLjI4NSwgMi4wXSwgWzQ3LjYwMzYsIC0xMjIuMzA1LCAxLjBdLCBbNDcuNjAzNywgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MDM3LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjYwMzgsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjAzOCwgLTEyMi4zMDMsIDEuMF0sIFs0Ny42MDM4LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjYwMzksIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjAzOSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNCwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MDQsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjA0LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYwNCwgLTEyMi4yODgsIDEuMF0sIFs0Ny42MDQxLCAtMTIyLjMxLCAxLjBdLCBbNDcuNjA0MiwgLTEyMi4zMDMsIDEuMF0sIFs0Ny42MDQ0LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjA0NCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNDUsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjA0NiwgLTEyMi4zLCAxLjBdLCBbNDcuNjA0NywgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MDQ4LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjA0OCwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNDksIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjA1LCAtMTIyLjMwNCwgMi4wXSwgWzQ3LjYwNSwgLTEyMi4zLCAxLjBdLCBbNDcuNjA1MSwgLTEyMi4zMTksIDEuMF0sIFs0Ny42MDUxLCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjYwNTEsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjA1MSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNTEsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjA1MiwgLTEyMi4zLCAxLjBdLCBbNDcuNjA1MywgLTEyMi4zMDYsIDEuMF0sIFs0Ny42MDUzLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYwNTQsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjA1NCwgLTEyMi4yOTUsIDEuMF0sIFs0Ny42MDU1LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjYwNTUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjA1NiwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MDU2LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjYwNTgsIC0xMjIuMzE5LCAxLjBdLCBbNDcuNjA1OCwgLTEyMi4zMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNTgsIC0xMjIuMywgMS4wXSwgWzQ3LjYwNTksIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MDU5LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjA1OSwgLTEyMi4zMDcsIDEuMF0sIFs0Ny42MDU5LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYwNTksIC0xMjIuMywgMS4wXSwgWzQ3LjYwNTksIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjA2LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjYwNiwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNjEsIC0xMjIuMjg2LCAxLjBdLCBbNDcuNjA2MywgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MDYzLCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjYwNjUsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjA2NiwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MDY3LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjA3LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjYwNywgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYwNzEsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjA3MywgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNzMsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjA3NCwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MDc0LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjA3NCwgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MDc1LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjYwNzYsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjA3NiwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwNzgsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjA3OCwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42MDc4LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYwODEsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjA4MiwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwODQsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjA4NSwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MDg2LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYwODYsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNjA4OCwgLTEyMi4zMTEsIDIuMF0sIFs0Ny42MDg4LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjYwODgsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNjA4OCwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwODksIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjA4OTk5OTk5OTk5OTk1LCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjYwODk5OTk5OTk5OTk5NSwgLTEyMi4zMDcsIDEuMF0sIFs0Ny42MDkyLCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjYwOTMsIC0xMjIuMzEsIDEuMF0sIFs0Ny42MDkzLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjA5NCwgLTEyMi4zMSwgMS4wXSwgWzQ3LjYwOTQsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjA5NSwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42MDk3LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjYwOTgsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjA5OSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYwOTksIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxMDEsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjEwMSwgLTEyMi4yOTUsIDEuMF0sIFs0Ny42MTAxLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYxMDIsIC0xMjIuMzE0LCAyLjBdLCBbNDcuNjEwMiwgLTEyMi4zMDksIDEuMF0sIFs0Ny42MTAyLCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjEwMiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42MTAzLCAtMTIyLjMsIDEuMF0sIFs0Ny42MTAzLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjEwMywgLTEyMi4yOTYsIDEuMF0sIFs0Ny42MTA0LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYxMDUsIC0xMjIuMzA5LCAyLjBdLCBbNDcuNjEwNiwgLTEyMi4zMSwgMy4wXSwgWzQ3LjYxMDgsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTA4LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYxMDksIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjEwOSwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42MTA5LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjYxMTEsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTEyLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjYxMTIsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjExMiwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxMTMsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjExMywgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxMTMsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjExNCwgLTEyMi4zMSwgMS4wXSwgWzQ3LjYxMTUsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjExNSwgLTEyMi4yODcsIDEuMF0sIFs0Ny42MTE3LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjExNywgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MTE3LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjExOCwgLTEyMi4zMDYsIDEuMF0sIFs0Ny42MTE4LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjExOCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxMTk5OTk5OTk5OTk5NSwgLTEyMi4yOTUsIDEuMF0sIFs0Ny42MTE5OTk5OTk5OTk5OTUsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjEyMSwgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MTIyLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjEyMiwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxMjIsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjEyMywgLTEyMi4zMTQsIDEuMF0sIFs0Ny42MTI0LCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjYxMjQsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjEyNCwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxMjYsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTI3LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjEyNywgLTEyMi4yODYsIDEuMF0sIFs0Ny42MTMxLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjEzMiwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjYxMzIsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjEzMywgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxMzMsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjEzMywgLTEyMi4yODcsIDEuMF0sIFs0Ny42MTM0LCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjEzNCwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxMzUsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjEzNSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxMzYsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjEzNiwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxMzYsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MTM4LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjEzOCwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNCwgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MTQyLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE0MywgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MTQ0LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjYxNDQsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTQ1LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjYxNDUsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTQ2LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYxNDYsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTQ3LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYxNDcsIC0xMjIuMywgMi4wXSwgWzQ3LjYxNDcsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTQ3LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjYxNDgsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjE0OSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNDksIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjE1LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYxNTQsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjE1NCwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNTUsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjE1NSwgLTEyMi4zLCAxLjBdLCBbNDcuNjE1NSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYxNTYsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTU2LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjE1NiwgLTEyMi4yOTUsIDEuMF0sIFs0Ny42MTU3LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYxNTcsIC0xMjIuMjksIDEuMF0sIFs0Ny42MTU3LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjYxNTgsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTU4LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYxNTksIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjE2MDAwMDAwMDAwMDEsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjE2MDAwMDAwMDAwMDEsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MTYxLCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYxNjIsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjE2MywgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MTYzLCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE2MywgLTEyMi4yOTIsIDEuMF0sIFs0Ny42MTYzLCAtMTIyLjA1NiwgMS4wXSwgWzQ3LjYxNjYsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjE2NiwgLTEyMi4yODcsIDEuMF0sIFs0Ny42MTY3LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjE2OCwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42MTY4LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYxNjgsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjE2OCwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNjgsIC0xMjIuMDQ1LCAxLjBdLCBbNDcuNjE2OSwgLTEyMi4zMDksIDEuMF0sIFs0Ny42MTY5LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjYxNjksIC0xMjIuMDQ1LCAxLjBdLCBbNDcuNjE3LCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjYxNywgLTEyMi4yOTEsIDEuMF0sIFs0Ny42MTcsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjE3LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYxNywgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxNywgLTEyMi4wNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNywgLTEyMi4wNTEsIDIuMF0sIFs0Ny42MTcxLCAtMTIyLjA0NCwgMS4wXSwgWzQ3LjYxNzIsIC0xMjIuMzEsIDEuMF0sIFs0Ny42MTcyLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE3MiwgLTEyMi4wNjEsIDEuMF0sIFs0Ny42MTczLCAtMTIyLjMxLCAxLjBdLCBbNDcuNjE3MywgLTEyMi4wNTYsIDEuMF0sIFs0Ny42MTc0LCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE3NSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MTc1LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE3NSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxNzUsIC0xMjIuMDU2LCAxLjBdLCBbNDcuNjE3NiwgLTEyMi4wNDUsIDEuMF0sIFs0Ny42MTc2LCAtMTIyLjA0NCwgMS4wXSwgWzQ3LjYxNzcsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTc4LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE3OCwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42MTc4LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjE3OCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNzgsIC0xMjIuMDU1LCAxLjBdLCBbNDcuNjE3OSwgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MTc5LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjE3OSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MTgsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjE4MSwgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MTgxLCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYxODEsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjE4MSwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxODEsIC0xMjIuMDU2LCAyLjBdLCBbNDcuNjE4MiwgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxODIsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNjE4MywgLTEyMi4zLCAxLjBdLCBbNDcuNjE4MywgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxODQsIC0xMjIuMzAxLCAyLjBdLCBbNDcuNjE4NSwgLTEyMi4yOTUsIDEuMF0sIFs0Ny42MTg2LCAtMTIyLjI4OCwgMi4wXSwgWzQ3LjYxODcsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjE4NywgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxODcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjE4NywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxODcsIC0xMjIuMDQ0LCAxLjBdLCBbNDcuNjE4NywgLTEyMi4wNCwgMS4wXSwgWzQ3LjYxODgsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTg4LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYxODgsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjE4OSwgLTEyMi4yODgsIDEuMF0sIFs0Ny42MTg5LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYxOSwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MTksIC0xMjIuMDU0LCAxLjBdLCBbNDcuNjE5MSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MTkxLCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjYxOTEsIC0xMjIuMDQyOTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42MTkyLCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYxOTIsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNjE5MiwgLTEyMi4zMDcsIDEuMF0sIFs0Ny42MTkyLCAtMTIyLjMwMSwgMi4wXSwgWzQ3LjYxOTIsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTkyLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjE5MiwgLTEyMi4wNTUsIDEuMF0sIFs0Ny42MTkzLCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjYxOTQsIC0xMjIuMDQyLCAxLjBdLCBbNDcuNjE5NCwgLTEyMi4wNDEsIDEuMF0sIFs0Ny42MTk1LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE5NSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxOTYsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTk2LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjE5NiwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42MTk2LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYxOTcsIC0xMjIuMywgMS4wXSwgWzQ3LjYxOTcsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjE5OCwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxOTgsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjE5OCwgLTEyMi4wNTMsIDEuMF0sIFs0Ny42MTk4LCAtMTIyLjA0NCwgMS4wXSwgWzQ3LjYxOTgsIC0xMjIuMDM4LCAxLjBdLCBbNDcuNjE5OSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MTk5LCAtMTIyLjMsIDIuMF0sIFs0Ny42MiwgLTEyMi4zMDksIDEuMF0sIFs0Ny42MiwgLTEyMi4zLCAxLjBdLCBbNDcuNjIsIC0xMjIuMDUyLCAxLjBdLCBbNDcuNjIsIC0xMjIuMDQyOTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42MjAxLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjYyMDQsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNjIwNCwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42MjA0LCAtMTIyLjA0NCwgMS4wXSwgWzQ3LjYyMDUsIC0xMjIuMywgMS4wXSwgWzQ3LjYyMDUsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjA1LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjYyMDUsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjIwNiwgLTEyMi4zLCAxLjBdLCBbNDcuNjIwNiwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyMDYsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjA2LCAtMTIyLjA1NiwgMS4wXSwgWzQ3LjYyMDcsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjIwNywgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyMDcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjIwNywgLTEyMi4wNDIsIDEuMF0sIFs0Ny42MjA4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYyMDksIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjA5LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYyMDksIC0xMjIuMDUyLCAxLjBdLCBbNDcuNjIxLCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYyMSwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42MjEsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42MjEyLCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjIxMiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42MjEyLCAtMTIyLjA1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIxMiwgLTEyMi4wNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyMTIsIC0xMjIuMDM4LCAxLjBdLCBbNDcuNjIxNSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYyMTYsIC0xMjIuMDU2LCAxLjBdLCBbNDcuNjIxNywgLTEyMi4zMDksIDEuMF0sIFs0Ny42MjIsIC0xMjIuMywgMS4wXSwgWzQ3LjYyMiwgLTEyMi4wNTksIDEuMF0sIFs0Ny42MjIxLCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjYyMjEsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjIyMSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYyMjIsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjIyMywgLTEyMi4yOTQsIDEuMF0sIFs0Ny42MjIzLCAtMTIyLjA0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIyNCwgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyMjQsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjIyNCwgLTEyMi4wNDUsIDEuMF0sIFs0Ny42MjI1LCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjYyMjUsIC0xMjIuMDU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjI1LCAtMTIyLjAzOCwgMS4wXSwgWzQ3LjYyMjYsIC0xMjIuMzE5LCAxLjBdLCBbNDcuNjIyNiwgLTEyMi4zMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyMjYsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjIyNywgLTEyMi4zMjMsIDEuMF0sIFs0Ny42MjI3LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjIyOCwgLTEyMi4zMTksIDEuMF0sIFs0Ny42MjI4LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjYyMjksIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjMwMDAwMDAwMDAwMSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MjMxLCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIzMSwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42MjMxLCAtMTIyLjA0NCwgMS4wXSwgWzQ3LjYyMzEsIC0xMjIuMDQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjMyLCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIzMiwgLTEyMi4wNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyMzIsIC0xMjIuMDQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjMzLCAtMTIyLjMxOSwgMy4wXSwgWzQ3LjYyMzMsIC0xMjIuMywgMS4wXSwgWzQ3LjYyMzMsIC0xMjIuMDQ2LCAyLjBdLCBbNDcuNjIzNCwgLTEyMi4wNTksIDEuMF0sIFs0Ny42MjM1LCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjYyMzUsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjIzNSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyMzYsIC0xMjIuMzE4LCAxLjBdLCBbNDcuNjIzNiwgLTEyMi4zMDYsIDIuMF0sIFs0Ny42MjM2LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIzNiwgLTEyMi4wNCwgMS4wXSwgWzQ3LjYyMzgsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjIzOSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyMzksIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjM5LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjIzOTk5OTk5OTk5OTk1LCAtMTIyLjA1LCAxLjBdLCBbNDcuNjIzOTk5OTk5OTk5OTk1LCAtMTIyLjA0Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI0MSwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MjQyLCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYyNDIsIC0xMjIuMDQsIDEuMF0sIFs0Ny42MjQ0LCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI0NSwgLTEyMi4zMTYsIDEuMF0sIFs0Ny42MjQ2LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI0NywgLTEyMi4wNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNDgsIC0xMjIuMzE5LCAxLjBdLCBbNDcuNjI0OCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MjQ4LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjI0OSwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MjQ5LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI1LCAtMTIyLjMsIDEuMF0sIFs0Ny42MjUsIC0xMjIuMDQyLCAxLjBdLCBbNDcuNjI1MSwgLTEyMi4zLCAxLjBdLCBbNDcuNjI1MywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNTQsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjI1NCwgLTEyMi4wNSwgMS4wXSwgWzQ3LjYyNTQsIC0xMjIuMDQ1LCAxLjBdLCBbNDcuNjI1NSwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42MjU1LCAtMTIyLjA1OSwgMS4wXSwgWzQ3LjYyNTYsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjU2LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI1NiwgLTEyMi4wNTksIDEuMF0sIFs0Ny42MjU2LCAtMTIyLjA0MiwgMS4wXSwgWzQ3LjYyNTgsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjU4LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjYyNTgsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNjI1OCwgLTEyMi4wNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNTgsIC0xMjIuMDM4LCAxLjBdLCBbNDcuNjI1OSwgLTEyMi4zMjEsIDEuMF0sIFs0Ny42MjU5LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYyNTksIC0xMjIuMDUxLCAxLjBdLCBbNDcuNjI2MDAwMDAwMDAwMDEsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjI2MDAwMDAwMDAwMDEsIC0xMjIuMDQ3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjYxLCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjYyNjEsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNjI2MSwgLTEyMi4wNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyNjIsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjI2MiwgLTEyMi4wNDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNjIsIC0xMjIuMDQ1LCAxLjBdLCBbNDcuNjI2MywgLTEyMi4zMTQsIDIuMF0sIFs0Ny42MjYzLCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI2MywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNjQsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjI2NCwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42MjY0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYyNjQsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjY0LCAtMTIyLjA1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI2NSwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42MjY1LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI2NSwgLTEyMi4zLCAxLjBdLCBbNDcuNjI2NSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNjUsIC0xMjIuMjk2LCAyLjBdLCBbNDcuNjI2NSwgLTEyMi4yOTUsIDEuMF0sIFs0Ny42MjY3LCAtMTIyLjA0NiwgMS4wXSwgWzQ3LjYyNjgsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjY4LCAtMTIyLjA1LCAxLjBdLCBbNDcuNjI2OCwgLTEyMi4wMzgsIDEuMF0sIFs0Ny42MjY5LCAtMTIyLjA2LCAxLjBdLCBbNDcuNjI2OTk5OTk5OTk5OTk1LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjYyNjk5OTk5OTk5OTk5NSwgLTEyMi4zMDMsIDEuMF0sIFs0Ny42MjcxLCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjYyNzIsIC0xMjIuMzE4LCAxLjBdLCBbNDcuNjI3MiwgLTEyMi4zMTYsIDEuMF0sIFs0Ny42MjcyLCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI3MiwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyNzIsIC0xMjIuMjksIDEuMF0sIFs0Ny42MjczLCAtMTIyLjA1OSwgMS4wXSwgWzQ3LjYyNzQsIC0xMjIuMDQsIDEuMF0sIFs0Ny42Mjc1LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjYyNzUsIC0xMjIuMzE1LCAxLjBdLCBbNDcuNjI3NSwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNzUsIC0xMjIuMzA1LCAxLjBdLCBbNDcuNjI3NiwgLTEyMi4wNTMsIDIuMF0sIFs0Ny42Mjc3LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjYyNzgsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjI3OSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42Mjc5LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjYyOCwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyODIsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjI4MiwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyODMsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjgzLCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYyODQsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mjg0LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYyODUsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mjg1LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjYyODUsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjI4NSwgLTEyMi4zMDksIDEuMF0sIFs0Ny42Mjg1LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjYyODUsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjI4NSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42Mjg3LCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjYyODcsIC0xMjIuMywgMS4wXSwgWzQ3LjYyODcsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjI4OCwgLTEyMi4zMjEsIDEuMF0sIFs0Ny42Mjg4LCAtMTIyLjMsIDEuMF0sIFs0Ny42Mjg4LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI4OCwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyODksIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjI5MSwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42MjkxLCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjYyOTIsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjI5MiwgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyOTUsIC0xMjIuMzIsIDIuMF0sIFs0Ny42Mjk1LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYyOTUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjI5NiwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyOTYsIC0xMjIuMywgMS4wXSwgWzQ3LjYyOTgsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mjk4LCAtMTIyLjMyMywgMS4wXSwgWzQ3LjYyOTgsIC0xMjIuMzIxLCAxLjBdLCBbNDcuNjI5OCwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYyOTksIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjI5OSwgLTEyMi4yOCwgMS4wXSwgWzQ3LjYzLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzMDEsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjMwMSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MzAzLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMwNCwgLTEyMi4zMDksIDEuMF0sIFs0Ny42MzA0LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzMDUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMwNSwgLTEyMi4zLCAxLjBdLCBbNDcuNjMwNiwgLTEyMi4yODgsIDEuMF0sIFs0Ny42MzA3LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjYzMDcsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMwOCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzA4LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMwOCwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42MzA4LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjYzMDgsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjMxLCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYzMTEsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzEyLCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjYzMTIsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjMxMiwgLTEyMi4zMDcsIDEuMF0sIFs0Ny42MzEyLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYzMTMsIC0xMjIuMzcsIDEuMF0sIFs0Ny42MzEzLCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjYzMTMsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjMxMywgLTEyMi4zMDMsIDEuMF0sIFs0Ny42MzE0LCAtMTIyLjM1MywgMS4wXSwgWzQ3LjYzMTUsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzE1LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjYzMTUsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzE1LCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMxNiwgLTEyMi4zMDMsIDEuMF0sIFs0Ny42MzE2LCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMxNywgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMTcsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjMxNywgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MzE4LCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjYzMTgsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjMxOSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMTksIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjMyLCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjYzMiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42MzIsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzIsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjMyLCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjYzMjEsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzIxLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjYzMjEsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjMyMSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MzI0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYzMjUsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzI1LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYzMjUsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzI5LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjMzLCAtMTIyLjM3LCAxLjBdLCBbNDcuNjMzLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjMzLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMzMSwgLTEyMi4zMSwgMS4wXSwgWzQ3LjYzMzIsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzMyLCAtMTIyLjI5LCAxLjBdLCBbNDcuNjMzMiwgLTEyMi4yODEsIDEuMF0sIFs0Ny42MzMzLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMzNCwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMzQsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzM2LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjMzNywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMzcsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjMzNywgLTEyMi4yOCwgMS4wXSwgWzQ3LjYzMzgsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzM4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYzMzksIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMzOSwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMzk5OTk5OTk5OTk5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzNDEsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjM0MSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MzQxLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM0MiwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42MzQyLCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYzNDIsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzQzLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjYzNDMsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzQzLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM0NCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzQ0LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjYzNDUsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjM0NSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42MzQ1LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYzNDYsIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzQ3LCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjYzNDcsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjM0OCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MzQ4LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjYzNDksIC0xMjIuMzE4LCAxLjBdLCBbNDcuNjM0OSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYzNSwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42MzUsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzUxLCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1MSwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42MzUxLCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjYzNTIsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzUyLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjYzNTIsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzUyLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1MywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNTMsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjM1NCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42MzU1LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjYzNTYsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzU2LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjYzNTYsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNjM1NywgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNTcsIC0xMjIuMzI0LCAyLjBdLCBbNDcuNjM1NywgLTEyMi4zMjIsIDEuMF0sIFs0Ny42MzU4LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1OCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNTgsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjM1OSwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42MzU5LCAtMTIyLjMsIDEuMF0sIFs0Ny42MzYsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNjM2LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzNjEsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzYxLCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjYzNjEsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjM2MiwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzYyLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzNjIsIC0xMjIuMzI0LCAxLjBdLCBbNDcuNjM2MiwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42MzYyLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM2MywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjYzNjMsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjM2MywgLTEyMi4zMiwgMS4wXSwgWzQ3LjYzNjQsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzY0LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzNjQsIC0xMjIuMzUxLCAyLjBdLCBbNDcuNjM2NCwgLTEyMi4zLCAxLjBdLCBbNDcuNjM2NCwgLTEyMi4yOCwgMS4wXSwgWzQ3LjYzNjUsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjM2NiwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzY2LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjYzNjYsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjM2NiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNjcsIC0xMjIuMzcsIDEuMF0sIFs0Ny42MzY3LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM2NywgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MzY3LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzNjgsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzY4LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjM2OCwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42MzY4LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjYzNjgsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjM2OCwgLTEyMi4zMDYsIDEuMF0sIFs0Ny42MzY4LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjYzNjksIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjM2OSwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42MzcsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzcsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjM3LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzNywgLTEyMi4zNjEsIDEuMF0sIFs0Ny42MzcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjM3LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjYzNzEsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjM3MiwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42MzcyLCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjYzNzMsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM3MywgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNzMsIC0xMjIuMywgMS4wXSwgWzQ3LjYzNzQsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM3NCwgLTEyMi4zMjQsIDEuMF0sIFs0Ny42Mzc0LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjYzNzQsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjM3NCwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNzQsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjM3NCwgLTEyMi4yNzksIDEuMF0sIFs0Ny42Mzc1LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjYzNzYsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjM3NiwgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNzYsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjM3OCwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNzgsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjM3OSwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MzgwMDAwMDAwMDAwMSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzODAwMDAwMDAwMDAxLCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM4MDAwMDAwMDAwMDEsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjM4MDAwMDAwMDAwMDEsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzgwMDAwMDAwMDAwMSwgLTEyMi4yODgsIDEuMF0sIFs0Ny42MzgxLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjYzODEsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjM4MiwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42MzgyLCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjYzODIsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjM4NCwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzODQsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjM4NSwgLTEyMi4zNywgMS4wXSwgWzQ3LjYzODUsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNjM4NiwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42Mzg2LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjM4NywgLTEyMi4zNjMsIDEuMF0sIFs0Ny42Mzg3LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM4OCwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42Mzg4LCAtMTIyLjMsIDEuMF0sIFs0Ny42Mzg5LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM4OSwgLTEyMi4yODgsIDEuMF0sIFs0Ny42Mzg5OTk5OTk5OTk5OTYsIC0xMjIuMzEsIDEuMF0sIFs0Ny42MzkxLCAtMTIyLjM2LCAxLjBdLCBbNDcuNjM5MiwgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MzkzLCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM5MywgLTEyMi4zMTEsIDEuMF0sIFs0Ny42MzkzLCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM5MywgLTEyMi4zMDEsIDEuMF0sIFs0Ny42Mzk0LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjYzOTQsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Mzk0LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM5NCwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYzOTQsIC0xMjIuMjgsIDEuMF0sIFs0Ny42Mzk1LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjYzOTgsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mzk4LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYzOTksIC0xMjIuMzIsIDIuMF0sIFs0Ny42Mzk5LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY0LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjY0MDEsIC0xMjIuMzIsIDEuMF0sIFs0Ny42NDAxLCAtMTIyLjMwMywgMS4wXSwgWzQ3LjY0MDEsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDAyLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQwMiwgLTEyMi4zNywgMS4wXSwgWzQ3LjY0MDIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDAyLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjY0MDMsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDA0LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY0MDQsIC0xMjIuMzI1LCAxLjBdLCBbNDcuNjQwNSwgLTEyMi4zNjcsIDEuMF0sIFs0Ny42NDA1LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQwNSwgLTEyMi4zMjQsIDEuMF0sIFs0Ny42NDA1LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjY0MDYsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDA2LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjY0MDgsIC0xMjIuMzA3LCAyLjBdLCBbNDcuNjQwOCwgLTEyMi4zLCAxLjBdLCBbNDcuNjQwOSwgLTEyMi40MDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0MDksIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjQxMDAwMDAwMDAwMDEsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDExLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQxMywgLTEyMi40MDUsIDEuMF0sIFs0Ny42NDEzLCAtMTIyLjQwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQxMywgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MTMsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDEzLCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY0MTUsIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDE1LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQxNSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MTYsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjQxNiwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0MTYsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjQxNiwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42NDE3LCAtMTIyLjQxMSwgMS4wXSwgWzQ3LjY0MTcsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjQxOCwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MTgsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDE4LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQxOCwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42NDE5LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQxOSwgLTEyMi4zNzQsIDEuMF0sIFs0Ny42NDE5LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY0MTksIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjQxOTk5OTk5OTk5OTk2LCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY0MTk5OTk5OTk5OTk5NiwgLTEyMi4zNzQsIDEuMF0sIFs0Ny42NDE5OTk5OTk5OTk5OTYsIC0xMjIuMzYsIDEuMF0sIFs0Ny42NDE5OTk5OTk5OTk5OTYsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDIyLCAtMTIyLjQwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQyMiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42NDIyLCAtMTIyLjM2LCAxLjBdLCBbNDcuNjQyMywgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MjMsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjQyMywgLTEyMi4zNywgMS4wXSwgWzQ3LjY0MjMsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjQyNCwgLTEyMi40MTEsIDIuMF0sIFs0Ny42NDI0LCAtMTIyLjQwNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQyNCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0MjQsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjQyNSwgLTEyMi40MDYsIDEuMF0sIFs0Ny42NDI1LCAtMTIyLjM3NCwgMi4wXSwgWzQ3LjY0MjUsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDI2LCAtMTIyLjQwODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQyNiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MjYsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDI3LCAtMTIyLjQwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQyNywgLTEyMi4zOTcsIDEuMF0sIFs0Ny42NDI3LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY0MjgsIC0xMjIuNDExLCAxLjBdLCBbNDcuNjQyOSwgLTEyMi40MTIsIDEuMF0sIFs0Ny42NDI5LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQyOSwgLTEyMi4zNiwgMS4wXSwgWzQ3LjY0MywgLTEyMi40MTIsIDEuMF0sIFs0Ny42NDMsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjQzLCAtMTIyLjM3LCAxLjBdLCBbNDcuNjQzLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY0MywgLTEyMi4zNTksIDEuMF0sIFs0Ny42NDMsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjQzMSwgLTEyMi4zNTksIDEuMF0sIFs0Ny42NDMxLCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQzMywgLTEyMi40MDQsIDEuMF0sIFs0Ny42NDMzLCAtMTIyLjQwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQzNCwgLTEyMi40MTIsIDEuMF0sIFs0Ny42NDM0LCAtMTIyLjQwODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQzNCwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MzUsIC0xMjIuMzk5LCAxLjBdLCBbNDcuNjQzNSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MzUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjQzNSwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42NDM2LCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY0MzYsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjQzNywgLTEyMi4zOTYsIDEuMF0sIFs0Ny42NDM3LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQzOCwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MzgsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDM4LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjQ0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY0NCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42NDQsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjQ0MSwgLTEyMi4zODUsIDEuMF0sIFs0Ny42NDQyLCAtMTIyLjQwNiwgMS4wXSwgWzQ3LjY0NDIsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDQyLCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjY0NDMsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjQ0NCwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NDUsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjQ0NiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NDgsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDQ4LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ0OSwgLTEyMi4zODksIDEuMF0sIFs0Ny42NDUsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDUxLCAtMTIyLjM4NywgMS4wXSwgWzQ3LjY0NTEsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjQ1MiwgLTEyMi40MTEsIDEuMF0sIFs0Ny42NDUyLCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ1MiwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NTIsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjQ1MiwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42NDUzLCAtMTIyLjQxLCAxLjBdLCBbNDcuNjQ1MywgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NTMsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDUzLCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ1NCwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NTQsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjQ1NSwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NTUsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjQ1NiwgLTEyMi4zODksIDEuMF0sIFs0Ny42NDU2LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjY0NTYsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjQ1NywgLTEyMi40MTIsIDEuMF0sIFs0Ny42NDU3LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY0NiwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY0NiwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NDYsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjQ2LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ2MSwgLTEyMi4zOTcsIDEuMF0sIFs0Ny42NDYxLCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ2MSwgLTEyMi4zODksIDEuMF0sIFs0Ny42NDYxLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ2MSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NjEsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDYyLCAtMTIyLjQwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ2MiwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NDYyLCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjY0NjMsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDY0LCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY0NjQsIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDY0LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY0NjUsIC0xMjIuNDA0LCAxLjBdLCBbNDcuNjQ2NSwgLTEyMi4zOTUsIDEuMF0sIFs0Ny42NDY1LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ2NSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NjYsIC0xMjIuNDA0LCAxLjBdLCBbNDcuNjQ2NiwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42NDY2LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY0NjcsIC0xMjIuMzkzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDY5LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY0NywgLTEyMi40MSwgMS4wXSwgWzQ3LjY0NywgLTEyMi4zNjksIDEuMF0sIFs0Ny42NDcxLCAtMTIyLjQxMiwgMS4wXSwgWzQ3LjY0NzEsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjQ3MiwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NzIsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjQ3MiwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NzIsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjQ3MiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NzIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDczLCAtMTIyLjQwNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ3MywgLTEyMi40MDYsIDEuMF0sIFs0Ny42NDczLCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY0NzQsIC0xMjIuNDEyLCAxLjBdLCBbNDcuNjQ3NCwgLTEyMi40MTEsIDEuMF0sIFs0Ny42NDc0LCAtMTIyLjQwMiwgMS4wXSwgWzQ3LjY0NzQsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNjQ3NCwgLTEyMi4zODksIDEuMF0sIFs0Ny42NDc0LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ3NSwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NzUsIC0xMjIuMzk3LCAyLjBdLCBbNDcuNjQ3NSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny42NDc1LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ3NSwgLTEyMi4zNTksIDEuMF0sIFs0Ny42NDc2LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ3OCwgLTEyMi4zOTYsIDEuMF0sIFs0Ny42NDc4LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjY0NzksIC0xMjIuNDA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDc5LCAtMTIyLjM2NywgMi4wXSwgWzQ3LjY0OCwgLTEyMi40MSwgMS4wXSwgWzQ3LjY0OCwgLTEyMi4zOTcsIDEuMF0sIFs0Ny42NDgsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNjQ4LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY0OCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0OCwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NDgsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDgxLCAtMTIyLjQwNiwgMS4wXSwgWzQ3LjY0ODEsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjQ4MiwgLTEyMi40MTIsIDEuMF0sIFs0Ny42NDgyLCAtMTIyLjQwODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ4MiwgLTEyMi40MDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0ODIsIC0xMjIuNDAyLCAxLjBdLCBbNDcuNjQ4MiwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0ODIsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDgzLCAtMTIyLjQwNCwgMS4wXSwgWzQ3LjY0ODUsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjQ4NywgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0ODgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjQ4OSwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0ODk5OTk5OTk5OTk5NCwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0ODk5OTk5OTk5OTk5NCwgLTEyMi4zODMsIDEuMF0sIFs0Ny42NDkxLCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY0OTEsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjQ5MiwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42NDkzLCAtMTIyLjQxMywgMS4wXSwgWzQ3LjY0OTMsIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42NDkzLCAtMTIyLjM5OCwgMS4wXSwgWzQ3LjY0OTMsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjQ5NCwgLTEyMi4zOTUsIDEuMF0sIFs0Ny42NDk1LCAtMTIyLjQwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ5NiwgLTEyMi40MTMsIDEuMF0sIFs0Ny42NDk2LCAtMTIyLjQwNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ5NiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0OTYsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDk2LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY0OTcsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDk4LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjY0OTgsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDk5LCAtMTIyLjQxMywgMS4wXSwgWzQ3LjY0OTksIC0xMjIuMzkxLCAxLjBdLCBbNDcuNjQ5OSwgLTEyMi4zNywgMS4wXSwgWzQ3LjY1LCAtMTIyLjQxNSwgMS4wXSwgWzQ3LjY1MDEsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTAxLCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY1MDEsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjUwMSwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NTAxLCAtMTIyLjM3LCAxLjBdLCBbNDcuNjUwMywgLTEyMi40MSwgMi4wXSwgWzQ3LjY1MDMsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTAzLCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY1MDMsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjUwNSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY1MDUsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjUwNSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42NTA2LCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY1MDYsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNjUwNiwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NTA2LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjY1MDgsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjUwOCwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1MDgsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjUwOSwgLTEyMi4zNywgMS4wXSwgWzQ3LjY1MSwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1MSwgLTEyMi4zOTksIDEuMF0sIFs0Ny42NTEsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjUxLCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjY1MSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42NTExLCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY1MTEsIC0xMjIuNCwgMS4wXSwgWzQ3LjY1MTEsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjUxMywgLTEyMi40MDYsIDEuMF0sIFs0Ny42NTEzLCAtMTIyLjQsIDEuMF0sIFs0Ny42NTEzLCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUxMywgLTEyMi4zODUsIDEuMF0sIFs0Ny42NTEzLCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY1MTMsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjUxNCwgLTEyMi40MTMsIDEuMF0sIFs0Ny42NTE0LCAtMTIyLjQwNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUxNCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1MTQsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjUxNCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42NTE1LCAtMTIyLjQsIDEuMF0sIFs0Ny42NTE1LCAtMTIyLjM5OSwgMS4wXSwgWzQ3LjY1MTUsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjUxNSwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NTE1LCAtMTIyLjM3NSwgMi4wXSwgWzQ3LjY1MTYsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTE2LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY1MTcsIC0xMjIuNDA2LCAxLjBdLCBbNDcuNjUxNywgLTEyMi4zNzQsIDEuMF0sIFs0Ny42NTE4LCAtMTIyLjQsIDEuMF0sIFs0Ny42NTE4LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY1MTksIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjUyLCAtMTIyLjQwNCwgMS4wXSwgWzQ3LjY1MiwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NTIxLCAtMTIyLjQxMywgMS4wXSwgWzQ3LjY1MjEsIC0xMjIuNCwgMi4wXSwgWzQ3LjY1MjEsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNjUyMiwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42NTIzLCAtMTIyLjQxMiwgMS4wXSwgWzQ3LjY1MjQsIC0xMjIuNDA0LCAxLjBdLCBbNDcuNjUyNCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42NTI2LCAtMTIyLjM4NCwgMi4wXSwgWzQ3LjY1MjYsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTI3LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUyOCwgLTEyMi40MDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1MjgsIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTI5LCAtMTIyLjQxMSwgMS4wXSwgWzQ3LjY1MjksIC0xMjIuNDA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTI5LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY1MjksIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTI5LCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjY1MywgLTEyMi40MTYsIDEuMF0sIFs0Ny42NTMsIC0xMjIuNDE1LCAxLjBdLCBbNDcuNjUzLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY1MzEsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjUzMSwgLTEyMi40MDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1MzEsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTMxLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY1MzMsIC0xMjIuMzQ2LCAzLjBdLCBbNDcuNjUzMywgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1MzQsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTM0LCAtMTIyLjM1NSwgMi4wXSwgWzQ3LjY1MzQsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjUzNCwgLTEyMi4zNDYsIDIuMF0sIFs0Ny42NTM1LCAtMTIyLjM1MywgMS4wXSwgWzQ3LjY1MzYsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTM2LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUzNiwgLTEyMi4zNTQsIDQuMF0sIFs0Ny42NTM2LCAtMTIyLjM0MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUzNiwgLTEyMi4zNCwgMS4wXSwgWzQ3LjY1MzcsIC0xMjIuMzQsIDEuMF0sIFs0Ny42NTM3LCAtMTIyLjMzNSwgMS4wXSwgWzQ3LjY1MzgsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTM4LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUzOCwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42NTM5LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY1MzksIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjUzOSwgLTEyMi4zMjksIDEuMF0sIFs0Ny42NTM5OTk5OTk5OTk5OTYsIC0xMjIuMzUsIDEuMF0sIFs0Ny42NTM5OTk5OTk5OTk5OTYsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjUzOTk5OTk5OTk5OTk2LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjUzOTk5OTk5OTk5OTk2LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjY1NDEsIC0xMjIuMzI5LCAxLjBdLCBbNDcuNjU0MiwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1NDIsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjU0MiwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42NTQzLCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjU0MywgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1NDMsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTQzLCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY1NDMsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTQzLCAtMTIyLjM0NSwgMS4wXSwgWzQ3LjY1NDQsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTQ1LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY1NDUsIC0xMjIuMzQ0LCAxLjBdLCBbNDcuNjU0NSwgLTEyMi4zMzYsIDEuMF0sIFs0Ny42NTQ1LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY1NDYsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTQ2LCAtMTIyLjM0NCwgMS4wXSwgWzQ3LjY1NDYsIC0xMjIuMzM3LCAxLjBdLCBbNDcuNjU0NywgLTEyMi4zNCwgMS4wXSwgWzQ3LjY1NDksIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjU0OSwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42NTUsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNjU1LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjU1MSwgLTEyMi4zNiwgMS4wXSwgWzQ3LjY1NTEsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjU1MSwgLTEyMi4zNDQsIDEuMF0sIFs0Ny42NTU0LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY1NTYsIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjU1NywgLTEyMi4zNDgsIDEuMF0sIFs0Ny42NTU3LCAtMTIyLjM0NCwgMS4wXSwgWzQ3LjY1NTcsIC0xMjIuMzI3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTU4LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY1NTgsIC0xMjIuMzM0LCAxLjBdLCBbNDcuNjU1OCwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42NTU5LCAtMTIyLjM0NCwgMS4wXSwgWzQ3LjY1NTksIC0xMjIuMzM5LCAxLjBdLCBbNDcuNjU1OSwgLTEyMi4zMzgsIDEuMF0sIFs0Ny42NTYwMDAwMDAwMDAwMSwgLTEyMi4zNTksIDEuMF0sIFs0Ny42NTYxLCAtMTIyLjM0MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjU2MSwgLTEyMi4zMzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1NjIsIC0xMjIuMzU0LCAyLjBdLCBbNDcuNjU2MywgLTEyMi4zNTMsIDEuMF0sIFs0Ny42NTYzLCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjY1NjUsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjU2NSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42NTY2LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjU2NiwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1NjcsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTY3LCAtMTIyLjMyNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjU2OSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42NTY5LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY1NywgLTEyMi4zNDUsIDEuMF0sIFs0Ny42NTcyLCAtMTIyLjM0NiwgMi4wXSwgWzQ3LjY1NzIsIC0xMjIuMzM1LCAxLjBdLCBbNDcuNjU3MiwgLTEyMi4zMzMsIDEuMF0sIFs0Ny42NTc0LCAtMTIyLjM0NSwgMS4wXSwgWzQ3LjY1NzUsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjU3NiwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42NTgxLCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY1ODIsIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjU4MiwgLTEyMi4zMzYsIDEuMF0sIFs0Ny42NTgzLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjU4MywgLTEyMi4zNTEsIDEuMF0sIFs0Ny42NTgzLCAtMTIyLjM0NCwgMS4wXSwgWzQ3LjY1ODQsIC0xMjIuMzUsIDEuMF0sIFs0Ny42NTg0LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY1ODUsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjU4NSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42NTg2LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjU4NywgLTEyMi4zNDQsIDEuMF0sIFs0Ny42NTg4LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjU4OCwgLTEyMi4zNDQsIDEuMF0sIFs0Ny42NTg5LCAtMTIyLjM0MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjU5LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY1OSwgLTEyMi4zMzksIDEuMF0sIFs0Ny42NTkyLCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjU5MywgLTEyMi4zMzcsIDEuMF0sIFs0Ny42NTkzLCAtMTIyLjMyNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjU5NCwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42NTk0LCAtMTIyLjM0NSwgMS4wXSwgWzQ3LjY1OTUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjU5NSwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42NTk1LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjY1OTYsIC0xMjIuMzM4LCAxLjBdLCBbNDcuNjU5NywgLTEyMi4zNTUsIDEuMF0sIFs0Ny42NTk3LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY1OTcsIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjU5NywgLTEyMi4zMzEsIDEuMF0sIFs0Ny42NTk4LCAtMTIyLjM1NSwgMi4wXSwgWzQ3LjY1OTgsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTk4LCAtMTIyLjM0OCwgMi4wXSwgWzQ3LjY1OTksIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjYwMSwgLTEyMi4zMzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2MDEsIC0xMjIuMzMxLCAxLjBdLCBbNDcuNjYwNCwgLTEyMi4zNTIsIDIuMF0sIFs0Ny42NjA1LCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjY2MDUsIC0xMjIuMzM1LCAxLjBdLCBbNDcuNjYwNSwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42NjA2LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY2MDYsIC0xMjIuMzMxLCAxLjBdLCBbNDcuNjYwNywgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2MDcsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjYwNywgLTEyMi4zNTIsIDEuMF0sIFs0Ny42NjA3LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY2MDgsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNjYwOCwgLTEyMi4zMzUsIDEuMF0sIFs0Ny42NjA4LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY2MDksIC0xMjIuMzQ0LCAxLjBdLCBbNDcuNjYwOSwgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY2MSwgLTEyMi4zNDQsIDEuMF0sIFs0Ny42NjEsIC0xMjIuMzM4LCAxLjBdLCBbNDcuNjYxMSwgLTEyMi4zNDYsIDEuMF0sIFs0Ny42NjEyLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY2MTYsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjE3LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjYxNywgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMy4wXSwgWzQ3LjY2MTgsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjYxOSwgLTEyMi4zNTIsIDIuMF0sIFs0Ny42NjE5LCAtMTIyLjM1MSwgMi4wXSwgWzQ3LjY2MTksIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NjIsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjYyLCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjYyLCAtMTIyLjMyOSwgMS4wXSwgWzQ3LjY2MjEsIC0xMjIuMzMsIDEuMF0sIFs0Ny42NjIzLCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjY2MjQsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjYyNCwgLTEyMi4zNTQsIDEuMF0sIFs0Ny42NjI0LCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjYyNCwgLTEyMi4zNDgsIDIuMF0sIFs0Ny42NjI0LCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjYyNCwgLTEyMi4zMzYsIDEuMF0sIFs0Ny42NjI0LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY2MjUsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjYyNiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42NjI2LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjY2MjcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjYyOSwgLTEyMi4zMjksIDEuMF0sIFs0Ny42NjMwMDAwMDAwMDAwMDQsIC0xMjIuMzM1LCAxLjBdLCBbNDcuNjYzMSwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42NjMxLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY2MzEsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjMxLCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY2MzEsIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjYzMSwgLTEyMi4zMzcsIDEuMF0sIFs0Ny42NjMzLCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjYzNCwgLTEyMi4zNTksIDEuMF0sIFs0Ny42NjM0LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjYzNCwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42NjM0LCAtMTIyLjM0LCAxLjBdLCBbNDcuNjYzNSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2MzYsIC0xMjIuMzMsIDEuMF0sIFs0Ny42NjM3LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY2MzgsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjM4LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY2MzgsIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjYzOCwgLTEyMi4zMjksIDEuMF0sIFs0Ny42NjM5LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY0MSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2NDIsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjQ1LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjY0NSwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY2NDUsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjQ1LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY0NiwgLTEyMi4zNjcsIDIuMF0sIFs0Ny42NjQ2LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY2NDYsIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NjQ2LCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjY2NDcsIC0xMjIuMzM3LCAxLjBdLCBbNDcuNjY0OCwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42NjQ4LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY0OCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42NjQ4LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY0OSwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42NjQ5LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjY2NTEsIC0xMjIuMzY4LCAzLjBdLCBbNDcuNjY1MiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2NTIsIC0xMjIuMzM4LCA0LjBdLCBbNDcuNjY1MywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2NTMsIC0xMjIuMzM4LCAxLjBdLCBbNDcuNjY1MywgLTEyMi4zMzMsIDEuMF0sIFs0Ny42NjU0LCAtMTIyLjM1NSwgMi4wXSwgWzQ3LjY2NTUsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjU1LCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjY2NTUsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjY1NSwgLTEyMi4zNCwgMS4wXSwgWzQ3LjY2NTYsIC0xMjIuMzM1LCAxLjBdLCBbNDcuNjY1OCwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42NjU5LCAtMTIyLjM2OSwgMi4wXSwgWzQ3LjY2NTksIC0xMjIuMzU5LCAxLjBdLCBbNDcuNjY1OSwgLTEyMi4yODcsIDEuMF0sIFs0Ny42NjYwMDAwMDAwMDAwMDQsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjY2MDAwMDAwMDAwMDA0LCAtMTIyLjM1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY2MDAwMDAwMDAwMDA0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjY2NjEsIC0xMjIuMzcsIDEuMF0sIFs0Ny42NjYyLCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY2NjIsIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NjYyLCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY2MywgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2NjMsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjY1LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjY2NjYsIC0xMjIuMzcsIDEuMF0sIFs0Ny42NjY2LCAtMTIyLjM2NywgMi4wXSwgWzQ3LjY2NjYsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjY2NiwgLTEyMi4yODUsIDEuMF0sIFs0Ny42NjY2LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjY2NjcsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjY2NywgLTEyMi4zNiwgMS4wXSwgWzQ3LjY2NjcsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjY4LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY2OCwgLTEyMi4yODUsIDEuMF0sIFs0Ny42NjcsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjY3MywgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2NzQsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjY3NCwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY2NzQsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjY3NSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42Njc1LCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjY2NzUsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjY3NywgLTEyMi4zMjQsIDEuMF0sIFs0Ny42Njc3LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjY2NzcsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNjY3OCwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42NjgxLCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY2ODEsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjgxLCAtMTIyLjM2LCAxLjBdLCBbNDcuNjY4MSwgLTEyMi4zNTUsIDIuMF0sIFs0Ny42NjgxLCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjY2ODEsIC0xMjIuMzIsIDEuMF0sIFs0Ny42NjgxLCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY2ODIsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjY4MywgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2ODMsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjY4MywgLTEyMi4yODYsIDEuMF0sIFs0Ny42Njg0LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY2ODQsIC0xMjIuMzY1LCAyLjBdLCBbNDcuNjY4NCwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42Njg0LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY4NCwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42Njg0LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY2ODQsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNjY4NCwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42Njg0LCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjY2ODUsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjY4NSwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42Njg3LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjY2ODgsIC0xMjIuMzI0LCAxLjBdLCBbNDcuNjY4OCwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42Njg4LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjY4OCwgLTEyMi4zMDksIDEuMF0sIFs0Ny42Njg4LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjY2ODksIC0xMjIuMzcsIDEuMF0sIFs0Ny42Njg5LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjY2ODksIC0xMjIuMzYzLCAzLjBdLCBbNDcuNjY4OSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2ODksIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Njg5LCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjY2OSwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42NjksIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjkxLCAtMTIyLjM2LCAxLjBdLCBbNDcuNjY5MSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY2OTEsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNjY5MSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjY2OTIsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjkyLCAtMTIyLjM5LCAxLjBdLCBbNDcuNjY5MiwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjY2OTIsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNjY5MiwgLTEyMi4zMjUsIDEuMF0sIFs0Ny42NjkyLCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY5MiwgLTEyMi4yNzksIDEuMF0sIFs0Ny42NjkzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY5MywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2OTMsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjY5MywgLTEyMi4yODksIDEuMF0sIFs0Ny42Njk0LCAtMTIyLjMsIDEuMF0sIFs0Ny42Njk0LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjY5NCwgLTEyMi4yNzksIDEuMF0sIFs0Ny42Njk1LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjY5NSwgLTEyMi4zMiwgMS4wXSwgWzQ3LjY2OTYsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Njk2LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY5NiwgLTEyMi4zMjQsIDIuMF0sIFs0Ny42Njk2LCAtMTIyLjMyLCAxLjBdLCBbNDcuNjY5NywgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY2OTcsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Njk3LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY5NywgLTEyMi4yOSwgMS4wXSwgWzQ3LjY2OTcsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNjY5OCwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42Njk4LCAtMTIyLjMyNSwgMi4wXSwgWzQ3LjY2OTgsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjY5OCwgLTEyMi4zLCAxLjBdLCBbNDcuNjY5OSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY2OTksIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjY5OSwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NywgLTEyMi4zOTEsIDEuMF0sIFs0Ny42NywgLTEyMi4zODksIDEuMF0sIFs0Ny42NywgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjY3LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcwMSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny42NzAxLCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjY3MDEsIC0xMjIuMjg2LCAxLjBdLCBbNDcuNjcwMiwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MDIsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzAyLCAtMTIyLjMsIDEuMF0sIFs0Ny42NzA0LCAtMTIyLjM5MSwgMi4wXSwgWzQ3LjY3MDQsIC0xMjIuMzksIDEuMF0sIFs0Ny42NzA0LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjcwNCwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MDUsIC0xMjIuMzksIDIuMF0sIFs0Ny42NzA2LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjY3MDYsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzA2LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjcwNiwgLTEyMi4zLCAxLjBdLCBbNDcuNjcwNywgLTEyMi4zODEsIDEuMF0sIFs0Ny42NzA3LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcwNywgLTEyMi4yODgsIDEuMF0sIFs0Ny42NzA4LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjcwOSwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MTEsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNjcxMSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY3MTEsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzExLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcxMSwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MTEsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzExLCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcxMSwgLTEyMi4zMjQsIDEuMF0sIFs0Ny42NzExLCAtMTIyLjMxNSwgMS4wXSwgWzQ3LjY3MTIsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzEyLCAtMTIyLjM5LCAxLjBdLCBbNDcuNjcxMiwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY3MTIsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzEyLCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjY3MTMsIC0xMjIuMzgzLCAyLjBdLCBbNDcuNjcxMywgLTEyMi4zLCAxLjBdLCBbNDcuNjcxNCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MTQsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNjcxNCwgLTEyMi4zMiwgMS4wXSwgWzQ3LjY3MTQsIC0xMjIuMjgsIDIuMF0sIFs0Ny42NzE1LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcxNSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MTUsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjcxNSwgLTEyMi4zODIsIDEuMF0sIFs0Ny42NzE1LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY3MTYsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjcxNywgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MTcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjcxOCwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MTgsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzE4LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjcxOCwgLTEyMi4zODIsIDEuMF0sIFs0Ny42NzE4LCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY3MTgsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjcxOSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny42NzE5LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjcxOSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY3MTksIC0xMjIuMzgyLCAyLjBdLCBbNDcuNjcxOSwgLTEyMi4zODEsIDEuMF0sIFs0Ny42NzE5LCAtMTIyLjM4LCAyLjBdLCBbNDcuNjcxOSwgLTEyMi4zNzQsIDEuMF0sIFs0Ny42NzE5LCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjY3MTksIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjcyLCAtMTIyLjM5NywgMS4wXSwgWzQ3LjY3MjEsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjcyMSwgLTEyMi4zODcsIDEuMF0sIFs0Ny42NzIxLCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY3MjEsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjcyMiwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42NzIyLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY3MjIsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjcyMiwgLTEyMi4zODEsIDEuMF0sIFs0Ny42NzIyLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcyMiwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MjQsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzI0LCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjY3MjQsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjcyNSwgLTEyMi4zOTYsIDIuMF0sIFs0Ny42NzI1LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcyNiwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY3MjYsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzI2LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjY3MjYsIC0xMjIuMzgsIDEuMF0sIFs0Ny42NzI2LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjcyNiwgLTEyMi4zMiwgMS4wXSwgWzQ3LjY3MjYsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzI2LCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjY3MjcsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjcyNywgLTEyMi4zOTYsIDEuMF0sIFs0Ny42NzI3LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY3MjcsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjcyNywgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MjcsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjcyNywgLTEyMi4zNTEsIDEuMF0sIFs0Ny42NzI3LCAtMTIyLjMyLCAxLjBdLCBbNDcuNjcyNywgLTEyMi4zMTYsIDEuMF0sIFs0Ny42NzI3LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcyNywgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MjcsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzI4LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjY3MjgsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzI4LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY3MjgsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzI4LCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjY3MjksIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjcyOSwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY3MjksIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzI5LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcyOSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MjksIC0xMjIuMzYzLCAyLjBdLCBbNDcuNjcyOSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MjksIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzMxLCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjczMSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42NzMyLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY3MzIsIC0xMjIuMjksIDEuMF0sIFs0Ny42NzMzLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY3MzMsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjczMywgLTEyMi4zNywgMS4wXSwgWzQ3LjY3MzMsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzMzLCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjY3MzMsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzMzLCAtMTIyLjI5NiwgMS4wXSwgWzQ3LjY3MzQsIC0xMjIuMzgsIDEuMF0sIFs0Ny42NzM1LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjczNSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42NzM1LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjczNiwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY3MzYsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjczNiwgLTEyMi4zMjMsIDIuMF0sIFs0Ny42NzM3LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY3MzcsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzM3LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY3MzcsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjczNywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3MzcsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzM4LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjczOCwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42NzM4LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjY3MzksIC0xMjIuMzk2LCAxLjBdLCBbNDcuNjczOSwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42NzQsIC0xMjIuMzg3LCAyLjBdLCBbNDcuNjc0LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjc0LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc0LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc0LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY3NDEsIC0xMjIuMzk2LCAxLjBdLCBbNDcuNjc0MSwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NzQxLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY3NDEsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjc0MSwgLTEyMi4zMjMsIDIuMF0sIFs0Ny42NzQyLCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjY3NDIsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzQyLCAtMTIyLjMwMywgMS4wXSwgWzQ3LjY3NDIsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzQzLCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY3NDMsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzQzLCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjY3NDMsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzQzLCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc0MywgLTEyMi4zMDEsIDEuMF0sIFs0Ny42NzQzLCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc0MywgLTEyMi4yODUsIDEuMF0sIFs0Ny42NzQ0LCAtMTIyLjM4MywgMi4wXSwgWzQ3LjY3NDQsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjc0NCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42NzQ0LCAtMTIyLjM1MywgMi4wXSwgWzQ3LjY3NDQsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjc0NSwgLTEyMi4zMSwgMS4wXSwgWzQ3LjY3NDUsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjc0NiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NDYsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzQ2LCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjY3NDYsIC0xMjIuMzIsIDEuMF0sIFs0Ny42NzQ2LCAtMTIyLjMxNSwgMS4wXSwgWzQ3LjY3NDYsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzQ3LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc0NywgLTEyMi4zOTEsIDEuMF0sIFs0Ny42NzQ3LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjY3NDcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjc0OCwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NDgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjc0OCwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NzQ4LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc0OCwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NDgsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzQ4LCAtMTIyLjM1MiwgMi4wXSwgWzQ3LjY3NDgsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjc0OSwgLTEyMi4zOTgsIDEuMF0sIFs0Ny42NzQ5LCAtMTIyLjMyMywgMS4wXSwgWzQ3LjY3NSwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY3NSwgLTEyMi4zODcsIDEuMF0sIFs0Ny42NzUsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzUsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjc1MSwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY3NTEsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42NzUxLCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc1MSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NTEsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjc1MiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42NzUyLCAtMTIyLjMsIDEuMF0sIFs0Ny42NzUyLCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjY3NTMsIC0xMjIuMzYzLCAyLjBdLCBbNDcuNjc1MywgLTEyMi4zNiwgMS4wXSwgWzQ3LjY3NTMsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjc1MywgLTEyMi4zMDQsIDEuMF0sIFs0Ny42NzU0LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY3NTQsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjc1NCwgLTEyMi4zNTksIDEuMF0sIFs0Ny42NzU0LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc1NCwgLTEyMi4zMjUsIDEuMF0sIFs0Ny42NzU0LCAtMTIyLjMxNSwgMS4wXSwgWzQ3LjY3NTQsIC0xMjIuMzA3LCAyLjBdLCBbNDcuNjc1NCwgLTEyMi4zLCAxLjBdLCBbNDcuNjc1NCwgLTEyMi4yOTQsIDEuMF0sIFs0Ny42NzU1LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjc1NSwgLTEyMi4zNjksIDEuMF0sIFs0Ny42NzU1LCAtMTIyLjM2NywgMi4wXSwgWzQ3LjY3NTUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjc1NiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42NzU2LCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc1NiwgLTEyMi4zMDYsIDEuMF0sIFs0Ny42NzU2LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY3NTYsIC0xMjIuMywgMS4wXSwgWzQ3LjY3NTYsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzU3LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY3NTcsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzU3LCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc1NywgLTEyMi4zMDksIDEuMF0sIFs0Ny42NzU4LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc1OCwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY3NTgsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjc1OCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42NzU5LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjY3NiwgLTEyMi4zMDksIDEuMF0sIFs0Ny42NzYsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzYsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjc2LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjY3NjEsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjc2MSwgLTEyMi4zLCAxLjBdLCBbNDcuNjc2MSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NjIsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzYyLCAtMTIyLjM2LCAxLjBdLCBbNDcuNjc2MiwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NjIsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjc2MiwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NjIsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjc2MiwgLTEyMi4yODUsIDEuMF0sIFs0Ny42NzYzLCAtMTIyLjM3LCAxLjBdLCBbNDcuNjc2MywgLTEyMi4zNTIsIDEuMF0sIFs0Ny42NzYzLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc2NCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NjQsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjc2NCwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42NzY0LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjY3NjUsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjc2NSwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NjUsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjc2NSwgLTEyMi4zODEsIDEuMF0sIFs0Ny42NzY1LCAtMTIyLjMyLCAyLjBdLCBbNDcuNjc2NSwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NjUsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNjc2NSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42NzY1LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjY3NjUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjc2NiwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42NzY3LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc2NywgLTEyMi4zOCwgMS4wXSwgWzQ3LjY3NjcsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjc2NywgLTEyMi4zNTEsIDEuMF0sIFs0Ny42NzY3LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY3NjcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjc2NywgLTEyMi4yODYsIDEuMF0sIFs0Ny42NzY4LCAtMTIyLjM5LCAxLjBdLCBbNDcuNjc2OCwgLTEyMi4zNjcsIDEuMF0sIFs0Ny42NzY4LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY3NjgsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzY4LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjc2OCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42NzY4LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjY3NjksIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjc2OSwgLTEyMi4zODMsIDEuMF0sIFs0Ny42NzY5LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjY3NjksIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzY5LCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjY3NjksIC0xMjIuMzYsIDEuMF0sIFs0Ny42NzY5LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjY3NywgLTEyMi4zMjUsIDEuMF0sIFs0Ny42NzcxLCAtMTIyLjM5OCwgMS4wXSwgWzQ3LjY3NzEsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjc3MSwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NzEsIC0xMjIuMzE5LCAyLjBdLCBbNDcuNjc3MiwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NzIsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjc3MiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42NzcyLCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjY3NzMsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNjc3MywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NzQsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Nzc0LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY3NzQsIC0xMjIuMzI1LCAyLjBdLCBbNDcuNjc3NCwgLTEyMi4zMjQsIDEuMF0sIFs0Ny42Nzc0LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc3NSwgLTEyMi4zOTgsIDEuMF0sIFs0Ny42Nzc1LCAtMTIyLjM5LCAxLjBdLCBbNDcuNjc3NSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3NzUsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjc3NSwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NzUsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjc3NiwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42Nzc2LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjc3NiwgLTEyMi4zMjQsIDEuMF0sIFs0Ny42Nzc2LCAtMTIyLjMxOCwgMi4wXSwgWzQ3LjY3NzcsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjc3NywgLTEyMi4zMDUsIDEuMF0sIFs0Ny42Nzc3LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjY3NzgsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjc3OCwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42Nzc5LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY3NzksIC0xMjIuMzc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Nzc5LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc3OSwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NzksIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjc3OSwgLTEyMi4yOTQsIDIuMF0sIFs0Ny42NzgwMDAwMDAwMDAwMDQsIC0xMjIuMzc1LCAyLjBdLCBbNDcuNjc4MDAwMDAwMDAwMDA0LCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc4MDAwMDAwMDAwMDA0LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY3ODAwMDAwMDAwMDAwNCwgLTEyMi4zNiwgMS4wXSwgWzQ3LjY3ODAwMDAwMDAwMDAwNCwgLTEyMi4zMjIsIDEuMF0sIFs0Ny42NzgwMDAwMDAwMDAwMDQsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjc4MSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny42NzgxLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY3ODEsIC0xMjIuMzc0LCAyLjBdLCBbNDcuNjc4MSwgLTEyMi4zNjksIDEuMF0sIFs0Ny42NzgxLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY3ODIsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjc4MiwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42NzgyLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY3ODIsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjc4MiwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42NzgyLCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjY3ODIsIC0xMjIuMywgMS4wXSwgWzQ3LjY3ODIsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzgyLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjY3ODMsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjc4MywgLTEyMi4zNzYsIDEuMF0sIFs0Ny42NzgzLCAtMTIyLjM2NiwgMi4wXSwgWzQ3LjY3ODMsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjc4MywgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3ODQsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Nzg0LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY3ODQsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjc4NCwgLTEyMi4yODUsIDEuMF0sIFs0Ny42Nzg1LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjY3ODUsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Nzg1LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc4NSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42Nzg1LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc4NSwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42Nzg2LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY3ODYsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Nzg2LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY3ODYsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Nzg2LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY3ODYsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjc4NiwgLTEyMi4zNiwgMS4wXSwgWzQ3LjY3ODYsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Nzg3LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY3ODcsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjc4NywgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3ODcsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjc4NywgLTEyMi4yODUsIDEuMF0sIFs0Ny42Nzg4LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc4OCwgLTEyMi4zNDYsIDEuMF0sIFs0Ny42Nzg4LCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjc4OSwgLTEyMi4zOTcsIDEuMF0sIFs0Ny42Nzg5LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY3ODksIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjc4OSwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42Nzg5LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjY3ODk5OTk5OTk5OTk5NSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjY3ODk5OTk5OTk5OTk5NSwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3ODk5OTk5OTk5OTk5NSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42Nzg5OTk5OTk5OTk5OTUsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Nzg5OTk5OTk5OTk5OTUsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjc4OTk5OTk5OTk5OTk1LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY3ODk5OTk5OTk5OTk5NSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42Nzg5OTk5OTk5OTk5OTUsIC0xMjIuMywgMS4wXSwgWzQ3LjY3ODk5OTk5OTk5OTk5NSwgLTEyMi4yODgsIDEuMF0sIFs0Ny42NzkxLCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY3OTEsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzkxLCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY3OTIsIC0xMjIuMzEsIDEuMF0sIFs0Ny42NzkyLCAtMTIyLjMwMywgMS4wXSwgWzQ3LjY3OTIsIC0xMjIuMjg3LCAyLjBdLCBbNDcuNjc5MywgLTEyMi4zOTcsIDEuMF0sIFs0Ny42NzkzLCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY3OTMsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzkzLCAtMTIyLjM4LCAxLjBdLCBbNDcuNjc5MywgLTEyMi4zMTksIDEuMF0sIFs0Ny42NzkzLCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjY3OTMsIC0xMjIuMywgMS4wXSwgWzQ3LjY3OTQsIC0xMjIuMzcsIDEuMF0sIFs0Ny42Nzk0LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc5NCwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42Nzk1LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc5NSwgLTEyMi4zNTQsIDEuMF0sIFs0Ny42Nzk1LCAtMTIyLjM1MywgMS4wXSwgWzQ3LjY3OTYsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Nzk2LCAtMTIyLjM5LCAxLjBdLCBbNDcuNjc5NiwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42Nzk2LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjY3OTcsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjc5NywgLTEyMi4zODQsIDEuMF0sIFs0Ny42Nzk3LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc5NywgLTEyMi4zNTksIDIuMF0sIFs0Ny42Nzk3LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc5NywgLTEyMi4yOTIsIDEuMF0sIFs0Ny42Nzk4LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY3OTgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjc5OCwgLTEyMi4zNTQsIDEuMF0sIFs0Ny42Nzk4LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY3OTgsIC0xMjIuMzQ4LCAxLjBdLCBbNDcuNjc5OCwgLTEyMi4yOTIsIDEuMF0sIFs0Ny42Nzk5LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY3OTksIC0xMjIuMzgxLCAxLjBdLCBbNDcuNjc5OSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42Nzk5LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY3OTksIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjc5OSwgLTEyMi4yNzksIDEuMF0sIFs0Ny42OCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42OCwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42OCwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42OCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MDEsIC0xMjIuMzgsIDEuMF0sIFs0Ny42ODAxLCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgwMSwgLTEyMi4zNTQsIDEuMF0sIFs0Ny42ODAxLCAtMTIyLjM0OCwgMy4wXSwgWzQ3LjY4MDEsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjgwMSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42ODAxLCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY4MDIsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODAyLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY4MDIsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODAyLCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjY4MDIsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODAyLCAtMTIyLjI5LCAxLjBdLCBbNDcuNjgwMywgLTEyMi4zODcsIDEuMF0sIFs0Ny42ODAzLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgwMywgLTEyMi4zMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MDMsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjgwMywgLTEyMi4yODcsIDEuMF0sIFs0Ny42ODA0LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgwNCwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MDQsIC0xMjIuMzE5LCAxLjBdLCBbNDcuNjgwNCwgLTEyMi4zMTYsIDEuMF0sIFs0Ny42ODA0LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY4MDQsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjgwNSwgLTEyMi4zODUsIDEuMF0sIFs0Ny42ODA1LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY4MDUsIC0xMjIuMzgsIDEuMF0sIFs0Ny42ODA1LCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjY4MDUsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjgwNSwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MDUsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjgwNSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42ODA2LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgwNiwgLTEyMi4zNiwgMi4wXSwgWzQ3LjY4MDYsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjgwNywgLTEyMi4zNjksIDEuMF0sIFs0Ny42ODA3LCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjY4MDcsIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODA3LCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjY4MDcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjgwOCwgLTEyMi4zODQsIDEuMF0sIFs0Ny42ODA4LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjY4MDgsIC0xMjIuMzgsIDEuMF0sIFs0Ny42ODA4LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjY4MDksIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjgwOSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42ODA5LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY4MDksIC0xMjIuMzA1LCAxLjBdLCBbNDcuNjgwOSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MDksIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjgwOSwgLTEyMi4yODYsIDEuMF0sIFs0Ny42ODEwMDAwMDAwMDAwMDQsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42ODEwMDAwMDAwMDAwMDQsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODEwMDAwMDAwMDAwMDQsIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODEwMDAwMDAwMDAwMDQsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjgxMDAwMDAwMDAwMDA0LCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY4MTAwMDAwMDAwMDAwNCwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42ODExLCAtMTIyLjM4MywgMS4wXSwgWzQ3LjY4MTEsIC0xMjIuMzE4LCAxLjBdLCBbNDcuNjgxMSwgLTEyMi4yODgsIDEuMF0sIFs0Ny42ODExLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgxMywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MTMsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjgxMywgLTEyMi4zNDYsIDEuMF0sIFs0Ny42ODEzLCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgxNCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MTQsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNjgxNCwgLTEyMi4yODgsIDEuMF0sIFs0Ny42ODE1LCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjY4MTUsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNjgxNSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny42ODE2LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY4MTYsIC0xMjIuMzEsIDEuMF0sIFs0Ny42ODE3LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjgxOCwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MTgsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjgxOSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MTksIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjgxOSwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42ODE5LCAtMTIyLjI4NywgMi4wXSwgWzQ3LjY4MTk5OTk5OTk5OTk5NSwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MTk5OTk5OTk5OTk5NSwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42ODE5OTk5OTk5OTk5OTUsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODE5OTk5OTk5OTk5OTUsIC0xMjIuMzQ4LCAyLjBdLCBbNDcuNjgxOTk5OTk5OTk5OTk1LCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgyMSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MjEsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODIxLCAtMTIyLjM4LCAxLjBdLCBbNDcuNjgyMSwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MjEsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODIxLCAtMTIyLjMxLCAxLjBdLCBbNDcuNjgyMSwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MjEsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODIyLCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgyMiwgLTEyMi4zMDksIDEuMF0sIFs0Ny42ODIzLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgyMywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MjMsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjgyMywgLTEyMi4zNjUsIDEuMF0sIFs0Ny42ODIzLCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgyMywgLTEyMi4zNDgsIDEuMF0sIFs0Ny42ODIzLCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgyMywgLTEyMi4zMjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MjMsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODI0LCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY4MjQsIC0xMjIuMzgsIDEuMF0sIFs0Ny42ODI0LCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgyNCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42ODI0LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjY4MjUsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNjgyNSwgLTEyMi4zNzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MjUsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjgyNiwgLTEyMi4zOTM5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjY4MjYsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODI2LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjY4MjYsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjgyNiwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MjcsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNjgyNywgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MjcsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjgyNywgLTEyMi4zMjUsIDEuMF0sIFs0Ny42ODI3LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjY4MjcsIC0xMjIuMzEsIDEuMF0sIFs0Ny42ODI3LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgyNywgLTEyMi4yODksIDEuMF0sIFs0Ny42ODI4LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY4MjgsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODI4LCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgyOCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42ODI4LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY4MjgsIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjgyOCwgLTEyMi4zMjksIDEuMF0sIFs0Ny42ODI4LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjY4MjksIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODI5LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgyOSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42ODMsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjgzMSwgLTEyMi4zODEsIDEuMF0sIFs0Ny42ODMxLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjgzMSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MzEsIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODMxLCAtMTIyLjMyOSwgMS4wXSwgWzQ3LjY4MzEsIC0xMjIuMzI1LCAxLjBdLCBbNDcuNjgzMiwgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MzIsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjgzMiwgLTEyMi4zMDQsIDIuMF0sIFs0Ny42ODMyLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjY4MzMsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNjgzMywgLTEyMi4zOTUsIDEuMF0sIFs0Ny42ODMzLCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgzNCwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42ODM0LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjY4MzQsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNjgzNSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MzUsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNjgzNSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MzUsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42ODM1LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgzNSwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MzYsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjgzNiwgLTEyMi4zODcsIDEuMF0sIFs0Ny42ODM3LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY4MzcsIC0xMjIuMzIxLCAxLjBdLCBbNDcuNjgzNywgLTEyMi4zMiwgMS4wXSwgWzQ3LjY4MzcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjgzNywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4MzgsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODM4LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgzOCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42ODM4LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY4MzgsIC0xMjIuMzMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODM4LCAtMTIyLjMyNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgzOCwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42ODM4LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjgzOSwgLTEyMi4zNzYsIDEuMF0sIFs0Ny42ODM5LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY4MzksIC0xMjIuMzQ1LCAxLjBdLCBbNDcuNjgzOSwgLTEyMi4yODEsIDEuMF0sIFs0Ny42ODQsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODQsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODQsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODQsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNjg0LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg0LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjY4NCwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NCwgLTEyMi4yODEsIDEuMF0sIFs0Ny42ODQxLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjY4NDEsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42ODQxLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjY4NDIsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42ODQyLCAtMTIyLjM4NywgMS4wXSwgWzQ3LjY4NDIsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjg0MiwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42ODQyLCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY4NDIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODQyLCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY4NDIsIC0xMjIuMzQ0LCAxLjBdLCBbNDcuNjg0MiwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDIsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjg0MywgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDMsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODQzLCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY4NDMsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODQ0LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NCwgLTEyMi4zODcsIDQuMF0sIFs0Ny42ODQ0LCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NCwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDQsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjg0NCwgLTEyMi4zMDIsIDEuMF0sIFs0Ny42ODQ0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjY4NDUsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjg0NSwgLTEyMi4zNTksIDEuMF0sIFs0Ny42ODQ1LCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42ODQ1LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjg0NiwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDYsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODQ2LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NiwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42ODQ2LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY4NDYsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODQ2LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY4NDYsIC0xMjIuMzQ1LCAyLjBdLCBbNDcuNjg0NiwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjY4NDYsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNjg0NiwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NDcsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNjg0NywgLTEyMi4zODEsIDEuMF0sIFs0Ny42ODQ3LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NywgLTEyMi4zNywgMS4wXSwgWzQ3LjY4NDcsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjg0NywgLTEyMi4zMTgsIDEuMF0sIFs0Ny42ODQ3LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjY4NDgsIC0xMjIuMzk4LCAxLjBdLCBbNDcuNjg0OCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDgsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNjg0OCwgLTEyMi4zODksIDEuMF0sIFs0Ny42ODQ4LCAtMTIyLjM4NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0OCwgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDksIC0xMjIuMzc2LCAxLjBdLCBbNDcuNjg0OSwgLTEyMi4zNzQsIDEuMF0sIFs0Ny42ODQ5LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjY4NDksIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjg0OSwgLTEyMi4zMzUsIDEuMF0sIFs0Ny42ODQ5LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY4NDksIC0xMjIuMzI3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODUsIC0xMjIuMzYzLCAyLjBdLCBbNDcuNjg1LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjg1LCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY4NTEsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjg1MSwgLTEyMi4zMjIsIDEuMF0sIFs0Ny42ODUyLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg1MiwgLTEyMi4zMjksIDEuMF0sIFs0Ny42ODUyLCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg1MiwgLTEyMi4zMTEsIDEuMF0sIFs0Ny42ODUzLCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjY4NTMsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODUzLCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY4NTMsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42ODUzLCAtMTIyLjM1LCAxLjBdLCBbNDcuNjg1MywgLTEyMi4zMzEsIDEuMF0sIFs0Ny42ODUzLCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY4NTMsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODUzLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjY4NTQsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODU0LCAtMTIyLjMzMSwgMS4wXSwgWzQ3LjY4NTQsIC0xMjIuMzI5LCAxLjBdLCBbNDcuNjg1NCwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NTQsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjg1NCwgLTEyMi4yODgsIDEuMF0sIFs0Ny42ODU1LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY4NTUsIC0xMjIuMzIxLCAyLjBdLCBbNDcuNjg1NSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny42ODU1LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjY4NTYsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjg1NiwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42ODU2LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg1NywgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NTcsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNjg1NywgLTEyMi4zNTMsIDEuMF0sIFs0Ny42ODU3LCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg1NywgLTEyMi4zNDgsIDEuMF0sIFs0Ny42ODU3LCAtMTIyLjM0MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg1NywgLTEyMi4zMzksIDEuMF0sIFs0Ny42ODU3LCAtMTIyLjMzNCwgMS4wXSwgWzQ3LjY4NTcsIC0xMjIuMzMxLCAxLjBdLCBbNDcuNjg1NywgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NTgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjg1OCwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42ODU4LCAtMTIyLjMzNiwgMS4wXSwgWzQ3LjY4NTgsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjg1OCwgLTEyMi4yODYsIDEuMF0sIFs0Ny42ODU5LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY4NTksIC0xMjIuMzcsIDEuMF0sIFs0Ny42ODU5LCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjY4NTksIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjg2MDAwMDAwMDAwMDEsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODYwMDAwMDAwMDAwMSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42ODYwMDAwMDAwMDAwMSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NjEsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjg2MSwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42ODYyLCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY4NjIsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODYyLCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY4NjIsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODYyLCAtMTIyLjMxLCAxLjBdLCBbNDcuNjg2MywgLTEyMi4zODIsIDEuMF0sIFs0Ny42ODY0LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY4NjQsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjg2NCwgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NjQsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODY0LCAtMTIyLjM0NSwgMS4wXSwgWzQ3LjY4NjQsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODY0LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY4NjUsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjg2NSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny42ODY1LCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjg2NSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42ODY1LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjY4NjUsIC0xMjIuMzIsIDEuMF0sIFs0Ny42ODY1LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjg2NiwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NjYsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODY3LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg2NywgLTEyMi4zNywgMS4wXSwgWzQ3LjY4NjcsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjg2NywgLTEyMi4zNTQsIDIuMF0sIFs0Ny42ODY3LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjY4NjcsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODY3LCAtMTIyLjM0NSwgMS4wXSwgWzQ3LjY4NjgsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42ODY4LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY4NjgsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODY4LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjY4NjksIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODY5LCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg3LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY4NywgLTEyMi4zODksIDEuMF0sIFs0Ny42ODcsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42ODcsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNjg3LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg3LCAtMTIyLjMyLCAxLjBdLCBbNDcuNjg3LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjY4NzEsIC0xMjIuMzkxLCAxLjBdLCBbNDcuNjg3MSwgLTEyMi4zNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NzEsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNjg3MSwgLTEyMi4zMzYsIDEuMF0sIFs0Ny42ODcxLCAtMTIyLjMzNCwgMS4wXSwgWzQ3LjY4NzEsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjg3MiwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42ODcyLCAtMTIyLjM1LCAxLjBdLCBbNDcuNjg3MiwgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjY4NzIsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjg3MiwgLTEyMi4zNDQsIDEuMF0sIFs0Ny42ODcyLCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjY4NzIsIC0xMjIuMzMzLCAxLjBdLCBbNDcuNjg3MiwgLTEyMi4yOTYsIDEuMF0sIFs0Ny42ODczLCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY4NzMsIC0xMjIuMzM2LCAxLjBdLCBbNDcuNjg3MywgLTEyMi4zMzMsIDEuMF0sIFs0Ny42ODczLCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg3MywgLTEyMi4zMjEsIDEuMF0sIFs0Ny42ODczLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjY4NzMsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjg3MywgLTEyMi4yODYsIDEuMF0sIFs0Ny42ODc0LCAtMTIyLjMzNiwgMS4wXSwgWzQ3LjY4NzQsIC0xMjIuMzE1LCAxLjBdLCBbNDcuNjg3NSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NzUsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODc1LCAtMTIyLjM4NywgMS4wXSwgWzQ3LjY4NzUsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODc1LCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjY4NzUsIC0xMjIuMzM2LCAxLjBdLCBbNDcuNjg3NSwgLTEyMi4zMywgMS4wXSwgWzQ3LjY4NzUsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjg3NSwgLTEyMi4yODcsIDEuMF0sIFs0Ny42ODc2LCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY4NzYsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjg3NiwgLTEyMi4zODEsIDEuMF0sIFs0Ny42ODc2LCAtMTIyLjM4LCAxLjBdLCBbNDcuNjg3NiwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42ODc2LCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjY4NzYsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODc2LCAtMTIyLjI5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg3NywgLTEyMi4zOTUsIDEuMF0sIFs0Ny42ODc3LCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjY4NzcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNjg3NywgLTEyMi4yODksIDEuMF0sIFs0Ny42ODc3LCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg3OCwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY4NzgsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjg3OCwgLTEyMi4zNSwgMS4wXSwgWzQ3LjY4NzgsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODc4LCAtMTIyLjMzMSwgMS4wXSwgWzQ3LjY4NzgsIC0xMjIuMjksIDEuMF0sIFs0Ny42ODc4LCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjY4NzgsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjg3OSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NzksIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjg3OSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4OCwgLTEyMi4zODMsIDEuMF0sIFs0Ny42ODgsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODgsIC0xMjIuMzIsIDEuMF0sIFs0Ny42ODgsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODgsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNjg4MSwgLTEyMi4zNjcsIDEuMF0sIFs0Ny42ODgxLCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg4MSwgLTEyMi4zMiwgMS4wXSwgWzQ3LjY4ODEsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODgyLCAtMTIyLjM4MiwgMS4wXSwgWzQ3LjY4ODIsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjg4MiwgLTEyMi4zMDYsIDEuMF0sIFs0Ny42ODgzLCAtMTIyLjM5NSwgMS4wXSwgWzQ3LjY4ODMsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjg4MywgLTEyMi4zMzUsIDEuMF0sIFs0Ny42ODgzLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjY4ODMsIC0xMjIuMywgMS4wXSwgWzQ3LjY4ODMsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODg0LCAtMTIyLjM3NCwgMS4wXSwgWzQ3LjY4ODQsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjg4NSwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjY4ODUsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNjg4NSwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY4ODUsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNjg4NSwgLTEyMi4zMzcsIDEuMF0sIFs0Ny42ODg1LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjY4ODYsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODg2LCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg4NiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42ODg2LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg4NiwgLTEyMi4zNjEsIDIuMF0sIFs0Ny42ODg2LCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY4ODYsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjg4NiwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42ODg2LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY4ODYsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjg4NywgLTEyMi4zNjgsIDEuMF0sIFs0Ny42ODg3LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY4ODcsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODg3LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjY4ODgsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODg4LCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg4OSwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY4ODksIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjg4OSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4ODksIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjg4OSwgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4ODksIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODg5LCAtMTIyLjMzLCAxLjBdLCBbNDcuNjg4OSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42ODg5OTk5OTk5OTk5OSwgLTEyMi4zOTYsIDEuMF0sIFs0Ny42ODg5OTk5OTk5OTk5OSwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42ODg5OTk5OTk5OTk5OSwgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4OTEsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjg5MSwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42ODkxLCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjY4OTIsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODkyLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY4OTIsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODkyLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg5MiwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42ODkyLCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY4OTIsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjg5MiwgLTEyMi4zMDYsIDEuMF0sIFs0Ny42ODkyLCAtMTIyLjMsIDEuMF0sIFs0Ny42ODkzLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg5MywgLTEyMi4zNjUsIDEuMF0sIFs0Ny42ODkzLCAtMTIyLjM1OSwgMS4wXSwgWzQ3LjY4OTMsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjg5MywgLTEyMi4zNSwgMS4wXSwgWzQ3LjY4OTMsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODkzLCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY4OTMsIC0xMjIuMzI1LCAxLjBdLCBbNDcuNjg5NCwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42ODk0LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY4OTQsIC0xMjIuMzUsIDEuMF0sIFs0Ny42ODk0LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg5NSwgLTEyMi4zODIsIDEuMF0sIFs0Ny42ODk1LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjg5NSwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42ODk2LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjY4OTYsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjg5NiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4OTYsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODk2LCAtMTIyLjMzMSwgMS4wXSwgWzQ3LjY4OTYsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjg5NywgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4OTcsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjg5NywgLTEyMi4zNTQsIDEuMF0sIFs0Ny42ODk3LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjY4OTcsIC0xMjIuMjk0LCAxLjBdLCBbNDcuNjg5NywgLTEyMi4yODUsIDEuMF0sIFs0Ny42ODk4LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY4OTgsIC0xMjIuMzI3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42ODk4LCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg5OSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42ODk5LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjY4OTksIC0xMjIuMzIsIDEuMF0sIFs0Ny42ODk5LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY4OTksIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42OSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5LCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjY5MDEsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNjkwMSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjY5MDEsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTAxLCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkwMSwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MDEsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNjkwMSwgLTEyMi4zMzEsIDEuMF0sIFs0Ny42OTAxLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkwMiwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MDIsIC0xMjIuMzg3LCAyLjBdLCBbNDcuNjkwMiwgLTEyMi4zODIsIDEuMF0sIFs0Ny42OTAyLCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY5MDIsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNjkwMiwgLTEyMi4zMjUsIDEuMF0sIFs0Ny42OTAyLCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjY5MDMsIC0xMjIuMzkzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTAzLCAtMTIyLjM4LCAxLjBdLCBbNDcuNjkwMywgLTEyMi4zNywgMS4wXSwgWzQ3LjY5MDMsIC0xMjIuMzQsIDEuMF0sIFs0Ny42OTAzLCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkwNCwgLTEyMi4zOTcsIDEuMF0sIFs0Ny42OTA0LCAtMTIyLjM5NSwgMi4wXSwgWzQ3LjY5MDQsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTA0LCAtMTIyLjM3LCAyLjBdLCBbNDcuNjkwNCwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MDQsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTA0LCAtMTIyLjM0NiwgMi4wXSwgWzQ3LjY5MDQsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42OTA0LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjY5MDUsIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTA1LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY5MDcsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjkwNywgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjY5MDcsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTA3LCAtMTIyLjM0LCAxLjBdLCBbNDcuNjkwOCwgLTEyMi4zOTcsIDEuMF0sIFs0Ny42OTA4LCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY5MDgsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjkwOCwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MDksIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjkwOSwgLTEyMi4zODEsIDEuMF0sIFs0Ny42OTA5LCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjY5MSwgLTEyMi4zODcsIDIuMF0sIFs0Ny42OTEsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTEsIC0xMjIuMzM3LCAxLjBdLCBbNDcuNjkxLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjY5MSwgLTEyMi4zMDMsIDEuMF0sIFs0Ny42OTExLCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkxMSwgLTEyMi4zNjcsIDEuMF0sIFs0Ny42OTExLCAtMTIyLjM1LCAxLjBdLCBbNDcuNjkxMSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42OTExLCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkxMSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MTEsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTExLCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjkxMiwgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MTIsIC0xMjIuMzQsIDEuMF0sIFs0Ny42OTEyLCAtMTIyLjMzNCwgMS4wXSwgWzQ3LjY5MTIsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTEzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjkxMywgLTEyMi4zODEsIDEuMF0sIFs0Ny42OTEzLCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjY5MTMsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjkxMywgLTEyMi4zMzMsIDEuMF0sIFs0Ny42OTE0LCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkxNCwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42OTE0LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjkxNCwgLTEyMi4zMSwgMS4wXSwgWzQ3LjY5MTQsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjkxNCwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MTUsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNjkxNSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42OTE1LCAtMTIyLjM0NSwgMS4wXSwgWzQ3LjY5MTUsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNjkxNSwgLTEyMi4zMjEsIDEuMF0sIFs0Ny42OTE1LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkxNiwgLTEyMi4zOCwgMS4wXSwgWzQ3LjY5MTYsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTE2LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjkxNiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MTYsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjkxNiwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MTYsIC0xMjIuMzQsIDEuMF0sIFs0Ny42OTE3LCAtMTIyLjM4LCAxLjBdLCBbNDcuNjkxNywgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MTgsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjkxOCwgLTEyMi4zMiwgMS4wXSwgWzQ3LjY5MTksIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTE5LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjkyLCAtMTIyLjMzOCwgMS4wXSwgWzQ3LjY5MiwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MjEsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNjkyMSwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MjEsIC0xMjIuMzgsIDEuMF0sIFs0Ny42OTIyLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY5MjIsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTIzLCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkyMywgLTEyMi4zNjksIDEuMF0sIFs0Ny42OTIzLCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjkyNCwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MjQsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjkyNCwgLTEyMi4zMjEsIDIuMF0sIFs0Ny42OTI0LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjkyNSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MjUsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjkyNSwgLTEyMi4zMiwgMS4wXSwgWzQ3LjY5MjYsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjkyNiwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42OTI2LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkyNiwgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MjYsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjkyNywgLTEyMi4zOTUsIDEuMF0sIFs0Ny42OTI3LCAtMTIyLjMzOCwgMS4wXSwgWzQ3LjY5MjgsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNjkyOCwgLTEyMi4zMzcsIDEuMF0sIFs0Ny42OTI4LCAtMTIyLjMyMywgMS4wXSwgWzQ3LjY5MjksIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTI5LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjkyOSwgLTEyMi4zNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MjksIC0xMjIuMzMzLCAxLjBdLCBbNDcuNjkyOSwgLTEyMi4zMjEsIDEuMF0sIFs0Ny42OTI5LCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjY5MzAwMDAwMDAwMDAxLCAtMTIyLjM4MywgMS4wXSwgWzQ3LjY5MzAwMDAwMDAwMDAxLCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY5MzEsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTMxLCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkzMSwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42OTMxLCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkzMSwgLTEyMi4zMywgMS4wXSwgWzQ3LjY5MzIsIC0xMjIuMzk1LCAxLjBdLCBbNDcuNjkzMiwgLTEyMi4zNDYsIDEuMF0sIFs0Ny42OTMzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjkzMywgLTEyMi4zNjEsIDEuMF0sIFs0Ny42OTMzLCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkzNCwgLTEyMi4zNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MzQsIC0xMjIuMzI3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTM0LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjY5MzUsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjkzNSwgLTEyMi4zNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MzUsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTM1LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkzNywgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MzcsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTM3LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjY5MzcsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTM3LCAtMTIyLjMzOCwgMS4wXSwgWzQ3LjY5MzgsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTM4LCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkzOCwgLTEyMi4zNCwgMS4wXSwgWzQ3LjY5MzgsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTM4LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjkzOSwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42OTM5LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjY5MzksIC0xMjIuMzQ4LCAxLjBdLCBbNDcuNjkzOSwgLTEyMi4zNCwgMS4wXSwgWzQ3LjY5Mzk5OTk5OTk5OTk5NiwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42OTM5OTk5OTk5OTk5OTYsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjkzOTk5OTk5OTk5OTk2LCAtMTIyLjM0LCAxLjBdLCBbNDcuNjkzOTk5OTk5OTk5OTk2LCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk0MSwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42OTQyLCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjY5NDIsIC0xMjIuMzE5LCAyLjBdLCBbNDcuNjk0MiwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42OTQyLCAtMTIyLjMwNCwgMi4wXSwgWzQ3LjY5NDMsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjk0NCwgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjY5NDQsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjk0NCwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42OTQ0LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjY5NDUsIC0xMjIuMzgsIDEuMF0sIFs0Ny42OTQ1LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk0NSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42OTQ1LCAtMTIyLjMzLCAxLjBdLCBbNDcuNjk0NSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42OTQ2LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk0NiwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NDYsIC0xMjIuMzM2LCAxLjBdLCBbNDcuNjk0NiwgLTEyMi4zMjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5NDcsIC0xMjIuMzQ2LCAyLjBdLCBbNDcuNjk0NywgLTEyMi4zNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NDcsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjk0NywgLTEyMi4zMTUsIDEuMF0sIFs0Ny42OTQ3LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY5NDcsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjk0OCwgLTEyMi4zOTUsIDEuMF0sIFs0Ny42OTQ4LCAtMTIyLjM5Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk0OCwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5NDgsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTQ4LCAtMTIyLjMyNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk0OCwgLTEyMi4zMjIsIDEuMF0sIFs0Ny42OTQ5LCAtMTIyLjM5NiwgMS4wXSwgWzQ3LjY5NDksIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjk0OSwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NDksIC0xMjIuMzcsIDEuMF0sIFs0Ny42OTUsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjk1LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk1LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjY5NTEsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjk1MSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42OTUxLCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY5NTEsIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTUxLCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY5NTEsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNjk1MSwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42OTUyLCAtMTIyLjM5LCAxLjBdLCBbNDcuNjk1MiwgLTEyMi4zMjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NTIsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTUyLCAtMTIyLjMyMywgMS4wXSwgWzQ3LjY5NTMsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTUzLCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk1NCwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42OTU1LCAtMTIyLjM3NiwgMi4wXSwgWzQ3LjY5NTUsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTU1LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjY5NTYsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTU2LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjk1NiwgLTEyMi4zMzUsIDEuMF0sIFs0Ny42OTU2LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjY5NTYsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjk1NiwgLTEyMi4zMDMsIDEuMF0sIFs0Ny42OTU3LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjY5NTcsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjk1NywgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5NTgsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42OTU4LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk1OSwgLTEyMi4zNzYsIDIuMF0sIFs0Ny42OTU5LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY5NTksIC0xMjIuMzYzOTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42OTYwMDAwMDAwMDAwMSwgLTEyMi4zMTksIDEuMF0sIFs0Ny42OTYwMDAwMDAwMDAwMSwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42OTYxLCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNjk2MSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjY5NjEsIC0xMjIuMzE4LCAxLjBdLCBbNDcuNjk2MSwgLTEyMi4zMTYsIDIuMF0sIFs0Ny42OTYyLCAtMTIyLjM5NywgMS4wXSwgWzQ3LjY5NjIsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTYyLCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY5NjIsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTYyLCAtMTIyLjM0LCAxLjBdLCBbNDcuNjk2MiwgLTEyMi4zMjEsIDEuMF0sIFs0Ny42OTYyLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjY5NjIsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTYzLCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk2MywgLTEyMi4zNjcsIDEuMF0sIFs0Ny42OTYzLCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY5NjMsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTYzLCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjY5NjQsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjk2NCwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NjQsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTY0LCAtMTIyLjMzNiwgMS4wXSwgWzQ3LjY5NjQsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjk2NSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5NjUsIC0xMjIuMzUsIDEuMF0sIFs0Ny42OTY1LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjk2NSwgLTEyMi4zNCwgMS4wXSwgWzQ3LjY5NjUsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTY2LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY5NjYsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjk2NiwgLTEyMi4zNSwgMS4wXSwgWzQ3LjY5NjYsIC0xMjIuMzI0LCAzLjBdLCBbNDcuNjk2NiwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42OTY3LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY5NjcsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTY4LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY5NjgsIC0xMjIuMzQsIDEuMF0sIFs0Ny42OTY5LCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY5NjksIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjk2OSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42OTY5LCAtMTIyLjM1LCAxLjBdLCBbNDcuNjk2OSwgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NjksIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTY5LCAtMTIyLjMzOCwgMS4wXSwgWzQ3LjY5NjksIC0xMjIuMzEsIDEuMF0sIFs0Ny42OTY5OTk5OTk5OTk5OTYsIC0xMjIuMzksIDEuMF0sIFs0Ny42OTY5OTk5OTk5OTk5OTYsIC0xMjIuMzcsIDEuMF0sIFs0Ny42OTY5OTk5OTk5OTk5OTYsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjk2OTk5OTk5OTk5OTk2LCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk2OTk5OTk5OTk5OTk2LCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk2OTk5OTk5OTk5OTk2LCAtMTIyLjMyMiwgMS4wXSwgWzQ3LjY5Njk5OTk5OTk5OTk5NiwgLTEyMi4zMTksIDEuMF0sIFs0Ny42OTY5OTk5OTk5OTk5OTYsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTcxLCAtMTIyLjM5MSwgMS4wXSwgWzQ3LjY5NzEsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTcxLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk3MSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5NzEsIC0xMjIuMzQsIDEuMF0sIFs0Ny42OTcxLCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk3MSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42OTcyLCAtMTIyLjM1LCAxLjBdLCBbNDcuNjk3MiwgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5NzIsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42OTczLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY5NzMsIC0xMjIuMzYsIDEuMF0sIFs0Ny42OTczLCAtMTIyLjM1LCAxLjBdLCBbNDcuNjk3NCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42OTc0LCAtMTIyLjM1NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk3NCwgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjY5NzQsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTc1LCAtMTIyLjM1NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk3NSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42OTc1LCAtMTIyLjM0LCAxLjBdLCBbNDcuNjk3NSwgLTEyMi4zMTYsIDEuMF0sIFs0Ny42OTc2LCAtMTIyLjM4MSwgMS4wXSwgWzQ3LjY5NzYsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjk3NiwgLTEyMi4zNTksIDEuMF0sIFs0Ny42OTc3LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjY5NzcsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTc4LCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjY5NzgsIC0xMjIuMzE2LCAyLjBdLCBbNDcuNjk3OCwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42OTc5LCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjY5NzksIC0xMjIuMzIsIDEuMF0sIFs0Ny42OTc5LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjk4LCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjY5OCwgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5OCwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42OTgsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny42OTgsIC0xMjIuMzQsIDEuMF0sIFs0Ny42OTgxLCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjY5ODEsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjk4MSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42OTgxLCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk4MSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5ODEsIC0xMjIuMzQsIDIuMF0sIFs0Ny42OTgxLCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjY5ODIsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTgyLCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk4MiwgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5ODMsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjk4MywgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5ODMsIC0xMjIuMzg5LCAxLjBdLCBbNDcuNjk4MywgLTEyMi4zNjcsIDIuMF0sIFs0Ny42OTg0LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjY5ODUsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNjk4NSwgLTEyMi4zODIsIDEuMF0sIFs0Ny42OTg1LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk4NSwgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5ODUsIC0xMjIuMzQ4LCAyLjBdLCBbNDcuNjk4NSwgLTEyMi4zNCwgMS4wXSwgWzQ3LjY5ODcsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjk4NywgLTEyMi4zNjUsIDIuMF0sIFs0Ny42OTg3LCAtMTIyLjM2LCAxLjBdLCBbNDcuNjk4NywgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5ODgsIC0xMjIuMzg3LCAxLjBdLCBbNDcuNjk4OCwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5ODgsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjk4OCwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42OTg4LCAtMTIyLjMyLCAxLjBdLCBbNDcuNjk4OCwgLTEyMi4zMTksIDEuMF0sIFs0Ny42OTg4LCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk4OCwgLTEyMi4zMTYsIDEuMF0sIFs0Ny42OTg4LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk4OSwgLTEyMi4zNjksIDEuMF0sIFs0Ny42OTg5LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjY5ODksIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDMuMF0sIFs0Ny42OTksIC0xMjIuMzUsIDEuMF0sIFs0Ny42OTksIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42OTksIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjk5MSwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5OTEsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjk5MiwgLTEyMi4zODcsIDEuMF0sIFs0Ny42OTkyLCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY5OTIsIC0xMjIuMzc4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTkyLCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk5MiwgLTEyMi4zMDksIDEuMF0sIFs0Ny42OTkzLCAtMTIyLjM3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk5MywgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5OTMsIC0xMjIuMzc2LCAxLjBdLCBbNDcuNjk5MywgLTEyMi4zNDYsIDIuMF0sIFs0Ny42OTk0LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY5OTQsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTk0LCAtMTIyLjM0NzAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjk5NCwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5OTUsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNjk5NSwgLTEyMi4zODMsIDEuMF0sIFs0Ny42OTk1LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjY5OTUsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjk5NiwgLTEyMi4zOTEsIDEuMF0sIFs0Ny42OTk2LCAtMTIyLjMxLCAxLjBdLCBbNDcuNjk5NiwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5OTcsIC0xMjIuMzIsIDEuMF0sIFs0Ny42OTk3LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY5OTcsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNjk5NywgLTEyMi4zMTQsIDEuMF0sIFs0Ny42OTk3LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjY5OTgsIC0xMjIuMzY3LCAyLjBdLCBbNDcuNjk5OCwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42OTk5LCAtMTIyLjM5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk5OSwgLTEyMi4zOTEsIDIuMF0sIFs0Ny42OTk5LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk5OSwgLTEyMi4zMDUsIDIuMF0sIFs0Ny43LCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjcsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNzAwMSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMDEsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNzAwMSwgLTEyMi4zMTEsIDEuMF0sIFs0Ny43MDAxLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjcwMDIsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNzAwMywgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMDMsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNzAwNCwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMDQsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNzAwNCwgLTEyMi4zMDUsIDEuMF0sIFs0Ny43MDA0LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjcwMDUsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDA1LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjcwMDUsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNzAwNSwgLTEyMi4zMTQsIDEuMF0sIFs0Ny43MDA1LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzAwNiwgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMDYsIC0xMjIuMzUsIDEuMF0sIFs0Ny43MDA2LCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjcwMDcsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNzAwNywgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMDgsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDA4LCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjcwMDgsIC0xMjIuMzM4LCAxLjBdLCBbNDcuNzAwOSwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMSwgLTEyMi4zOSwgMS4wXSwgWzQ3LjcwMSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny43MDEsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDEsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNzAxLCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjcwMTEsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDExLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjcwMTEsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNzAxMSwgLTEyMi4zMzYsIDEuMF0sIFs0Ny43MDEyLCAtMTIyLjMyMiwgMS4wXSwgWzQ3LjcwMTMsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDEzLCAtMTIyLjM2OSwgMS4wXSwgWzQ3LjcwMTMsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNzAxMywgLTEyMi4zNTksIDEuMF0sIFs0Ny43MDEzLCAtMTIyLjM1LCAxLjBdLCBbNDcuNzAxNCwgLTEyMi4zODEsIDEuMF0sIFs0Ny43MDE0LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzAxNSwgLTEyMi4zMiwgMS4wXSwgWzQ3LjcwMTUsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDE1LCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjcwMTcsIC0xMjIuMzYsIDEuMF0sIFs0Ny43MDE3LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjcwMTgsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNzAxOSwgLTEyMi4zNjYsIDEuMF0sIFs0Ny43MDE5LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzAxOSwgLTEyMi4zMTEsIDIuMF0sIFs0Ny43MDIxLCAtMTIyLjMyLCAxLjBdLCBbNDcuNzAyMiwgLTEyMi4zNzQsIDEuMF0sIFs0Ny43MDIzLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzAyNCwgLTEyMi4zNTksIDEuMF0sIFs0Ny43MDI0LCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjcwMjQsIC0xMjIuMzUsIDEuMF0sIFs0Ny43MDI0LCAtMTIyLjM0NiwgMi4wXSwgWzQ3LjcwMjQsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNzAyNSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMjcsIC0xMjIuMzc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDI3LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjcwMjcsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNzAyNywgLTEyMi4zNTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwMjcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNzAyNywgLTEyMi4zNDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcwMjcsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDI3LCAtMTIyLjM0NiwgMi4wXSwgWzQ3LjcwMjgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNzAyOCwgLTEyMi4zMjEsIDEuMF0sIFs0Ny43MDMsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNzAzLCAtMTIyLjI4LCAxLjBdLCBbNDcuNzAzMSwgLTEyMi4yNzksIDEuMF0sIFs0Ny43MDMyLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjcwMzQsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDM3LCAtMTIyLjI5NiwgMi4wXSwgWzQ3LjcwMzcsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDM5OTk5OTk5OTk5OSwgLTEyMi4zLCAxLjBdLCBbNDcuNzA0MSwgLTEyMi4yODgsIDEuMF0sIFs0Ny43MDQ1LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjcwNDYsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDQ4LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjcwNSwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcwNTQsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDU1LCAtMTIyLjMsIDEuMF0sIFs0Ny43MDU4LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjcwNTksIC0xMjIuMzAxLCAxLjBdLCBbNDcuNzA2LCAtMTIyLjI5LCAxLjBdLCBbNDcuNzA2NSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwNjUsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNzA2NywgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcwNjksIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDcxLCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjcwNzQsIC0xMjIuMjksIDEuMF0sIFs0Ny43MDc2LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNzA3NywgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcwNzgsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNzA4NSwgLTEyMi4yODYsIDEuMF0sIFs0Ny43MDg1LCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzA4OCwgLTEyMi4zMDIsIDEuMF0sIFs0Ny43MDkxLCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjcwOTIsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNzA5OCwgLTEyMi4yOTEsIDEuMF0sIFs0Ny43MDk4LCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzEsIC0xMjIuMjg2LCAxLjBdLCBbNDcuNzEwMiwgLTEyMi4yODEsIDEuMF0sIFs0Ny43MTA1LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjcxMDYsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNzEwNiwgLTEyMi4yODYsIDEuMF0sIFs0Ny43MTA2LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjcxMDcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNzEwNywgLTEyMi4yODksIDEuMF0sIFs0Ny43MTA4LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjcxMTMsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNzExNCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny43MTE0LCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNzExNywgLTEyMi4yODYsIDEuMF0sIFs0Ny43MTE4LCAtMTIyLjI5LCAxLjBdLCBbNDcuNzExOTk5OTk5OTk5OTk2LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjcxMjEsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNzEyMiwgLTEyMi4yOSwgMS4wXSwgWzQ3LjcxMjMsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNzEyNCwgLTEyMi4zMDEsIDEuMF0sIFs0Ny43MTI0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjcxMjcsIC0xMjIuMjk2LCAxLjBdLCBbNDcuNzEzLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzEzMSwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxMzIsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTMyLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzEzNCwgLTEyMi4yOTIsIDEuMF0sIFs0Ny43MTM0LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjcxMzUsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNzEzNiwgLTEyMi4yOSwgMS4wXSwgWzQ3LjcxMzYsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNzEzOSwgLTEyMi4yOTIsIDEuMF0sIFs0Ny43MTM5LCAtMTIyLjI4OCwgMi4wXSwgWzQ3LjcxNCwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxNDIsIC0xMjIuMjg2LCAyLjBdLCBbNDcuNzE0MiwgLTEyMi4yODEsIDEuMF0sIFs0Ny43MTQ0LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzE0NSwgLTEyMi4yODUsIDEuMF0sIFs0Ny43MTQ2LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjcxNDgsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNzE0OCwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcxNSwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcxNTIsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MTUzLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzE1NSwgLTEyMi4zMTUsIDIuMF0sIFs0Ny43MTU1LCAtMTIyLjIxNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzE1NSwgLTEyMi4yMTEsIDEuMF0sIFs0Ny43MTU2LCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzE1OCwgLTEyMi4xNjcsIDEuMF0sIFs0Ny43MTU5LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjcxNTksIC0xMjIuMTY1LCAxLjBdLCBbNDcuNzE2LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjcxNjEsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MTYxLCAtMTIyLjIxOSwgMS4wXSwgWzQ3LjcxNjEsIC0xMjIuMTcsIDEuMF0sIFs0Ny43MTYyLCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjcxNjIsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNzE2MiwgLTEyMi4xNjYsIDEuMF0sIFs0Ny43MTYzLCAtMTIyLjMwMywgMS4wXSwgWzQ3LjcxNjMsIC0xMjIuMjI5LCAxLjBdLCBbNDcuNzE2MywgLTEyMi4yMTIsIDEuMF0sIFs0Ny43MTY0LCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjcxNjQsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNzE2NSwgLTEyMi4zMTksIDEuMF0sIFs0Ny43MTY1LCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzE2NiwgLTEyMi4yMjUsIDEuMF0sIFs0Ny43MTY3LCAtMTIyLjE3LCAxLjBdLCBbNDcuNzE2OCwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxNywgLTEyMi4yMjYsIDEuMF0sIFs0Ny43MTcsIC0xMjIuMTYzLCAxLjBdLCBbNDcuNzE3MSwgLTEyMi4zMiwgMS4wXSwgWzQ3LjcxNzEsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNzE3NCwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcxNzQsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTc2LCAtMTIyLjIxNCwgMS4wXSwgWzQ3LjcxNzcsIC0xMjIuMjIzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTc4LCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjcxNzgsIC0xMjIuMTY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTc5LCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjcxOCwgLTEyMi4xNjEsIDEuMF0sIFs0Ny43MTgxLCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjcxODEsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTgzLCAtMTIyLjMxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzE4MywgLTEyMi4zMDYsIDEuMF0sIFs0Ny43MTgzLCAtMTIyLjIxMywgMS4wXSwgWzQ3LjcxODQsIC0xMjIuMjI2LCAxLjBdLCBbNDcuNzE4NSwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcxODUsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTg1LCAtMTIyLjIyNiwgMS4wXSwgWzQ3LjcxODYsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNzE4NiwgLTEyMi4yMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxODcsIC0xMjIuMjE5LCAxLjBdLCBbNDcuNzE4OCwgLTEyMi4zMjEsIDEuMF0sIFs0Ny43MTg4LCAtMTIyLjE3NywgMi4wXSwgWzQ3LjcxODksIC0xMjIuMjI1LCAxLjBdLCBbNDcuNzE4OSwgLTEyMi4xODEsIDEuMF0sIFs0Ny43MTg5LCAtMTIyLjE3OCwgMS4wXSwgWzQ3LjcxODk5OTk5OTk5OTk5NCwgLTEyMi4zMjEsIDEuMF0sIFs0Ny43MTg5OTk5OTk5OTk5OTQsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MTg5OTk5OTk5OTk5OTQsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNzE4OTk5OTk5OTk5OTk0LCAtMTIyLjIxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzE4OTk5OTk5OTk5OTk0LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzE5MSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny43MTkxLCAtMTIyLjMwOSwgMi4wXSwgWzQ3LjcxOTIsIC0xMjIuMzIsIDIuMF0sIFs0Ny43MTkyLCAtMTIyLjE3NiwgMS4wXSwgWzQ3LjcxOTMsIC0xMjIuMjE3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTkzLCAtMTIyLjIxNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzE5NCwgLTEyMi4xNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcxOTYsIC0xMjIuMjIzLCAxLjBdLCBbNDcuNzE5NywgLTEyMi4yMTksIDEuMF0sIFs0Ny43MTk3LCAtMTIyLjIxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzE5NywgLTEyMi4xNzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxOTgsIC0xMjIuMTcxLCAxLjBdLCBbNDcuNzE5OSwgLTEyMi4xOCwgMS4wXSwgWzQ3LjcyLCAtMTIyLjMyLCAxLjBdLCBbNDcuNzIsIC0xMjIuMzE4LCAxLjBdLCBbNDcuNzIsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MiwgLTEyMi4yMjksIDEuMF0sIFs0Ny43MiwgLTEyMi4yMTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyLCAtMTIyLjIxMywgMS4wXSwgWzQ3LjcyMDEsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjAyLCAtMTIyLjIyMywgMS4wXSwgWzQ3LjcyMDIsIC0xMjIuMjIsIDEuMF0sIFs0Ny43MjAzLCAtMTIyLjMyMywgMS4wXSwgWzQ3LjcyMDMsIC0xMjIuMjI5LCAxLjBdLCBbNDcuNzIwNCwgLTEyMi4xOCwgMS4wXSwgWzQ3LjcyMDUsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNzIwNSwgLTEyMi4yMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyMDUsIC0xMjIuMTY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjA2LCAtMTIyLjIxMSwgMS4wXSwgWzQ3LjcyMDYsIC0xMjIuMTcxLCAxLjBdLCBbNDcuNzIwNywgLTEyMi4zMiwgMS4wXSwgWzQ3LjcyMDcsIC0xMjIuMjIyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MjA3LCAtMTIyLjE2NSwgMS4wXSwgWzQ3LjcyMDksIC0xMjIuMzE4LCAxLjBdLCBbNDcuNzIwOSwgLTEyMi4xNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyMTAwMDAwMDAwMDAwNCwgLTEyMi4zMTksIDEuMF0sIFs0Ny43MjEwMDAwMDAwMDAwMDQsIC0xMjIuMjI4LCAxLjBdLCBbNDcuNzIxMDAwMDAwMDAwMDA0LCAtMTIyLjE3OSwgMS4wXSwgWzQ3LjcyMTEsIC0xMjIuMzEsIDEuMF0sIFs0Ny43MjExLCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzIxMSwgLTEyMi4xNjksIDEuMF0sIFs0Ny43MjExLCAtMTIyLjE2NiwgMS4wXSwgWzQ3LjcyMTIsIC0xMjIuMjI2LCAxLjBdLCBbNDcuNzIxMiwgLTEyMi4yMjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyMTIsIC0xMjIuMjExLCAxLjBdLCBbNDcuNzIxMiwgLTEyMi4xNzksIDEuMF0sIFs0Ny43MjEyLCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzIxMywgLTEyMi4zMSwgMS4wXSwgWzQ3LjcyMTMsIC0xMjIuMTcxLCAxLjBdLCBbNDcuNzIxNCwgLTEyMi4yMjcsIDEuMF0sIFs0Ny43MjE0LCAtMTIyLjE2NiwgMS4wXSwgWzQ3LjcyMTUsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNzIxNiwgLTEyMi4yMTksIDEuMF0sIFs0Ny43MjE4LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzIxOCwgLTEyMi4zMDYsIDEuMF0sIFs0Ny43MjE4LCAtMTIyLjE3NiwgMS4wXSwgWzQ3LjcyMjEsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MjIxLCAtMTIyLjE2MiwgMS4wXSwgWzQ3LjcyMjIsIC0xMjIuMzIsIDEuMF0sIFs0Ny43MjIyLCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjcyMjIsIC0xMjIuMjI5LCAxLjBdLCBbNDcuNzIyMiwgLTEyMi4xNzcsIDEuMF0sIFs0Ny43MjIyLCAtMTIyLjE2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzIyMywgLTEyMi4zMjEsIDEuMF0sIFs0Ny43MjIzLCAtMTIyLjIwOSwgMS4wXSwgWzQ3LjcyMjMsIC0xMjIuMTczOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjIzLCAtMTIyLjE3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzIyMywgLTEyMi4xNiwgMS4wXSwgWzQ3LjcyMjUsIC0xMjIuMTc5LCAxLjBdLCBbNDcuNzIyNiwgLTEyMi4yMjcsIDEuMF0sIFs0Ny43MjI2LCAtMTIyLjIxOSwgMS4wXSwgWzQ3LjcyMjcsIC0xMjIuMTc5LCAxLjBdLCBbNDcuNzIyNywgLTEyMi4xNiwgMS4wXSwgWzQ3LjcyMjgsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNzIyOCwgLTEyMi4zMSwgMS4wXSwgWzQ3LjcyMjksIC0xMjIuMTgsIDEuMF0sIFs0Ny43MjMsIC0xMjIuMjE1LCAxLjBdLCBbNDcuNzIzLCAtMTIyLjE2MiwgMS4wXSwgWzQ3LjcyMywgLTEyMi4xNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyMzEsIC0xMjIuMjI5LCAxLjBdLCBbNDcuNzIzMSwgLTEyMi4xNywgMS4wXSwgWzQ3LjcyMzIsIC0xMjIuMTczOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjMyLCAtMTIyLjE2MSwgMS4wXSwgWzQ3LjcyMzMsIC0xMjIuMjIxLCAxLjBdLCBbNDcuNzIzMywgLTEyMi4xNjcsIDEuMF0sIFs0Ny43MjM1LCAtMTIyLjMyNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzIzNiwgLTEyMi4xNzM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyMzcsIC0xMjIuMTgxLCAxLjBdLCBbNDcuNzIzOCwgLTEyMi4xNjUsIDEuMF0sIFs0Ny43MjM5LCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzIzOSwgLTEyMi4yMjcsIDEuMF0sIFs0Ny43MjM5LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzIzOSwgLTEyMi4xNjQsIDEuMF0sIFs0Ny43MjQsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNzI0MSwgLTEyMi4xNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyNDIsIC0xMjIuMjE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MjQyLCAtMTIyLjE3LCAxLjBdLCBbNDcuNzI0MiwgLTEyMi4xNjEsIDEuMF0sIFs0Ny43MjQzLCAtMTIyLjIyMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzI0MywgLTEyMi4yMSwgMS4wXSwgWzQ3LjcyNDMsIC0xMjIuMTYxLCAxLjBdLCBbNDcuNzI0NCwgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNDQsIC0xMjIuMjI2LCAxLjBdLCBbNDcuNzI0NCwgLTEyMi4yMjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNDQsIC0xMjIuMTc5LCAxLjBdLCBbNDcuNzI0NSwgLTEyMi4yMiwgMS4wXSwgWzQ3LjcyNDUsIC0xMjIuMTYzLCAxLjBdLCBbNDcuNzI0NiwgLTEyMi4yMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNDYsIC0xMjIuMTcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjQ3LCAtMTIyLjE4MSwgMS4wXSwgWzQ3LjcyNDcsIC0xMjIuMTY1LCAxLjBdLCBbNDcuNzI0OCwgLTEyMi4zMjUsIDEuMF0sIFs0Ny43MjQ4LCAtMTIyLjE3OSwgMS4wXSwgWzQ3LjcyNDgsIC0xMjIuMTYyLCAxLjBdLCBbNDcuNzI0OSwgLTEyMi4xNzgsIDEuMF0sIFs0Ny43MjUsIC0xMjIuMTczOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjUsIC0xMjIuMTcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MjUxLCAtMTIyLjE3NSwgMS4wXSwgWzQ3LjcyNTEsIC0xMjIuMTY3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjUzLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjcyNTMsIC0xMjIuMTY0LCAxLjBdLCBbNDcuNzI1NCwgLTEyMi4zMSwgMS4wXSwgWzQ3LjcyNTQsIC0xMjIuMjI3LCAxLjBdLCBbNDcuNzI1NCwgLTEyMi4xNzUsIDEuMF0sIFs0Ny43MjU1LCAtMTIyLjE2NCwgMS4wXSwgWzQ3LjcyNTYsIC0xMjIuMjExLCAxLjBdLCBbNDcuNzI1NiwgLTEyMi4xOCwgMS4wXSwgWzQ3LjcyNTYsIC0xMjIuMTY2LCAxLjBdLCBbNDcuNzI1NywgLTEyMi4xNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNTcsIC0xMjIuMTYyLCAxLjBdLCBbNDcuNzI1OCwgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNTgsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNzI1OCwgLTEyMi4xNzcsIDEuMF0sIFs0Ny43MjU5LCAtMTIyLjIyNiwgMS4wXSwgWzQ3LjcyNjAwMDAwMDAwMDAwNiwgLTEyMi4zMTYsIDEuMF0sIFs0Ny43MjYwMDAwMDAwMDAwMDYsIC0xMjIuMjIxLCAxLjBdLCBbNDcuNzI2MDAwMDAwMDAwMDA2LCAtMTIyLjIyLCAxLjBdLCBbNDcuNzI2MDAwMDAwMDAwMDA2LCAtMTIyLjIxLCAxLjBdLCBbNDcuNzI2MSwgLTEyMi4zMDYsIDEuMF0sIFs0Ny43MjYxLCAtMTIyLjIyLCAxLjBdLCBbNDcuNzI2MSwgLTEyMi4yMTksIDEuMF0sIFs0Ny43MjYxLCAtMTIyLjE3LCAxLjBdLCBbNDcuNzI2MiwgLTEyMi4zMTYsIDEuMF0sIFs0Ny43MjYyLCAtMTIyLjIyMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzI2MiwgLTEyMi4yMjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNjMsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNzI2MywgLTEyMi4yMjksIDEuMF0sIFs0Ny43MjY0LCAtMTIyLjMxLCAxLjBdLCBbNDcuNzI2NCwgLTEyMi4zMDcsIDEuMF0sIFs0Ny43MjY1LCAtMTIyLjMxNSwgMS4wXSwgWzQ3LjcyNjUsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNzI2NSwgLTEyMi4yMDYsIDEuMF0sIFs0Ny43MjY2LCAtMTIyLjE3NSwgMS4wXSwgWzQ3LjcyNjcsIC0xMjIuMjI3LCAxLjBdLCBbNDcuNzI2NywgLTEyMi4yMjYsIDIuMF0sIFs0Ny43MjY4LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjcyNjgsIC0xMjIuMjI5LCAxLjBdLCBbNDcuNzI2OCwgLTEyMi4yMTMsIDEuMF0sIFs0Ny43MjY5LCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjcyNywgLTEyMi4yMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNywgLTEyMi4xNzYsIDEuMF0sIFs0Ny43MjcxLCAtMTIyLjIyOCwgMS4wXSwgWzQ3LjcyNzEsIC0xMjIuMjE1LCAxLjBdLCBbNDcuNzI3MiwgLTEyMi4zMTEsIDEuMF0sIFs0Ny43MjczLCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjcyNzMsIC0xMjIuMjExLCAxLjBdLCBbNDcuNzI3NCwgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyNzQsIC0xMjIuMzA1LCAxLjBdLCBbNDcuNzI3NCwgLTEyMi4xODEsIDEuMF0sIFs0Ny43Mjc1LCAtMTIyLjIxOSwgMS4wXSwgWzQ3LjcyNzYsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjc2LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjcyNzYsIC0xMjIuMTgxLCAxLjBdLCBbNDcuNzI3NywgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyNzcsIC0xMjIuMjA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjc4LCAtMTIyLjIyNiwgMS4wXSwgWzQ3LjcyNzgsIC0xMjIuMjE3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43Mjc4LCAtMTIyLjE3NywgMS4wXSwgWzQ3LjcyNzksIC0xMjIuMTk1LCAxLjBdLCBbNDcuNzI3OSwgLTEyMi4xOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyOCwgLTEyMi4zMzksIDEuMF0sIFs0Ny43MjgxLCAtMTIyLjMzNSwgMS4wXSwgWzQ3LjcyODEsIC0xMjIuMjM1LCAxLjBdLCBbNDcuNzI4MSwgLTEyMi4yMzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyODEsIC0xMjIuMjMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MjgyLCAtMTIyLjMzNywgMS4wXSwgWzQ3LjcyODIsIC0xMjIuMTg2LCAxLjBdLCBbNDcuNzI4MywgLTEyMi4yMzgsIDEuMF0sIFs0Ny43MjgzLCAtMTIyLjE5NCwgMS4wXSwgWzQ3LjcyODQsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNzI4NCwgLTEyMi4zMzYsIDEuMF0sIFs0Ny43Mjg0LCAtMTIyLjI0MSwgMS4wXSwgWzQ3LjcyODUsIC0xMjIuMjQsIDEuMF0sIFs0Ny43Mjg1LCAtMTIyLjE5OCwgMS4wXSwgWzQ3LjcyODcsIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjg4LCAtMTIyLjMzMSwgMS4wXSwgWzQ3LjcyODgsIC0xMjIuMTkxLCAxLjBdLCBbNDcuNzI4OSwgLTEyMi4zNCwgMS4wXSwgWzQ3LjcyODksIC0xMjIuMzM4LCAxLjBdLCBbNDcuNzI4OSwgLTEyMi4zMzcsIDEuMF0sIFs0Ny43Mjg5LCAtMTIyLjMzNSwgMS4wXSwgWzQ3LjcyODksIC0xMjIuMjMyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43Mjg5LCAtMTIyLjIwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzI4OSwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyOSwgLTEyMi4zNSwgMS4wXSwgWzQ3LjcyOSwgLTEyMi4yMzUsIDEuMF0sIFs0Ny43MjkxLCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjcyOTEsIC0xMjIuMzM3LCAxLjBdLCBbNDcuNzI5MSwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcyOTIsIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjkyLCAtMTIyLjIzMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzI5MywgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjcyOTQsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjk0LCAtMTIyLjMyOSwgMS4wXSwgWzQ3LjcyOTQsIC0xMjIuMTg2LCAxLjBdLCBbNDcuNzI5NSwgLTEyMi4zMzQsIDEuMF0sIFs0Ny43Mjk1LCAtMTIyLjMzMSwgMS4wXSwgWzQ3LjcyOTYsIC0xMjIuMzMzLCAxLjBdLCBbNDcuNzI5NiwgLTEyMi4yNDEsIDEuMF0sIFs0Ny43Mjk2LCAtMTIyLjI0LCAxLjBdLCBbNDcuNzI5NiwgLTEyMi4yMzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcyOTYsIC0xMjIuMjMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjk2LCAtMTIyLjE5OSwgMS4wXSwgWzQ3LjcyOTcsIC0xMjIuMzI3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjk4LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzI5OCwgLTEyMi4zNCwgMS4wXSwgWzQ3LjcyOTgsIC0xMjIuMjMxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjk5LCAtMTIyLjE5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMsIC0xMjIuMzM1LCAyLjBdLCBbNDcuNzMsIC0xMjIuMTkxLCAxLjBdLCBbNDcuNzMwMSwgLTEyMi4yLCAxLjBdLCBbNDcuNzMwMiwgLTEyMi4xOTcsIDEuMF0sIFs0Ny43MzAzLCAtMTIyLjMyOSwgMS4wXSwgWzQ3LjczMDMsIC0xMjIuMjM4LCAxLjBdLCBbNDcuNzMwMywgLTEyMi4yMzYsIDEuMF0sIFs0Ny43MzAzLCAtMTIyLjE5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMwNCwgLTEyMi4yNCwgMS4wXSwgWzQ3LjczMDQsIC0xMjIuMjMxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzA2LCAtMTIyLjIzMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzMwNywgLTEyMi4zMzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMDcsIC0xMjIuMjM4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MzA3LCAtMTIyLjE4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzMwOCwgLTEyMi4zNCwgMS4wXSwgWzQ3LjczMDgsIC0xMjIuMTk0LCAxLjBdLCBbNDcuNzMwOCwgLTEyMi4xOSwgMS4wXSwgWzQ3LjczMDksIC0xMjIuMzM2LCAxLjBdLCBbNDcuNzMwOSwgLTEyMi4zMzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMSwgLTEyMi4zNDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMSwgLTEyMi4xOTEsIDEuMF0sIFs0Ny43MzExLCAtMTIyLjIzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMxMSwgLTEyMi4yMDQsIDEuMF0sIFs0Ny43MzExLCAtMTIyLjE5OSwgMS4wXSwgWzQ3LjczMTIsIC0xMjIuMzM0LCAxLjBdLCBbNDcuNzMxMiwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMTMsIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MzE0LCAtMTIyLjIzNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMxNiwgLTEyMi4zMzgsIDEuMF0sIFs0Ny43MzE3LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzMxNywgLTEyMi4yNDEsIDEuMF0sIFs0Ny43MzE3LCAtMTIyLjIzMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMxNywgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMTcsIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzMxOCwgLTEyMi4yMzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczMTksIC0xMjIuMzM1LCAxLjBdLCBbNDcuNzMxOSwgLTEyMi4yMDQsIDEuMF0sIFs0Ny43MzIsIC0xMjIuMjMxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzIsIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzMyMSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMjEsIC0xMjIuMzQsIDEuMF0sIFs0Ny43MzIxLCAtMTIyLjMzNCwgMy4wXSwgWzQ3LjczMjEsIC0xMjIuMTk4LCAxLjBdLCBbNDcuNzMyMSwgLTEyMi4xOTUsIDEuMF0sIFs0Ny43MzIxLCAtMTIyLjE4NSwgMS4wXSwgWzQ3LjczMjIsIC0xMjIuMjM2LCAxLjBdLCBbNDcuNzMyMiwgLTEyMi4xODUsIDEuMF0sIFs0Ny43MzI1LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNzMyNSwgLTEyMi4zMzcsIDEuMF0sIFs0Ny43MzI1LCAtMTIyLjI0NSwgMS4wXSwgWzQ3LjczMjUsIC0xMjIuMjQyLCAxLjBdLCBbNDcuNzMyNiwgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczMjYsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzI2LCAtMTIyLjIzNCwgMi4wXSwgWzQ3LjczMjYsIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzMyNiwgLTEyMi4xOTQsIDEuMF0sIFs0Ny43MzI3LCAtMTIyLjIzMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzMyOCwgLTEyMi4zNCwgMS4wXSwgWzQ3LjczMjgsIC0xMjIuMjAxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzI4LCAtMTIyLjIsIDEuMF0sIFs0Ny43MzI4LCAtMTIyLjE5OCwgMS4wXSwgWzQ3LjczMjksIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MzI5LCAtMTIyLjIzNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMzMSwgLTEyMi4xODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczMzIsIC0xMjIuMjMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzMyLCAtMTIyLjE5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMzMywgLTEyMi4yNDYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMzUsIC0xMjIuMTkzLCAxLjBdLCBbNDcuNzMzNiwgLTEyMi4zMzksIDEuMF0sIFs0Ny43MzM2LCAtMTIyLjIzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMzNiwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczMzYsIC0xMjIuMTksIDEuMF0sIFs0Ny43MzM2LCAtMTIyLjE4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzMzNywgLTEyMi4zNDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczMzgsIC0xMjIuMzMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzM4LCAtMTIyLjIzNCwgMS4wXSwgWzQ3LjczMzksIC0xMjIuMzM5LCAxLjBdLCBbNDcuNzMzOSwgLTEyMi4zMzcsIDEuMF0sIFs0Ny43MzM5LCAtMTIyLjMzNiwgMS4wXSwgWzQ3LjczMzksIC0xMjIuMTgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MzM5OTk5OTk5OTk5OTUsIC0xMjIuMTkzLCAxLjBdLCBbNDcuNzMzOTk5OTk5OTk5OTk1LCAtMTIyLjE5LCAxLjBdLCBbNDcuNzMzOTk5OTk5OTk5OTk1LCAtMTIyLjE4MiwgMS4wXSwgWzQ3LjczNDEsIC0xMjIuMjQyLCAxLjBdLCBbNDcuNzM0MSwgLTEyMi4yMzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczNDEsIC0xMjIuMjM0LCAxLjBdLCBbNDcuNzM0MSwgLTEyMi4yLCAxLjBdLCBbNDcuNzM0MiwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczNDMsIC0xMjIuMzQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzQzLCAtMTIyLjMzNSwgMS4wXSwgWzQ3LjczNDMsIC0xMjIuMTkzLCAxLjBdLCBbNDcuNzM0NCwgLTEyMi4zMzYsIDEuMF0sIFs0Ny43MzQ0LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjczNDQsIC0xMjIuMjQ2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzQ2LCAtMTIyLjI0NCwgMS4wXSwgWzQ3LjczNDcsIC0xMjIuMjM2LCAxLjBdLCBbNDcuNzM0NywgLTEyMi4xOTMsIDEuMF0sIFs0Ny43MzQ3LCAtMTIyLjE5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzM0OCwgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczNDgsIC0xMjIuMzQsIDEuMF0sIFs0Ny43MzQ5LCAtMTIyLjIzOCwgMS4wXSwgWzQ3LjczNDksIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzM1LCAtMTIyLjI0NCwgMS4wXSwgWzQ3LjczNSwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczNTEsIC0xMjIuMTk5LCAxLjBdLCBbNDcuNzM1MSwgLTEyMi4xODksIDEuMF0sIFs0Ny43MzUxLCAtMTIyLjE4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzM1MiwgLTEyMi4zMzcsIDEuMF0sIFs0Ny43MzUyLCAtMTIyLjMzLCAxLjBdLCBbNDcuNzM1MiwgLTEyMi4zMjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczNTIsIC0xMjIuMjMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzUyLCAtMTIyLjIwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzM1MywgLTEyMi4zMzQsIDEuMF0sIFs0Ny43MzUzLCAtMTIyLjMzMSwgMS4wXSwgWzQ3LjczNTQsIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MzU1LCAtMTIyLjIzMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzM1NSwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczNTYsIC0xMjIuMzQsIDEuMF0sIFs0Ny43MzU2LCAtMTIyLjMzNSwgMS4wXSwgWzQ3LjczNTYsIC0xMjIuMTk4LCAxLjBdLCBbNDcuNzM1NiwgLTEyMi4xOTEsIDEuMF0sIFs0Ny43MzU3LCAtMTIyLjMzMywgMS4wXSwgWzQ3LjczNTcsIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzM1NywgLTEyMi4xOTUsIDEuMF0sIFs0Ny43MzU3LCAtMTIyLjE5MywgMS4wXSwgWzQ3LjczNTgsIC0xMjIuMzQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzU4LCAtMTIyLjMzNSwgMS4wXSwgWzQ3LjczNTksIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzYwMDAwMDAwMDAwMDQsIC0xMjIuMjMxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzYwMDAwMDAwMDAwMDQsIC0xMjIuMTk5LCAxLjBdLCBbNDcuNzM2MSwgLTEyMi4yMzQsIDEuMF0sIFs0Ny43MzYyLCAtMTIyLjMzMywgMS4wXSwgWzQ3LjczNjIsIC0xMjIuMjQxLCAxLjBdLCBbNDcuNzM2MywgLTEyMi4yNTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczNjQsIC0xMjIuMTkzLCAxLjBdLCBbNDcuNzM2NSwgLTEyMi4zMzksIDEuMF0sIFs0Ny43MzY1LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjczNjUsIC0xMjIuMjQyLCAxLjBdLCBbNDcuNzM2NiwgLTEyMi4zNSwgMS4wXSwgWzQ3LjczNjYsIC0xMjIuMzMzLCAxLjBdLCBbNDcuNzM2NiwgLTEyMi4yMzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczNjYsIC0xMjIuMjA0LCAxLjBdLCBbNDcuNzM2NywgLTEyMi4xOTUsIDEuMF0sIFs0Ny43MzY4LCAtMTIyLjMzNywgMS4wXSwgWzQ3LjczNjgsIC0xMjIuMjMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MzY4LCAtMTIyLjIzMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzM2OCwgLTEyMi4xOTUsIDEuMF0sIFs0Ny43MzY4LCAtMTIyLjE4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzM2OSwgLTEyMi4zMzgsIDEuMF0sIFs0Ny43MzY5OTk5OTk5OTk5OTUsIC0xMjIuMzM0LCAxLjBdLCBbNDcuNzM2OTk5OTk5OTk5OTk1LCAtMTIyLjI0MywgMS4wXSwgWzQ3LjczNjk5OTk5OTk5OTk5NSwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjczNzIsIC0xMjIuMzUsIDEuMF0sIFs0Ny43MzcyLCAtMTIyLjIsIDEuMF0sIFs0Ny43MzczLCAtMTIyLjM1LCAxLjBdLCBbNDcuNzM3MywgLTEyMi4yNDEsIDEuMF0sIFs0Ny43MzczLCAtMTIyLjE5OCwgMS4wXSwgWzQ3LjczNzMsIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzM3NCwgLTEyMi4xODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczNzUsIC0xMjIuMTk0LCAxLjBdLCBbNDcuNzM3NiwgLTEyMi4yNDIsIDEuMF0sIFs0Ny43Mzc3LCAtMTIyLjE5NywgMS4wXSwgWzQ3LjczNzgsIC0xMjIuMjQ3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43Mzc5LCAtMTIyLjI0Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzM3OSwgLTEyMi4yMzI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczNzksIC0xMjIuMTg1LCAxLjBdLCBbNDcuNzM4LCAtMTIyLjIzNSwgMS4wXSwgWzQ3LjczODIsIC0xMjIuMTgyLCAxLjBdLCBbNDcuNzM4NCwgLTEyMi4zNDgsIDEuMF0sIFs0Ny43Mzg1LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjczODUsIC0xMjIuMjUsIDEuMF0sIFs0Ny43Mzg2LCAtMTIyLjIzOCwgMS4wXSwgWzQ3LjczODYsIC0xMjIuMjM1LCAxLjBdLCBbNDcuNzM4NywgLTEyMi4zMzcsIDEuMF0sIFs0Ny43Mzg4LCAtMTIyLjMzOSwgMS4wXSwgWzQ3LjczOSwgLTEyMi4yNTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjczOSwgLTEyMi4yNDksIDEuMF0sIFs0Ny43MzksIC0xMjIuMjQzLCAxLjBdLCBbNDcuNzM5LCAtMTIyLjIwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzM5LCAtMTIyLjE5NCwgMS4wXSwgWzQ3LjczOTEsIC0xMjIuMjQxLCAyLjBdLCBbNDcuNzM5MiwgLTEyMi4xOTQsIDEuMF0sIFs0Ny43MzkzLCAtMTIyLjI0NCwgMS4wXSwgWzQ3LjczOTMsIC0xMjIuMiwgMS4wXSwgWzQ3LjczOTQsIC0xMjIuMjUyMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny43Mzk0LCAtMTIyLjIwNCwgMS4wXSwgWzQ3LjczOTYsIC0xMjIuMjQ3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mzk3LCAtMTIyLjI0MiwgMS4wXSwgWzQ3LjczOTksIC0xMjIuMzMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mzk5LCAtMTIyLjIzNCwgMS4wXSwgWzQ3LjczOTksIC0xMjIuMTk3LCAxLjBdLCBbNDcuNzQsIC0xMjIuMzQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43NCwgLTEyMi4zMzksIDEuMF0sIFs0Ny43NDAxLCAtMTIyLjM0OCwgMS4wXSwgWzQ3Ljc0MDEsIC0xMjIuMjQ5LCAxLjBdLCBbNDcuNzQwMiwgLTEyMi4zNDgsIDEuMF1dLAogICAgICAgICAgICAgICAgeyJibHVyIjogMTUsICJtYXgiOiAxLjAsICJtYXhab29tIjogMTMsICJtaW5PcGFjaXR5IjogMC41LCAicmFkaXVzIjogOH0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMTM5NjM2ZmRhNWU1NGI0NzhjMDdiZmE4OWViY2FjOWIpOwogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Notice the congregation of sales in the Seattle city center. This areas very desirable because of their proximity to work centers and entertainment venues. Also notice the smaller blobs in the west of the Seattle. These features may show established suburbs, or a new development that came on line in the timeframe on the map.

#### Areas of High Density and High Price.
Below is a map of show areas with high volume (above 75 sales) and high prices, area with the 10% zone means.


```python
base_map = folium.Map(
    location=[data.lat.mean()-data.lat.std(), data.long.median()],
   
    zoom_start=9)

HeatMap(data= data[['lat', 'long', 'ones']].loc[(data.zone_count> 75) 
    & (data.zone_mean > data.zone_mean.quantile(.9))].groupby(['lat', 'long']).
        sum().reset_index().values.tolist(), radius=10, max_zoom=13).add_to(base_map)
scat_plot(data.loc[(data.zone_count> 75) & (data.zone_mean > data.zone_mean.quantile(.9))], 'sqft_living')
plt.title('Sales in High Density and High Cost Area\n V. Living Space')
plt.xlabel('sqft')
plt.ylabel('$')
base_map

```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF80OGQ4NTgxNWIwYzM0NjQ0OGJmZDI5YmIxYjI5MGE5OSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9sZWFmbGV0LmdpdGh1Yi5pby9MZWFmbGV0LmhlYXQvZGlzdC9sZWFmbGV0LWhlYXQuanMiPjwvc2NyaXB0Pgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzQ4ZDg1ODE1YjBjMzQ2NDQ4YmZkMjliYjFiMjkwYTk5IiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF80OGQ4NTgxNWIwYzM0NjQ0OGJmZDI5YmIxYjI5MGE5OSA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF80OGQ4NTgxNWIwYzM0NjQ0OGJmZDI5YmIxYjI5MGE5OSIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDcuNDIxNTQxMjI2MjI0MywgLTEyMi4yMzEwMDAwMDAwMDAwMV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA5LAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfZDk1ZGMwNjA0ZGZkNGQxMmE5ZGU2Zjk0MTkwOTg3ODkgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIkRhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzQ4ZDg1ODE1YjBjMzQ2NDQ4YmZkMjliYjFiMjkwYTk5KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgaGVhdF9tYXBfNGFiYzIyM2E1YzBkNDkzZmJmMTNhZDQ1MjZiYjlkNWMgPSBMLmhlYXRMYXllcigKICAgICAgICAgICAgICAgIFtbNDcuNjE2MDAwMDAwMDAwMDEsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjE2OCwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42MTY4LCAtMTIyLjMwMywgMS4wXSwgWzQ3LjYxNjksIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjE3LCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjYxNzIsIC0xMjIuMzEsIDEuMF0sIFs0Ny42MTczLCAtMTIyLjMxLCAxLjBdLCBbNDcuNjE3NCwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxNzcsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTc5LCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjYxNzksIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MTgsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjE4MSwgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MTgyLCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjE4OCwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxODgsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjE5LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjYxOTIsIC0xMjIuMzE5LCAxLjBdLCBbNDcuNjE5MiwgLTEyMi4zMTYsIDEuMF0sIFs0Ny42MTkyLCAtMTIyLjMwNywgMS4wXSwgWzQ3LjYxOTMsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjE5NSwgLTEyMi4zMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYxOTksIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjIsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNjIwMSwgLTEyMi4zMDksIDEuMF0sIFs0Ny42MjA4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYyMDksIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjEsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjIxNywgLTEyMi4zMDksIDEuMF0sIFs0Ny42MjIxLCAtMTIyLjMyNSwgMS4wXSwgWzQ3LjYyMjEsIC0xMjIuMzE0LCAxLjBdLCBbNDcuNjIyNCwgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyMjQsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjIyNSwgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MjI2LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYyMjYsIC0xMjIuMzEyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjI3LCAtMTIyLjMyMywgMS4wXSwgWzQ3LjYyMjcsIC0xMjIuMzEsIDEuMF0sIFs0Ny42MjI4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYyMjgsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjIzMDAwMDAwMDAwMDEsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjIzMywgLTEyMi4zMTksIDMuMF0sIFs0Ny42MjM1LCAtMTIyLjMwOSwgMS4wXSwgWzQ3LjYyMzUsIC0xMjIuMzA2LCAxLjBdLCBbNDcuNjIzNiwgLTEyMi4zMTgsIDEuMF0sIFs0Ny42MjM2LCAtMTIyLjMwNiwgMi4wXSwgWzQ3LjYyMzgsIC0xMjIuMzExLCAxLjBdLCBbNDcuNjI0MSwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MjQyLCAtMTIyLjMwNiwgMS4wXSwgWzQ3LjYyNDQsIC0xMjIuMzI2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjQ1LCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjYyNDYsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjQ4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYyNDksIC0xMjIuMzExLCAxLjBdLCBbNDcuNjI1NSwgLTEyMi4zMTQsIDEuMF0sIFs0Ny42MjU4LCAtMTIyLjMxMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI1OCwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42MjU5LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjYyNjAwMDAwMDAwMDAxLCAtMTIyLjMyMywgMS4wXSwgWzQ3LjYyNjEsIC0xMjIuMzI0LCAxLjBdLCBbNDcuNjI2MiwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42MjYzLCAtMTIyLjMxNCwgMi4wXSwgWzQ3LjYyNjMsIC0xMjIuMzEyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjY0LCAtMTIyLjMyMywgMS4wXSwgWzQ3LjYyNjUsIC0xMjIuMzIzLCAxLjBdLCBbNDcuNjI2NSwgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyNjk5OTk5OTk5OTk5NSwgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MjY5OTk5OTk5OTk5OTUsIC0xMjIuMzAzLCAxLjBdLCBbNDcuNjI3MSwgLTEyMi4zMjQsIDEuMF0sIFs0Ny42MjcyLCAtMTIyLjMxOCwgMS4wXSwgWzQ3LjYyNzIsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNjI3MiwgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyNzUsIC0xMjIuMzIxLCAxLjBdLCBbNDcuNjI3NSwgLTEyMi4zMTUsIDEuMF0sIFs0Ny42Mjc1LCAtMTIyLjMwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI3NSwgLTEyMi4zMDUsIDEuMF0sIFs0Ny42Mjc5LCAtMTIyLjMxNSwgMS4wXSwgWzQ3LjYyODIsIC0xMjIuMzIyLCAxLjBdLCBbNDcuNjI4MywgLTEyMi4zMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyODQsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNjI4NSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyODUsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjI4NSwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42Mjg3LCAtMTIyLjM1MSwgMS4wXSwgWzQ3LjYyODcsIC0xMjIuMywgMS4wXSwgWzQ3LjYyODcsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjI4OCwgLTEyMi4zLCAxLjBdLCBbNDcuNjI4OCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyODgsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjkxLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYyOTEsIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjI5MiwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42Mjk1LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjYyOTYsIC0xMjIuMywgMS4wXSwgWzQ3LjYyOTgsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mjk4LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjI5OSwgLTEyMi4zNTQsIDEuMF0sIFs0Ny42Mjk5LCAtMTIyLjI4LCAxLjBdLCBbNDcuNjMsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNjMwMSwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzAxLCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzMDMsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzA0LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzMDUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMwNSwgLTEyMi4zLCAxLjBdLCBbNDcuNjMwNiwgLTEyMi4yODgsIDEuMF0sIFs0Ny42MzA3LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjYzMDcsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMwOCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzA4LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYzMDgsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjMwOCwgLTEyMi4yNzksIDEuMF0sIFs0Ny42MzEyLCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjYzMTIsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjMxMiwgLTEyMi4yOTEsIDEuMF0sIFs0Ny42MzEzLCAtMTIyLjM3LCAxLjBdLCBbNDcuNjMxMywgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzEzLCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjYzMTQsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjMxNSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMTUsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzE1LCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMxNiwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMTcsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjMxOCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzE4LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjYzMTksIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzE5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzMiwgLTEyMi4zNjksIDEuMF0sIFs0Ny42MzIsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjMyLCAtMTIyLjI5LCAxLjBdLCBbNDcuNjMyLCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYzMiwgLTEyMi4yNzksIDEuMF0sIFs0Ny42MzIxLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMyMSwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42MzI0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYzMjUsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzI1LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjMzLCAtMTIyLjM3LCAxLjBdLCBbNDcuNjMzLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjMzLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMzMiwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMzIsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzMyLCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjYzMzMsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzM0LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMzNCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMzYsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzM3LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjMzNywgLTEyMi4zNjUsIDEuMF0sIFs0Ny42MzM3LCAtMTIyLjI4LCAxLjBdLCBbNDcuNjMzOCwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzMzksIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMzOSwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMzk5OTk5OTk5OTk5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzNDEsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjM0MSwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNDIsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjM0MiwgLTEyMi4yODksIDEuMF0sIFs0Ny42MzQyLCAtMTIyLjI4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM0MywgLTEyMi4zNTIsIDEuMF0sIFs0Ny42MzQzLCAtMTIyLjI4Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM0MywgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNDQsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM0NSwgLTEyMi4zNjcsIDEuMF0sIFs0Ny42MzQ1LCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjYzNDUsIC0xMjIuMzAyLCAxLjBdLCBbNDcuNjM0NiwgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNDcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjM0NywgLTEyMi4zNTIsIDEuMF0sIFs0Ny42MzQ4LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzNDgsIC0xMjIuMjgxLCAxLjBdLCBbNDcuNjM0OSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYzNSwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42MzUsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzUxLCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1MSwgLTEyMi4zNTIsIDEuMF0sIFs0Ny42MzUxLCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjYzNTIsIC0xMjIuMzU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzUyLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjYzNTIsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzUyLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1MywgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNTMsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjM1NCwgLTEyMi4zNTMsIDEuMF0sIFs0Ny42MzU2LCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1NiwgLTEyMi4zNjcsIDEuMF0sIFs0Ny42MzU2LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjYzNTcsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MzU4LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1OCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNTksIC0xMjIuMzUxLCAxLjBdLCBbNDcuNjM1OSwgLTEyMi4zLCAxLjBdLCBbNDcuNjM2LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYzNiwgLTEyMi4zMDEsIDEuMF0sIFs0Ny42MzYxLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM2MSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny42MzYxLCAtMTIyLjM2NiwgMS4wXSwgWzQ3LjYzNjIsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM2MiwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42MzYyLCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYzNjIsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzYzLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjM2MywgLTEyMi4zNjgsIDEuMF0sIFs0Ny42MzY0LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjM2NCwgLTEyMi4zNjMsIDEuMF0sIFs0Ny42MzY0LCAtMTIyLjM1MSwgMi4wXSwgWzQ3LjYzNjQsIC0xMjIuMywgMS4wXSwgWzQ3LjYzNjQsIC0xMjIuMjgsIDEuMF0sIFs0Ny42MzY1LCAtMTIyLjM2MSwgMS4wXSwgWzQ3LjYzNjYsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM2NiwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42MzY2LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzNjYsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzY3LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjM2NywgLTEyMi4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNjcsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjM2OCwgLTEyMi4zNzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNjgsIC0xMjIuMzcsIDEuMF0sIFs0Ny42MzY4LCAtMTIyLjM1NSwgMS4wXSwgWzQ3LjYzNjgsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjM2OCwgLTEyMi4zNTEsIDEuMF0sIFs0Ny42MzY4LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjYzNjksIC0xMjIuMzY1LCAxLjBdLCBbNDcuNjM2OSwgLTEyMi4zNTUsIDEuMF0sIFs0Ny42MzcsIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzcsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjM3LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzNywgLTEyMi4zNjEsIDEuMF0sIFs0Ny42MzcsIC0xMjIuMzU1LCAxLjBdLCBbNDcuNjM3MSwgLTEyMi4yNzksIDEuMF0sIFs0Ny42MzcyLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjYzNzMsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM3MywgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNzMsIC0xMjIuMywgMS4wXSwgWzQ3LjYzNzQsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM3NCwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzNzQsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjM3NCwgLTEyMi4yNzksIDEuMF0sIFs0Ny42Mzc1LCAtMTIyLjM1NCwgMS4wXSwgWzQ3LjYzNzYsIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjM3OCwgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzNzgsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNjM4MDAwMDAwMDAwMDEsIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzgwMDAwMDAwMDAwMSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYzODAwMDAwMDAwMDAxLCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzODIsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNjM4MiwgLTEyMi4zNTksIDEuMF0sIFs0Ny42Mzg0LCAtMTIyLjM3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM4NCwgLTEyMi4yNzksIDEuMF0sIFs0Ny42Mzg1LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjM4NSwgLTEyMi4yODEsIDEuMF0sIFs0Ny42Mzg2LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjYzODYsIC0xMjIuMzYsIDEuMF0sIFs0Ny42Mzg3LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzODcsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Mzg4LCAtMTIyLjMwMiwgMS4wXSwgWzQ3LjYzODgsIC0xMjIuMywgMS4wXSwgWzQ3LjYzODksIC0xMjIuMzcxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mzg5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzOTEsIC0xMjIuMzYsIDEuMF0sIFs0Ny42MzkzLCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjYzOTQsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Mzk0LCAtMTIyLjI5LCAxLjBdLCBbNDcuNjM5NCwgLTEyMi4yOCwgMS4wXSwgWzQ3LjYzOTUsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNjM5OCwgLTEyMi4zNTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0LCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjY0MDEsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDAyLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQwMiwgLTEyMi4zNywgMS4wXSwgWzQ3LjY0MDIsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDAzLCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQwNCwgLTEyMi4zNjEsIDEuMF0sIFs0Ny42NDA1LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjY0MDUsIC0xMjIuMzU2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDA1LCAtMTIyLjMwMSwgMS4wXSwgWzQ3LjY0MDgsIC0xMjIuMywgMS4wXSwgWzQ3LjY0MDksIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDEzLCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY0MTMsIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDE1LCAtMTIyLjQwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQxNywgLTEyMi40MTEsIDEuMF0sIFs0Ny42NDE5OTk5OTk5OTk5OTYsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjQyMiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MjQsIC0xMjIuNDExLCAyLjBdLCBbNDcuNjQyNCwgLTEyMi40MDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0MjUsIC0xMjIuNDA2LCAxLjBdLCBbNDcuNjQyNiwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0MjYsIC0xMjIuNDAyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDI3LCAtMTIyLjQwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQyOCwgLTEyMi40MTEsIDEuMF0sIFs0Ny42NDI5LCAtMTIyLjQxMiwgMS4wXSwgWzQ3LjY0MywgLTEyMi40MTIsIDEuMF0sIFs0Ny42NDMzLCAtMTIyLjQwNCwgMS4wXSwgWzQ3LjY0MzMsIC0xMjIuNDAyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDM0LCAtMTIyLjQxMiwgMS4wXSwgWzQ3LjY0MzQsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDM1LCAtMTIyLjM5OSwgMS4wXSwgWzQ3LjY0MzYsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjQzOCwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NDIsIC0xMjIuNDA2LCAxLjBdLCBbNDcuNjQ0NiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NDgsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDUyLCAtMTIyLjQxMSwgMS4wXSwgWzQ3LjY0NTMsIC0xMjIuNDEsIDEuMF0sIFs0Ny42NDUzLCAtMTIyLjQwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ1NSwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NTcsIC0xMjIuNDEyLCAxLjBdLCBbNDcuNjQ2MiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NjQsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjQ2NCwgLTEyMi40MDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NjUsIC0xMjIuNDA0LCAxLjBdLCBbNDcuNjQ2NiwgLTEyMi40MDQsIDEuMF0sIFs0Ny42NDcsIC0xMjIuNDEsIDEuMF0sIFs0Ny42NDcxLCAtMTIyLjQxMiwgMS4wXSwgWzQ3LjY0NzMsIC0xMjIuNDA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDczLCAtMTIyLjQwNiwgMS4wXSwgWzQ3LjY0NzQsIC0xMjIuNDEyLCAxLjBdLCBbNDcuNjQ3NCwgLTEyMi40MTEsIDEuMF0sIFs0Ny42NDc0LCAtMTIyLjQwMiwgMS4wXSwgWzQ3LjY0NzUsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDc5LCAtMTIyLjQwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ4LCAtMTIyLjQxLCAxLjBdLCBbNDcuNjQ4MSwgLTEyMi40MDYsIDEuMF0sIFs0Ny42NDgxLCAtMTIyLjQwNSwgMS4wXSwgWzQ3LjY0ODIsIC0xMjIuNDEyLCAxLjBdLCBbNDcuNjQ4MiwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0ODIsIC0xMjIuNDA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDgyLCAtMTIyLjQwMiwgMS4wXSwgWzQ3LjY0ODMsIC0xMjIuNDA0LCAxLjBdLCBbNDcuNjQ4OTk5OTk5OTk5OTk0LCAtMTIyLjQwMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ5MSwgLTEyMi40MDUsIDEuMF0sIFs0Ny42NDkzLCAtMTIyLjQxMywgMS4wXSwgWzQ3LjY0OTMsIC0xMjIuNDAxMDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42NDk1LCAtMTIyLjQwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ5NiwgLTEyMi40MTMsIDEuMF0sIFs0Ny42NDk2LCAtMTIyLjQwNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjQ5NiwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0OTksIC0xMjIuNDEzLCAxLjBdLCBbNDcuNjUsIC0xMjIuNDE1LCAxLjBdLCBbNDcuNjUwMSwgLTEyMi40MDg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1MDEsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjUwMywgLTEyMi40MSwgMi4wXSwgWzQ3LjY1MDYsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjUxLCAtMTIyLjQwODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjUxLCAtMTIyLjM5OSwgMS4wXSwgWzQ3LjY1MTEsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjUxMSwgLTEyMi40LCAxLjBdLCBbNDcuNjUxMywgLTEyMi40MDYsIDEuMF0sIFs0Ny42NTEzLCAtMTIyLjQsIDEuMF0sIFs0Ny42NTE0LCAtMTIyLjQxMywgMS4wXSwgWzQ3LjY1MTQsIC0xMjIuNDA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTE1LCAtMTIyLjQsIDEuMF0sIFs0Ny42NTE1LCAtMTIyLjM5OSwgMS4wXSwgWzQ3LjY1MTYsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NTE3LCAtMTIyLjQwNiwgMS4wXSwgWzQ3LjY1MTgsIC0xMjIuNCwgMS4wXSwgWzQ3LjY1MiwgLTEyMi40MDQsIDEuMF0sIFs0Ny42NTIxLCAtMTIyLjQxMywgMS4wXSwgWzQ3LjY1MjEsIC0xMjIuNCwgMi4wXSwgWzQ3LjY1MjMsIC0xMjIuNDEyLCAxLjBdLCBbNDcuNjUyNCwgLTEyMi40MDQsIDEuMF0sIFs0Ny42NTI4LCAtMTIyLjQwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjUyOCwgLTEyMi40MDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1MjksIC0xMjIuNDExLCAxLjBdLCBbNDcuNjUyOSwgLTEyMi40MDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1MywgLTEyMi40MTYsIDEuMF0sIFs0Ny42NTMsIC0xMjIuNDE1LCAxLjBdLCBbNDcuNjUzMSwgLTEyMi40MDUsIDEuMF0sIFs0Ny42NTMxLCAtMTIyLjQwMTAwMDAwMDAwMDAxLCAxLjBdXSwKICAgICAgICAgICAgICAgIHsiYmx1ciI6IDE1LCAibWF4IjogMS4wLCAibWF4Wm9vbSI6IDEzLCAibWluT3BhY2l0eSI6IDAuNSwgInJhZGl1cyI6IDEwfQogICAgICAgICAgICApLmFkZFRvKG1hcF80OGQ4NTgxNWIwYzM0NjQ0OGJmZDI5YmIxYjI5MGE5OSk7CiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




![png](student_files/student_245_1.png)


The above shows areas in Seattle north of downtown showing both high sale density and high pricing. The scatter plot shows a fairly linearity relation between living space and price, and heteroscedasticity is less pronounced. When examining like individual or grouped areas, many of the problem of the entire dataset seems to diminish.

#### Mapping Sales in the 99th Percentile  of Price


```python
base_map = folium.Map(
    location=[data.lat.mean()-data.lat.std(), data.long.median()],
    tiles='Stamen Terrain',
    zoom_start=9)

HeatMap(data= data[['lat', 'long', 'ones']].loc[(data.price > data.price.quantile(.99))].groupby(['lat', 'long']).
        sum().reset_index().values.tolist(), radius=15, max_zoom=9).add_to(base_map)
print('$',data.price.quantile(.99), ' to ', data.price.max())
base_map

```

    $ 1970000.0  to  7700000.0
    




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF84YjU5ZTFlMDRmMTg0ZGFiOWRiYjgwZDU0NjQ2ODMyYyB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9sZWFmbGV0LmdpdGh1Yi5pby9MZWFmbGV0LmhlYXQvZGlzdC9sZWFmbGV0LWhlYXQuanMiPjwvc2NyaXB0Pgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzhiNTllMWUwNGYxODRkYWI5ZGJiODBkNTQ2NDY4MzJjIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF84YjU5ZTFlMDRmMTg0ZGFiOWRiYjgwZDU0NjQ2ODMyYyA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF84YjU5ZTFlMDRmMTg0ZGFiOWRiYjgwZDU0NjQ2ODMyYyIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDcuNDIxNTQxMjI2MjI0MywgLTEyMi4yMzEwMDAwMDAwMDAwMV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA5LAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfNDNiYmUwMmViOTQ1NGVlMzk5NzgxMzMzNDExNGM1ZTUgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3N0YW1lbi10aWxlcy17c30uYS5zc2wuZmFzdGx5Lm5ldC90ZXJyYWluL3t6fS97eH0ve3l9LmpwZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIk1hcCB0aWxlcyBieSBcdTAwM2NhIGhyZWY9XCJodHRwOi8vc3RhbWVuLmNvbVwiXHUwMDNlU3RhbWVuIERlc2lnblx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vY3JlYXRpdmVjb21tb25zLm9yZy9saWNlbnNlcy9ieS8zLjBcIlx1MDAzZUNDIEJZIDMuMFx1MDAzYy9hXHUwMDNlLiBEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vY3JlYXRpdmVjb21tb25zLm9yZy9saWNlbnNlcy9ieS1zYS8zLjBcIlx1MDAzZUNDIEJZIFNBXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzhiNTllMWUwNGYxODRkYWI5ZGJiODBkNTQ2NDY4MzJjKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgaGVhdF9tYXBfMTdmYWUyYzZhZTM2NGU4NTkwYzk0ZDI1YjZhNWYzNGQgPSBMLmhlYXRMYXllcigKICAgICAgICAgICAgICAgIFtbNDcuNDE2OSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny40NTU4LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTI4MywgLTEyMi4yMDUsIDEuMF0sIFs0Ny41Mjg1LCAtMTIyLjIwNSwgMS4wXSwgWzQ3LjUzMTYsIC0xMjIuMjMyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MzE3LCAtMTIyLjIzMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM0OCwgLTEyMi4yNDMsIDEuMF0sIFs0Ny41MzU1LCAtMTIyLjI0LCAxLjBdLCBbNDcuNTM1OCwgLTEyMi4yMTMsIDEuMF0sIFs0Ny41MzcxLCAtMTIxLjk4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTM3MSwgLTEyMS43NTYsIDEuMF0sIFs0Ny41Mzc5LCAtMTIyLjI2NCwgMS4wXSwgWzQ3LjU0MDYsIC0xMjEuOTgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDMxLCAtMTIyLjExMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQ1NSwgLTEyMi4yMTEsIDEuMF0sIFs0Ny41NDU3LCAtMTIxLjk5MSwgMS4wXSwgWzQ3LjU0NjMsIC0xMjIuMzk3LCAxLjBdLCBbNDcuNTQ3NywgLTEyMi4xMjYsIDEuMF0sIFs0Ny41NDkxLCAtMTIyLjEwNCwgMS4wXSwgWzQ3LjU1MTUsIC0xMjIuMTEzLCAxLjBdLCBbNDcuNTUxNiwgLTEyMi4zOTgsIDEuMF0sIFs0Ny41NTQ3LCAtMTIyLjE0Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTU1MiwgLTEyMi4yMzEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1NTQsIC0xMjIuMDc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NTY5OTk5OTk5OTk5OTUsIC0xMjIuMjEsIDEuMF0sIFs0Ny41NjAyLCAtMTIyLjIyNywgMS4wXSwgWzQ3LjU2MTIsIC0xMjIuMjI5LCAxLjBdLCBbNDcuNTYyLCAtMTIyLjE2MiwgMS4wXSwgWzQ3LjU2MzEsIC0xMjIuMjEsIDEuMF0sIFs0Ny41NjM2LCAtMTIyLjIzMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTY0OCwgLTEyMi4yMSwgMS4wXSwgWzQ3LjU2NzUsIC0xMjIuMTg5LCAxLjBdLCBbNDcuNTY4MiwgLTEyMi4wNTksIDEuMF0sIFs0Ny41NjgzLCAtMTIyLjE4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY4NCwgLTEyMi4xOSwgMS4wXSwgWzQ3LjU2OTIsIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NjkyLCAtMTIyLjE4OSwgMS4wXSwgWzQ3LjU2OTYsIC0xMjIuMDksIDEuMF0sIFs0Ny41NzAxLCAtMTIyLjE4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTcwMywgLTEyMi4yOCwgMS4wXSwgWzQ3LjU3MDgsIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NzE2LCAtMTIyLjIwNCwgMS4wXSwgWzQ3LjU3MTk5OTk5OTk5OTk5NiwgLTEyMi4xMDIsIDEuMF0sIFs0Ny41NzIxLCAtMTIyLjIzODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTcyMiwgLTEyMi4yMzYsIDEuMF0sIFs0Ny41NzI0LCAtMTIyLjEwNCwgMS4wXSwgWzQ3LjU3MjgsIC0xMjIuMjA1LCAxLjBdLCBbNDcuNTc0NCwgLTEyMi4yODI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU3NDcsIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Nzg2LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU3OTUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNTgwMiwgLTEyMi4yMTIsIDEuMF0sIFs0Ny41ODExLCAtMTIyLjQsIDEuMF0sIFs0Ny41ODM1LCAtMTIyLjIwMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTg1LCAtMTIyLjIyMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTg4NCwgLTEyMi4wODYsIDEuMF0sIFs0Ny41OSwgLTEyMi4yMjksIDEuMF0sIFs0Ny41OTE5LCAtMTIyLjI1MSwgMS4wXSwgWzQ3LjU5MjIsIC0xMjIuMjA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41OTI1LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU5MjgsIC0xMjIuMDg2LCAxLjBdLCBbNDcuNTk0MywgLTEyMi4xMSwgMS4wXSwgWzQ3LjU5NTQsIC0xMjIuMjA2LCAxLjBdLCBbNDcuNTk2NCwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU5NjgsIC0xMjIuMDgzLCAxLjBdLCBbNDcuNjA0MiwgLTEyMi4xMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYwNSwgLTEyMi4xMTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYwNTMsIC0xMjIuMDc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MDcxLCAtMTIyLjIxMiwgMS4wXSwgWzQ3LjYwODMsIC0xMjIuMTEsIDEuMF0sIFs0Ny42MDkyLCAtMTIyLjA3MywgMS4wXSwgWzQ3LjYxMDIsIC0xMjIuMjI1LCAxLjBdLCBbNDcuNjE0NiwgLTEyMi4yMTMsIDEuMF0sIFs0Ny42MTUxLCAtMTIyLjIyMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjE1NCwgLTEyMi4yMSwgMS4wXSwgWzQ3LjYxNTUsIC0xMjIuMjM4LCAxLjBdLCBbNDcuNjE2NSwgLTEyMi4yMzYsIDEuMF0sIFs0Ny42MTY2LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjYxNjgsIC0xMjIuMjE2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MTcyLCAtMTIyLjIzLCAxLjBdLCBbNDcuNjE3NywgLTEyMi4yMjksIDEuMF0sIFs0Ny42MTc4LCAtMTIyLjIwOSwgMS4wXSwgWzQ3LjYxODEsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTgzLCAtMTIyLjIyNywgMS4wXSwgWzQ3LjYxODcsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MTkzLCAtMTIyLjIyOSwgMS4wXSwgWzQ3LjYxOTYsIC0xMjIuMjg2LCAxLjBdLCBbNDcuNjIsIC0xMjIuMjA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjA1LCAtMTIyLjIxNCwgMS4wXSwgWzQ3LjYyMDgsIC0xMjIuMjE5LCAxLjBdLCBbNDcuNjIwOSwgLTEyMi4yMzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyMTcsIC0xMjIuMjM4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjE4LCAtMTIyLjIzNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjIyMSwgLTEyMi4yMDgsIDEuMF0sIFs0Ny42MjI0LCAtMTIyLjIxNSwgMS4wXSwgWzQ3LjYyMjcsIC0xMjIuMjE2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjI5LCAtMTIyLjIyLCAxLjBdLCBbNDcuNjIzMDAwMDAwMDAwMDEsIC0xMjIuMjE3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjMyLCAtMTIyLjIyLCAxLjBdLCBbNDcuNjIzNiwgLTEyMi4yMzUsIDEuMF0sIFs0Ny42MjM5OTk5OTk5OTk5OTUsIC0xMjIuMjIxLCAxLjBdLCBbNDcuNjI0MiwgLTEyMi4yMzg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyNDUsIC0xMjIuMjEsIDEuMF0sIFs0Ny42MjUxLCAtMTIyLjIxNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI1OSwgLTEyMi4yOTEsIDEuMF0sIFs0Ny42MjYwMDAwMDAwMDAwMSwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42MjYzLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjYyNjMsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjY0LCAtMTIyLjI4OSwgMS4wXSwgWzQ3LjYyNzUsIC0xMjIuMzE1LCAxLjBdLCBbNDcuNjI3NywgLTEyMi4yODYsIDEuMF0sIFs0Ny42MjgxLCAtMTIyLjIxNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI4NSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyODUsIC0xMjIuMzYxLCAxLjBdLCBbNDcuNjI4NSwgLTEyMi4zMjIsIDEuMF0sIFs0Ny42Mjg1LCAtMTIyLjMwNCwgMS4wXSwgWzQ3LjYyODcsIC0xMjIuMjg3LCAxLjBdLCBbNDcuNjI4OCwgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYyODgsIC0xMjIuMjgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mjg5LCAtMTIyLjIzMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjI4OSwgLTEyMi4yMDUsIDEuMF0sIFs0Ny42MjkxLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYyOTUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjI5NSwgLTEyMi4yMzYsIDEuMF0sIFs0Ny42Mjk3LCAtMTIyLjIxNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjI5OCwgLTEyMi4zMjMsIDEuMF0sIFs0Ny42Mjk5LCAtMTIyLjI4LCAxLjBdLCBbNDcuNjMwMywgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMDQsIC0xMjIuMjM2LCAxLjBdLCBbNDcuNjMwNSwgLTEyMi4zNTQsIDEuMF0sIFs0Ny42MzA1LCAtMTIyLjI0LCAxLjBdLCBbNDcuNjMwNiwgLTEyMi4yODgsIDEuMF0sIFs0Ny42MzA3LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjYzMDcsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjMxMiwgLTEyMi4zNjYsIDEuMF0sIFs0Ny42MzEyLCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjYzMTIsIC0xMjIuMjIzLCAxLjBdLCBbNDcuNjMxNiwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMTksIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjMyMSwgLTEyMi4zOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYzMjEsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjMyMywgLTEyMi4xOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYzMjYsIC0xMjIuMjE2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MzM4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjYzMzgsIC0xMjIuMDcyLCAxLjBdLCBbNDcuNjMzOTk5OTk5OTk5OTksIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjM0NCwgLTEyMi4yMTUsIDEuMF0sIFs0Ny42MzQ0LCAtMTIyLjIxNCwgMS4wXSwgWzQ3LjYzNDUsIC0xMjIuMzY3LCAxLjBdLCBbNDcuNjM0OSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYzNTIsIC0xMjIuMjksIDEuMF0sIFs0Ny42MzU0LCAtMTIyLjIyMSwgMS4wXSwgWzQ3LjYzNjEsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjM2NCwgLTEyMi4yMTQsIDEuMF0sIFs0Ny42MzY0LCAtMTIyLjIwOSwgMS4wXSwgWzQ3LjYzNzQsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjM3NywgLTEyMi4yMTksIDEuMF0sIFs0Ny42Mzc5LCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjYzODAwMDAwMDAwMDAxLCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzODcsIC0xMjIuNDA1LCAxLjBdLCBbNDcuNjM4OCwgLTEyMi40MDYsIDEuMF0sIFs0Ny42Mzg5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjYzOTMsIC0xMjIuMDk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Mzk1LCAtMTIyLjIzNCwgMS4wXSwgWzQ3LjYzOTgsIC0xMjIuMjM3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDA5LCAtMTIyLjI0MSwgMS4wXSwgWzQ3LjY0MDksIC0xMjIuMjIxLCAxLjBdLCBbNDcuNjQxMywgLTEyMi4yNCwgMS4wXSwgWzQ3LjY0MTUsIC0xMjIuMjg1LCAxLjBdLCBbNDcuNjQyNywgLTEyMi40MDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NDgsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDUzLCAtMTIyLjQxLCAxLjBdLCBbNDcuNjQ1NCwgLTEyMi4yMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0NjUsIC0xMjIuMzE5LCAxLjBdLCBbNDcuNjQ4LCAtMTIyLjIxNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ4OCwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0OTksIC0xMjIuMjE2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NSwgLTEyMi4yMTQsIDEuMF0sIFs0Ny42NTE1LCAtMTIyLjI3NywgMS4wXSwgWzQ3LjY1NDcsIC0xMjIuMjAyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjA4LCAtMTIyLjI2ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjYxLCAtMTIyLjI2ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjYyMiwgLTEyMi4yNzIsIDEuMF0sIFs0Ny42NjQ2LCAtMTIyLjIwOCwgMS4wXSwgWzQ3LjY2NzUsIC0xMjEuOTg2LCAxLjBdLCBbNDcuNjY5NiwgLTEyMi4yNjEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3MDEsIC0xMjIuMjU5LCAxLjBdLCBbNDcuNjcwMiwgLTEyMi4yNiwgMS4wXSwgWzQ3LjY3MDksIC0xMjIuNDA2LCAxLjBdLCBbNDcuNjcxNSwgLTEyMi40MDYsIDEuMF0sIFs0Ny42NzY3LCAtMTIyLjIxMSwgMS4wXSwgWzQ3LjY3OTUsIC0xMjEuOTkxLCAxLjBdLCBbNDcuNjgwMywgLTEyMi4yMTQsIDEuMF0sIFs0Ny42ODY0LCAtMTIyLjIxNSwgMS4wXSwgWzQ3LjY4NzgsIC0xMjIuMjE1LCAxLjBdLCBbNDcuNjg4MiwgLTEyMi4yMTQsIDEuMF0sIFs0Ny42ODk5LCAtMTIyLjIxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjkzNCwgLTEyMi4yNzEsIDEuMF0sIFs0Ny42OTY3LCAtMTIyLjIxNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk5MSwgLTEyMi4yMzUsIDEuMF0sIFs0Ny42OTk3LCAtMTIyLjI0LCAxLjBdLCBbNDcuNzAxMSwgLTEyMi4yNDQsIDEuMF0sIFs0Ny43MDIyLCAtMTIyLjIyMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzA2NywgLTEyMi4zOCwgMS4wXSwgWzQ3LjcwODcsIC0xMjIuMjc2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MTU5LCAtMTIyLjI1MSwgMS4wXSwgWzQ3LjcxNzUsIC0xMjIuMjc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MjA1LCAtMTIyLjI2LCAxLjBdLCBbNDcuNzI4OCwgLTEyMi4zNywgMS4wXSwgWzQ3LjcyOTUsIC0xMjIuMzcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Mjk2LCAtMTIyLjM3LCAxLjBdLCBbNDcuNzMzNCwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3Ljc0OTMsIC0xMjIuMjgsIDEuMF0sIFs0Ny43NjEyLCAtMTIyLjM4MSwgMS4wXV0sCiAgICAgICAgICAgICAgICB7ImJsdXIiOiAxNSwgIm1heCI6IDEuMCwgIm1heFpvb20iOiA5LCAibWluT3BhY2l0eSI6IDAuNSwgInJhZGl1cyI6IDE1fQogICAgICAgICAgICApLmFkZFRvKG1hcF84YjU5ZTFlMDRmMTg0ZGFiOWRiYjgwZDU0NjQ2ODMyYyk7CiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



The high sales price tend to be concentrated around the waterfront. There seems be a particular concentration around the Lake Washington in Bellevue and a piece of Seattle West across the Lake.

#### Mapping Sales
Below is a heatmap marking individual with one weight (a value 1), showing density sale throughout the county.


```python
base_map = folium.Map(
    location=[data.lat.mean()-data.lat.std(), data.long.median()],
    tiles='Stamen Terrain',
    zoom_start=9)

HeatMap(data= data[['lat', 'long', 'ones']].groupby(['lat', 'long']).
        sum().reset_index().values.tolist(), radius=7, max_zoom=9, min_opacity=.99).add_to(base_map)
base_map
```







#### Mapping Sales Density of New Homes
The map shows high density sales areas on new homes (built since 2012). The weight for each sales is one. 


```python
base_map = folium.Map(
    location=[data.lat.mean()-data.lat.std(), data.long.median()],
    tiles='Stamen Terrain',
    zoom_start=9)


HeatMap(data= data[['lat', 'long', 'ones']].loc[(data.yr_built >= 2012)].groupby(['lat', 'long']).
        sum().reset_index().values.tolist(), radius=10, max_zoom=9, min_opacity=.99).add_to(base_map)
base_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF80ZjExMDllMmE5NTA0MGEzYjFkYWQ5MzI3MTgxMTdiZiB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9sZWFmbGV0LmdpdGh1Yi5pby9MZWFmbGV0LmhlYXQvZGlzdC9sZWFmbGV0LWhlYXQuanMiPjwvc2NyaXB0Pgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzRmMTEwOWUyYTk1MDQwYTNiMWRhZDkzMjcxODExN2JmIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF80ZjExMDllMmE5NTA0MGEzYjFkYWQ5MzI3MTgxMTdiZiA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF80ZjExMDllMmE5NTA0MGEzYjFkYWQ5MzI3MTgxMTdiZiIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDcuNDIxNTQxMjI2MjI0MywgLTEyMi4yMzEwMDAwMDAwMDAwMV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA5LAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfN2M0YWEwYTI4NjA4NDViN2JjNDQwZWRhYTIwOTIzYzIgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3N0YW1lbi10aWxlcy17c30uYS5zc2wuZmFzdGx5Lm5ldC90ZXJyYWluL3t6fS97eH0ve3l9LmpwZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIk1hcCB0aWxlcyBieSBcdTAwM2NhIGhyZWY9XCJodHRwOi8vc3RhbWVuLmNvbVwiXHUwMDNlU3RhbWVuIERlc2lnblx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vY3JlYXRpdmVjb21tb25zLm9yZy9saWNlbnNlcy9ieS8zLjBcIlx1MDAzZUNDIEJZIDMuMFx1MDAzYy9hXHUwMDNlLiBEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vY3JlYXRpdmVjb21tb25zLm9yZy9saWNlbnNlcy9ieS1zYS8zLjBcIlx1MDAzZUNDIEJZIFNBXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzRmMTEwOWUyYTk1MDQwYTNiMWRhZDkzMjcxODExN2JmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgaGVhdF9tYXBfYTE1NzVlNTMzNTgxNGI2NDg5ZDk2ZjY3NzgwNGRhMjggPSBMLmhlYXRMYXllcigKICAgICAgICAgICAgICAgIFtbNDcuMTk0OCwgLTEyMS45NzUsIDIuMF0sIFs0Ny4xOTQ4LCAtMTIxLjk3Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMjEzNywgLTEyMS45ODg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjI1NzcsIC0xMjIuMTk4LCAxLjBdLCBbNDcuMjU4MSwgLTEyMi4yLCAxLjBdLCBbNDcuMjU4MiwgLTEyMi4xOTgsIDEuMF0sIFs0Ny4yNTgyLCAtMTIyLjE5NiwgMS4wXSwgWzQ3LjI1ODMsIC0xMjIuMjAyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4yNTgzLCAtMTIyLjE5OCwgMS4wXSwgWzQ3LjI1ODQsIC0xMjIuMTk2LCAxLjBdLCBbNDcuMjU4NSwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjI1ODYsIC0xMjIuMiwgMS4wXSwgWzQ3LjI1ODYsIC0xMjIuMTk0LCAxLjBdLCBbNDcuMjU4OSwgLTEyMi4yNTYsIDEuMF0sIFs0Ny4yNTg5OTk5OTk5OTk5OSwgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjI1OTIsIC0xMjIuMTk0LCAxLjBdLCBbNDcuMjU5NiwgLTEyMi4xOTQsIDEuMF0sIFs0Ny4yNTk3LCAtMTIyLjE5OSwgMS4wXSwgWzQ3LjI1OTksIC0xMjIuMjgyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4yNTk5LCAtMTIyLjI1NiwgMS4wXSwgWzQ3LjI1OTksIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4yNjAyLCAtMTIyLjI4Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMjYwMiwgLTEyMi4yNTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjI2MDIsIC0xMjIuMjQ2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4yNjAzLCAtMTIyLjE5NSwgMS4wXSwgWzQ3LjI2MDMsIC0xMjIuMTk0LCAyLjBdLCBbNDcuMjYwNiwgLTEyMi4yNTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjI2MTEsIC0xMjIuMTk4LCAxLjBdLCBbNDcuMjYxNSwgLTEyMi4xOTgsIDEuMF0sIFs0Ny4yNjc2LCAtMTIyLjI1NiwgMS4wXSwgWzQ3LjI2ODMsIC0xMjEuOTQ2LCAxLjBdLCBbNDcuMjcxNiwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjI3NTIsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4yODI0LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjI4NTEsIC0xMjIuMjc3LCAxLjBdLCBbNDcuMjg3MywgLTEyMi4xNzcsIDEuMF0sIFs0Ny4yOTA0LCAtMTIyLjI2NCwgMS4wXSwgWzQ3LjI5MDUsIC0xMjIuMjY0LCAxLjBdLCBbNDcuMjkxMSwgLTEyMi4yNjYsIDEuMF0sIFs0Ny4yOTMxLCAtMTIyLjI2NCwgMi4wXSwgWzQ3LjI5NTMsIC0xMjIuMjY1LCAxLjBdLCBbNDcuMjk1NiwgLTEyMi4zNSwgMS4wXSwgWzQ3LjI5NTgsIC0xMjIuMjY1LCAxLjBdLCBbNDcuMjk2MDAwMDAwMDAwMDEsIC0xMjIuMzUsIDEuMF0sIFs0Ny4yOTYxLCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjI5NjIsIC0xMjIuMzUsIDEuMF0sIFs0Ny4yOTY2LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjI5NywgLTEyMi4zNTEsIDEuMF0sIFs0Ny4yOTcyLCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjI5NzQsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4yOTc1LCAtMTIyLjM1LCAyLjBdLCBbNDcuMzAwMywgLTEyMi4yNjI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjMwMDgsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zMDA5LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjMwNTIsIC0xMjIuMjc2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zMDU0LCAtMTIyLjI3NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzA3NywgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjMwOTUsIC0xMjIuMDAyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zMTYyLCAtMTIyLjI3MiwgMS4wXSwgWzQ3LjMxOTgsIC0xMjIuMTc4LCAxLjBdLCBbNDcuMzIsIC0xMjIuMTc4LCAxLjBdLCBbNDcuMzIwNCwgLTEyMi4xNzgsIDEuMF0sIFs0Ny4zMjA1LCAtMTIyLjE3OCwgMS4wXSwgWzQ3LjMyMTEsIC0xMjIuMTgyLCAxLjBdLCBbNDcuMzI1MiwgLTEyMi4xNjcsIDEuMF0sIFs0Ny4zMjU5LCAtMTIyLjE2NywgMS4wXSwgWzQ3LjMyNjEsIC0xMjIuMTYzLCAxLjBdLCBbNDcuMzI2MiwgLTEyMi4xNjUsIDEuMF0sIFs0Ny4zMjY5LCAtMTIyLjE2NSwgMS4wXSwgWzQ3LjMyNzMsIC0xMjIuMTYzLCAxLjBdLCBbNDcuMzI3NiwgLTEyMi4xNjMsIDEuMF0sIFs0Ny4zMjgxLCAtMTIyLjE2NCwgMS4wXSwgWzQ3LjMzMDcsIC0xMjIuMTg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zMzU5LCAtMTIyLjI1NywgMS4wXSwgWzQ3LjMzOCwgLTEyMi4xODEsIDEuMF0sIFs0Ny4zMzg0LCAtMTIyLjE4LCAxLjBdLCBbNDcuMzM5MywgLTEyMi4xODEsIDEuMF0sIFs0Ny4zNDA0LCAtMTIyLjE4MSwgMS4wXSwgWzQ3LjM0MSwgLTEyMi4xOCwgMS4wXSwgWzQ3LjM0MSwgLTEyMi4xNzksIDEuMF0sIFs0Ny4zNDEyLCAtMTIyLjE3OSwgMS4wXSwgWzQ3LjM0MTIsIC0xMjIuMTc4LCAxLjBdLCBbNDcuMzQxMywgLTEyMi4xOCwgMS4wXSwgWzQ3LjM0MTMsIC0xMjIuMTc4LCAxLjBdLCBbNDcuMzQyMSwgLTEyMi4xNzksIDEuMF0sIFs0Ny4zNDI1LCAtMTIyLjE3OSwgMS4wXSwgWzQ3LjM0MjcsIC0xMjIuMjc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNDI4LCAtMTIyLjI3Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzQyOCwgLTEyMi4xNzksIDEuMF0sIFs0Ny4zNDMsIC0xMjIuMDU2LCAxLjBdLCBbNDcuMzQ0MywgLTEyMi4zMjcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM0NDgsIC0xMjIuMjE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNDQ4LCAtMTIyLjAyNCwgMS4wXSwgWzQ3LjM0NSwgLTEyMi4yMSwgMS4wXSwgWzQ3LjM0NSwgLTEyMi4yMDksIDEuMF0sIFs0Ny4zNDUxLCAtMTIyLjIxNCwgMS4wXSwgWzQ3LjM0NTEsIC0xMjIuMjA5LCAxLjBdLCBbNDcuMzQ1MiwgLTEyMi4yMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM0NTIsIC0xMjIuMjEsIDEuMF0sIFs0Ny4zNDUyLCAtMTIyLjIwOSwgMS4wXSwgWzQ3LjM0NTQsIC0xMjIuMjE0LCAyLjBdLCBbNDcuMzQ1NSwgLTEyMi4yMSwgMS4wXSwgWzQ3LjM0NTUsIC0xMjIuMDIzLCAxLjBdLCBbNDcuMzQ1NywgLTEyMi4yMTcwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjM0NjAwMDAwMDAwMDAwNCwgLTEyMi4yMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM0NjEsIC0xMjIuMTUyLCAxLjBdLCBbNDcuMzQ3OSwgLTEyMi4wMjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM0ODIsIC0xMjIuMDQyLCAxLjBdLCBbNDcuMzQ4MywgLTEyMi4wNDIsIDEuMF0sIFs0Ny4zNTA3LCAtMTIyLjI5MSwgMS4wXSwgWzQ3LjM1MTEsIC0xMjIuMjc1LCAxLjBdLCBbNDcuMzUxMiwgLTEyMi4xMzUsIDEuMF0sIFs0Ny4zNTEzLCAtMTIyLjE3Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzUxNSwgLTEyMi4yNzUsIDEuMF0sIFs0Ny4zNTE1LCAtMTIyLjEzNCwgMS4wXSwgWzQ3LjM1MTgsIC0xMjIuMjc1LCAxLjBdLCBbNDcuMzUxOSwgLTEyMi4xOTcsIDEuMF0sIFs0Ny4zNTE5LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzUyLCAtMTIyLjE5NywgMS4wXSwgWzQ3LjM1MjIsIC0xMjIuMjc1LCAxLjBdLCBbNDcuMzUyMiwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM1MjMsIC0xMjIuMjc1LCAxLjBdLCBbNDcuMzUyNCwgLTEyMi4xOTgsIDIuMF0sIFs0Ny4zNTI0LCAtMTIyLjE3NSwgMS4wXSwgWzQ3LjM1MjUsIC0xMjIuMjc1LCAxLjBdLCBbNDcuMzUzMiwgLTEyMi4yMTEsIDEuMF0sIFs0Ny4zNTM1LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzU0NSwgLTEyMi4wNTYsIDEuMF0sIFs0Ny4zNTQ3LCAtMTIyLjA2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzU1MSwgLTEyMi4wNjEsIDEuMF0sIFs0Ny4zNTUyLCAtMTIyLjAyMywgMS4wXSwgWzQ3LjM1NTUsIC0xMjIuMDYyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNTU1LCAtMTIyLjA2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzU1NSwgLTEyMi4wNjEsIDEuMF0sIFs0Ny4zNTU1LCAtMTIyLjA1NCwgMS4wXSwgWzQ3LjM1NTcsIC0xMjIuMDYyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNTU5LCAtMTIyLjA2Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzU1OSwgLTEyMi4wMjMsIDEuMF0sIFs0Ny4zNTU5LCAtMTIyLjAyMiwgMS4wXSwgWzQ3LjM1NiwgLTEyMi4wNTcsIDIuMF0sIFs0Ny4zNTYxLCAtMTIyLjA2Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzU2MiwgLTEyMi4wMjIsIDEuMF0sIFs0Ny4zNTY4LCAtMTIyLjE2MywgMS4wXSwgWzQ3LjM1NjgsIC0xMjIuMDU1LCAxLjBdLCBbNDcuMzU3NCwgLTEyMi4zMTgsIDEuMF0sIFs0Ny4zNTg0LCAtMTIyLjE2MywgMS4wXSwgWzQ3LjM1ODQsIC0xMjIuMDgzLCAxLjBdLCBbNDcuMzU4NywgLTEyMi4xNjMsIDEuMF0sIFs0Ny4zNTg4LCAtMTIyLjE2MywgMS4wXSwgWzQ3LjM1ODgsIC0xMjIuMDgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTg5LCAtMTIyLjA4MywgMS4wXSwgWzQ3LjM1ODk5OTk5OTk5OTk5NSwgLTEyMi4wODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM1ODk5OTk5OTk5OTk5NSwgLTEyMi4wODEsIDEuMF0sIFs0Ny4zNTkxLCAtMTIyLjIwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzU5NCwgLTEyMi4yMDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM1OTQsIC0xMjIuMDgyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNTk1LCAtMTIyLjA0NSwgMS4wXSwgWzQ3LjM1OTUsIC0xMjIuMDQyLCAxLjBdLCBbNDcuMzU5NiwgLTEyMi4wODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM1OTcsIC0xMjIuMDQ1LCAxLjBdLCBbNDcuMzU5OSwgLTEyMi4wODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjM2MDEsIC0xMjIuMDMyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNjAyLCAtMTIyLjA4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzYwMywgLTEyMi4wODEsIDEuMF0sIFs0Ny4zNjEyLCAtMTIyLjEwMywgMS4wXSwgWzQ3LjM2MTIsIC0xMjIuMDgxLCAxLjBdLCBbNDcuMzYxMywgLTEyMi4wODEsIDEuMF0sIFs0Ny4zNjU4LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzY2MDAwMDAwMDAwMDEsIC0xMjIuMzA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjY3LCAtMTIyLjMwNywgMS4wXSwgWzQ3LjM2NzEsIC0xMjIuMTEzLCAxLjBdLCBbNDcuMzY4MiwgLTEyMi4xMTcsIDEuMF0sIFs0Ny4zNjg2LCAtMTIyLjExNywgMS4wXSwgWzQ3LjM2ODksIC0xMjIuMDU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNjksIC0xMjIuMjg3LCAxLjBdLCBbNDcuMzY5MywgLTEyMi4yODYsIDEuMF0sIFs0Ny4zNjkzLCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuMzY5NiwgLTEyMi4wMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM2OTcsIC0xMjIuMTE3LCAxLjBdLCBbNDcuMzY5OSwgLTEyMi4wMTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM3LCAtMTIyLjAxODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzcwNiwgLTEyMi4wMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM3MDYsIC0xMjIuMDE3MDAwMDAwMDAwMDEsIDMuMF0sIFs0Ny4zNzEsIC0xMjIuMDE3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny4zNzEsIC0xMjIuMDE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zNzE0LCAtMTIyLjE5NywgMS4wXSwgWzQ3LjM3MiwgLTEyMi4xMywgMS4wXSwgWzQ3LjM3MjYsIC0xMjIuMTU4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny4zNzMxLCAtMTIyLjE1ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuMzczOCwgLTEyMi4xNjEsIDEuMF0sIFs0Ny4zNzQ2LCAtMTIyLjE4OSwgMS4wXSwgWzQ3LjM3NTEsIC0xMjIuMTg5LCAxLjBdLCBbNDcuMzc1MSwgLTEyMi4xODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjM3OSwgLTEyMi4xODQsIDEuMF0sIFs0Ny4zODIsIC0xMjIuMTk3LCAxLjBdLCBbNDcuMzgyNiwgLTEyMi4wMzYsIDEuMF0sIFs0Ny4zODMsIC0xMjIuMDk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny4zODM1LCAtMTIyLjA5NSwgMS4wXSwgWzQ3LjM4NSwgLTEyMi4xODksIDEuMF0sIFs0Ny4zOTM4LCAtMTIyLjMyMSwgMS4wXSwgWzQ3LjM5OTQsIC0xMjIuMzExLCAxLjBdLCBbNDcuNDIxMywgLTEyMi4xOTQsIDEuMF0sIFs0Ny40Mjk1LCAtMTIyLjIwNSwgMS4wXSwgWzQ3LjQyOTksIC0xMjIuMjA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40MzEwMDAwMDAwMDAwMDQsIC0xMjIuMzM1LCAxLjBdLCBbNDcuNDM0OSwgLTEyMi4zMjc5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjQzNSwgLTEyMi4zMjksIDEuMF0sIFs0Ny40MzgsIC0xMjIuMzQ0LCAxLjBdLCBbNDcuNDQwOCwgLTEyMi4xNjEsIDEuMF0sIFs0Ny40NDY3LCAtMTIyLjI4NywgMS4wXSwgWzQ3LjQ0NzcsIC0xMjIuMjA0LCAxLjBdLCBbNDcuNDQ5MywgLTEyMi4yODEsIDEuMF0sIFs0Ny40NTEzLCAtMTIyLjI2NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDUxNCwgLTEyMi4yNzMsIDEuMF0sIFs0Ny40NTE3LCAtMTIyLjIwNCwgMS4wXSwgWzQ3LjQ1MzYsIC0xMjIuMjc0LCAxLjBdLCBbNDcuNDU1NywgLTEyMi4xMywgMS4wXSwgWzQ3LjQ1NzksIC0xMjIuNDQzLCAxLjBdLCBbNDcuNDU5MywgLTEyMi4xMzYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ1OTUsIC0xMjIuMjA4LCAxLjBdLCBbNDcuNDY5MiwgLTEyMi4zNTEsIDEuMF0sIFs0Ny40Njk0LCAtMTIxLjk4ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDY5OCwgLTEyMi4xMjEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ3MDIsIC0xMjIuMjc1LCAxLjBdLCBbNDcuNDcyNiwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ3MzksIC0xMjIuMTEsIDEuMF0sIFs0Ny40NzUsIC0xMjEuNzM3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40NzU5LCAtMTIxLjczNSwgMS4wXSwgWzQ3LjQ3NTksIC0xMjEuNzM0LCAyLjBdLCBbNDcuNDc2MSwgLTEyMS43MzQsIDEuMF0sIFs0Ny40NzY1LCAtMTIxLjczNCwgMS4wXSwgWzQ3LjQ3NjYsIC0xMjEuNzM1LCAxLjBdLCBbNDcuNDc3NSwgLTEyMi4xMjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ3NzYsIC0xMjIuMTIyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40Nzc5LCAtMTIyLjEyMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNDc4MywgLTEyMi4xMjI5OTk5OTk5OTk5OSwgMi4wXSwgWzQ3LjQ3ODQsIC0xMjIuMTIyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40Nzg2LCAtMTIyLjEyMjk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNDc4NiwgLTEyMi4xMjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ3ODcsIC0xMjIuMTIyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny40NzksIC0xMjIuMTI0LCAxLjBdLCBbNDcuNDc5NSwgLTEyMi4xMjYsIDEuMF0sIFs0Ny40Nzk3LCAtMTIyLjIzLCAxLjBdLCBbNDcuNDc5NywgLTEyMi4xMjYsIDEuMF0sIFs0Ny40ODEyLCAtMTIyLjEyMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDgxNSwgLTEyMi4xNTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjQ4MjIsIC0xMjIuMTMxLCAxLjBdLCBbNDcuNDgyMywgLTEyMi4xMjQsIDEuMF0sIFs0Ny40ODIzLCAtMTIxLjc3MiwgMS4wXSwgWzQ3LjQ4MjMsIC0xMjEuNzcxLCAxLjBdLCBbNDcuNDgyNiwgLTEyMS43NzEsIDEuMF0sIFs0Ny40ODI3LCAtMTIyLjEzMSwgMi4wXSwgWzQ3LjQ4MjcsIC0xMjEuNzczLCAxLjBdLCBbNDcuNDgzMSwgLTEyMi4xMzUsIDEuMF0sIFs0Ny40ODMxLCAtMTIxLjc3MywgMi4wXSwgWzQ3LjQ4MzIsIC0xMjEuNzc0LCAxLjBdLCBbNDcuNDgzMiwgLTEyMS43NzIsIDEuMF0sIFs0Ny40ODM0LCAtMTIxLjc3MywgMS4wXSwgWzQ3LjQ4MzYsIC0xMjEuNzY4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODM4LCAtMTIxLjc3MywgMS4wXSwgWzQ3LjQ4MzgsIC0xMjEuNzY4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODM5OTk5OTk5OTk5OTUsIC0xMjEuNzcxLCAxLjBdLCBbNDcuNDg0MSwgLTEyMS43NzIsIDEuMF0sIFs0Ny40ODQyLCAtMTIxLjc3MiwgMS4wXSwgWzQ3LjQ4NTMsIC0xMjIuMTU0LCAxLjBdLCBbNDcuNDg1NSwgLTEyMi4xNDksIDEuMF0sIFs0Ny40ODU2LCAtMTIyLjE1NCwgMS4wXSwgWzQ3LjQ4NTcsIC0xMjIuMTUyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODYyLCAtMTIyLjE2MywgMS4wXSwgWzQ3LjQ4NjMsIC0xMjIuMTQyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40ODY1LCAtMTIyLjE0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDg2NiwgLTEyMi4zMTksIDEuMF0sIFs0Ny40ODY2LCAtMTIyLjE1NSwgMS4wXSwgWzQ3LjQ4Njk5OTk5OTk5OTk5NSwgLTEyMi4zMiwgMi4wXSwgWzQ3LjQ4NzMsIC0xMjIuMTY2LCAxLjBdLCBbNDcuNDg4OSwgLTEyMi4zMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ5MDksIC0xMjIuMTQzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTE0LCAtMTIyLjMzNCwgMS4wXSwgWzQ3LjQ5MTQsIC0xMjIuMjUyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny40OTE3LCAtMTIyLjE3LCAxLjBdLCBbNDcuNDkxOSwgLTEyMi4zMzYsIDEuMF0sIFs0Ny40OTIyLCAtMTIyLjE1Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNDk0NywgLTEyMi4xNzEsIDEuMF0sIFs0Ny40OTUsIC0xMjIuMTQ1LCAxLjBdLCBbNDcuNDk1NCwgLTEyMi4xNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjQ5NTQsIC0xMjIuMTUyLCAxLjBdLCBbNDcuNDk2LCAtMTIyLjE2OSwgMS4wXSwgWzQ3LjQ5NjksIC0xMjIuMTQ1LCAxLjBdLCBbNDcuNDk3MywgLTEyMi4zNDYsIDEuMF0sIFs0Ny40OTk5LCAtMTIyLjIzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTAwMSwgLTEyMi4yMzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUwMTQsIC0xMjIuMzQxMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MDE0LCAtMTIyLjM0LCAxLjBdLCBbNDcuNTAxNiwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUwMTcsIC0xMjIuMzQsIDEuMF0sIFs0Ny41MDE5LCAtMTIyLjM0LCAyLjBdLCBbNDcuNTAxOSwgLTEyMi4xNTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUwMzMsIC0xMjIuMTkzLCAxLjBdLCBbNDcuNTA0NCwgLTEyMi4xNywgMS4wXSwgWzQ3LjUwNDcsIC0xMjIuMTcsIDEuMF0sIFs0Ny41MDQ5LCAtMTIyLjE3LCAxLjBdLCBbNDcuNTA1MSwgLTEyMi4xNTUsIDEuMF0sIFs0Ny41MDU5LCAtMTIyLjE0NiwgMS4wXSwgWzQ3LjUwNiwgLTEyMi4xNDUsIDEuMF0sIFs0Ny41MDY1LCAtMTIyLjE3MSwgMS4wXSwgWzQ3LjUwNywgLTEyMi4xODEsIDEuMF0sIFs0Ny41MDgyLCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTA4OTk5OTk5OTk5OTksIC0xMjIuMjQsIDEuMF0sIFs0Ny41MTA3LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTEwOCwgLTEyMi4zNjMsIDIuMF0sIFs0Ny41MTA4LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTExNCwgLTEyMi4xOTYsIDEuMF0sIFs0Ny41MTE3LCAtMTIyLjM2NSwgMS4wXSwgWzQ3LjUxMjMsIC0xMjIuMzE2LCAxLjBdLCBbNDcuNTEyNywgLTEyMi4xNjksIDEuMF0sIFs0Ny41MTM0LCAtMTIyLjM0MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTEzNCwgLTEyMi4yLCAxLjBdLCBbNDcuNTEzNSwgLTEyMi4xNjksIDEuMF0sIFs0Ny41MTM3LCAtMTIyLjE2OSwgMS4wXSwgWzQ3LjUxMzcsIC0xMjIuMTY3LCAxLjBdLCBbNDcuNTE1LCAtMTIxLjg3MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTE1LCAtMTIxLjg3LCAxLjBdLCBbNDcuNTE1LCAtMTIxLjg2OSwgMi4wXSwgWzQ3LjUxNjIsIC0xMjEuODc0LCAxLjBdLCBbNDcuNTE2NCwgLTEyMi4xOSwgMi4wXSwgWzQ3LjUxNjQsIC0xMjEuODc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MTY1LCAtMTIyLjM0NiwgMi4wXSwgWzQ3LjUxNjUsIC0xMjEuODgzLCAxLjBdLCBbNDcuNTE2NywgLTEyMi4zNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUxNjcsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNTE2OCwgLTEyMi4zNzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUxNjgsIC0xMjIuMTg5LCAxLjBdLCBbNDcuNTE2OCwgLTEyMS44ODMsIDIuMF0sIFs0Ny41MTY4LCAtMTIxLjg3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTE2OSwgLTEyMS44ODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUxNjksIC0xMjEuODg0LCAxLjBdLCBbNDcuNTE2OSwgLTEyMS44Nzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUxNjk5OTk5OTk5OTk5NiwgLTEyMS44NzYsIDEuMF0sIFs0Ny41MTcxLCAtMTIyLjM0NzAwMDAwMDAwMDAxLCA0LjBdLCBbNDcuNTE3MSwgLTEyMi4zNDYsIDEuMF0sIFs0Ny41MTcxLCAtMTIxLjg3NiwgMS4wXSwgWzQ3LjUxNzIsIC0xMjIuMjA4LCAxLjBdLCBbNDcuNTE3MywgLTEyMi4yODYsIDEuMF0sIFs0Ny41MTc4LCAtMTIxLjg4NywgMS4wXSwgWzQ3LjUxODEsIC0xMjEuODg1LCAxLjBdLCBbNDcuNTE4MywgLTEyMS44ODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUxODQsIC0xMjEuODg2MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny41MTg1LCAtMTIxLjg4NSwgMS4wXSwgWzQ3LjUxODUsIC0xMjEuODg0LCAxLjBdLCBbNDcuNTE4NywgLTEyMi4yMDgsIDEuMF0sIFs0Ny41MTksIC0xMjEuODc3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MTkzLCAtMTIxLjg3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTE5NiwgLTEyMS44Nzc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUyMDEsIC0xMjIuMjA0LCAxLjBdLCBbNDcuNTIwMSwgLTEyMS44ODUsIDEuMF0sIFs0Ny41MjAyLCAtMTIxLjg4NSwgMS4wXSwgWzQ3LjUyMDMsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MjM1LCAtMTIyLjE2Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTIzOTk5OTk5OTk5OTk0LCAtMTIyLjIsIDEuMF0sIFs0Ny41MjQxLCAtMTIyLjI4NSwgMS4wXSwgWzQ3LjUyNDUsIC0xMjIuMTg0LCAxLjBdLCBbNDcuNTI1MiwgLTEyMi4xOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyNTQsIC0xMjEuODg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MjU5LCAtMTIyLjI3OSwgMS4wXSwgWzQ3LjUyNjUsIC0xMjEuODg3LCAxLjBdLCBbNDcuNTI4MywgLTEyMi4zODUsIDEuMF0sIFs0Ny41MjgzLCAtMTIyLjIwNSwgMS4wXSwgWzQ3LjUyODUsIC0xMjIuMjA1LCAxLjBdLCBbNDcuNTI4NiwgLTEyMi4xODcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUyOTQsIC0xMjIuMTU0LCAxLjBdLCBbNDcuNTI5NywgLTEyMi4xNTUsIDEuMF0sIFs0Ny41Mjk3LCAtMTIyLjA3MywgMS4wXSwgWzQ3LjUyOTksIC0xMjIuMjAyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41MywgLTEyMi4wNzMsIDIuMF0sIFs0Ny41MzA1LCAtMTIyLjE4NCwgMS4wXSwgWzQ3LjUzMDgsIC0xMjIuMTg0LCAxLjBdLCBbNDcuNTMxMSwgLTEyMi4wNzQsIDEuMF0sIFs0Ny41MzExLCAtMTIyLjA3MywgMS4wXSwgWzQ3LjUzMTMsIC0xMjIuMDc2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzEzLCAtMTIyLjA3NCwgMS4wXSwgWzQ3LjUzMiwgLTEyMi4zODQsIDEuMF0sIFs0Ny41MzIxLCAtMTIyLjAzMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTMyMywgLTEyMi4wNzEsIDMuMF0sIFs0Ny41MzIzLCAtMTIyLjA3LCAxLjBdLCBbNDcuNTMyNSwgLTEyMi4wNzIsIDEuMF0sIFs0Ny41MzM2LCAtMTIxLjg0MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMzNiwgLTEyMS44MzgsIDEuMF0sIFs0Ny41MzM3LCAtMTIxLjg0MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTMzOCwgLTEyMS44NDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNCwgLTEyMS44NDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNDIsIC0xMjEuODQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41MzQyLCAtMTIxLjg0MTAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNTM2MywgLTEyMi4zNzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjUzNywgLTEyMi4xODUsIDEuMF0sIFs0Ny41MzcxLCAtMTIyLjM4Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM4MDAwMDAwMDAwMDA0LCAtMTIyLjExMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTM4MiwgLTEyMi4xMTM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjUzODIsIC0xMjIuMTExLCAxLjBdLCBbNDcuNTM4NywgLTEyMi4zNjcsIDEuMF0sIFs0Ny41Mzg4LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjUzODgsIC0xMjIuMTEzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzg5LCAtMTIyLjM2NywgMS4wXSwgWzQ3LjUzOTQsIC0xMjIuMDI3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Mzk0LCAtMTIyLjAyNywgMi4wXSwgWzQ3LjU0MDIsIC0xMjIuMzg3LCAzLjBdLCBbNDcuNTQwNCwgLTEyMi4yNjc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MDksIC0xMjIuMzksIDEuMF0sIFs0Ny41NDE4LCAtMTIyLjAxLCAxLjBdLCBbNDcuNTQxOSwgLTEyMi4zODc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0MjEsIC0xMjIuMDExMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDI5LCAtMTIyLjAxMiwgMS4wXSwgWzQ3LjU0MywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0MywgLTEyMi4zNjgsIDEuMF0sIFs0Ny41NDMsIC0xMjIuMDEsIDEuMF0sIFs0Ny41NDMxLCAtMTIyLjAxMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQzMiwgLTEyMi4zNzYsIDEuMF0sIFs0Ny41NDMzLCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjU0MzQsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNTQzOCwgLTEyMi4zODUsIDEuMF0sIFs0Ny41NDM5LCAtMTIyLjAxMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTQzOSwgLTEyMS44NjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU0NDEsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNTQ0MSwgLTEyMi4xMTcsIDEuMF0sIFs0Ny41NDQxLCAtMTIyLjAxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTQ0MiwgLTEyMi4xMTYsIDEuMF0sIFs0Ny41NDQzLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjU0NDQsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNTQ0NSwgLTEyMi4wMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU0NDYsIC0xMjIuMDE3MDAwMDAwMDAwMDEsIDQuMF0sIFs0Ny41NDQ2LCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjU0NDksIC0xMjIuMDExMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDUsIC0xMjIuMzY4LCAxLjBdLCBbNDcuNTQ1NSwgLTEyMi4zNjgsIDEuMF0sIFs0Ny41NDU1LCAtMTIyLjIxMSwgMS4wXSwgWzQ3LjU0NTksIC0xMjIuMzc3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDU5LCAtMTIyLjM2OCwgMS4wXSwgWzQ3LjU0NjIsIC0xMjIuMTgyLCAxLjBdLCBbNDcuNTQ2MywgLTEyMi4xODIsIDEuMF0sIFs0Ny41NDcsIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NDgxLCAtMTIyLjEwMywgMS4wXSwgWzQ3LjU0ODIsIC0xMjIuMzg3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NDg3LCAtMTIyLjI3MiwgMS4wXSwgWzQ3LjU0OTEsIC0xMjIuMTA0LCAxLjBdLCBbNDcuNTQ5MiwgLTEyMi4zODcsIDEuMF0sIFs0Ny41NDkzLCAtMTIyLjM4NywgMS4wXSwgWzQ3LjU0OTksIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTEzLCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUxNCwgLTEyMi4wMjMsIDEuMF0sIFs0Ny41NTE0LCAtMTIxLjk5Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUxNywgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MTgsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41NTE5LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU1MjEsIC0xMjEuOTk5LCAxLjBdLCBbNDcuNTUyMSwgLTEyMS45OTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU1MjIsIC0xMjIuMzgyLCAxLjBdLCBbNDcuNTUyMiwgLTEyMi4wMDEsIDEuMF0sIFs0Ny41NTIyLCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU1MjMsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTI0LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU1MjQsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTI1LCAtMTIyLjI3MywgMS4wXSwgWzQ3LjU1MjUsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTI2LCAtMTIxLjk5OSwgMS4wXSwgWzQ3LjU1MjYsIC0xMjEuOTk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NTI4LCAtMTIyLjA3NjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTUyOCwgLTEyMS45OTksIDIuMF0sIFs0Ny41NTI5LCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTUyOSwgLTEyMi4yODEsIDEuMF0sIFs0Ny41NTMwMDAwMDAwMDAwMDQsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNTUzMiwgLTEyMi4yODIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU1MzgsIC0xMjIuMTg5LCAxLjBdLCBbNDcuNTUzOTk5OTk5OTk5OTk1LCAtMTIyLjE5LCAxLjBdLCBbNDcuNTUzOTk5OTk5OTk5OTk1LCAtMTIyLjE4OSwgMS4wXSwgWzQ3LjU1NDIsIC0xMjIuMzU5LCAxLjBdLCBbNDcuNTU0MywgLTEyMi4zNTksIDEuMF0sIFs0Ny41NTU0LCAtMTIyLjI2NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTU1NSwgLTEyMi4zNjMsIDIuMF0sIFs0Ny41NTU1LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTU1NiwgLTEyMi4zNjMsIDEuMF0sIFs0Ny41NTU2LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTU2OSwgLTEyMi4yMjUsIDEuMF0sIFs0Ny41NTczLCAtMTIyLjI4NywgMS4wXSwgWzQ3LjU1NzMsIC0xMjIuMDIsIDEuMF0sIFs0Ny41NTc4LCAtMTIyLjM2MywgMS4wXSwgWzQ3LjU1OTUsIC0xMjIuMzY1LCAxLjBdLCBbNDcuNTYwNiwgLTEyMi4wMTEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2MDcsIC0xMjIuMDMxLCAxLjBdLCBbNDcuNTYxOCwgLTEyMi4wMjcsIDEuMF0sIFs0Ny41NjE5LCAtMTIyLjAyNywgMS4wXSwgWzQ3LjU2MjIsIC0xMjIuMjkyLCAxLjBdLCBbNDcuNTYyNiwgLTEyMi4wMDUsIDEuMF0sIFs0Ny41NjMzLCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjU2MzcsIC0xMjIuMjk1LCAxLjBdLCBbNDcuNTYzOCwgLTEyMi4yOTUsIDEuMF0sIFs0Ny41NjM5LCAtMTIyLjI5MiwgMS4wXSwgWzQ3LjU2MzksIC0xMjIuMjIzLCAxLjBdLCBbNDcuNTY0MiwgLTEyMi4yOTIsIDEuMF0sIFs0Ny41NjQ2LCAtMTIyLjI5NSwgMS4wXSwgWzQ3LjU2NDcsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjQ4LCAtMTIyLjI5NCwgMi4wXSwgWzQ3LjU2NTYsIC0xMjIuNDAyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjYyLCAtMTIyLjM3ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTY2NCwgLTEyMi4xMjg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU2NjYsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NjY3LCAtMTIyLjI5NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTY2OSwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU2NzEsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41Njc0LCAtMTIyLjM5MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTY4MywgLTEyMi4zMTQsIDEuMF0sIFs0Ny41Njg0LCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjU2ODcsIC0xMjIuMjkxLCAxLjBdLCBbNDcuNTY5OSwgLTEyMi4yOTYsIDEuMF0sIFs0Ny41Njk5LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU3LCAtMTIyLjI4OCwgMS4wXSwgWzQ3LjU3MDcsIC0xMjIuMjE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41NzExLCAtMTIyLjI4NiwgMS4wXSwgWzQ3LjU3MjcsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzUzLCAtMTIyLjI5NCwgMS4wXSwgWzQ3LjU3NTQsIC0xMjIuMDcxLCAxLjBdLCBbNDcuNTc1NSwgLTEyMi4wNzEsIDEuMF0sIFs0Ny41NzU2LCAtMTIyLjMxNiwgMS4wXSwgWzQ3LjU3NTYsIC0xMjIuMDcxLCAxLjBdLCBbNDcuNTc2MSwgLTEyMi4yMDUsIDEuMF0sIFs0Ny41NzYyLCAtMTIyLjQxNSwgMS4wXSwgWzQ3LjU3NjMsIC0xMjIuNDA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny41NzY0LCAtMTIyLjQwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTc2NSwgLTEyMi4wNzIsIDEuMF0sIFs0Ny41Nzc0LCAtMTIyLjQxMiwgMS4wXSwgWzQ3LjU3NzQsIC0xMjIuMjIyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41Nzc1LCAtMTIyLjQxLCAxLjBdLCBbNDcuNTc4NCwgLTEyMi4yMjYsIDEuMF0sIFs0Ny41Nzg5OTk5OTk5OTk5OSwgLTEyMi4zODksIDEuMF0sIFs0Ny41ODAyLCAtMTIyLjAzOSwgMS4wXSwgWzQ3LjU4MDYsIC0xMjIuMDU1LCAxLjBdLCBbNDcuNTgxLCAtMTIyLjAyMiwgMS4wXSwgWzQ3LjU4MTYsIC0xMjIuMDIxLCAxLjBdLCBbNDcuNTgxNywgLTEyMi4wNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU4MTgsIC0xMjIuNDAyLCAzLjBdLCBbNDcuNTgxOCwgLTEyMi4wNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU4MTgsIC0xMjEuOTk2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41ODE5LCAtMTIyLjA0NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTgyMSwgLTEyMi4wNDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU4NjksIC0xMjIuMzExLCAxLjBdLCBbNDcuNTg2OSwgLTEyMi4wNTQsIDEuMF0sIFs0Ny41ODcyLCAtMTIyLjA1NSwgMS4wXSwgWzQ3LjU4NzYsIC0xMjIuMzA5LCAxLjBdLCBbNDcuNTg4NSwgLTEyMS45MzksIDEuMF0sIFs0Ny41OTAyLCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjU5MTQsIC0xMjIuMDI3LCAxLjBdLCBbNDcuNTkxNywgLTEyMi4zODYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjU5MTksIC0xMjEuOTc1LCAxLjBdLCBbNDcuNTkyLCAtMTIyLjI5Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkyMywgLTEyMi4wNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5MjMsIC0xMjEuOTczLCAxLjBdLCBbNDcuNTkyOSwgLTEyMi4zMDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5MjksIC0xMjIuMDU3LCAxLjBdLCBbNDcuNTkyOSwgLTEyMS45NzM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5MywgLTEyMi4wNTcsIDEuMF0sIFs0Ny41OTMzLCAtMTIyLjA2MSwgMS4wXSwgWzQ3LjU5MzMsIC0xMjIuMDYsIDEuMF0sIFs0Ny41OTM0LCAtMTIxLjk3Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTkzNSwgLTEyMi4wNjEsIDEuMF0sIFs0Ny41OTM3LCAtMTIyLjA2MSwgMS4wXSwgWzQ3LjU5NDEsIC0xMjEuOTczLCAxLjBdLCBbNDcuNTk0MiwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5NTEsIC0xMjIuMzAxLCAxLjBdLCBbNDcuNTk1OSwgLTEyMi4wMTQsIDEuMF0sIFs0Ny41OTYwMDAwMDAwMDAwMDQsIC0xMjIuMjk3OTk5OTk5OTk5OTksIDIuMF0sIFs0Ny41OTYzLCAtMTIyLjIsIDEuMF0sIFs0Ny41OTY0LCAtMTIyLjIwMTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNTk2NSwgLTEyMi4wMTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjU5NzQsIC0xMjIuMjAyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny41OTc0LCAtMTIyLjAxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNTk3NSwgLTEyMS45ODUsIDEuMF0sIFs0Ny41OTksIC0xMjIuMTk3LCAxLjBdLCBbNDcuNTk5MSwgLTEyMi4yLCAxLjBdLCBbNDcuNjAwMSwgLTEyMi4yOTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjYwMTgsIC0xMjIuMjk3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MDE4LCAtMTIyLjA2LCAxLjBdLCBbNDcuNjAyLCAtMTIyLjMxMSwgMS4wXSwgWzQ3LjYwMjIsIC0xMjIuMDYsIDEuMF0sIFs0Ny42MDI2LCAtMTIyLjA2LCAxLjBdLCBbNDcuNjAzNiwgLTEyMi4wNTksIDEuMF0sIFs0Ny42MDM5LCAtMTIyLjE5NCwgMS4wXSwgWzQ3LjYwNDcsIC0xMjIuMzA1LCAxLjBdLCBbNDcuNjA1LCAtMTIyLjMwNCwgMi4wXSwgWzQ3LjYwNTEsIC0xMjIuMzA0LCAxLjBdLCBbNDcuNjA2NywgLTEyMi4wNTMsIDIuMF0sIFs0Ny42MDY3LCAtMTIyLjA1MiwgMS4wXSwgWzQ3LjYwNywgLTEyMi4wNTMsIDIuMF0sIFs0Ny42MDcyLCAtMTIyLjA1NCwgMS4wXSwgWzQ3LjYwNzIsIC0xMjIuMDUzLCAxLjBdLCBbNDcuNjA3MywgLTEyMi4zMDQsIDEuMF0sIFs0Ny42MDc0LCAtMTIyLjMwNSwgMS4wXSwgWzQ3LjYwODgsIC0xMjIuMzExLCAyLjBdLCBbNDcuNjEwMiwgLTEyMi4yMjUsIDEuMF0sIFs0Ny42MTEzLCAtMTIyLjMxNCwgMS4wXSwgWzQ3LjYxMjIsIC0xMjIuMDU5LCAxLjBdLCBbNDcuNjE2OCwgLTEyMi4yMTYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYxNzIsIC0xMjIuMzEsIDEuMF0sIFs0Ny42MTczLCAtMTIyLjMxLCAxLjBdLCBbNDcuNjE3MywgLTEyMi4wNTYsIDEuMF0sIFs0Ny42MTc1LCAtMTIyLjA1NiwgMS4wXSwgWzQ3LjYxNzgsIC0xMjIuMjksIDEuMF0sIFs0Ny42MTc4LCAtMTIyLjA1NSwgMS4wXSwgWzQ3LjYxODEsIC0xMjIuMDU2LCAyLjBdLCBbNDcuNjE5NiwgLTEyMi4yOTcwMDAwMDAwMDAwMSwgMi4wXSwgWzQ3LjYxOTgsIC0xMjIuMjg5LCAxLjBdLCBbNDcuNjE5OSwgLTEyMi4zLCAxLjBdLCBbNDcuNjIsIC0xMjIuMjA3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjA1LCAtMTIyLjMsIDEuMF0sIFs0Ny42MjA1LCAtMTIyLjI5ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIwNiwgLTEyMi4zLCAxLjBdLCBbNDcuNjIwOSwgLTEyMi4yMzcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjYyMTIsIC0xMjIuMDU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjIxLCAtMTIyLjI5LCAxLjBdLCBbNDcuNjIyMSwgLTEyMi4yMDgsIDEuMF0sIFs0Ny42MjI1LCAtMTIyLjA1Nzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjIyOCwgLTEyMi4zMDcsIDEuMF0sIFs0Ny42MjMyLCAtMTIyLjAyMywgMS4wXSwgWzQ3LjYyMzUsIC0xMjIuMTkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjM2LCAtMTIyLjAyMywgMS4wXSwgWzQ3LjYyMzcsIC0xMjIuMDIzLCAxLjBdLCBbNDcuNjIzOSwgLTEyMi4yOSwgMS4wXSwgWzQ3LjYyNTYsIC0xMjIuMjk4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42MjU4LCAtMTIyLjAwNSwgMS4wXSwgWzQ3LjYyNTksIC0xMjIuMTQyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42MjYyLCAtMTIyLjMyMywgMS4wXSwgWzQ3LjYyNzEsIC0xMjIuMzI0LCAxLjBdLCBbNDcuNjI4OSwgLTEyMi4yMDUsIDEuMF0sIFs0Ny42MjkxLCAtMTIyLjM2MywgMS4wXSwgWzQ3LjYzMDIsIC0xMjIuMzQ0LCAxLjBdLCBbNDcuNjMxMywgLTEyMi4zNywgMS4wXSwgWzQ3LjYzMjIsIC0xMjIuMjA5LCAxLjBdLCBbNDcuNjMzMiwgLTEyMi4xOTksIDEuMF0sIFs0Ny42MzUxLCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM1MywgLTEyMi4zNTMsIDEuMF0sIFs0Ny42MzU0LCAtMTIyLjM0NiwgMS4wXSwgWzQ3LjYzNTQsIC0xMjIuMTk4LCAxLjBdLCBbNDcuNjM1NiwgLTEyMi4zNDYsIDEuMF0sIFs0Ny42MzU2LCAtMTIyLjI4MSwgMS4wXSwgWzQ3LjYzNjIsIC0xMjIuMzY5LCAxLjBdLCBbNDcuNjM2OCwgLTEyMi4yNzksIDEuMF0sIFs0Ny42MzcyLCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjYzODQsIC0xMjIuMzcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42Mzg0LCAtMTIyLjM0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjM5NSwgLTEyMi4zMjksIDEuMF0sIFs0Ny42Mzk2LCAtMTIyLjMyOSwgMS4wXSwgWzQ3LjY0MDgsIC0xMjIuMzA3LCAxLjBdLCBbNDcuNjQxMDAwMDAwMDAwMDEsIC0xMjIuMzU3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NDIzLCAtMTIyLjM3NSwgMS4wXSwgWzQ3LjY0MjQsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjQyNSwgLTEyMi4zNzQsIDIuMF0sIFs0Ny42NDM0LCAtMTIyLjMyNCwgMS4wXSwgWzQ3LjY0MzUsIC0xMjIuMzU0LCAxLjBdLCBbNDcuNjQ0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY0NTEsIC0xMjIuMzc1LCAxLjBdLCBbNDcuNjQ2MSwgLTEyMi4zNjIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY0NzIsIC0xMjIuMzgzLCAxLjBdLCBbNDcuNjQ3OSwgLTEyMi40MDc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY0ODIsIC0xMjIuNDA4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NDgyLCAtMTIyLjQwNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjQ4OTk5OTk5OTk5OTk0LCAtMTIyLjM4MywgMS4wXSwgWzQ3LjY0OTMsIC0xMjIuMzg0LCAxLjBdLCBbNDcuNjQ5NywgLTEyMi4zMzksIDEuMF0sIFs0Ny42NTA2LCAtMTIyLjE5NSwgMS4wXSwgWzQ3LjY1MTQsIC0xMjIuMzg2MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NTE0LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY1MjEsIC0xMjIuNCwgMi4wXSwgWzQ3LjY1MzIsIC0xMjIuMzQ4LCAxLjBdLCBbNDcuNjUzNiwgLTEyMi4zNTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY1MzYsIC0xMjIuMzU0LCA0LjBdLCBbNDcuNjU0NywgLTEyMi4yMDIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY1NSwgLTEyMi4wMDksIDEuMF0sIFs0Ny42NTU4LCAtMTIyLjM0OCwgMS4wXSwgWzQ3LjY1NzIsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjU3NSwgLTEyMi4wMDYsIDEuMF0sIFs0Ny42NTgsIC0xMjIuMDA2LCAxLjBdLCBbNDcuNjU4NSwgLTEyMi4zNDgsIDEuMF0sIFs0Ny42NjA3LCAtMTIyLjM2MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjYxMSwgLTEyMi4zNDYsIDEuMF0sIFs0Ny42NjI4LCAtMTIyLjE0Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjYzNiwgLTEyMi4zMTksIDIuMF0sIFs0Ny42NjM4LCAtMTIyLjMxOSwgMS4wXSwgWzQ3LjY2NDQsIC0xMjIuMjkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NjQ2LCAtMTIyLjM2NywgMi4wXSwgWzQ3LjY2NDcsIC0xMjIuMjgzOTk5OTk5OTk5OTksIDQuMF0sIFs0Ny42NjQ4LCAtMTIyLjM1MywgMS4wXSwgWzQ3LjY2NDksIC0xMjIuMzUzLCAxLjBdLCBbNDcuNjY1MSwgLTEyMi40MDI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY2NzksIC0xMjIuMTcyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Njg1LCAtMTIyLjMzMjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjY4OCwgLTEyMi40MDIsIDEuMF0sIFs0Ny42Njg5LCAtMTIyLjM2MywgMy4wXSwgWzQ3LjY2ODksIC0xMjIuMzYyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NjksIC0xMjIuNDAyLCAyLjBdLCBbNDcuNjY5MSwgLTEyMi40MDIsIDEuMF0sIFs0Ny42NjkyLCAtMTIyLjM3MjAwMDAwMDAwMDAxLCAyLjBdLCBbNDcuNjcsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42NzA2LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjcwOSwgLTEyMi4xNjYsIDEuMF0sIFs0Ny42NzIyLCAtMTIyLjM4NCwgMS4wXSwgWzQ3LjY3MjIsIC0xMjIuMzgxLCAxLjBdLCBbNDcuNjcyNSwgLTEyMi4zMywgMS4wXSwgWzQ3LjY3MzUsIC0xMjIuMDU3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzQsIC0xMjIuMzg3LCAyLjBdLCBbNDcuNjc0LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc0LCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc0MSwgLTEyMi4zODQsIDEuMF0sIFs0Ny42NzQzLCAtMTIyLjE4NCwgMS4wXSwgWzQ3LjY3NDcsIC0xMjIuMzkyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42NzQ4LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc1NywgLTEyMi4zMjYwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NTgsIC0xMjIuMzgsIDEuMF0sIFs0Ny42NzU5LCAtMTIyLjE1MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc2LCAtMTIyLjE5NywgMS4wXSwgWzQ3LjY3NjEsIC0xMjIuMTUyLCAxLjBdLCBbNDcuNjc2NCwgLTEyMi4zOTIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY3NjUsIC0xMjIuMzIsIDIuMF0sIFs0Ny42NzcsIC0xMjIuMzMsIDEuMF0sIFs0Ny42NzcxLCAtMTIyLjE4NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjc4MDAwMDAwMDAwMDA0LCAtMTIyLjExNywgMS4wXSwgWzQ3LjY3ODEsIC0xMjIuMTE3LCAxLjBdLCBbNDcuNjc4MiwgLTEyMi4yOTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY3ODUsIC0xMjIuMzkyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny42Nzg5LCAtMTIyLjI3MywgMS4wXSwgWzQ3LjY3ODk5OTk5OTk5OTk5NSwgLTEyMi4zNzUsIDEuMF0sIFs0Ny42Nzk3LCAtMTIyLjM4NSwgMS4wXSwgWzQ3LjY3OTgsIC0xMjIuMzg1LCAxLjBdLCBbNDcuNjc5OSwgLTEyMi4zODUsIDEuMF0sIFs0Ny42ODAyLCAtMTIyLjIwOCwgMS4wXSwgWzQ3LjY4MDUsIC0xMjIuMzQ2LCAxLjBdLCBbNDcuNjgxMywgLTEyMi4zNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MTQsIC0xMjIuMjg4LCAxLjBdLCBbNDcuNjgxOSwgLTEyMi4xNzIwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4MjYsIC0xMjIuMzkzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODI2LCAtMTIyLjE4NiwgMS4wXSwgWzQ3LjY4MjgsIC0xMjIuMTg2LCAxLjBdLCBbNDcuNjgzLCAtMTIyLjAxNSwgMS4wXSwgWzQ3LjY4NDIsIC0xMjIuMTk2LCAxLjBdLCBbNDcuNjg0MywgLTEyMi4zMDUsIDEuMF0sIFs0Ny42ODQzLCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjY4NDQsIC0xMjIuMDE2LCAxLjBdLCBbNDcuNjg0NSwgLTEyMi4zNTksIDEuMF0sIFs0Ny42ODQ1LCAtMTIyLjIwNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NSwgLTEyMi4wMTcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY4NDYsIC0xMjIuMzQ4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODQ3LCAtMTIyLjAxNzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjg0NywgLTEyMi4wMTYsIDEuMF0sIFs0Ny42ODQ4LCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjY4NDksIC0xMjIuMDE3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42ODQ5LCAtMTIyLjAxNiwgMS4wXSwgWzQ3LjY4NSwgLTEyMi4wMTYsIDEuMF0sIFs0Ny42ODUyLCAtMTIyLjIxLCAxLjBdLCBbNDcuNjg1MywgLTEyMi4yMSwgMi4wXSwgWzQ3LjY4NTMsIC0xMjIuMjA0LCAxLjBdLCBbNDcuNjg2OSwgLTEyMi4wMTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4NzMsIC0xMjIuMzMzLCAxLjBdLCBbNDcuNjg3NSwgLTEyMi4zMywgMS4wXSwgWzQ3LjY4OCwgLTEyMi4wMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY4ODQsIC0xMjIuMzc0LCAxLjBdLCBbNDcuNjg4NSwgLTEyMi4wMjEsIDEuMF0sIFs0Ny42ODg2LCAtMTIyLjM2Mzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjg5OCwgLTEyMi4wMTUsIDEuMF0sIFs0Ny42OTAxLCAtMTIyLjAxNzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjkwMSwgLTEyMi4wMTUsIDEuMF0sIFs0Ny42OTA2LCAtMTIyLjAxODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjkwOCwgLTEyMi4zNjUsIDEuMF0sIFs0Ny42OTExLCAtMTIyLjMxMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjkxMSwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5MTQsIC0xMjIuMjEsIDEuMF0sIFs0Ny42OTE2LCAtMTIyLjM3LCAxLjBdLCBbNDcuNjkyMywgLTEyMS44NjksIDEuMF0sIFs0Ny42OTMxLCAtMTIyLjAyMiwgMS4wXSwgWzQ3LjY5MzUsIC0xMjIuMzUyLCAxLjBdLCBbNDcuNjkzNSwgLTEyMi4zNDEwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjY5MzYsIC0xMjIuMTksIDEuMF0sIFs0Ny42OTM3LCAtMTIyLjMzOCwgMS4wXSwgWzQ3LjY5NDQsIC0xMjIuMzc3MDAwMDAwMDAwMDEsIDIuMF0sIFs0Ny42OTQ1LCAtMTIyLjM3NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk0NSwgLTEyMi4wMTc5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjY5NDgsIC0xMjIuMTc5LCAxLjBdLCBbNDcuNjk0OCwgLTEyMi4xNzgsIDIuMF0sIFs0Ny42OTUsIC0xMjIuMTgsIDEuMF0sIFs0Ny42OTUyLCAtMTIyLjE3OCwgMS4wXSwgWzQ3LjY5NTYsIC0xMjIuMzYsIDEuMF0sIFs0Ny42OTU3LCAtMTIyLjM3NiwgMS4wXSwgWzQ3LjY5NTksIC0xMjIuMzYzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny42OTU5LCAtMTIyLjEzMywgMS4wXSwgWzQ3LjY5NjUsIC0xMjIuMTM0LCAxLjBdLCBbNDcuNjk2NiwgLTEyMi4xMzMsIDEuMF0sIFs0Ny42OTcxLCAtMTIyLjM3MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNjk3NCwgLTEyMi4zNjksIDEuMF0sIFs0Ny42OTgzLCAtMTIyLjM4OSwgMS4wXSwgWzQ3LjY5ODcsIC0xMjIuMzY2LCAxLjBdLCBbNDcuNjk4NywgLTEyMi4zNjUsIDIuMF0sIFs0Ny42OTk0LCAtMTIyLjM0ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNjk5OCwgLTEyMi4zNjcsIDEuMF0sIFs0Ny43MDAyLCAtMTIyLjEwMywgMS4wXSwgWzQ3LjcwMDYsIC0xMjIuMzM5LCAxLjBdLCBbNDcuNzAxNiwgLTEyMi4xNywgMS4wXSwgWzQ3LjcwMTYsIC0xMjIuMTAzLCAxLjBdLCBbNDcuNzAzOSwgLTEyMi4zNiwgMS4wXSwgWzQ3LjcwNDMsIC0xMjIuMTIyMDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43MDQzLCAtMTIyLjExMzk5OTk5OTk5OTk5LCAyLjBdLCBbNDcuNzA0NiwgLTEyMi4xMjI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcwNDcsIC0xMjIuMzQsIDEuMF0sIFs0Ny43MDQ3LCAtMTIyLjEyMjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzA0NywgLTEyMi4xMTMsIDEuMF0sIFs0Ny43MDQ4LCAtMTIyLjM0LCAxLjBdLCBbNDcuNzA0OCwgLTEyMi4xMTMsIDEuMF0sIFs0Ny43MDQ4LCAtMTIyLjExMSwgMS4wXSwgWzQ3LjcwNDksIC0xMjIuMzQsIDIuMF0sIFs0Ny43MDQ5LCAtMTIyLjEyNiwgMS4wXSwgWzQ3LjcwNDksIC0xMjIuMTI1LCAyLjBdLCBbNDcuNzA0OSwgLTEyMi4xMjI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcwNSwgLTEyMi4xMjYsIDEuMF0sIFs0Ny43MDUsIC0xMjIuMTEzOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MDUzLCAtMTIyLjEyNiwgMi4wXSwgWzQ3LjcwNTMsIC0xMjIuMTEzLCAxLjBdLCBbNDcuNzA3NCwgLTEyMS4zNjM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcwNzYsIC0xMjIuMTE1LCAxLjBdLCBbNDcuNzA3NywgLTEyMi4xMiwgMS4wXSwgWzQ3LjcwODksIC0xMjIuMjQ1LCAxLjBdLCBbNDcuNzA5NywgLTEyMi4xMDcwMDAwMDAwMDAwMSwgMS4wXSwgWzQ3LjcxLCAtMTIyLjExMywgMS4wXSwgWzQ3LjcxMDEsIC0xMjIuMTA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTAzLCAtMTIyLjEwOSwgMS4wXSwgWzQ3LjcxMDMsIC0xMjIuMTA3OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43MTA3LCAtMTIyLjExLCAxLjBdLCBbNDcuNzExMiwgLTEyMi4wMTg5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxMTUsIC0xMjIuMDE0LCAxLjBdLCBbNDcuNzEzMiwgLTEyMi4yOTI5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3LjcxMzMsIC0xMjIuMzExLCAxLjBdLCBbNDcuNzE1MywgLTEyMi4yMjEsIDEuMF0sIFs0Ny43MTU5LCAtMTIyLjI1MSwgMS4wXSwgWzQ3LjcxOCwgLTEyMi4xNTYsIDEuMF0sIFs0Ny43MTg0LCAtMTIyLjE1NiwgMS4wXSwgWzQ3LjcxODcsIC0xMjIuMDI0LCAxLjBdLCBbNDcuNzE4OSwgLTEyMi4yMjUsIDEuMF0sIFs0Ny43MTk0LCAtMTIyLjE5OSwgMS4wXSwgWzQ3LjcyMDIsIC0xMjIuMjIzLCAxLjBdLCBbNDcuNzIxNCwgLTEyMi4yODksIDEuMF0sIFs0Ny43MjIxLCAtMTIyLjM1MiwgMS4wXSwgWzQ3LjcyNDYsIC0xMjIuMzYzLCAxLjBdLCBbNDcuNzI5NSwgLTEyMi4zMzQsIDEuMF0sIFs0Ny43MywgLTEyMi4zMzUsIDIuMF0sIFs0Ny43MzE4LCAtMTIxLjk4MjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzMyLCAtMTIyLjE3OCwgMS4wXSwgWzQ3LjczMjEsIC0xMjIuMzM0LCAxLjBdLCBbNDcuNzMyMywgLTEyMi4xNjUsIDEuMF0sIFs0Ny43MzI1LCAtMTIyLjE2NSwgMS4wXSwgWzQ3LjczMzYsIC0xMjIuMjEsIDEuMF0sIFs0Ny43MzM2LCAtMTIxLjk2NSwgMS4wXSwgWzQ3LjczMzgsIC0xMjIuMjA4LCAxLjBdLCBbNDcuNzMzOSwgLTEyMi4yMDksIDEuMF0sIFs0Ny43MzQ1LCAtMTIxLjk2NzAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzM0OCwgLTEyMi4zNCwgMS4wXSwgWzQ3LjczNjQsIC0xMjIuMjg2LCAxLjBdLCBbNDcuNzM2OCwgLTEyMi4yOTUsIDEuMF0sIFs0Ny43MzczLCAtMTIxLjk2OSwgMS4wXSwgWzQ3LjczODQsIC0xMjIuMzQ4LCAxLjBdLCBbNDcuNzM5NywgLTEyMi4zMTYsIDEuMF0sIFs0Ny43Mzk3LCAtMTIyLjIyMzk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzQwMSwgLTEyMi4yMjYsIDEuMF0sIFs0Ny43NDQsIC0xMjIuMTgxLCAxLjBdLCBbNDcuNzQ0MiwgLTEyMS45ODQsIDEuMF0sIFs0Ny43NDQ3LCAtMTIxLjk4NCwgMS4wXSwgWzQ3Ljc0ODQsIC0xMjIuMzE3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43NTA0LCAtMTIyLjMwNywgMS4wXSwgWzQ3Ljc1MDgsIC0xMjIuMjM4OTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43NTIxLCAtMTIyLjMzNCwgMS4wXSwgWzQ3Ljc1MzgsIC0xMjIuMzI1LCAxLjBdLCBbNDcuNzU1LCAtMTIyLjIxNjAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzU1NywgLTEyMi4zMDksIDEuMF0sIFs0Ny43NjE4LCAtMTIyLjI2MTAwMDAwMDAwMDAxLCAxLjBdLCBbNDcuNzY1NywgLTEyMi4yODM5OTk5OTk5OTk5OSwgMS4wXSwgWzQ3Ljc2NzgsIC0xMjIuMjM3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43Njg1LCAtMTIyLjIzNiwgMS4wXSwgWzQ3Ljc2ODUsIC0xMjIuMTYsIDEuMF0sIFs0Ny43Njg1LCAtMTIyLjE1ODk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzY5NCwgLTEyMi4wNjQsIDEuMF0sIFs0Ny43Njk4LCAtMTIyLjM4NCwgMS4wXSwgWzQ3Ljc3MjcsIC0xMjIuMjM3MDAwMDAwMDAwMDEsIDEuMF0sIFs0Ny43NzM3LCAtMTIyLjIzOCwgMS4wXSwgWzQ3Ljc3Mzk5OTk5OTk5OTk5NCwgLTEyMi4yOCwgMS4wXSwgWzQ3Ljc3NDMsIC0xMjIuMjc5LCAxLjBdLCBbNDcuNzc0NSwgLTEyMi4yMjUsIDEuMF0sIFs0Ny43NzQ4LCAtMTIyLjI0NCwgMS4wXSwgWzQ3Ljc3NTQsIC0xMjIuMTcyOTk5OTk5OTk5OTksIDEuMF0sIFs0Ny43NzU3LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdLCBbNDcuNzc2LCAtMTIyLjE3Mjk5OTk5OTk5OTk5LCAxLjBdXSwKICAgICAgICAgICAgICAgIHsiYmx1ciI6IDE1LCAibWF4IjogMS4wLCAibWF4Wm9vbSI6IDksICJtaW5PcGFjaXR5IjogMC45OSwgInJhZGl1cyI6IDEwfQogICAgICAgICAgICApLmFkZFRvKG1hcF80ZjExMDllMmE5NTA0MGEzYjFkYWQ5MzI3MTgxMTdiZik7CiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



According to National Association of Home Builders, after sale spending averages $2,368. more in new home then older homes. Pinpoint where new homes built will allow vendors, retails, contractors, and other professional the sale into the residential market to better target their. 


```python

```