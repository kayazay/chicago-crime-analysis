# A COMPREHENSIVE ANALYSIS OF CHICAGO CRIME DATA _(2001-2022)_

I took on a mission to explore the crime data of a state in the U.S - Chicago, as a Capstone Project to a training program; my aim was to determine the following:
+ __What kinds__ of crime happen the most in Chicago?
+ __When__ is the state the most vulnerable to crime?
+ .. And __where__?
+ __What places__ does crime go unpunished the most?

Contained in this repository are:
+ A Jupyter Notebook _(ipynb)_,
+ Datasets used for analysis,
+ Answers to the questions above, and others.

Enough preamble given, let's dive right into it then!

<img src = "https://user-images.githubusercontent.com/60517587/193845348-305d2338-6c0f-4487-8d74-4fb2edbbda95.gif" width=100% height=20%/>

## DATA PRE-PROCESSING

The following are the libraries installed for this study and a brief why:
+ __pandas__ - manipulation of tabular data
+ __math__ - handling null values
+ __matplotlib__ & __seaborn__ - plotting charts
+ __re__ - REGular EXpressions used for string manipulation

```py
import pandas as pd
import math
import seaborn as sns
import re
import matplotlib.pyplot as plt
```

### Loading the Dataset
```py
filepath  =  "C:/Users/user/Desktop/eze/"
crime  =  pd.read_csv(filepath+"crime_data_Proj1.csv",index_col=0)
```

_**Note:** The variable `filepath` may be tweaked in accordance with the directory that holds the datasets._

### Overview
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

Inferences from this statistic would be highlighted in the next phase. First, though, I switch variables to `crime_clean`.

```py
crime_clean  =  crime
```
This is common practice in my codes - and I do this for readability, to prevent the dataset from mis-updating, and for easier debugging.

## DATA CONVERSION & CLEANING
<img src = "https://user-images.githubusercontent.com/60517587/193684399-a6035d40-2420-4cff-b664-62439da2c3bc.gif" width=50% height=10%/>

### 1. `Date`

> Values in these column are recognized as strings. They should be converted from _object_ to _datetime_.
```py
time_format  =  '%m/%d/%Y %H:%M:%S %p'
crime_clean['Date']  = crime_clean['Date'].apply(lambda  x:  x  if  type(x)!=str  else  dt.strptime(x,  time_format))
```
### 2. `District`, `Ward`, `Community Area` & `Beat`

> These are recognized as floats since they are technically numbers; as opposed to objects since they are really elements of location.
>
>  A custom function - `to_str()` was created to explicitly perform the conversion.

```py
# Created a function that returns null if the value in a column is null & converts to a string if it is not
def to_str(_unit):
    if isinstance(_unit, str) or math.isnan(_unit):
        unit_done = _unit
    else:
        unit_done = str(int(_unit))
    return unit_done

# APPLIED the to_str function on relevant columns
BCDW = ['Beat', 'Community Area', 'District', 'Ward']
crime_clean[BCDW] = crime_clean[BCDW].applymap(to_str)
```

### 3. Investigate for Missing Values

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
```py
  crime_clean = crime_clean.dropna(subset=['Case Number'])
  ```
  > Why? This column holds semi-unique values. i.e a crime incident that doesn't have a `Case Number` is an error that __might__ indicate the crime didn't happen.

+ Other columns that have missing values are:-

  + `Location Description`,
  + `Ward`,
  + `Community Area`,
  + `X & Y Coordinates`,
  + `Lat` & `Long`.

 > All these are elements of location and a missing value indicates the location was not recorded for some reason. Not necessarily that the crime didn't happen.

## DATA MANIPULATION

As per my ritual, once again - variable switch 👌🏽

 `crime_tweak = crime_clean`

Here, we look for more features or columns to extract from our dataset for it to hold more _"analytical water"_. __OBSERVE the following new columns...__

<img src = "https://user-images.githubusercontent.com/60517587/193694512-ccd5d4a4-8a0a-4e30-a41c-62d9ec078a3d.gif" width=50% height=10%/>

### Street Type

This column holds the full address of the location a reported crime occurred. 

