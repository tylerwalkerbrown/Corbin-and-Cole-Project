# Exploring Statcast data containing Gerrit Cole and Corbin Burnes

- Hypothesis Testing 
- Probability 
- Correlation 
- Linear Regression ( Speed VS Spin Rate)

# Packages (Fangraphs)


```python
import pandas as pd
from baseball_scraper import statcast
from baseball_scraper import playerid_lookup
from baseball_scraper import statcast_pitcher
from baseball_scraper import pitching_stats
from baseball_scraper import standings
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
```


```python
#data = standings(2016)
#print(data)
```

# Player Lookup


```python
#Looking at Gerrit Cole
playerid_lookup('cole', 'gerrit')
```

    Gathering player lookup table. This may take a moment.


    /Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/baseball_scraper/playerid_lookup.py:39: FutureWarning: In a future version of pandas all arguments of DataFrame.drop except for the argument 'labels' will be keyword-only.
      results = results.reset_index().drop('index', 1)





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
      <th>name_last</th>
      <th>name_first</th>
      <th>key_mlbam</th>
      <th>key_retro</th>
      <th>key_bbref</th>
      <th>key_fangraphs</th>
      <th>mlb_played_first</th>
      <th>mlb_played_last</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>cole</td>
      <td>gerrit</td>
      <td>543037</td>
      <td>coleg001</td>
      <td>colege01</td>
      <td>13125</td>
      <td>2013.0</td>
      <td>2022.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Looking at up Corbin Burns
playerid_lookup('burnes', 'corbin')
```

    Gathering player lookup table. This may take a moment.


    /Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/baseball_scraper/playerid_lookup.py:39: FutureWarning: In a future version of pandas all arguments of DataFrame.drop except for the argument 'labels' will be keyword-only.
      results = results.reset_index().drop('index', 1)





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
      <th>name_last</th>
      <th>name_first</th>
      <th>key_mlbam</th>
      <th>key_retro</th>
      <th>key_bbref</th>
      <th>key_fangraphs</th>
      <th>mlb_played_first</th>
      <th>mlb_played_last</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>burnes</td>
      <td>corbin</td>
      <td>669203</td>
      <td>burnc002</td>
      <td>burneco01</td>
      <td>19361</td>
      <td>2018.0</td>
      <td>2022.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Pulling 2022 statistics for gerrit cole and corbin burnes
gerrit = statcast_pitcher('2022-04-01', '2022-09-01', 543037)
corbin = statcast_pitcher('2022-04-01', '2022-09-01', 669203)
```

    Gathering Player Data
    Gathering Player Data


# Storing Data in SQL


```python
import mysql.connector
import sqlite3
# import the module
import pymysql
from sqlalchemy import create_engine
```


```python
# create sqlalchemy engine
engine = create_engine("mysql+pymysql://{user}:{password}@localhost/{database}"
                       .format(user = '',
                              password = '',
                              database = ''))
```


```python
#Connector information
mydb = mysql.connector.connect(host = "
              user = 'root',
              password = '',
              database = ''
              )
```


```python
#Creating cursor 
mycursor = mydb.cursor(buffered=True)
```


```python
#Creating database for pitching data 
#mycursor.execute("create database mlb_pitchers")
```


```python
#Reading data to sql
corbin.to_sql('corbin_burns', con = engine, if_exists = 'append', chunksize = len(corbin))
gerrit.to_sql('gerrit_cole', con = engine, if_exists = 'append', chunksize = len(gerrit))

```




    2730



# Reading in Data from SQL


```python
cole_query="""SELECT game_date,sz_top,sz_bot,pitch_name,release_speed,home_team,away_team,stand,release_spin_rate,description
        ,plate_x,plate_z,hc_x,hc_y
        FROM mlb_pitchers.gerrit_cole
        WHERE pitch_name is NOT NULL"""
corbin_query="""SELECT game_date,sz_top,sz_bot,pitch_name,release_speed,home_team,away_team,stand,release_spin_rate,description 
        ,plate_x,plate_z,hc_x,hc_y
        FROM mlb_pitchers.corbin_burns
        WHERE pitch_name is NOT NULL"""
```


