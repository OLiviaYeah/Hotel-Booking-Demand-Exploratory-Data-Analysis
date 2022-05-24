# Hotel-Booking-Demand-Exploratory-Data-Analysis

## 1.LOAD DATASET AND USEFUL PACKAGES

```
#load packages
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt 
import seaborn as sns
import plotly.express as px
​
#visulizaion package
%matplotlib inline
​
#negative number 
plt.rcParams['axes.unicode_minus']=False
​
#warnings
import warnings
warnings.filterwarnings('ignore')
```

```
#load data
df=pd.read_csv('../input/hotel-booking-demand/hotel_bookings.csv')
```

## 2.FIRST IMPRESSION OF DATA

```
df.head()
```
<img width="841" alt="1" src="https://user-images.githubusercontent.com/101906347/170097605-fbdc6f10-3bee-430a-96ac-f72c3b4cd6b8.png">

```
df.info()
```
<img width="502" alt="2" src="https://user-images.githubusercontent.com/101906347/170097652-a6793652-96c4-4f3f-b2cf-91fca3cf89aa.png">

> This data set has 119390 rows and 32 columns,and there are some missing values.
> 
> * hotel: The name of the hotel, city hotel or resort hotel
> * is_canceled: Boolean value stored as int64, indicates the hotel is canceled or not
> * lead_time: Number of days that elapsed between the entering date of the booking into the PMS and the arrival date
> * arrival_date*: Information of the arrival date
> * stay in_weekend_nights: Number of weekend nights (Saturday or Sunday) the guest stayed or booked to stay at the hotel
> * stay in_week_nights: Number of week nights (Monday to Friday) the guest stayed or booked to stay at the hotel
> * adults/children/babies: Number of adults/children/babies
> * meal: Type of meal booked. Categories:
> * Undefined/SC – no meal package;
> * BB – Bed & Breakfast;
> * HB – Half board (breakfast and one other meal – usually dinner);
> * FB – Full board (breakfast, lunch and dinner)
> * country: Country of origin.
> * market_segment: Market segment designation. In categories, the term “TA” means “Travel Agents” and “TO” means “Tour Operators”
> * distribution_channel: Booking distribution channel. The term “TA” means “Travel Agents” and “TO” means “Tour Operators”
> * is_repeated_guest: Value indicating if the booking name was from a repeated guest (1) or not (0)

## 3.DATA CLEANSING 

* Fix missing values
* Fix exception values
* Change the data type

```
#missing value
df.isnull().sum()[df.isnull().sum()!=0]
```
<img width="149" alt="3" src="https://user-images.githubusercontent.com/101906347/170098306-2a6aaf02-9ca6-43ca-be42-acd2fd665474.png">

> Children, country, agent, company have missing values.
> 
> * **children** is probably because there are no children check-in, so the missing values can be filled with 0;
> 
> * **country** can be replaced by mode;
> 
> * there are too many missing values in **agent** and **company**, the observation index can be deleted.

```
#fix missing values
df['children']=df['children'].fillna(0)
df['country']=df['country'].fillna(value=df.country.mode()[0])
df.drop(['agent'],axis=1,inplace=True)
df.drop(['company'],axis=1,inplace=True)
```

```
#check
df.isnull().sum()[df.isnull().sum()!=0]
```
<img width="205" alt="4" src="https://user-images.githubusercontent.com/101906347/170098700-4c13f461-ab1e-42ae-8f72-7c5070674d5c.png">

```
#Exception handling
​
# 0 customer check-in
​
zero_guest=df[df[['adults', 'children', 'babies']].sum(axis=1)==0]
df.drop(zero_guest.index, inplace=True)
​
# stay 0 night
zero_days = df[df[['stays_in_weekend_nights',
                       'stays_in_week_nights']].sum(axis=1) == 0]
df.drop(zero_days.index, inplace=True)
​
# combine Undefined/SC
df.meal.replace("Undefined", "SC", inplace=True)
df.info()#Exception handling
​
# 0 customer check-in
​
zero_guest=df[df[['adults', 'children', 'babies']].sum(axis=1)==0]
df.drop(zero_guest.index, inplace=True)
​
# stay 0 night
zero_days = df[df[['stays_in_weekend_nights',
                       'stays_in_week_nights']].sum(axis=1) == 0]
df.drop(zero_days.index, inplace=True)
​
# combine Undefined/SC
df.meal.replace("Undefined", "SC", inplace=True)
df.info()
```