> `crime_tweak['Block'].head(40).values` gives us the first few records of this column:

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

There is a pattern in the last word for most records in this column.

> __AVE__, __ST__, __BLVD__, __RD__, ...

This indicates the type of street the crime occurred. What if... 🌟💡 a new column were created to hold this information.

+ _**Note**: These street forms are abbreviated as seen in the array; thus, for ease of compression, there is a need to also output their full meanings._

The __USPS__ suffix abbreviations was referenced to accomplish this.

  > This table is a universal standard for comparing street abbreviations and their full forms.
  > 
  > Link: _https://www.pb.com/docs/us/pdf/sis/mail-services/usps-suffix-abbreviations.pdf_

<details>
    <summary>
        Here are few records from this table for context
    </summary>

| | Primary Street | Commonly Used Street | Postal Service Standard|
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

</details>

For value matching, this was converted into a __dictionary__ with `Commonly Used Street` as __key__ and `Primary Street` as __values__.

  ```py
  # CREATED a dictionary containing USPS suffix abbreviations
  ABBREV = pd.read_excel(filepath + 'USPS SUFFIX ABBREVIATIONS.xlsx')
  # We would convert all values in this dataframe to lower case
  abbrev = ABBREV.apply(lambda x: x.astype(str).str.lower())
  # also we would convert the dataframe to a dictionary for value matching
  abbrev_dict = dict(zip(abbrev['Commonly Used Street'], abbrev['Primary Street']))
  ```

### Actual Street

After this, the function `func_block()` was devised to extract the street types from __`Block`__, cross match with the USPS abbreviations, and output their corresponding full meanings as a new column.

```py
# CREATED a function to extract street type out of the 'Block' column
def func_block(itB):
    eachB = str(itB)
    # ran a regex search on the Block column and output the type of Block.
    searched_txt = re.search("\w*$", itB).group().lower()
    # if-statement that outputs the corresponding full name if there's a match
    if searched_txt in abbrev_dict.keys():
        block_done = abbrev_dict[searched_txt]
    else:
        block_done = searched_txt
    # another if-statement that collates all unmatched regex expressions as 'Not Found'
    return block_done if block_done in abbrev_dict.values() else 'Not Found'

# APPLIED function on the relevant column
crime_tweak["blockType"] = crime_tweak["Block"].apply(func_block)
crime_tweak = crime_tweak.assign(blockType = crime_tweak['Block'].apply(func_block))
```

Further streamlining was needed to also discover the __exact street__ a reported crime occurred. This is imperative to conduct analysis by street level, not just street-type level. The function `street_func` was devised to do just this.
```py
def street_func(it2B):
    # PERFORMED regex search and replace apropriately
    regex1st = re.search("\s[N|S|W|E]\s.*$", it2B).group()
    regexOut = re.sub("^\s[N|S|W|E]\s", '', regex1st)
    return regexOut

# APPLIED function on relevant column
crime_tweak['actualStreet'] = crime_tweak['Block'].apply(street_func)
```

### Components of Date

The following features could be extracted from the `Date` column:-

+ __Time of day__ the crime occured - Morning, Afternoon, Evening,.;
+ __Hour__ of the day the crime occured;
+ __Day of the week__;
+ __Month__.

Another function `datefunc` was engineered for this purpose.
```py
# CREATED a function to extract all the features mentioned above
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
date_df = pd.DataFrame(datefunc_applied.tolist())

# CREATED new columns and fill with values gotten from the above operation.
crime_tweak[['dayOfWeek','hourDay','timeOfDay','month']] = date_df
```

At this point, the dataset has been fully prepared, thus we commence an actual __Analysis__.

<img src = "https://user-images.githubusercontent.com/60517587/193717269-041a3c02-e1aa-4627-a51f-ba5a7da01513.gif" width=40% height=10%/>

Before that, though, a final variable-switch to lock in all changes - `crime_df = crime_tweak`.

<details>
    <summary>
    Finally,we get an overview of the dataset along with all new features extracted.
    </summary>