```python
corbin = pd.read_sql(corbin_query,con=engine)
cole = pd.read_sql(cole_query,con=engine)
```

    Exception during reset or similar
    Traceback (most recent call last):
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 756, in _write_bytes
        self._sock.sendall(data)
    BrokenPipeError: [Errno 32] Broken pipe
    
    During handling of the above exception, another exception occurred:
    
    Traceback (most recent call last):
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sqlalchemy/pool/base.py", line 739, in _finalize_fairy
        fairy._reset(pool)
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sqlalchemy/pool/base.py", line 988, in _reset
        pool._dialect.do_rollback(self)
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sqlalchemy/engine/default.py", line 682, in do_rollback
        dbapi_connection.rollback()
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 479, in rollback
        self._execute_command(COMMAND.COM_QUERY, "ROLLBACK")
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 814, in _execute_command
        self._write_bytes(packet)
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 759, in _write_bytes
        raise err.OperationalError(
    pymysql.err.OperationalError: (2006, "MySQL server has gone away (BrokenPipeError(32, 'Broken pipe'))")



```python
cole.head()
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
      <th>game_date</th>
      <th>sz_top</th>
      <th>sz_bot</th>
      <th>pitch_name</th>
      <th>release_speed</th>
      <th>home_team</th>
      <th>away_team</th>
      <th>stand</th>
      <th>release_spin_rate</th>
      <th>description</th>
      <th>plate_x</th>
      <th>plate_z</th>
      <th>hc_x</th>
      <th>hc_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-08-31</td>
      <td>3.50</td>
      <td>1.67</td>
      <td>4-Seam Fastball</td>
      <td>97.5</td>
      <td>LAA</td>
      <td>NYY</td>
      <td>R</td>
      <td>2402.0</td>
      <td>hit_into_play</td>
      <td>0.11</td>
      <td>2.66</td>
      <td>184.71</td>
      <td>132.78</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-08-31</td>
      <td>3.53</td>
      <td>1.78</td>
      <td>4-Seam Fastball</td>
      <td>98.0</td>
      <td>LAA</td>
      <td>NYY</td>
      <td>R</td>
      <td>2370.0</td>
      <td>ball</td>
      <td>0.50</td>
      <td>1.03</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-08-31</td>
      <td>3.50</td>
      <td>1.78</td>
      <td>4-Seam Fastball</td>
      <td>96.7</td>
      <td>LAA</td>
      <td>NYY</td>
      <td>R</td>
      <td>2350.0</td>
      <td>ball</td>
      <td>-1.02</td>
      <td>3.47</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-08-31</td>
      <td>3.65</td>
      <td>1.78</td>
      <td>4-Seam Fastball</td>
      <td>97.5</td>
      <td>LAA</td>
      <td>NYY</td>
      <td>R</td>
      <td>2404.0</td>
      <td>ball</td>
      <td>0.02</td>
      <td>4.13</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-08-31</td>
      <td>3.68</td>
      <td>1.79</td>
      <td>Slider</td>
      <td>88.9</td>
      <td>LAA</td>
      <td>NYY</td>
      <td>R</td>
      <td>2523.0</td>
      <td>hit_into_play</td>
      <td>0.76</td>
      <td>1.63</td>
      <td>115.77</td>
      <td>162.23</td>
    </tr>
  </tbody>
</table>
</div>




```python
corbin.head()
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
      <th>game_date</th>
      <th>sz_top</th>
      <th>sz_bot</th>
      <th>pitch_name</th>
      <th>release_speed</th>
      <th>home_team</th>
      <th>away_team</th>
      <th>stand</th>
      <th>release_spin_rate</th>
      <th>description</th>
      <th>plate_x</th>
      <th>plate_z</th>
      <th>hc_x</th>
      <th>hc_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-08-29</td>
      <td>3.43</td>
      <td>1.62</td>
      <td>Curveball</td>
      <td>80.1</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2867.0</td>
      <td>hit_into_play</td>
      <td>-0.96</td>
      <td>2.38</td>
      <td>148.66</td>
      <td>165.89</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-08-29</td>
      <td>3.34</td>
      <td>1.52</td>
      <td>Cutter</td>
      <td>94.9</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2651.0</td>
      <td>ball</td>
      <td>-1.92</td>
      <td>1.86</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-08-29</td>
      <td>3.44</td>
      <td>1.61</td>
      <td>Curveball</td>
      <td>81.5</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2759.0</td>
      <td>blocked_ball</td>
      <td>-0.47</td>
      <td>0.32</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-08-29</td>
      <td>3.43</td>
      <td>1.62</td>
      <td>Cutter</td>
      <td>93.4</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2547.0</td>
      <td>foul_bunt</td>
      <td>-0.27</td>
      <td>3.03</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-08-29</td>
      <td>3.51</td>
      <td>1.63</td>
      <td>Cutter</td>
      <td>94.3</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2577.0</td>
      <td>called_strike</td>
      <td>-0.61</td>
      <td>2.09</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
cole.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2730 entries, 0 to 2729
    Data columns (total 14 columns):
     #   Column             Non-Null Count  Dtype  
    ---  ------             --------------  -----  
     0   game_date          2730 non-null   object 
     1   sz_top             2730 non-null   float64
     2   sz_bot             2730 non-null   float64
     3   pitch_name         2730 non-null   object 
     4   release_speed      2730 non-null   float64
     5   home_team          2730 non-null   object 
     6   away_team          2730 non-null   object 
     7   stand              2730 non-null   object 
     8   release_spin_rate  2728 non-null   float64
     9   description        2730 non-null   object 
     10  plate_x            2730 non-null   float64
     11  plate_z            2730 non-null   float64
     12  hc_x               413 non-null    float64
     13  hc_y               413 non-null    float64
    dtypes: float64(8), object(6)
    memory usage: 298.7+ KB


# Data Understanding


```python
from pylab import rcParams
import scipy.stats as st
rcParams['figure.figsize'] = 12, 7
```


```python
#Corben Burnes descriptive statistics
corbin.groupby(['pitch_name'])[['release_speed','release_spin_rate']].describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="8" halign="left">release_speed</th>
      <th colspan="8" halign="left">release_spin_rate</th>
    </tr>
    <tr>
      <th></th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>pitch_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4-Seam Fastball</th>
      <td>10.0</td>
      <td>96.190000</td>
      <td>1.006037</td>
      <td>94.8</td>
      <td>95.6</td>
      <td>96.1</td>
      <td>96.65</td>
      <td>98.0</td>
      <td>10.0</td>
      <td>2593.200000</td>
      <td>129.875838</td>
      <td>2368.0</td>
      <td>2550.0</td>
      <td>2614.0</td>
      <td>2667.5</td>
      <td>2756.0</td>
    </tr>
    <tr>
      <th>Changeup</th>
      <td>239.0</td>
      <td>90.221757</td>
      <td>0.947413</td>
      <td>86.8</td>
      <td>89.5</td>
      <td>90.3</td>
      <td>90.90</td>
      <td>92.7</td>
      <td>239.0</td>
      <td>2013.054393</td>
      <td>185.340594</td>
      <td>1224.0</td>
      <td>1885.0</td>
      <td>2028.0</td>
      <td>2126.5</td>
      <td>2991.0</td>
    </tr>
    <tr>
      <th>Curveball</th>
      <td>493.0</td>
      <td>81.730629</td>
      <td>1.209135</td>
      <td>78.3</td>
      <td>80.9</td>
      <td>81.7</td>
      <td>82.60</td>
      <td>85.2</td>
      <td>493.0</td>
      <td>2755.087221</td>
      <td>117.022332</td>
      <td>1540.0</td>
      <td>2703.0</td>
      <td>2756.0</td>
      <td>2815.0</td>
      <td>3000.0</td>
    </tr>
    <tr>
      <th>Cutter</th>
      <td>1458.0</td>
      <td>94.990878</td>
      <td>1.031717</td>
      <td>91.3</td>
      <td>94.3</td>
      <td>95.0</td>
      <td>95.70</td>
      <td>98.4</td>
      <td>1457.0</td>
      <td>2613.632807</td>
      <td>145.432050</td>
      <td>1146.0</td>
      <td>2552.0</td>
      <td>2626.0</td>
      <td>2696.0</td>
      <td>2995.0</td>
    </tr>
    <tr>
      <th>Sinker</th>
      <td>185.0</td>
      <td>96.250811</td>
      <td>0.913493</td>
      <td>93.7</td>
      <td>95.6</td>
      <td>96.3</td>
      <td>96.80</td>
      <td>99.5</td>
      <td>185.0</td>
      <td>2491.902703</td>
      <td>92.063844</td>
      <td>2068.0</td>
      <td>2438.0</td>
      <td>2500.0</td>
      <td>2559.0</td>
      <td>2699.0</td>
    </tr>
    <tr>
      <th>Slider</th>
      <td>227.0</td>
      <td>87.937885</td>
      <td>1.611071</td>
      <td>84.1</td>
      <td>86.9</td>
      <td>87.9</td>
      <td>88.90</td>
      <td>92.1</td>
      <td>227.0</td>
      <td>2792.048458</td>
      <td>123.831217</td>
      <td>2011.0</td>
      <td>2748.5</td>
      <td>2806.0</td>
      <td>2865.0</td>
      <td>2987.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Gerrit cole descriptive statistics 
cole.groupby(['pitch_name'])[['release_speed','release_spin_rate']].describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="8" halign="left">release_speed</th>
      <th colspan="8" halign="left">release_spin_rate</th>
    </tr>
    <tr>
      <th></th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>pitch_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4-Seam Fastball</th>
      <td>1423.0</td>
      <td>97.737597</td>
      <td>1.163153</td>
      <td>93.9</td>
      <td>97.0</td>
      <td>97.8</td>
      <td>98.50</td>
      <td>101.0</td>
      <td>1423.0</td>
      <td>2435.891075</td>
      <td>79.510626</td>
      <td>2134.0</td>
      <td>2384.0</td>
      <td>2439.0</td>
      <td>2492.00</td>
      <td>2678.0</td>
    </tr>
    <tr>
      <th>Changeup</th>
      <td>242.0</td>
      <td>89.719421</td>
      <td>1.135396</td>
      <td>86.2</td>
      <td>89.0</td>
      <td>89.7</td>
      <td>90.50</td>
      <td>92.6</td>
      <td>241.0</td>
      <td>1603.443983</td>
      <td>126.992216</td>
      <td>1293.0</td>
      <td>1525.0</td>
      <td>1603.0</td>
      <td>1692.00</td>
      <td>2011.0</td>
    </tr>
    <tr>
      <th>Cutter</th>
      <td>201.0</td>
      <td>92.107463</td>
      <td>1.316204</td>
      <td>88.3</td>
      <td>91.4</td>
      <td>92.1</td>
      <td>93.00</td>
      <td>95.0</td>
      <td>201.0</td>
      <td>2481.378109</td>
      <td>106.603782</td>
      <td>1559.0</td>
      <td>2436.0</td>
      <td>2482.0</td>
      <td>2543.00</td>
      <td>2694.0</td>
    </tr>
    <tr>
      <th>Knuckle Curve</th>
      <td>267.0</td>
      <td>82.804869</td>
      <td>1.453908</td>
      <td>78.0</td>
      <td>81.8</td>
      <td>83.0</td>
      <td>83.75</td>
      <td>86.3</td>
      <td>267.0</td>
      <td>2791.550562</td>
      <td>89.513767</td>
      <td>2519.0</td>
      <td>2737.5</td>
      <td>2796.0</td>
      <td>2856.50</td>
      <td>2975.0</td>
    </tr>
    <tr>
      <th>Slider</th>
      <td>597.0</td>
      <td>88.572194</td>
      <td>1.433491</td>
      <td>83.2</td>
      <td>87.7</td>
      <td>88.8</td>
      <td>89.60</td>
      <td>92.5</td>
      <td>596.0</td>
      <td>2575.543624</td>
      <td>86.028819</td>
      <td>2308.0</td>
      <td>2521.0</td>
      <td>2578.5</td>
      <td>2638.25</td>
      <td>2841.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
corbin.head()
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
      <th>game_date</th>
      <th>sz_top</th>
      <th>sz_bot</th>
      <th>pitch_name</th>
      <th>release_speed</th>
      <th>home_team</th>
      <th>away_team</th>
      <th>stand</th>
      <th>release_spin_rate</th>
      <th>description</th>
      <th>plate_x</th>
      <th>plate_z</th>
      <th>hc_x</th>
      <th>hc_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-08-29</td>
      <td>3.43</td>
      <td>1.62</td>
      <td>Curveball</td>
      <td>80.1</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2867.0</td>
      <td>hit_into_play</td>
      <td>-0.96</td>
      <td>2.38</td>
      <td>148.66</td>
      <td>165.89</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-08-29</td>
      <td>3.34</td>
      <td>1.52</td>
      <td>Cutter</td>
      <td>94.9</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2651.0</td>
      <td>ball</td>
      <td>-1.92</td>
      <td>1.86</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-08-29</td>
      <td>3.44</td>
      <td>1.61</td>
      <td>Curveball</td>
      <td>81.5</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2759.0</td>
      <td>blocked_ball</td>
      <td>-0.47</td>
      <td>0.32</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-08-29</td>
      <td>3.43</td>
      <td>1.62</td>
      <td>Cutter</td>
      <td>93.4</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2547.0</td>
      <td>foul_bunt</td>
      <td>-0.27</td>
      <td>3.03</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-08-29</td>
      <td>3.51</td>
      <td>1.63</td>
      <td>Cutter</td>
      <td>94.3</td>
      <td>MIL</td>
      <td>PIT</td>
      <td>L</td>
      <td>2577.0</td>
      <td>called_strike</td>
      <td>-0.61</td>
      <td>2.09</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
corbin.tail()
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
      <th>game_date</th>
      <th>sz_top</th>
      <th>sz_bot</th>
      <th>pitch_name</th>
      <th>release_speed</th>
      <th>home_team</th>
      <th>away_team</th>
      <th>stand</th>
      <th>release_spin_rate</th>
      <th>description</th>
      <th>plate_x</th>
      <th>plate_z</th>
      <th>hc_x</th>
      <th>hc_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2607</th>
      <td>2022-04-07</td>
      <td>3.32</td>
      <td>1.51</td>
      <td>Cutter</td>
      <td>95.3</td>
      <td>CHC</td>
      <td>MIL</td>
      <td>L</td>
      <td>2577.0</td>
      <td>ball</td>
      <td>-1.48</td>
      <td>3.16</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2608</th>
      <td>2022-04-07</td>
      <td>3.13</td>
      <td>1.44</td>
      <td>Cutter</td>
      <td>95.1</td>
      <td>CHC</td>
      <td>MIL</td>
      <td>L</td>
      <td>2580.0</td>
      <td>called_strike</td>
      <td>-0.41</td>
      <td>2.54</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2609</th>
      <td>2022-04-07</td>
      <td>3.23</td>
      <td>1.40</td>
      <td>Cutter</td>
      <td>92.9</td>
      <td>CHC</td>
      <td>MIL</td>
      <td>L</td>
      <td>2624.0</td>
      <td>ball</td>
      <td>0.99</td>
      <td>0.70</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2610</th>
      <td>2022-04-07</td>
      <td>3.35</td>
      <td>1.51</td>
      <td>Cutter</td>
      <td>93.8</td>
      <td>CHC</td>
      <td>MIL</td>
      <td>L</td>
      <td>2468.0</td>
      <td>ball</td>
      <td>0.93</td>
      <td>1.14</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2611</th>
      <td>2022-04-07</td>
      <td>3.10</td>
      <td>1.42</td>
      <td>Cutter</td>
      <td>93.8</td>
      <td>CHC</td>
      <td>MIL</td>
      <td>L</td>
      <td>2602.0</td>
      <td>ball</td>
      <td>1.75</td>
      <td>1.58</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Looking at the graph you can see a very low spin rate in a particular part of the data around pitches 800.


```python
#Finding unique values in pitch name
```


```python
different_pitches = """SELECT DISTINCT(pitch_name)
from mlb_pitchers.corbin_burns"""
```


```python
#Here we have a list of distinct pitches that corbin throws so we can plot
pd.read_sql(different_pitches, con= engine)
```

    Exception during reset or similar
    Traceback (most recent call last):
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 756, in _write_bytes
        self._sock.sendall(data)
    BrokenPipeError: [Errno 32] Broken pipe
    
    During handling of the above exception, another exception occurred:
    
    Traceback (most recent call last):
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sqlalchemy/pool/base.py", line 739, in _finalize_fairy
        fairy._reset(pool)
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sqlalchemy/pool/base.py", line 988, in _reset
        pool._dialect.do_rollback(self)
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sqlalchemy/engine/default.py", line 682, in do_rollback
        dbapi_connection.rollback()
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 479, in rollback
        self._execute_command(COMMAND.COM_QUERY, "ROLLBACK")
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 814, in _execute_command
        self._write_bytes(packet)
      File "/Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/pymysql/connections.py", line 759, in _write_bytes
        raise err.OperationalError(
    pymysql.err.OperationalError: (2006, "MySQL server has gone away (BrokenPipeError(32, 'Broken pipe'))")





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
      <th>pitch_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Curveball</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cutter</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sinker</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Changeup</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Slider</td>
    </tr>
    <tr>
      <th>5</th>
      <td>4-Seam Fastball</td>
    </tr>
    <tr>
      <th>6</th>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Exploring the spin rate of the curveball over the 2022 season 
plt.plot(corbin[corbin['pitch_name'] == 'Sinker'].release_spin_rate, label = 'Sinker', alpha = .4)
plt.plot(corbin[corbin['pitch_name'] == 'Cutter'].release_spin_rate, label = 'Cutter', alpha = .4)
plt.plot(corbin[corbin['pitch_name'] == 'Changeup'].release_spin_rate, label = 'Changeup', alpha = .4)
plt.plot(corbin[corbin['pitch_name'] == 'Slider'].release_spin_rate, label = 'Slider', alpha = .4)
plt.plot(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate, label = 'Curveball', alpha = .4)
plt.title('Pitches')
plt.xlabel('Number of pitches')
plt.ylabel('Spin Rate')
plt.legend()


```




    <matplotlib.legend.Legend at 0x7fbf3fb78be0>




    
![png](output_32_1.png)
    


Here we calculated the parobability of that low of a spin rate occuring compared to Corbin burns average spin rate

# Probability of Specific Spin Rates


```python
#Grouping by the date and the pitch name to collect spin rate averages 
speed_stats = corbin.groupby(['game_date','pitch_name'])['release_speed'].describe()
spin_stats = corbin.groupby(['game_date','pitch_name'])['release_spin_rate'].describe()
```


```python
#Reading decriptive statistics into sql
speed_stats.to_sql('corbin_speed_stats', con = engine, if_exists = 'append', chunksize = len(pitch_stats))
spin_stats.to_sql('corbin_spin_stats', con = engine, if_exists = 'append', chunksize = len(pitch_stats))
```

The next two tables are used to automate the z-scores for give games based on the amount thrown of that particular pitch. This table also caputure the z scores of the particular game vs the average of the whole season of pitches based on pitch type. In the final subquery we join the data on pitch name to get the across the board average versus the particular pitch. Had to create the code in mysql to run because python returned an error on the dashes to call for column names that contain special characters.

This is the formula I used when calculating the Z-score:![image.png](attachment:image.png)
create table mlb_pitchers.corbin_spin_z_scores as (
with game_zs as
(SELECT game_date, pitch_name as pitch, count, mean, std, min, `25%`, `50%`, `75%`, max, 
((`25%` - mean )/`std`) as 1st_z_game,
((`50%`- mean)/`std`) as 2nd_z_game,
((`75%`- mean)/`std`) as 3rd_z_game,
((`max`- mean)/`std`) as max_z_game,
((`min`- mean)/`std`) as min_z_game
 FROM mlb_pitchers.corbin_spin_stats)
 ,total_z as 
 (select distinct(pitch_name), std(release_spin_rate) as total_spin_std, avg(release_spin_rate) as total_spin_avg
 from mlb_pitchers.corbin_burns
 group by pitch_name) 
 select  *,
 ((`25%` - total_spin_avg )/total_spin_std) as 1st_z_population,
((`50%`- total_spin_avg)/total_spin_std) as 2nd_z_population,
((`75%`- total_spin_avg)/total_spin_std) as 3rd_z_population,
((`max`- total_spin_avg)/total_spin_std) as max_z_population,
((`min`- total_spin_avg)/total_spin_std) as min_z_population
 from total_z
 join game_zs on total_z.pitch_name =game_zs.pitch
 )
 create table mlb_pitchers.corbin_speed_z_score as (
(with game_zs as
(SELECT game_date, pitch_name as pitch, count, mean, std, min, `25%`, `50%`, `75%`, max,
((`25%` - mean )/`std`) as 1st_z_game,
((`50%`- mean)/`std`) as 2nd_z_game,
((`75%`- mean)/`std`) as 3rd_z_game,
((`max`- mean)/`std`) as max_z_game,
((`min`- mean)/`std`) as min_z_game
 FROM mlb_pitchers.corbin_speed_stats)
 ,total_z as 
 (select distinct(pitch_name), std(release_speed) as total_speed_std, avg(release_speed) as total_speed_avg
 from mlb_pitchers.corbin_burns
 group by pitch_name) 
 select *,
 ((`25%` - total_speed_avg )/total_speed_std) as 1st_z_population,
((`50%`- total_speed_avg)/total_speed_std) as 2nd_z_population,
((`75%`- total_speed_avg)/total_speed_std) as 3rd_z_population,
((`max`- total_speed_avg)/total_speed_std) as max_z_population,
((`min`- total_speed_avg)/total_speed_std) as min_z_population
 from game_zs
 join total_z on total_z.pitch_name =game_zs.pitch)
 )


```python
#Queries for reading in the new z_score data
speed_z_scores = """select * from mlb_pitchers.corbin_speed_z_score"""
spin_z_scores = """select * from mlb_pitchers.corbin_spin_z_scores"""
```


```python
speed_z_scores = pd.read_sql(speed_z_scores,con = engine)
```


```python
spin_z_scores = pd.read_sql(spin_z_scores,con = engine)
```


```python
spin_z_scores.head()
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
      <th>pitch_name</th>
      <th>total_spin_std</th>
      <th>total_spin_avg</th>
      <th>game_date</th>
      <th>pitch</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>...</th>
      <th>1st_z_game</th>
      <th>2nd_z_game</th>
      <th>3rd_z_game</th>
      <th>max_z_game</th>
      <th>min_z_game</th>
      <th>1st_z_population</th>
      <th>2nd_z_population</th>
      <th>3rd_z_population</th>
      <th>max_z_population</th>
      <th>min_z_population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Curveball</td>
      <td>116.903588</td>
      <td>2755.087221</td>
      <td>2022-08-29</td>
      <td>Curveball</td>
      <td>19.0</td>
      <td>2747.894737</td>
      <td>97.487261</td>
      <td>2631.0</td>
      <td>2675.5</td>
      <td>...</td>
      <td>-0.742607</td>
      <td>-0.306653</td>
      <td>0.596029</td>
      <td>2.339847</td>
      <td>-1.199077</td>
      <td>-0.680794</td>
      <td>-0.317246</td>
      <td>0.435511</td>
      <td>1.889701</td>
      <td>-1.061449</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Curveball</td>
      <td>116.903588</td>
      <td>2755.087221</td>
      <td>2022-08-23</td>
      <td>Curveball</td>
      <td>21.0</td>
      <td>2754.047619</td>
      <td>77.104783</td>
      <td>2615.0</td>
      <td>2691.0</td>
      <td>...</td>
      <td>-0.817688</td>
      <td>-0.104373</td>
      <td>0.829422</td>
      <td>1.841032</td>
      <td>-1.803359</td>
      <td>-0.548206</td>
      <td>-0.077733</td>
      <td>0.538160</td>
      <td>1.205376</td>
      <td>-1.198314</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Curveball</td>
      <td>116.903588</td>
      <td>2755.087221</td>
      <td>2022-08-18</td>
      <td>Curveball</td>
      <td>19.0</td>
      <td>2797.631579</td>
      <td>73.242528</td>
      <td>2583.0</td>
      <td>2761.0</td>
      <td>...</td>
      <td>-0.500141</td>
      <td>0.264442</td>
      <td>0.489721</td>
      <td>1.984754</td>
      <td>-2.930423</td>
      <td>0.050578</td>
      <td>0.529605</td>
      <td>0.670747</td>
      <td>1.607417</td>
      <td>-1.472044</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Curveball</td>
      <td>116.903588</td>
      <td>2755.087221</td>
      <td>2022-08-13</td>
      <td>Curveball</td>
      <td>16.0</td>
      <td>2720.187500</td>
      <td>30.962814</td>
      <td>2646.0</td>
      <td>2704.0</td>
      <td>...</td>
      <td>-0.522805</td>
      <td>-0.054501</td>
      <td>0.607584</td>
      <td>2.125534</td>
      <td>-2.396019</td>
      <td>-0.437003</td>
      <td>-0.312969</td>
      <td>-0.137611</td>
      <td>0.264430</td>
      <td>-0.933138</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Curveball</td>
      <td>116.903588</td>
      <td>2755.087221</td>
      <td>2022-08-07</td>
      <td>Curveball</td>
      <td>23.0</td>
      <td>2729.652174</td>
      <td>69.212196</td>
      <td>2614.0</td>
      <td>2684.0</td>
      <td>...</td>
      <td>-0.659597</td>
      <td>-0.096113</td>
      <td>0.359009</td>
      <td>2.360102</td>
      <td>-1.670980</td>
      <td>-0.608084</td>
      <td>-0.274476</td>
      <td>-0.005023</td>
      <td>1.179714</td>
      <td>-1.206868</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>




```python
plt.figure(1 , figsize = (15 , 6))
n = 0 
for x in ['release_speed' ,'release_spin_rate']:
    n += 1
    plt.subplot(2 , 2 , n)
    plt.subplots_adjust(hspace =0.5 , wspace = 0.5)
    sns.distplot(corbin[x] , bins = 20)
    plt.title('Distplot of {}'.format(x))
plt.show()
```

    /Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/seaborn/distributions.py:2619: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/seaborn/distributions.py:2619: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)



    
![png](output_45_1.png)
    



```python
#Calculating the deviations the minimum is awaty from the mean 
deivations_away = """WITH A AS(
SELECT STD(release_spin_rate) as std,
MAX(release_spin_rate) as max, 
MIN(release_spin_rate) as min,
AVG(release_spin_rate) as mean
 FROM mlb_pitchers.corbin_burns)
 SELECT (mean - min)/std as min_z,
 (mean - max)/std as max_z
 from A"""
```


```python
#Reading in the deviations 
deivations_away = pd.read_sql(deivations_away,con=engine)
```

Below is the output of the z score and we can use that now to calculate the probability of this pitch happening.


```python
deivations_away
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
      <th>min_z</th>
      <th>max_z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.921685</td>
      <td>-1.669928</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Caculating the probability of the max spin rate 
print(corbin[corbin['pitch_name']== 'Curveball'].release_spin_rate.max(), 'spin rate has a probability of happening:')
print(round(st.norm.cdf(-1.669928) * 100,2),"% chance of Happening")
```

    3000.0 spin rate has a probability of happening:
    4.75 % chance of Happening



```python
print(corbin[corbin['pitch_name']== 'Curveball'].release_spin_rate.min(), 'spin rate has a probability of happening:')
print(round(st.norm.cdf(-5.921685) * 100,6),"% chance of Happening")
```

    1540.0 spin rate has a probability of happening:
    0.0 % chance of Happening


Below we looked at the point that we calculated the p value for. You can see how great of an outlier the observation is.


```python
#Plotting the standard deviations of spin rate
min_cb_spin = corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate.min()

plt.subplot(211)
plt.title("Spin Rate CB Distribution")
plt.xlabel("Spin Rates")

plt.hist(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate, bins = 150)

plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate), color='#FD4E40', linestyle='solid', linewidth=2, label = "Mean")

plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) + np.std(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate), color='#FFB908', linestyle='solid', linewidth=2, label = "Standard Deviations")
plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) - np.std(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate), color='#FFB908', linestyle='solid', linewidth=2)

plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) + np.std(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) * 2, color='#FFB908', linestyle='solid', linewidth=2)
plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) - np.std(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) * 2, color='#FFB908', linestyle='solid', linewidth=2)

plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) + np.std(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) * 3, color='#FFB908', linestyle='solid', linewidth=2)
plt.axvline(np.average(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) - np.std(corbin[corbin['pitch_name'] == 'Curveball'].release_spin_rate) * 3, color='#FFB908', linestyle='solid', linewidth=2)

plt.axvline(min_cb_spin, color='#62EDBF', linestyle='solid', linewidth=2, label = "Minimum Spin Rate")

plt.legend()
```




    <matplotlib.legend.Legend at 0x7fbf6d81fb20>




    
![png](output_53_1.png)
    


# Clustering


```python
#Selecting a sample of 50 columns each from eazch pitch type to have evenly spread clusters 
sample = """(SELECT pitch_name, release_spin_rate,release_speed
FROM mlb_pitchers.corbin_burns
where pitch_name = 'Curveball'
limit 50)
UNION 
(SELECT pitch_name, release_spin_rate,release_speed
FROM mlb_pitchers.corbin_burns
where pitch_name = 'Cutter'
limit 50)
UNION 
(SELECT pitch_name, release_spin_rate,release_speed
FROM mlb_pitchers.corbin_burns
where pitch_name = 'Sinker'
limit 50)
UNION 
(SELECT pitch_name, release_spin_rate,release_speed
FROM mlb_pitchers.corbin_burns
where pitch_name = 'Changeup'
limit 50)
UNION
(SELECT pitch_name, release_spin_rate,release_speed
FROM mlb_pitchers.corbin_burns
where pitch_name = 'Slider'
limit 50)
UNION
(SELECT pitch_name, release_spin_rate,release_speed
FROM mlb_pitchers.corbin_burns
where pitch_name = '4-Seam Fastball'
limit 50)"""
```


```python
#Reading in the deviations 
sample = pd.read_sql(sample,con=engine)
```


```python
#Clustering of different pitches comparing speed and spin rate
sns.relplot(
    x="release_spin_rate", y="release_speed", hue=sample['pitch_name'], data=sample, height=6,
);
```


    
![png](output_57_0.png)
    



```python
sns.scatterplot(data=corbin, x="plate_x", y="plate_z", hue="description")
plt.title('Pitch Result')
plt.legend()
```




    <matplotlib.legend.Legend at 0x7fbf6fc366a0>




    
![png](output_58_1.png)
    


Creating a Chart for balls and strikes


```python
sns.scatterplot(data=corbin[corbin['stand']=='L'], x="plate_x", y="plate_z", hue="pitch_name")
plt.title('Strike Calls VS Balls Corbin')
plt.legend()
```




    <matplotlib.legend.Legend at 0x7fbf3e411940>




    
![png](output_60_1.png)
    



```python
sns.lmplot(data=corbin[corbin['stand']=='L'], x="plate_x", y="plate_z", hue="pitch_name")
plt.title('Strike Calls VS Balls Corbin')
plt.legend()
```




    <matplotlib.legend.Legend at 0x7fbf8b10b370>




    
![png](output_61_1.png)
    



```python
sns.countplot(x=corbin["pitch_name"])
plt.title("Most Used Pitch")
```




    Text(0.5, 1.0, 'Most Used Pitch')




    
![png](output_62_1.png)
    



```python
#Subsetting to see most hit pitch
most_hit = corbin[corbin['description']=='hit_into_play']
sns.countplot(x=most_hit["pitch_name"])
plt.title("Most Hit Pitch")
```




    Text(0.5, 1.0, 'Most Hit Pitch')




    
![png](output_63_1.png)
    


The goal is to figure out what the hit percentage is against every pitch. What we have to do is collect the sum of pitches thrown and every individual type count. 


```python
pitch_usage_per = """with result as(
SELECT distinct(description), count(description) as result
 FROM mlb_pitchers.corbin_burns
 group by description)
 select * , result/{pitch_num} as percent 
 from result""".format(pitch_num = len(corbin))