<img width="526" alt="Screen Shot 2022-05-24 at 2 34 21 PM" src="https://user-images.githubusercontent.com/101906347/170107789-1b1304a0-8c91-475b-be79-51d1ade12348.png">

> Data cleaning is complete, and the dataset size becomes 118565×30

```
#Change the data type (change reservation_status_date to date type)
df['reservation_status_date']=df['reservation_status_date'].astype('datetime64[ns]')

#Add a column for 'arrival_date' and make data type as 'date'
df['arrival_date']=df['arrival_date_year'].map(str)+'/'+df['arrival_date_month'].map(str)+'/'+df['arrival_date_day_of_month'].map(str)
df['arrival_date']=df['arrival_date'].astype('datetime64[ns]')

#check data type 
df.info()
```
<img width="548" alt="6" src="https://user-images.githubusercontent.com/101906347/170099287-88e57b79-e4b9-4f12-9d63-32e18d35126d.png">

## 4.ANALYSIS AND VISULIZATION

**4.1 Basic Operation Analysis**

1) Compare the total revenue of Resort & City Hotel by year and month

```
#Comparison of the total annual revenue of two hotels
df['total_adr']=(df['stays_in_weekend_nights']+df['stays_in_week_nights'])*df['adr']
df.pivot_table(values='total_adr',index='arrival_date_year',columns='hotel',aggfunc='sum').plot.bar()
plt.show()
```
<img width="400" alt="7" src="https://user-images.githubusercontent.com/101906347/170099662-ddd152e3-2c21-4f3c-b44a-9d937f7ba809.png">

> City Hotel has **higher** annual sales than Resort Hotel, except in 2015.

```
#Comparison of the total monthly revenue of two hotels
df['arrival_date_month']=df['arrival_date_month'].map(
                                                      {'July':7,'August':8,'September':9,'October':10,'November':11,'December':12,
                                                       'January':1,'February':2,'March':3,'April':4,'May':5,'June':6,})
df.pivot_table(values='total_adr',index='arrival_date_month',columns='hotel',aggfunc='sum').plot.bar()
plt.show()
```
<img width="384" alt="8" src="https://user-images.githubusercontent.com/101906347/170099914-5618cdcc-782f-47f8-98a2-1c678a17c0a1.png">

> The price fluctuation of City Hotel is smaller.

> For Resort Hotel, revenue is extremely **cyclical**. The peak season is in summer (July and Augest)

2) Comparison of room daily rates

```
#Comparison of the average daily rate of two hotels by months
df.pivot_table(values='adr',index='arrival_date_month',columns='hotel',aggfunc='mean').plot()
```

<img width="395" alt="9" src="https://user-images.githubusercontent.com/101906347/170100157-0679953c-4dd0-4513-ba76-471ed43f8f5d.png">

> * Most of the time City Hotel has higher daily rates than Resort Hotel. 
> * Resort Hotel's daily price vary greatly, and the average daily rate in July and August are higher than City Hotel, which may be the reason why their monthly revenue is higher than City Hotels during this time.

```
#Comparison of the average daily rate of two hotels depends on room types
sns.factorplot(x='reserved_room_type',y='adr',hue='hotel',data=df.query('adr<1000'),size=6,kind='box',aspect=1)
plt.title("Average Daily Rate for Rooms in Different Room Types", fontsize=20)
plt.xlabel("Room Type", fontsize=16)
plt.ylabel("Daily Rate", fontsize=16)
plt.show()
```

<img width="590" alt="10" src="https://user-images.githubusercontent.com/101906347/170100422-a554f507-86c2-40bf-b9ee-99e7f389ca92.png">

