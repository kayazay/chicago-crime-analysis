# A COMPREHENSIVE ANALYSIS OF CHICAGO CRIME DATA _(2001-2022)_

I took on a mission to explore the crime data of a state in the U.S - Chicago, as a Capston Project to a Training Program; my aim was to determine the following:

+ __What kinds__ of crime happen the most in Chicago?
+ __When__ is the state the most vulnerable to crime?
+ .. And __where__?
+ __What places__ does crime go unpunished the most?

Contained in this repository are:

+ A Jupyter Notebook _(ipynb)_, and
+ Datasets used for analysis.

## PREREQUISITES

The following are the libraries installed for this study and a brief why:

+ pandas
+ math
+ seaborn
+ re - REGEX manipulation
+ matplotlib

```py
import pandas as pd
import math
import seaborn as sns
import re
import matplotlib.pyplot as plt
```
## LOADING THE DATASET
```py
filepath  =  "C:/Users/user/Desktop/eze/"
crime  =  pd.read_csv(filepath+"crime_data_Proj1.csv",index_col=0)
```
_**Note:** The variable `filepath` may be tweaked in accordance with your directory_

## DATA PREPROCESSING
`crime.info()` gives us an overview of the fields in this dataset and respective data types

```py
<class 'pandas.core.frame.DataFrame'>
Int64Index: 2278726 entries, 0 to 2278725
Data columns (total 22 columns):
 #   Column                Dtype  
---  ------                -----  
 0   ID                    int64  
 1   Case Number           object 
 2   Date                  object 
 3   Block                 object 
 4   IUCR                  object 
 5   Primary Type          object 
 6   Description           object 
 7   Location Description  object 
 8   Arrest                bool   
 9   Domestic              bool   
 10  Beat                  int64  
 11  District              float64
 12  Ward                  float64
 13  Community Area        float64
 14  FBI Code              object 
 15  X Coordinate          float64
 16  Y Coordinate          float64
 17  Year                  int64  
 18  Updated On            object 
 19  Latitude              float64
 20  Longitude             float64
 21  Location              object 
dtypes: bool(2), float64(7), int64(3), object(10)
memory usage: 369.4+ MB
```

First I switch variables to `crime_clean`.
This is a common practice in my codes - I do this for readability and to prevent the dataset from mis-updating.

```py
crime_clean  =  crime
```
#### What to Change?
<insert GIF for change>

__Date__

> Values in these column are recognized as strings.
>
> It should be converted from an object to datetime.

```py
crime_clean['Date'] = pd.to_datetime(crime['Date'])
```

__District, Ward, Community Area & Beat__

> These are recognized as floats since there are numbers; as opposed to objects since they are really elements of location.
>
> custom function - `to_str()` was created to do this.

```py
# Created a function that returns null if the value in a column is null & converts to a string if it is not
def to_str(_unit):
    if isinstance(_unit, str):
        return _unit
    else:
        if math.isnan(_unit):
            unit_done=_unit
        else:
            unit_done = str(int(_unit))
        return unit_done

# APPLIED the to_str function on relevant columns
crime_clean['District'] = crime['District'].apply(to_str)
crime_clean['Ward'] = crime['Ward'].apply(to_str)
crime_clean['Community Area'] = crime['Community Area'].apply(to_str)
crime_clean['Beat'] = crime['Beat'].apply(to_str)
```

#### Investigate for Missing Values

`crime_clean.isna().sum()` shows us how many records in each column have missing values.

```py
ID                           0
Case Number                  0
Date                         0
Block                        0
IUCR                         0
Primary Type                 0
Description                  0
Location Description      2877
Arrest                       0
Domestic                     0
Beat                         0
District                    12
Ward                    184695
Community Area          184267
FBI Code                     0
X Coordinate             23984
Y Coordinate             23984
Year                         0
Updated On                   0
Latitude                 23984
Longitude                23984
Location                 23984
blockType                    0
actualStreet                 0
dayOfWeek                    1
hourDay                      1
timeOfDay                    1
month                        1
dtype: int64
```