|                      | 0                             | 1                             | 2                             | 3                             | 4                                      |
|:---------------------|:------------------------------|:------------------------------|:------------------------------|:------------------------------|:---------------------------------------|
| ID                   | 6407111                       | 11398199                      | 5488785                       | 11389116                      | 12420431                               |
| Case Number          | HP485721                      | JB372830                      | HN308568                      | JB361368                      | JE297624                               |
| Date                 | 2008-07-26 02:30:00           | 2018-07-31 10:57:00           | 2007-04-27 10:30:00           | 2018-07-23 08:55:00           | 2021-07-11 06:40:00                    |
| Block                | 085XX S MUSKEGON AVE          | 092XX S ELLIS AVE             | 062XX N TRIPP AVE             | 0000X N KEELER AVE            | 016XX W HARRISON ST                    |
| IUCR                 | 1320                          | 143C                          | 0610                          | 0560                          | 051A                                   |
| Primary Type         | CRIMINAL DAMAGE               | WEAPONS VIOLATION             | BURGLARY                      | ASSAULT                       | ASSAULT                                |
| Description          | TO VEHICLE                    | UNLAWFUL POSS AMMUNITION      | FORCIBLE ENTRY                | SIMPLE                        | AGGRAVATED - HANDGUN                   |
| Location Description | STREET                        | POOL ROOM                     | RESIDENCE                     | NURSING HOME/RETIREMENT HOME  | PARKING LOT / GARAGE (NON RESIDENTIAL) |
| Arrest               | False                         | True                          | True                          | False                         | False                                  |
| Domestic             | False                         | False                         | False                         | False                         | False                                  |
| Beat                 | 423                           | 413                           | 1711                          | 1115                          | 1231                                   |
| District             | 4                             | 4                             | 17                            | 11                            | 12                                     |
| Ward                 | 10                            | 8                             | 39                            | 28                            | 27                                     |
| Community Area       | 46                            | 47                            | 12                            | 26                            | 28                                     |
| FBI Code             | 14                            | 15                            | 05                            | 08A                           | 04A                                    |
| X Coordinate         | 1196638.0                     | 1184499.0                     | 1146911.0                     | 1148388.0                     | 1165430.0                              |
| Y Coordinate         | 1848800.0                     | 1843935.0                     | 1941022.0                     | 1899882.0                     | 1897441.0                              |
| Year                 | 2008                          | 2018                          | 2007                          | 2018                          | 2021                                   |
| Updated On           | 02/28/2018 03:56:25 PM        | 08/07/2018 04:02:59 PM        | 02/28/2018 03:56:25 PM        | 07/30/2018 03:52:24 PM        | 07/18/2021 04:56:02 PM                 |
| Latitude             | 41.739979622                  | 41.726922145                  | 41.994137622                  | 41.881217483                  | 41.874173691                           |
| Longitude            | -87.555120042                 | -87.599746995                 | -87.734959049                 | -87.730589961                 | -87.668082118                          |
| Location             | (41.739979622, -87.555120042) | (41.726922145, -87.599746995) | (41.994137622, -87.734959049) | (41.881217483, -87.730589961) | (41.874173691, -87.668082118)          |
| blockType            | avenue                        | avenue                        | avenue                        | avenue                        | street                                 |
| actualStreet         | MUSKEGON AVE                  | ELLIS AVE                     | TRIPP AVE                     | KEELER AVE                    | HARRISON ST                            |
| dayOfWeek            | Saturday                      | Tuesday                       | Friday                        | Monday                        | Sunday                                 |
| hourDay              | 02                            | 10                            | 10                            | 08                            | 06                                     |
| timeOfDay            | Night                         | Morning                       | Morning                       | Morning                       | Morning                                |
| month                | 07                            | 07                            | 04                            | 07                            | 07                                     |

</details>

## DATA ANALYSIS

Let's dive into the data and gain trustworthy insights from charts & graphs!

### **1. How many unique crime reports are in this dataset?**

<details>
    <summary>
       Code
   </summary>

```py
year_range = crime_df['Year'].max() - crime_df['Year'].min()
total_crime = crime_df['ID'].nunique()
per_year = total_crime//year_range

print("There have been {0} unique crime reports.\nIn {1} years.\nThis is an estimate of {2} crime reports every year.".format(total_crime, year_range, per_year))
```
</details>