> * Resort Hotel is generally low-priced and have more room types. The price is generally around 100, which is affected by the minimum value;
> 
> * City Hotel's room consumption is higher than that of Resort Hotels, the median is at the middle level, and the extreme value has little impact.

```
#Comparison of the average daily rate for rooms in different market segments
sns.factorplot(x='market_segment',y='adr',hue='hotel',data=df.query('adr<400'),size=6,kind='box',aspect=1)
plt.title("Average Daily Rate for Rooms in Different Market Segments", fontsize=20)
plt.xlabel("Market Segments", fontsize=16)
plt.ylabel("Average Daily Room Rate ", fontsize=16)
plt.show()
```

<img width="627" alt="11" src="https://user-images.githubusercontent.com/101906347/170100804-3025affc-8174-4014-803e-f1d43d17f3f1.png">

> From the boxplot, it can be seen that most of the two hotels are booked **online** or **direct**, which is related to their reasonable prices. However, the prices of booking from airlines are high, and there are fewer bookings through this market segment.

3) Occupancy Rate vs Cancellation Rates

```
#Whether the reservations are cancelled or not
a=df[df['is_canceled']==0].groupby('hotel').is_canceled.count()
b=df[df['is_canceled']==1].groupby('hotel').is_canceled.count()
data=pd.DataFrame({'hotel':a.index,
                 '0':a.values,
                 '1':b.values
                  })
data['NOT cancell']=data['0']/(data['0']+data['1'])
data['IS cancelled']=data['1']/(data['0']+data['1'])
data
```

<img width="385" alt="12" src="https://user-images.githubusercontent.com/101906347/170101119-148caeb7-1702-45b1-a2e5-bbf36aa8960f.png">

> Relatively speaking, overall City Hotels have **higher cancellation rates** than Resort Hotels.

**Factors affecting cancellation rate**

1. previous cancellation
2. lead time
3. deposite type
4. distribution_channel

```
#previous cancellation

plt.figure(figsize=(10,6))
df1=df.groupby('previous_cancellations')['is_canceled'].describe()
sns.regplot(x=df1.index,y=df1['mean']*100,data=df1,color='g')
plt.title('how previous cancellation affecting cancellation rate',fontsize=20)
plt.xlabel('Number of previous bookings cancelled before the current booking',fontsize=16)
plt.ylabel('Cancellation rate (%)',fontsize=16)
plt.show()
```

<img width="636" alt="13" src="https://user-images.githubusercontent.com/101906347/170101350-4cfbbe97-bf16-4074-8fcd-e1ff9b5c665b.png">

> The higher the number of previous cancellation, the cancellation rate is higher.

```
#lead time 

plt.figure(figsize=(10,6))
df2=df.groupby('lead_time')['is_canceled'].describe()
sns.regplot(x=df2.index,y=df2['mean']*100,data=df2,color='g')
plt.title('How lead time affecting cancellation rate',fontsize=20)
plt.xlabel('Lead time',fontsize=16)
plt.ylabel('Cancellation rate (%)',fontsize=16)
plt.show()
```

<img width="680" alt="14" src="https://user-images.githubusercontent.com/101906347/170101561-7903948b-08ac-41a9-8154-6462231dd460.png">

> From the scatter plot, it can be found that the cancellation rate has a certain relationship with the lead time. The longer the advance booking time is, the higher the cancellation rate is.

```
#deposite type
df3=df.groupby('deposit_type')['is_canceled'].describe()
plt.figure(figsize=(8,6))
sns.barplot(x=df3.index,y=df3['mean']*100,data=df3)
plt.title("How deposit type affecting cancellation rate", fontsize=20)
plt.xlabel("Deposit type", fontsize=16)
plt.ylabel("Cancellation rate (%)", fontsize=16)
plt.show()
```
<img width="554" alt="15" src="https://user-images.githubusercontent.com/101906347/170101836-1df7be4d-74d3-423a-95f1-87665eb7896f.png">