+ `Case Number` has a missing value and this would be dropped.

  > This is because this column holds semi-unique values.
  >
  > i.e a crime incident that doesn't have a `Case Number` is an error that might indicate the crime didn't happen

  ```py
  crime_clean = crime_clean.dropna(subset=['Case Number'])
  ```

+ Other columns that have missing values are:-

  + `Location Description`,
  + `Ward`,
  + `Community Area`,
  + `X & Y Coordinates`,
  + `Lat` & `Long`.

  > All these are elements of location and a missing value means the location was not recorded for some reason.
  >
  > Not necessarily that the crime didn't happen

## DATA MANIPULATION

As per my ritual, once again - variable switch <insert finger down emoji>

 `crime_tweak = crime_clean`

Now, we look for more features or columns to extract from our dataset for it to hold more analytical water, as it were. 

To start, OBSERVE THE FOLLOWING COLUMNS...

<inserts drum rolls GIF>

### Block

```	py
array(['085XX S MUSKEGON AVE', '092XX S ELLIS AVE', '062XX N TRIPP AVE',
       '0000X N KEELER AVE', '016XX W HARRISON ST', '003XX W 28 PL',
       '006XX S CENTRAL AVE', '043XX W POTOMAC AVE', '032XX W PIERCE AVE',
       '021XX N CALIFORNIA AVE', '031XX W WARREN BLVD',
       '058XX S KEDZIE AVE', '014XX N ELK GROVE AVE',
       '076XX S HALSTED ST', '042XX S INDIANA AVE', '019XX W MADISON ST',
       '047XX W FULTON ST', '009XX W GRACE ST', '002XX N LOREL AVE',
       '023XX S STATE ST', '022XX N NATCHEZ AVE', '133XX S BRANDON AVE',
       '036XX S LAKE PARK AVE', '003XX S MICHIGAN AVE', '001XX W 72ND ST',
       '078XX S PHILLIPS AVE', '025XX W BERWYN AVE', '077XX S PEORIA ST',
       '008XX N MICHIGAN AVE', '0000X W DIVISION ST',
       '065XX S JUSTINE ST', '065XX S ELLIS AVE',
       '053XX W BLOOMINGDALE AVE', '011XX S WESTERN AVE',
       '005XX W DIVISION ST', '059XX S KEDVALE AVE', '031XX S HALSTED ST',
       '001XX S PULASKI RD', '001XX N DAMEN AVE', '048XX N AVERS AVE'],
      dtype=object)
```

This column holds the full address of the location a reported crime occurred. There is a pattern in the last word for most records in this column.

> __AVE__, __ST__, __BLVD__, __RD__, ...

This indicates the type of street the crime occurred. What if ... <insert bulb emoji> A new column were created to hold this information.

+ _**Note**: These street forms however are abbreviated as seen in the array; thus, for ease of compression, there is a need to also output their full meanings._

+ The __USPS suffix abbreviations__ was referenced to accomplish this.

  > This table is a universal standard for street abbreviations and their full forms. Link to table - https://www.pb.com/docs/us/pdf/sis/mail-services/usps-suffix-abbreviations.pdf


<details>
    <summary>
        Here are few records from this table for context
    </summary>
    | | Primary Street | Commonly Used Street | Postal Service Standard |
   |---:|:-----------------|:-----------------------|:--------------------------|
  | 0 | ALLEY | ALLEE | ALY | 
  | 1 | ALLEY | ALLEY | ALY | 
  | 2 | ALLEY | ALLY | ALY | 
  | 3 | ALLEY | ALY | ALY |
  | 4 | ANNEX | ANEX | ANX |    
  | 5 | ANNEX | ANNEX | ANX | 
  | 6 | ANNEX | ANNX | ANX | 
  | 7 | ANNEX | ANX | ANX | 
  | 8 | ARCADE | ARC | ARC | 
  | 9 | ARCADE | ARCADE | ARC | 
  | 10 | AVENUE | AV | AVE |
  | 11 | AVENUE | AVE | AVE |
  | 12 | AVENUE | AVEN | AVE | 
  | 13 | AVENUE | AVENU | AVE | 
  | 14 | AVENUE | AVENUE | AVE |
  | 15 | AVENUE | AVN | AVE | 
  | 16 | AVENUE | AVNUE | AVE | 
  | 17 | BAYOO | BAYOO | BYU |
  | 18 | BAYOO | BAYOU | BYU | 
  | 19 | BEACH | BCH | BCH |