```py
There have been 2278725 unique crime reports.
In 21 years.
This is an estimate of 108510 crime reports every year.
```

+ This is an interesting summary statistic of the dataset.
+ In the rest of this study, other analytical questions would be answered about demography of location of crime report, time of report, etc. 
+ Ultimately, the goal of this report is to advise the Chicago PD appropriately based on insights gleaned off the given dataset.

### **2. What are the most crime-prone locations?**

<details>
    <summary>
       Code
      </summary>

```py
blockG = pd.DataFrame(crime_df.groupby('Block').size(), columns = ['frequency'])
blockG = blockG.sort_values('frequency', ascending = False).reset_index()

blockG.head(16)
```
</details>


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

+ These addresses are masked for data protection reasons, and that's understandable. However, one clear insight from this is that some houses are targeted more than others by criminals.
+ Places like this would definitely require the concentrated efforts of the police department to curtail crime rate.
+ Something else is observed though- specific streets are being repeated, throughout these 15 locations.
+ Perhaps it's not just a house that's at the receiving end of high-rate crime, but the whole street? Next chart tells us more.

### **3. What streets have had the most crime reports?**

<details>
    <summary>
       Code to plot chart
      </summary>

```py
# CREATED a table to hold the street names grouped by frequency of occurence
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
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199010-64e46809-20fb-420e-a53b-afc36da3179f.png">

+ Out of the 15 streets shown on this chart, 2 stand out the most - __STATE ST__ & __MICHIGAN AVE__ with very high frequency of crime reports.
+ This is not by chance; if investigated properly, the reason for this gaping figures would be discovered, and appropriate actions taken.

### **4. Where does crime happen the most in Chicago?**

<details>
    <summary>
       Code to plot chart
      </summary>

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
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199013-44a01582-4e95-4f6e-8b83-ed4264439563.png">

+ Crime occurs the most in __streets__ & __avenues__ than it does any other place.

### **5. What time in the day does crime happen the most?**

<img src = "https://user-images.githubusercontent.com/60517587/193710030-fe8e74b5-1e04-4916-88e6-736772a8d6cb.gif" width=50% height=10%>

#### **5.1 Initial Time Analysis**

<details>
    <summary>
       Code to plot chart
      </summary>

```py
plt.figure(figsize=(8,5))
plt.title('Time of Crime Report', fontsize=10)

sns.countplot(data=crime_df, x='timeOfDay')
plt.xlabel('Time in the Day')
plt.ylabel('Number of Crimes Reported')
plt.show()
```
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199015-2e3cd898-7e19-4cae-9fa0-29c432a5d0c0.png">

+ Crime occurs the most at __Night__ in Chicago, which doesn't come as a shock.
+ But what does, is that the 2nd highest is the _Afternoon__. That's strange, why would there be high rate of crime in broad daylight?
+ Perhaps it has something to do with wrong shift-allocation or deployment of police personnel to high crime areas.
+ Could a more detailed chart provide more insight?

#### **5.2 Comprehensive Time Analysis**

<details>
    <summary>
       Code to plot chart
      </summary>

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
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199016-490fbcff-8c43-4649-8321-6d3ec91cb890.png">

+ This chart fortifies the previous understanding we had gotten from our initial time analysis.
+ Streamlining it down to the hour, crime occurs the most at __12 noon__ & __12 midnight__, as well as between __18:00 - 22:00__.
+ This could help in appropriate shift-planning and deployment strategy.

### **6. In what months have the highest number of crimes been reported?**

<details>
    <summary>
       Code to plot chart
      </summary>

```py
plt.figure(figsize=(10,5))
plt.title('Month of Crime Reports', fontsize=10)