> **Non refund** payment method has the highest cancelation rate.

```
#distribution channel

df4=df.groupby('distribution_channel')['is_canceled'].describe()
plt.figure(figsize=(8,6))
sns.barplot(x=df4.index,y=df4['mean']*100,data=df4)
plt.title("How distribution channel affecting cancellation rate", fontsize=20)
plt.xlabel("Distribution channel", fontsize=16)
plt.ylabel("Cancellation rate (%)", fontsize=16)
plt.show()
```

<img width="554" alt="16" src="https://user-images.githubusercontent.com/101906347/170102036-6a37bc69-75ee-458d-9abf-ba5f079d1c65.png">

> Customers booking under the **Undefined** channel have a higher cancellation rate.

4.2 Customer Analysis

* nationality distribution
* arrival date
* lead time
* total stay
* distribution channel
* meal options
* car parking
* returning customer

```
# nationality distribution

df5=pd.DataFrame({'Total number of customers':df['country'].value_counts()})
px.choropleth(df5,
        locations=df5.index,
        color='Total number of customers',
        hover_name='Total number of customers',
        color_continuous_scale=px.colors.sequential.Plasma,
        title="Customer nationality distribution").show()
```

<img width="907" alt="17" src="https://user-images.githubusercontent.com/101906347/170102440-5caf5253-c0e3-4ca2-9dbc-785e1e46194a.png">

> Customers are mainly concentrated in **Europe**.

```
#arrival date (by month)

plt.figure(figsize=(16,5))
plt.subplot(1,2,1)
df.query("hotel=='City Hotel'").arrival_date_month.value_counts().sort_index().plot.bar()
plt.title("City Hotel Arrival Date(by month)", fontsize=20)
plt.xlabel("Month", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.subplot(1,2,2)
df.query("hotel=='Resort Hotel'").arrival_date_month.value_counts().sort_index().plot.bar()
plt.title("Resort Hotel Arrival Date(by month)）", fontsize=20)
plt.xlabel("Month", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.show()#arrival date (by month)

plt.figure(figsize=(16,5))
plt.subplot(1,2,1)
df.query("hotel=='City Hotel'").arrival_date_month.value_counts().sort_index().plot.bar()
plt.title("City Hotel Arrival Date(by month)", fontsize=20)
plt.xlabel("Month", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.subplot(1,2,2)
df.query("hotel=='Resort Hotel'").arrival_date_month.value_counts().sort_index().plot.bar()
plt.title("Resort Hotel Arrival Date(by month)）", fontsize=20)
plt.xlabel("Month", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.show()
```

<img width="988" alt="18" src="https://user-images.githubusercontent.com/101906347/170102816-3307e31b-0a44-413f-812f-2c73cdbf5cb9.png">

> The frequency of hotel occupancy in **City Hotel** from April to October do not change much.  January, February, November and December are the off-season for occupancy;

> July and August are the peak occupancy season for **Resort Hotel**, which is likely to be related to the peak tourist season at this time.

```
#lead time

sns.factorplot(x='arrival_date_month',y='lead_time',hue='hotel',data=df,size=6)
plt.title("Lead time (by month)", fontsize=20)
plt.xlabel("Month", fontsize=16)
plt.ylabel("Lead Time", fontsize=16)
plt.show()
```

<img width="534" alt="19" src="https://user-images.githubusercontent.com/101906347/170103172-4499b2fa-d273-4275-a27b-0746fd534826.png">

> Customers who arrive in July and August (City Hotel), in May, June and September (Resort Hotel),they prefer book long time ago.