+ For value matching, this was converted into a dictionary with `Commonly Used Street` as __key__ and `Primary Street` as __values__.

  ```py
  # CREATED a dictionary containing USPS suffix abbreviations
  ABBREV = pd.read_excel(filepath + 'USPS SUFFIX ABBREVIATIONS.xlsx')
  # We would convert all values in this dataframe to lower case
  abbrev = ABBREV.apply(lambda x: x.astype(str).str.lower())
  # also we would convert the dataframe to a dictionary for value matching
  abbrev_dict = dict(zip(abbrev['Commonly Used Street'], abbrev['Primary Street']))
  ```

After this, the function `func_block()` was devised to extract the street types, cross match with the USPS abbreviations, and output their corresponding full meanings as a new column.

```py
def func_block(itB):
    eachB = str(itB)
    # ran a regex search on the Block column and output the type of Block.
    searched_txt = re.search("\w*$", itB).group().lower()
    # ran an if-statement that outputs the corresponding full name if there's a match
    if searched_txt in abbrev_dict.keys():
        block_done = abbrev_dict[searched_txt]
    else:
        block_done = searched_txt
    # another if-statement that collates all unmatched regex expressions as 'Not Found'
    return block_done if block_done in abbrev_dict.values() else 'Not Found'

# APPLIED function on the relevant column
crime_tweak['blockType'] = crime_tweak['Block'].apply(func_block)
```

Further streamlining was needed to also discover the exact street a reported crime occurred. This is imperative to conduct analysis by street level, in addition to street-type level.

+ The function `street_func` was created to do just this

```py
def street_func(it2B):
    # PERFORMED regex search and replace apropriately
    regex1st = re.search("\s[N|S|W|E]\s.*$", it2B).group()
    regexOut = re.sub("^\s[N|S|W|E]\s", '', regex1st)
    return regexOut

# APPLIED function on relevant column
crime_tweak['actualStreet'] = crime_tweak['Block'].apply(street_func)
```

### Date

The following features could be extracted from the `Date` column:-

+ Time of day the crime occured - Morning, Afternoon, Evening or Night;
+ Hour of the day the crime occured;
+ Day of the week;
+ Month.

A function `datefunc` was engineered for this purpose.

```py
def datefunc(itD):
    dow = itD.strftime("%A")
    mnth = itD.strftime("%m")
    _hour = itD.strftime("%H")
    int_hour = int(_hour)
    if int_hour < 6:
        tod = 'Night'
    elif int_hour < 12:
        tod = 'Morning'
    elif int_hour < 17:
        tod = 'Afternoon'
    elif int_hour < 20:
        tod = 'Evening'
    elif int_hour >= 20:
        tod = 'Night'
    return dow, _hour, tod, mnth

# APPLIED datefunc() on the relevant column to get a series of outputs
datefunc_applied = crime_tweak['Date'].apply(datefunc)
# CONVERTED series to a DataFrame
date_df = pd.DataFrame(datefunc_applied.tolist())

# CREATED new columns and fill with values gotten from the above operation.
crime_tweak[['dayOfWeek','hourDay','timeOfDay','month']] = date_df
```

And... WE'RE DONE!

<inserts AND ITS A WRAP GIF>

At this point, the dataset has been fully prepared, thus we commence an actual __Analysis__.

> Before that, a final variable-switch to lock in all changes <insert finger emoji> 
>
> `crime_df = crime_tweak`. 

Finally, `crime_df.head().T` gives us an overview of the dataset along with all new features extracted.