```


```python
#Reading in the deviations 
pitch_usage_per = pd.read_sql(pitch_usage_per,con=engine)
```


```python
#Breaking down by each result precent
pitch_usage_per
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
      <th>description</th>
      <th>result</th>
      <th>percent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>hit_into_play</td>
      <td>396</td>
      <td>0.1516</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ball</td>
      <td>896</td>
      <td>0.3430</td>
    </tr>
    <tr>
      <th>2</th>
      <td>blocked_ball</td>
      <td>65</td>
      <td>0.0249</td>
    </tr>
    <tr>
      <th>3</th>
      <td>foul_bunt</td>
      <td>2</td>
      <td>0.0008</td>
    </tr>
    <tr>
      <th>4</th>
      <td>called_strike</td>
      <td>454</td>
      <td>0.1738</td>
    </tr>
    <tr>
      <th>5</th>
      <td>swinging_strike</td>
      <td>360</td>
      <td>0.1378</td>
    </tr>
    <tr>
      <th>6</th>
      <td>foul</td>
      <td>389</td>
      <td>0.1489</td>
    </tr>
    <tr>
      <th>7</th>
      <td>foul_tip</td>
      <td>27</td>
      <td>0.0103</td>
    </tr>
    <tr>
      <th>8</th>
      <td>hit_by_pitch</td>
      <td>12</td>
      <td>0.0046</td>
    </tr>
    <tr>
      <th>9</th>
      <td>swinging_strike_blocked</td>
      <td>48</td>
      <td>0.0184</td>
    </tr>
    <tr>
      <th>10</th>
      <td>missed_bunt</td>
      <td>1</td>
      <td>0.0004</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Plotting the most ciommon results