```
#Total Stay
df['total_stay']=df['stays_in_weekend_nights']+df['stays_in_week_nights']
plt.figure(figsize=(12,5))
plt.subplot(1,2,1)
df.query("total_stay<30&hotel=='City Hotel'").total_stay.plot.hist(bins=15,color='r')
plt.title("Total Stay of City Hotel ", fontsize=20)
plt.xlabel("days）", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.subplot(1,2,2)
df.query("total_stay<30&hotel=='Resort Hotel'").total_stay.plot.hist(bins=15)
plt.title("Total Stay of Resort Hotel", fontsize=18)
plt.xlabel("days", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.show()#Total Stay
df['total_stay']=df['stays_in_weekend_nights']+df['stays_in_week_nights']
plt.figure(figsize=(12,5))
plt.subplot(1,2,1)
df.query("total_stay<30&hotel=='City Hotel'").total_stay.plot.hist(bins=15,color='r')
plt.title("Total Stay of City Hotel ", fontsize=20)
plt.xlabel("days）", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.subplot(1,2,2)
df.query("total_stay<30&hotel=='Resort Hotel'").total_stay.plot.hist(bins=15)
plt.title("Total Stay of Resort Hotel", fontsize=18)
plt.xlabel("days", fontsize=16)
plt.ylabel("Frequency", fontsize=16)
plt.show()
```

<img width="814" alt="20" src="https://user-images.githubusercontent.com/101906347/170103553-7c4a0c38-ffa8-4af2-8a79-17c848dd9e80.png">

> Most City Hotel stay within **5** days;
Most Resort Hotel stay within **7** days.

```
#different meal 

plt.figure(figsize=(12,6))
plt.subplot(1,2,1)
plt.pie(df.query("hotel=='City Hotel'").meal.value_counts(),labels=df.query("hotel=='City Hotel'").meal.value_counts().index,autopct='%.2f%%')
plt.title("different meal options in city hotels (%)", fontsize=20)
plt.legend()
plt.subplot(1,2,2)
plt.pie(df.query("hotel=='Resort Hotel'").meal.value_counts(),labels=df.query("hotel=='Resort Hotel'").meal.value_counts().index,autopct='%.2f%%')
plt.title("different meal options in Resort Hotel (%)", fontsize=20)
plt.legend()
plt.show()
```

<img width="802" alt="21" src="https://user-images.githubusercontent.com/101906347/170103798-8c9826d6-3262-4469-b280-17d9171aed65.png">

> City Hotel and Resort Hotel have roughly the same meal choices, **breakfast only (BB)** is the most popular option.

```
#booking channel
plt.figure(figsize=(12,6))
plt.subplot(1,2,1)
plt.pie(df.query("hotel=='City Hotel'").distribution_channel.value_counts(),labels=df.query("hotel=='City Hotel'").distribution_channel.value_counts().index,autopct='%.2f%%')
plt.title("Proportion of City Hotel booking channel selection (%)", fontsize=20)
plt.legend()
plt.subplot(1,2,2)
plt.pie(df.query("hotel=='Resort Hotel'").distribution_channel.value_counts(),labels=df.query("hotel=='Resort Hotel'").distribution_channel.value_counts().index,autopct='%.2f%%')
plt.title("Proportion of Resort Hotel booking channel selection (%)", fontsize=20)
plt.legend()
plt.show()
```

<img width="937" alt="22" src="https://user-images.githubusercontent.com/101906347/170106888-a4d0f527-cd55-415e-8220-044824fa240d.png">

> The choice of booking channels for City Hotel and Resort hotel is roughly the same, with the vast majority of bookings made through **travel agencies** and **tour operators.**

```
# car parking
sns.countplot(x='required_car_parking_spaces',hue='hotel',data=df)
```

<img width="537" alt="23" src="https://user-images.githubusercontent.com/101906347/170107113-6c4c12aa-02b7-4e0c-9b63-4bb4187926b4.png">

> The majority of customers do not need a parking space.
Resort Hotel customers require more parking spaces than City Hotels.

```
# returning customers
tick_label = ['New Guest', 'Repeated Guest']
sns.countplot(x='is_repeated_guest', hue='is_canceled', data=df)
plt.xticks([0, 1], tick_label)
```

<img width="463" alt="24" src="https://user-images.githubusercontent.com/101906347/170107391-4488c683-95f2-4f16-9f33-c2b578077f4b.png">

> Majority booking come from new guest.