|                      | 0                             | 1                             | 2                             | 3                             | 4                                      | 5                             | 6                             | 7                             | 8                             | 9                               |
| :------------------- | :---------------------------- | :---------------------------- | :---------------------------- | :---------------------------- | :------------------------------------- | :---------------------------- | :---------------------------- | :---------------------------- | :---------------------------- | :------------------------------ |
| ID                   | 6407111                       | 11398199                      | 5488785                       | 11389116                      | 12420431                               | 1699235                       | 5061155                       | 9876456                       | 7582927                       | 10566046                        |
| Case Number          | HP485721                      | JB372830                      | HN308568                      | JB361368                      | JE297624                               | G498287                       | HM660983                      | HX527438                      | HS386492                      | HZ313634                        |
| Date                 | 2008-07-26 14:30:00           | 2018-07-31 10:57:00           | 2007-04-27 10:30:00           | 2018-07-23 08:55:00           | 2021-07-11 06:40:00                    | 2001-08-21 00:00:00           | 2006-10-14 22:00:00           | 2014-12-02 11:48:00           | 2010-06-30 01:00:00           | 2016-06-18 23:15:00             |
| Block                | 085XX S MUSKEGON AVE          | 092XX S ELLIS AVE             | 062XX N TRIPP AVE             | 0000X N KEELER AVE            | 016XX W HARRISON ST                    | 003XX W 28 PL                 | 006XX S CENTRAL AVE           | 043XX W POTOMAC AVE           | 032XX W PIERCE AVE            | 021XX N CALIFORNIA AVE          |
| IUCR                 | 1320                          | 143C                          | 0610                          | 0560                          | 051A                                   | 0810                          | 0320                          | 1811                          | 0910                          | 1811                            |
| Primary Type         | CRIMINAL DAMAGE               | WEAPONS VIOLATION             | BURGLARY                      | ASSAULT                       | ASSAULT                                | THEFT                         | ROBBERY                       | NARCOTICS                     | MOTOR VEHICLE THEFT           | NARCOTICS                       |
| Description          | TO VEHICLE                    | UNLAWFUL POSS AMMUNITION      | FORCIBLE ENTRY                | SIMPLE                        | AGGRAVATED - HANDGUN                   | OVER $500                     | STRONGARM - NO WEAPON         | POSS: CANNABIS 30GMS OR LESS  | AUTOMOBILE                    | POSS: CANNABIS 30GMS OR LESS    |
| Location Description | STREET                        | POOL ROOM                     | RESIDENCE                     | NURSING HOME/RETIREMENT HOME  | PARKING LOT / GARAGE (NON RESIDENTIAL) | STREET                        | CTA PLATFORM                  | ALLEY                         | STREET                        | POLICE FACILITY/VEH PARKING LOT |
| Arrest               | False                         | True                          | True                          | False                         | False                                  | False                         | False                         | True                          | False                         | True                            |
| Domestic             | False                         | False                         | False                         | False                         | False                                  | False                         | False                         | False                         | False                         | False                           |
| Beat                 | 423                           | 413                           | 1711                          | 1115                          | 1231                                   | 2113                          | 1513                          | 2534                          | 1422                          | 1414                            |
| District             | 4                             | 4                             | 17                            | 11                            | 12                                     | 2                             | 15                            | 25                            | 14                            | 14                              |
| Ward                 | 10                            | 8                             | 39                            | 28                            | 27                                     | nan                           | 29                            | 37                            | 26                            | 35                              |
| Community Area       | 46                            | 47                            | 12                            | 26                            | 28                                     | nan                           | 25                            | 23                            | 23                            | 22                              |
| FBI Code             | 14                            | 15                            | 05                            | 08A                           | 04A                                    | 06                            | 03                            | 18                            | 07                            | 18                              |
| X Coordinate         | 1196638.0                     | 1184499.0                     | 1146911.0                     | 1148388.0                     | 1165430.0                              | 1174343.0                     | 1139154.0                     | 1147306.0                     | 1154458.0                     | 1157345.0                       |
| Y Coordinate         | 1848800.0                     | 1843935.0                     | 1941022.0                     | 1899882.0                     | 1897441.0                              | 1885951.0                     | 1896536.0                     | 1908305.0                     | 1910116.0                     | 1914452.0                       |
| Year                 | 2008                          | 2018                          | 2007                          | 2018                          | 2021                                   | 2001                          | 2006                          | 2014                          | 2010                          | 2016                            |
| Updated On           | 02/28/2018 03:56:25 PM        | 08/07/2018 04:02:59 PM        | 02/28/2018 03:56:25 PM        | 07/30/2018 03:52:24 PM        | 07/18/2021 04:56:02 PM                 | 08/17/2015 03:03:40 PM        | 02/28/2018 03:56:25 PM        | 02/10/2018 03:50:01 PM        | 02/10/2018 03:50:01 PM        | 02/10/2018 03:50:01 PM          |
| Latitude             | 41.739979622                  | 41.726922145                  | 41.994137622                  | 41.881217483                  | 41.874173691                           | 41.842450075                  | 41.872208575                  | 41.904351902                  | 41.909181404                  | 41.921021491                    |
| Longitude            | -87.555120042                 | -87.599746995                 | -87.734959049                 | -87.730589961                 | -87.668082118                          | -87.635700695                 | -87.764578577                 | -87.734347128                 | -87.708027242                 | -87.69730355                    |
| Location             | (41.739979622, -87.555120042) | (41.726922145, -87.599746995) | (41.994137622, -87.734959049) | (41.881217483, -87.730589961) | (41.874173691, -87.668082118)          | (41.842450075, -87.635700695) | (41.872208575, -87.764578577) | (41.904351902, -87.734347128) | (41.909181404, -87.708027242) | (41.921021491, -87.69730355)    |
| blockType            | avenue                        | avenue                        | avenue                        | avenue                        | street                                 | place                         | avenue                        | avenue                        | avenue                        | avenue                          |
| actualStreet         | MUSKEGON AVE                  | ELLIS AVE                     | TRIPP AVE                     | KEELER AVE                    | HARRISON ST                            | 28 PL                         | CENTRAL AVE                   | POTOMAC AVE                   | PIERCE AVE                    | CALIFORNIA AVE                  |
| dayOfWeek            | Saturday                      | Tuesday                       | Friday                        | Monday                        | Sunday                                 | Tuesday                       | Saturday                      | Tuesday                       | Wednesday                     | Saturday                        |
| hourDay              | 14                            | 10                            | 10                            | 08                            | 06                                     | 00                            | 22                            | 11                            | 01                            | 23                              |
| timeOfDay            | Afternoon                     | Morning                       | Morning                       | Morning                       | Morning                                | Night                         | Night                         | Morning                       | Night                         | Night                           |
| month                | 07                            | 07                            | 04                            | 07                            | 07                                     | 08                            | 10                            | 12                            | 06                            | 06                              |