sns.histplot(data = crime_df.sort_values('month'), x = 'month', color= 'grey')
plt.ylabel('Number of Crime Reports')
plt.xlabel('Months')
plt.show()
```
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199017-85788452-c687-4071-a453-3589b06924a2.png">

+ The middle of the year - between May and August - records more crime than all other month. Investigation on why this is so is beyond this study.

### **7. What days of the week is crime reported the most in?**

<details>
    <summary>
       Code to plot chart
      </summary>

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
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199021-c80ad6fa-882f-4af4-810b-803a45623bf0.png">

+ On _prima facie_, one might conclude that this chart holds no statistical inference.
+ However a closer look at the bars indicate that __Friday__ & __Saturday__ have a more frequent crime occurrence than all other days.
+ Might this be as a result of relaxation of security laws because of the weekends?

### **8. What types of crime happen most frequently in Chicago?**

<details>
    <summary>
       Code to plot chart
      </summary>

```py
# We do a groupby on 'Primary Type' to get a table category and frequency
pt_freq = pd.DataFrame(crime_df.groupby('Primary Type').size(),columns=['frequency'])
# We reset the index of this dataframe
pt_freq = pt_freq.reset_index()
pt_freq = pt_freq.sort_values('frequency', ascending=False)

# now to plot
plt.figure(figsize=(12,6))
plt.title('Top 10 Most Frequent Primary Types of Crime', fontsize=15)
#
sns.barplot(data=pt_freq[:10], x='Primary Type', y='frequency')
plt.ylabel('Number of Crimes Reported')
plt.xticks(rotation = 45)
plt.show()
```
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199025-758b4066-9212-4e7f-843d-b56818dd4b52.png">

### **Elements of Location against Rate of Arrest**

It's not enough to discover where crime is being reported the most in, it's worth also considering the "success rate" of arrests in these respective locations.
In other words, __where does crime go unpunished__ the most?
To make these insights worthwhile, 4 elements of location were considered:
+ __`District`, `Ward`, `Community Area` & `Beat`__ 

For ease and reusability, a function- `locate_func()` was created to accept two inputs: 
+ name of respective column,
+ bottom N parameter.

```py
#CREATED a function to accept elements of location and pivot it against Arrest column, as well as a parameter for Bottom N
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

Inferences from the resulting charts would serve as an advisement to the Chicago State Police Department as to where crime is left unchecked the most within Chicago.
Thus, there could be logical deployment of more military personnel in locations with the lowest Rate of Arrest.

<details>
    <summary>
       Code to plot chart
      </summary>

```py
# now we plot all 4 graphs
locate_func('Beat', 15)
locate_func('Ward', 15)
locate_func('Community Area', 15)
locate_func('District', 15)

plt.ylabel('Percentage of Rate of Arrest')
plt.show()
```
</details>

<img src = "https://user-images.githubusercontent.com/60517587/192199029-d9f8dd8e-e746-4282-b26e-58aca4f80f4a.png">

+ `Beat` __1214__, __1935__, __235__ have pretty low rates of arrest when compared to the others.
+ _An example of how the chart could be read:_ `Beat` __1214__ has a rate of arrest of around 7%, this could be interpreted as - Out of a 100 crime reports in this `Beat`, only 7 apprehensions were made. This, of course, is a very poor stat.

<img src = "https://user-images.githubusercontent.com/60517587/192198990-9ab59b2c-ec66-4e1e-8305-2a252cb6fc47.png">

+ `Ward` __43__ particularly has close to 12% rate of arrest.
+ However, other Wards have rate of arrest of 15% and above; this is still poor, yet comfortable compared to others.

<img src = "https://user-images.githubusercontent.com/60517587/192198997-2b2da0c4-3386-404e-b9b5-7b5daccab34c.png">

+ Similar poor statistics can be seen for `Community Area`.

<img src = "https://user-images.githubusercontent.com/60517587/192199006-833b6f31-7c15-4ae6-b28c-7b346d7deecb.png">

+ `District` have high enough rate of arrest, with the lowest being close to 20% and this is impressive.

## And... WE'RE DONE!

<img src = "https://user-images.githubusercontent.com/60517587/193709337-8d64de0e-e0ca-4c52-ad92-cd81b83d3c13.gif" width=50% height=10%>

To summarize, Chicago state is not the safest place to be- note that this is taken out of context from other states- and there seems to be multiple security issues that could be worked on.

This has been an interesting study, with a lot of analytical insights within as I'm sure you'd agree.

---

<p>&copy; 2022 Kingsley Izima</p>