sns.countplot(x=corbin["description"])
plt.title("Common Results")

```




    Text(0.5, 1.0, 'Common Results')




    
![png](output_68_1.png)
    


# Pitches Thrown Game to Game


```python
#Ordering the spin rates by date
spin = """select *
from mlb_pitchers.corbin_spin_stats
order by game_date"""
```


```python
#Indexing the date column
spin = pd.read_sql(ordered, con = engine)
spin = spin.set_index('game_date')
```


```python
#Game over game average spin rates
plt.plot(spin[spin['pitch_name'] == 'Curveball']['mean'], label = 'Curveball')
plt.plot(spin[spin['pitch_name'] == 'Sinker']['mean'], label = 'Sinker')
plt.plot(spin[spin['pitch_name'] == 'Cutter']['mean'], label = 'Cutter')
plt.plot(spin[spin['pitch_name'] == 'Changeup']['mean'], label = 'Changeup')
plt.plot(spin[spin['pitch_name'] == 'Slider']['mean'], label = 'Slider')
plt.title('Average Spin Rate Game over Game')
plt.xlabel('Games')
plt.legend()
plt.xticks(rotation=90)

```




    ([0,
      1,
      2,
      3,
      4,
      5,
      6,
      7,
      8,
      9,
      10,
      11,
      12,
      13,
      14,
      15,
      16,
      17,
      18,
      19,
      20,
      21,
      22,
      23,
      24,
      25],
     [Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, ''),
      Text(0, 0, '')])




    
![png](output_72_1.png)
    



```python
plt.plot(speed_stats[speed_stats['pitch_name'] == 'Sinker']['mean'], label = 'Sinker')
plt.plot(speed_stats[speed_stats['pitch_name'] == 'Cutter']['mean'], label = 'Cutter')
plt.plot(speed_stats[speed_stats['pitch_name'] == 'Changeup']['mean'], label = 'Changeup')
plt.plot(speed_stats[speed_stats['pitch_name'] == 'Slider']['mean'], label = 'Slider')
plt.plot(speed_stats[speed_stats['pitch_name'] == 'Curveball']['mean'], label = 'Curveball')
plt.title('Average Velocities Game over Game')
plt.xlabel('Games')
plt.xticks(rotation=90)

plt.legend()
```




    <matplotlib.legend.Legend at 0x7fbf8ab3dbb0>




    
![png](output_73_1.png)
    