## DATA ANALYSIS

Let's dive into the data and gain trustworthy insights from charts & graphs!

### **1. How many unique crime reports are in this dataset?**

```PY
year_range = crime_df['Year'].max() - crime_df['Year'].min()
total_crime = crime_df['ID'].nunique()
per_year = total_crime//year_range

print("There have been {0} unique crime reports.\nIn {1} years.\nThis is an estimate of {2} crime reports every year.".format(total_crime, year_range, per_year))
```

> ```PY
> There have been 2278725 unique crime reports.
> In 21 years.
> This is an estimate of 108510 crime reports every year.
> ```

+ This is an interesting summary statistic of the dataset.
+ In the rest of this study, other analytical questions would be answered about demography of location of crime report, time of report, etc. 
+ Ultimately, the goal of this report is to advise the Chicago PD appropriately based on insights gleaned off the given dataset.

### **2. What are the most crime-prone locations?**

```PY
blockG = pd.DataFrame(crime_df.groupby('Block').size(), columns = ['frequency'])
blockG = blockG.sort_values('frequency', ascending = False).reset_index()

blockG.head(16)
```

|      | Block                               | frequency |
| ---: | :---------------------------------- | --------: |
|    0 | 100XX W OHARE ST                    |      4878 |
|    1 | 001XX N STATE ST                    |      4362 |
|    2 | 076XX S CICERO AVE                  |      3010 |
|    3 | 008XX N MICHIGAN AVE                |      2800 |
|    4 | 0000X N STATE ST                    |      2587 |
|    5 | 0000X W TERMINAL ST                 |      1819 |
|    6 | 064XX S DR MARTIN LUTHER KING JR DR |      1692 |
|    7 | 063XX S DR MARTIN LUTHER KING JR DR |      1649 |
|    8 | 023XX S STATE ST                    |      1542 |
|    9 | 001XX W 87TH ST                     |      1346 |
|   10 | 008XX N STATE ST                    |      1320 |
|   11 | 012XX S WABASH AVE                  |      1301 |
|   12 | 0000X S STATE ST                    |      1282 |
|   13 | 022XX S STATE ST                    |      1281 |
|   14 | 006XX N MICHIGAN AVE                |      1276 |
|   15 | 057XX S CICERO AVE                  |      1233 |

### 3. What streets have had the most crime reports?

```PY
# CREATED a table to hold the street values and frequency of occurence
st_freq = pd.DataFrame(crime_df.groupby('actualStreet').size(), columns=['frequency'])
st_freq = st_freq.reset_index()

# FILTER off streets that appear only once in this table
st_freq = st_freq[st_freq['actualStreet']!=1].sort_values('frequency', ascending=False)

# now to plot
plt.figure(figsize=(15, 8))
plt.title('Top 15 Streets with the Highest Crime Rates', fontsize=13)
sns.barplot(data= st_freq[:15], x='actualStreet', y='frequency')
plt.xticks(rotation = 45)
plt.xlabel('Exact Street Crime Occured')
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199010-64e46809-20fb-420e-a53b-afc36da3179f.png">

### **4. Where does crime happen the most in Chicago?**

```py
# We do a groupby on 'blockType' to to get a table of types and frequency
blockTypeG = crime_df.groupby('blockType').size()
blockTypeG = pd.DataFrame(blockTypeG.sort_values(ascending=False),columns=['frequency'])
# We reset the index of this dataframe
blockTypeG = blockTypeG.reset_index()
# FILTER off 'Not Found' values
subBlock = blockTypeG[blockTypeG['blockType'] != 'Not Found']

# now to plot
plt.figure(figsize=(10, 5))
plt.title('Top 10 Street Types where Crime Happens The Most', fontsize=13)

sns.barplot(data=subBlock[:10], x='blockType', y='frequency')
plt.xlabel('Types of Street')
plt.ylabel('Number of Crimes Reported')
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199013-44a01582-4e95-4f6e-8b83-ed4264439563.png">

### **5. What time in the day does crime happen the most?**

#### **5.1 Initial Time Analysis**

```py
# now to plot
plt.figure(figsize=(8,5))
plt.title('Time of Crime Report', fontsize=10)

sns.countplot(data=crime_df, x='timeOfDay')
plt.xlabel('Time in the Day')
plt.ylabel('Number of Crimes Reported')
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199015-2e3cd898-7e19-4cae-9fa0-29c432a5d0c0.png">

#### **5.2 Comprehensive Time Analysis**

```py
crime_df = crime_df.sort_values('hourDay')
# now to plot
plt.figure(figsize=(10,5))
plt.title('Time of Crime Report', fontsize = 10)
#
sns.countplot(data = crime_df , x = 'hourDay', color='blue')
plt.xlabel('Hour in the Day')
plt.ylabel('Number of Crime Reports')
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199016-490fbcff-8c43-4649-8321-6d3ec91cb890.png">

### **6. In what months have the highest number of crimes been reported?**

```py
plt.figure(figsize=(10,5))
plt.title('Month of Crime Reports', fontsize=10)

sns.histplot(data = crime_df.sort_values('month'), x = 'month', color= 'grey')
plt.ylabel('Number of Crime Reports')
plt.xlabel('Months')
plt.show()
```

<img src = "">

### **7. What days of the week is crime reported the most in?**

```py
ordered_days = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']
#order dataframe so that days of the week comes out ordered
crime_df['dayOfWeek'] = crime_df['dayOfWeek'].astype('category').cat.reorder_categories(ordered_days)

#now to plot
plt.figure(figsize=(10,5))
plt.title('Time of Crime Report', fontsize = 10)
#
sns.countplot(data = crime_df , x = 'dayOfWeek')
plt.xlabel('Days of the Week')
plt.ylabel('Number of Crime Reports')
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199021-c80ad6fa-882f-4af4-810b-803a45623bf0.png">

### **8. What types of crime happen most frequently in Chicago?**

```py
# We do a groupby on 'Primary Type' to get a table category and frequency
pt_freq = pd.DataFrame(crime_df.groupby('Primary Type').size(),columns=['frequency'])
# We reset the index of this dataframe
pt_freq = pt_freq.reset_index()
pt_freq = pt_freq.sort_values('frequency', ascending=False)
# now to plot

plt.figure(figsize=(12,6))
plt.title('Top 10 Most Frequent Primary Types of Crime', fontsize=15)

#sns.barplot(data = pt_freq[pt_freq['frequency']>100000], x = 'Primary Type', y='frequency')

sns.barplot(data=pt_freq[:10], x='Primary Type', y='frequency')
plt.ylabel('Number of Crimes Reported')
plt.xticks(rotation = 45)
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199025-758b4066-9212-4e7f-843d-b56818dd4b52.png">

### **Analysis on Elements of Location against Rate of Arrest**

```py
#CREATED a function to accept columns of crime_df - elements of location - and pivots it against Arrest column, as well as a parameter for Bottom N
def locate_func(eachL, bottomN):
    bottomN = int(bottomN)
    # extracted a table of relevant eachL & Arrest values
    df = pd.DataFrame(crime_df.groupby([eachL, 'Arrest']).size())
    df = df.reset_index()
    # pivoted columns based on Arrest into two columns: True or False
    pivot_df  = df.pivot(index = eachL, columns = 'Arrest', values = 0)
    pivot_df = pivot_df.reset_index().rename_axis(None, axis=1)
    # calculated RateofArrest column in percent
    pivot_df['RateOfArrest'] = 100 * pivot_df[True] / (pivot_df[True] + pivot_df[False])
    pivot_df = pivot_df.sort_values('RateOfArrest')
    # plot charts appropriately with relevant parameters
    plt.figure(figsize=(10,4))
    title = "The {0}s with the {1} Lowest Rates of Arrest".format(eachL, bottomN)
    plt.title(label = title, fontsize=12)
    # now to plot
    return sns.barplot(data = pivot_df[:bottomN], x = eachL, y = 'RateOfArrest'), plt.ylabel('Percentage of Rate of Arrest')
```



```py
# now we plot all 4 graphs
locate_func('Beat', 15)
locate_func('Ward', 15)
locate_func('Community Area', 15)
locate_func('District', 15)

plt.ylabel('Percentage of Rate of Arrest')
plt.show()
```

<img src = "https://user-images.githubusercontent.com/60517587/192199029-d9f8dd8e-e746-4282-b26e-58aca4f80f4a.png">

<img src = "https://user-images.githubusercontent.com/60517587/192198990-9ab59b2c-ec66-4e1e-8305-2a252cb6fc47.png">

<img src = "https://user-images.githubusercontent.com/60517587/192198997-2b2da0c4-3386-404e-b9b5-7b5daccab34c.png">

<img src = "https://user-images.githubusercontent.com/60517587/192199006-833b6f31-7c15-4ae6-b28c-7b346d7deecb.png">
