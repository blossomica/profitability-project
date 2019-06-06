
# Evaluating Users Most Liked App - Project

The idea behind this project is to help developers understand which type of apps are likely to attract more users. 

There is a high chance that the most liked apps are also the ones that could lead to an increase in revenue.

To figure that out, I am exploring apps from Google Apps Store and the Apple Store. 

The data used to create this evaluation can be found on Kaggle ->  [Google Store data set](https://www.kaggle.com/lava18/google-play-store-apps/home) and [Apple Store data set](https://www.kaggle.com/ramamet4/app-store-apple-data-set-10k-apps/home). 

Without further ado, let's get started!

I'll begin by creating a function that helps print the rows and columns of a given data set. So that I can explore the data set I will be eventually importing.


```python
def explore_data(dataset, start, end, rows_and_columns=False):
    dataset_slice = dataset[start:end]
    for row in dataset_slice:
        print(row)
        
        # Adds new line after a row
        print('\n')
        
    if rows_and_columns:
        print("Number of rows:", len(dataset))
        print("Number of columns:", len(dataset[0]))
```

Now let's import the data we got from Kaggle, read it and store it as a list of lists. I'll store the header values separately.


```python
from csv import reader

# The Google Play data set #
opened_file = open('googleplaystore.csv')
read_file = reader(opened_file)
android = list(read_file)
android_header = android[0]
android = android[1:]

# The App Store data set #
opened_file = open('AppleStore.csv')
read_file = reader(opened_file)
ios = list(read_file)
ios_header = ios[0]
ios = ios[1:]
```

I want to explore our data by printing the number of rows, and the count of rows/column of each data set, to see what's in it.

I'll also print the headers separately to ensure we have stored the right information in the header.


```python
explore_data(android, 0, 3)
explore_data(ios, 0, 3)

# Printing number of rows/columns for android apps in Google Store
row_an = len(android)
column_an = len(android[1])
print("Number of colums: ", column_an, " and rows ", row_an, " for Google store")

# Printing number of rows/columns for iOS apps in Apple Store
row_ios = len(ios)
column_ios = len(ios[1])
print("Number of colums: ", column_ios, " and rows ", row_ios, " for Apple store")
print('\n')

# Printing column names for android and ios apps
print(android_header)
print('\n')
print(ios_header)


```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    Number of colums:  13  and rows  10841  for Google store
    Number of colums:  16  and rows  7197  for Apple store
    
    
    ['App', 'Category', 'Rating', 'Reviews', 'Size', 'Installs', 'Type', 'Price', 'Content Rating', 'Genres', 'Last Updated', 'Current Ver', 'Android Ver']
    
    
    ['id', 'track_name', 'size_bytes', 'currency', 'price', 'rating_count_tot', 'rating_count_ver', 'user_rating', 'user_rating_ver', 'ver', 'cont_rating', 'prime_genre', 'sup_devices.num', 'ipadSc_urls.num', 'lang.num', 'vpp_lic']


## Cleaning data

Some rows are incomplete, for example below we see the column of the rating comes as "19" which is wrong. So we need to clean our data by deleting that row to fix it.


```python
# Rating comes 19 which is wrong
print(android[10472]) 
```

    ['Life Made WI-Fi Touchscreen Photo Frame', '1.9', '19', '3.0M', '1,000+', 'Free', '0', 'Everyone', '', 'February 11, 2018', '1.0.19', '4.0 and up']



```python
#We delete the wrong row and check previous and current count
print(len(android))

del android[10472]  
print(len(android))
```

    10841
    10840


The data has a number of duplicate entries so we would have to delete them. Look below for an example of duplicate rows. In the dataset for android apps we see facebook existing twice. The only difference appears in column with index 3 which shows the numbers of ratings. The lesser number indicates the data was collected earlier than than that with a larger rating count.


```python
for app in android:
    name = app[0]
    if name == "Facebook":
        print(app)
```

    ['Facebook', 'SOCIAL', '4.1', '78158306', 'Varies with device', '1,000,000,000+', 'Free', '0', 'Teen', 'Social', 'August 3, 2018', 'Varies with device', 'Varies with device']
    ['Facebook', 'SOCIAL', '4.1', '78128208', 'Varies with device', '1,000,000,000+', 'Free', '0', 'Teen', 'Social', 'August 3, 2018', 'Varies with device', 'Varies with device']


I won't be removing the duplicates randomly. I will be removing only the ones with the lowest rating count number. That because, as said, indicates the data being older than that with a higher count.


```python
reviews_max = {}
for app in android:
    name = app[0]
    n_reviews = float(app[3])
    if name in reviews_max and (reviews_max[name] < n_reviews):
        reviews_max[name] = n_reviews
    if name not in reviews_max:
        reviews_max[name]=n_reviews
        
print(len(reviews_max))
```

    9659


Now we'll begin cleaning the data from duplicates.


```python
# Will store our clean data set here
android_clean = [] 
already_added=[]

for app in android:
    name = app[0]
    n_reviews = float(app[3])
    if name in reviews_max and name not in already_added:
        android_clean.append(app)
        already_added.append(name)
        
print(len(android_clean))
```

    9659


Some apps are not English, we want to eventually remove them. Below I am creating a function which tells me if the given input is a valid English word or not.


```python
# Function that checks if string is English
def belongs(word): 
    for char in word:
        if ord(char) > 127:
            return False
    return True
```

Let's check to see if our function can tell if a given input is or isn't an Eglish word.


```python
belongs("Instagram")
```




    True




```python
belongs("爱奇艺PPS -《欢乐颂2》电视剧热播")

```




    False




```python
belongs("Docs To Go™ Free Office Suite")

```




    False




```python
belongs("Instachat 😜")

```




    False



Our previous code has some **issues.** It thinks that things like emojis automatically make a word non English. To minimize such occurences, we will allow up to 3 non English characters to exist in a word before we qualify it as non-English. 

That still doesn't make things perfect but it reduces the errors.


```python
# Function that checks if string is English
def belongs(word): 
    count = 0
    for char in word:
        if ord(char) > 127:
            count+=1
            if count > 3:
                return False
    return True
```


```python
belongs("爱奇艺PPS -《欢乐颂2》电视剧热播")

```




    False




```python
belongs("Docs To Go™ Free Office Suite")
```




    True




```python
belongs("Instachat 😜")
```




    True




```python
android_english = []
ios_english = []

for app in android_clean:
    name = app[0]
    if belongs(name):
        android_english.append(app)
        
for app in ios:
    name = app[1]
    if belongs(name):
        ios_english.append(app)
        
explore_data(android_english, 0, 3, True)
print('\n')
explore_data(ios_english, 0, 3, True)


```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    Number of rows: 9614
    Number of columns: 13
    
    
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    Number of rows: 6183
    Number of columns: 16


## Remove Paid Apps

Currently we have both paid and none paid apps in our data. We need to only get the free apps. So let's go ahead and fix that.


```python
# Here I'll store the free android/ios apps
free_android = []
free_ios = []

# Checking if the price of the app equals to zero
for app in android_english:
    price = app[7]
    if price == '0':
        free_android.append(app)
        
for app in ios_english:
    price = app[4]
    if price == '0.0':
        free_ios.append(app)
        
        
explore_data(free_android, 0, 3, True)
print('\n')
explore_data(free_ios, 0, 3, True)

```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    Number of rows: 8862
    Number of columns: 13
    
    
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    Number of rows: 3222
    Number of columns: 16


## Figuring which apps attract more users

Our aim is to determine the kinds of apps that are likely to attract more users because our revenue is highly influenced by the number of people using our apps. Because our end goal is to add the app on both Google Play and the App Store, we need to find app profiles that are successful on both markets. 


```python
def freq_table(dataset, index):
    table = {}
    total = 0
    
    # If a value already exists in our table, we increase
    # its count or else we create that value and assign 
    # it a count of 1
    for row in dataset:
        total += 1
        value = row[index]
        if value in table:
            table[value] += 1
        else:
            table[value] = 1
    
    table_percentages = {}
    for key in table:
        percentage = (table[key] / total) * 100
        table_percentages[key] = percentage 
    
    return table_percentages
```

We want to view our data as tuples so we'd convert our table into a list of tuples through the display_table function below.


```python
def display_table(dataset,index):
    table = freq_table(dataset, index)
    table_display = []
    for key in table:
        key_val_as_tuple = (table[key], key)
        table_display.append(key_val_as_tuple)
        
    table_sorted = sorted(table_display, reverse = True)
    for entry in table_sorted:
        print(entry[1], ':', entry[0])
```


```python
display_table(free_android, 1)
```

    FAMILY : 18.449559918754233
    GAME : 9.873617693522906
    TOOLS : 8.440532611148726
    BUSINESS : 4.5926427443015125
    LIFESTYLE : 3.9043105393816293
    PRODUCTIVITY : 3.8930264048747465
    FINANCE : 3.7011961182577298
    MEDICAL : 3.5206499661475967
    SPORTS : 3.39652448657188
    PERSONALIZATION : 3.3175355450236967
    COMMUNICATION : 3.238546603475513
    HEALTH_AND_FITNESS : 3.080568720379147
    PHOTOGRAPHY : 2.945159106296547
    NEWS_AND_MAGAZINES : 2.798465357707064
    SOCIAL : 2.663055743624464
    TRAVEL_AND_LOCAL : 2.335815842924848
    SHOPPING : 2.2455427668697814
    BOOKS_AND_REFERENCE : 2.143985556307831
    DATING : 1.8618821936357481
    VIDEO_PLAYERS : 1.782893252087565
    MAPS_AND_NAVIGATION : 1.399232678853532
    EDUCATION : 1.2863913337846988
    FOOD_AND_DRINK : 1.2412547957571656
    ENTERTAINMENT : 1.128413450688332
    LIBRARIES_AND_DEMO : 0.9365831640713158
    AUTO_AND_VEHICLES : 0.9252990295644324
    HOUSE_AND_HOME : 0.8350259535093659
    WEATHER : 0.8011735499887158
    EVENTS : 0.7109004739336493
    ART_AND_DESIGN : 0.6770480704129994
    PARENTING : 0.6544798013992327
    COMICS : 0.6206273978785828
    BEAUTY : 0.598059128864816



```python
# Frequency table of genre
freq_table(free_android, 9) 
```




    {'Art & Design': 0.598059128864816,
     'Art & Design;Pretend Play': 0.011284134506883321,
     'Art & Design;Creativity': 0.06770480704129993,
     'Art & Design;Action & Adventure': 0.011284134506883321,
     'Auto & Vehicles': 0.9252990295644324,
     'Beauty': 0.598059128864816,
     'Books & Reference': 2.143985556307831,
     'Business': 4.5926427443015125,
     'Comics': 0.6093432633716994,
     'Comics;Creativity': 0.011284134506883321,
     'Communication': 3.238546603475513,
     'Dating': 1.8618821936357481,
     'Education;Education': 0.3385240352064997,
     'Education': 5.348679756262695,
     'Education;Creativity': 0.045136538027533285,
     'Education;Music & Video': 0.033852403520649964,
     'Education;Action & Adventure': 0.033852403520649964,
     'Education;Pretend Play': 0.056420672534416606,
     'Education;Brain Games': 0.033852403520649964,
     'Entertainment': 6.070864364703228,
     'Entertainment;Music & Video': 0.16926201760324985,
     'Entertainment;Brain Games': 0.07898894154818326,
     'Entertainment;Creativity': 0.033852403520649964,
     'Events': 0.7109004739336493,
     'Finance': 3.7011961182577298,
     'Food & Drink': 1.2412547957571656,
     'Health & Fitness': 3.080568720379147,
     'House & Home': 0.8350259535093659,
     'Libraries & Demo': 0.9365831640713158,
     'Lifestyle': 3.8930264048747465,
     'Lifestyle;Pretend Play': 0.011284134506883321,
     'Adventure;Action & Adventure': 0.033852403520649964,
     'Arcade': 1.8505980591288649,
     'Casual': 1.7603249830737984,
     'Card': 0.45136538027533285,
     'Casual;Pretend Play': 0.23696682464454977,
     'Action': 3.1031369893929135,
     'Strategy': 0.9140148950575491,
     'Puzzle': 1.128413450688332,
     'Sports': 3.4642292936131795,
     'Music': 0.2031144211238998,
     'Word': 0.2595350936583164,
     'Racing': 0.9930038366057323,
     'Casual;Creativity': 0.06770480704129993,
     'Casual;Action & Adventure': 0.13540961408259986,
     'Simulation': 2.0424283457458814,
     'Adventure': 0.6770480704129994,
     'Board': 0.3723764387271496,
     'Trivia': 0.41751297675468296,
     'Role Playing': 0.9365831640713158,
     'Action;Action & Adventure': 0.1015572105619499,
     'Casual;Brain Games': 0.13540961408259986,
     'Simulation;Action & Adventure': 0.07898894154818326,
     'Educational;Creativity': 0.033852403520649964,
     'Puzzle;Brain Games': 0.16926201760324985,
     'Educational;Education': 0.3949447077409162,
     'Educational;Brain Games': 0.06770480704129993,
     'Educational;Pretend Play': 0.09027307605506657,
     'Entertainment;Education': 0.011284134506883321,
     'Casual;Education': 0.022568269013766643,
     'Music;Music & Video': 0.022568269013766643,
     'Racing;Action & Adventure': 0.16926201760324985,
     'Arcade;Pretend Play': 0.011284134506883321,
     'Role Playing;Action & Adventure': 0.033852403520649964,
     'Simulation;Pretend Play': 0.022568269013766643,
     'Puzzle;Creativity': 0.022568269013766643,
     'Sports;Action & Adventure': 0.022568269013766643,
     'Educational;Action & Adventure': 0.033852403520649964,
     'Arcade;Action & Adventure': 0.12412547957571654,
     'Entertainment;Action & Adventure': 0.033852403520649964,
     'Puzzle;Action & Adventure': 0.033852403520649964,
     'Strategy;Action & Adventure': 0.011284134506883321,
     'Music & Audio;Music & Video': 0.011284134506883321,
     'Health & Fitness;Education': 0.011284134506883321,
     'Adventure;Education': 0.011284134506883321,
     'Board;Brain Games': 0.09027307605506657,
     'Board;Action & Adventure': 0.022568269013766643,
     'Casual;Music & Video': 0.011284134506883321,
     'Role Playing;Pretend Play': 0.045136538027533285,
     'Entertainment;Pretend Play': 0.022568269013766643,
     'Video Players & Editors;Creativity': 0.011284134506883321,
     'Medical': 3.5206499661475967,
     'Social': 2.663055743624464,
     'Shopping': 2.2455427668697814,
     'Photography': 2.945159106296547,
     'Travel & Local': 2.324531708417964,
     'Travel & Local;Action & Adventure': 0.011284134506883321,
     'Tools': 8.429248476641842,
     'Tools;Education': 0.011284134506883321,
     'Personalization': 3.3175355450236967,
     'Productivity': 3.8930264048747465,
     'Parenting': 0.49650191830286616,
     'Parenting;Music & Video': 0.06770480704129993,
     'Parenting;Education': 0.07898894154818326,
     'Parenting;Brain Games': 0.011284134506883321,
     'Weather': 0.8011735499887158,
     'Video Players & Editors': 1.7716091175806816,
     'Video Players & Editors;Music & Video': 0.022568269013766643,
     'News & Magazines': 2.798465357707064,
     'Maps & Navigation': 1.399232678853532,
     'Health & Fitness;Action & Adventure': 0.011284134506883321,
     'Educational': 0.3723764387271496,
     'Casino': 0.4287971112615662,
     'Trivia;Education': 0.011284134506883321,
     'Lifestyle;Education': 0.011284134506883321,
     'Card;Action & Adventure': 0.011284134506883321,
     'Books & Reference;Education': 0.011284134506883321,
     'Simulation;Education': 0.011284134506883321,
     'Puzzle;Education': 0.011284134506883321,
     'Role Playing;Brain Games': 0.011284134506883321,
     'Strategy;Education': 0.011284134506883321,
     'Racing;Pretend Play': 0.011284134506883321,
     'Communication;Creativity': 0.011284134506883321,
     'Strategy;Creativity': 0.011284134506883321}




```python
# Frequency table in category
display_table(free_android,1) 
```

    FAMILY : 18.449559918754233
    GAME : 9.873617693522906
    TOOLS : 8.440532611148726
    BUSINESS : 4.5926427443015125
    LIFESTYLE : 3.9043105393816293
    PRODUCTIVITY : 3.8930264048747465
    FINANCE : 3.7011961182577298
    MEDICAL : 3.5206499661475967
    SPORTS : 3.39652448657188
    PERSONALIZATION : 3.3175355450236967
    COMMUNICATION : 3.238546603475513
    HEALTH_AND_FITNESS : 3.080568720379147
    PHOTOGRAPHY : 2.945159106296547
    NEWS_AND_MAGAZINES : 2.798465357707064
    SOCIAL : 2.663055743624464
    TRAVEL_AND_LOCAL : 2.335815842924848
    SHOPPING : 2.2455427668697814
    BOOKS_AND_REFERENCE : 2.143985556307831
    DATING : 1.8618821936357481
    VIDEO_PLAYERS : 1.782893252087565
    MAPS_AND_NAVIGATION : 1.399232678853532
    EDUCATION : 1.2863913337846988
    FOOD_AND_DRINK : 1.2412547957571656
    ENTERTAINMENT : 1.128413450688332
    LIBRARIES_AND_DEMO : 0.9365831640713158
    AUTO_AND_VEHICLES : 0.9252990295644324
    HOUSE_AND_HOME : 0.8350259535093659
    WEATHER : 0.8011735499887158
    EVENTS : 0.7109004739336493
    ART_AND_DESIGN : 0.6770480704129994
    PARENTING : 0.6544798013992327
    COMICS : 0.6206273978785828
    BEAUTY : 0.598059128864816



```python
# Frequency table for prime genre
display_table(free_ios,11) 
```

    Games : 58.16263190564867
    Entertainment : 7.883302296710118
    Photo & Video : 4.9658597144630665
    Education : 3.662321539416512
    Social Networking : 3.2898820608317814
    Shopping : 2.60707635009311
    Utilities : 2.5139664804469275
    Sports : 2.1415270018621975
    Music : 2.0484171322160147
    Health & Fitness : 2.0173805090006205
    Productivity : 1.7380509000620732
    Lifestyle : 1.5828677839851024
    News : 1.3345747982619491
    Travel : 1.2414649286157666
    Finance : 1.1173184357541899
    Weather : 0.8690254500310366
    Food & Drink : 0.8069522036002483
    Reference : 0.5586592178770949
    Business : 0.5276225946617008
    Book : 0.4345127250155183
    Navigation : 0.186219739292365
    Medical : 0.186219739292365
    Catalogs : 0.12414649286157665


Now, I'm looking to find what genres are the most popular (have the most users) by calculating the average number of installs for each app genre. Let's start with the ios apps and procees to the android ones.


```python
# Below we are using the genre table
ios_prime_genre = freq_table(free_ios, 11)

for genre in ios_prime_genre:
    total = 0
    len_genre = 0
    for app in free_ios:
        genre_app = app[11]
        if genre_app == genre:
            total += float(app[5])
            len_genre += 1
    avg_total = total/len_genre
    print(genre, ':', avg_total)
```

    Social Networking : 71548.34905660378
    Photo & Video : 28441.54375
    Games : 22788.6696905016
    Music : 57326.530303030304
    Reference : 74942.11111111111
    Health & Fitness : 23298.015384615384
    Weather : 52279.892857142855
    Utilities : 18684.456790123455
    Travel : 28243.8
    Shopping : 26919.690476190477
    News : 21248.023255813954
    Navigation : 86090.33333333333
    Lifestyle : 16485.764705882353
    Entertainment : 14029.830708661417
    Food & Drink : 33333.92307692308
    Sports : 23008.898550724636
    Book : 39758.5
    Finance : 31467.944444444445
    Education : 7003.983050847458
    Productivity : 21028.410714285714
    Business : 7491.117647058823
    Catalogs : 4004.0
    Medical : 612.0


The recommended ios app profile for profitability is the **Navigation** section.

Now let's calculate the average number of installs per app genre for the Google Play data set.


```python
display_table(free_android,1)
```

    FAMILY : 18.449559918754233
    GAME : 9.873617693522906
    TOOLS : 8.440532611148726
    BUSINESS : 4.5926427443015125
    LIFESTYLE : 3.9043105393816293
    PRODUCTIVITY : 3.8930264048747465
    FINANCE : 3.7011961182577298
    MEDICAL : 3.5206499661475967
    SPORTS : 3.39652448657188
    PERSONALIZATION : 3.3175355450236967
    COMMUNICATION : 3.238546603475513
    HEALTH_AND_FITNESS : 3.080568720379147
    PHOTOGRAPHY : 2.945159106296547
    NEWS_AND_MAGAZINES : 2.798465357707064
    SOCIAL : 2.663055743624464
    TRAVEL_AND_LOCAL : 2.335815842924848
    SHOPPING : 2.2455427668697814
    BOOKS_AND_REFERENCE : 2.143985556307831
    DATING : 1.8618821936357481
    VIDEO_PLAYERS : 1.782893252087565
    MAPS_AND_NAVIGATION : 1.399232678853532
    EDUCATION : 1.2863913337846988
    FOOD_AND_DRINK : 1.2412547957571656
    ENTERTAINMENT : 1.128413450688332
    LIBRARIES_AND_DEMO : 0.9365831640713158
    AUTO_AND_VEHICLES : 0.9252990295644324
    HOUSE_AND_HOME : 0.8350259535093659
    WEATHER : 0.8011735499887158
    EVENTS : 0.7109004739336493
    ART_AND_DESIGN : 0.6770480704129994
    PARENTING : 0.6544798013992327
    COMICS : 0.6206273978785828
    BEAUTY : 0.598059128864816



```python
# Below we are using the category column
android_categ = freq_table(free_android,1) 

for category in android_categ:
    total = 0
    len_category = 0
    for app in free_android:
        category_app = app[1]
        if category_app == category:
            num_installs = app[5]
            num_installs = num_installs.replace("+","")
            num_installs = num_installs.replace(",","")
            total += float(num_installs)
            len_category += 1
    avg_total_and = total/len_category
    print(category, ':', avg_total_and)
            
            
```

    ART_AND_DESIGN : 1905351.6666666667
    AUTO_AND_VEHICLES : 647317.8170731707
    BEAUTY : 513151.88679245283
    BOOKS_AND_REFERENCE : 8767811.894736841
    BUSINESS : 1712290.1474201474
    COMICS : 817657.2727272727
    COMMUNICATION : 38456119.167247385
    DATING : 854028.8303030303
    EDUCATION : 3082017.543859649
    ENTERTAINMENT : 21134600.0
    EVENTS : 253542.22222222222
    FINANCE : 1387692.475609756
    FOOD_AND_DRINK : 1924897.7363636363
    HEALTH_AND_FITNESS : 4188821.9853479853
    HOUSE_AND_HOME : 1313681.9054054054
    LIBRARIES_AND_DEMO : 638503.734939759
    LIFESTYLE : 1437816.2687861272
    GAME : 15837565.085714286
    FAMILY : 2691618.159021407
    MEDICAL : 120616.48717948717
    SOCIAL : 23253652.127118643
    SHOPPING : 7036877.311557789
    PHOTOGRAPHY : 17805627.643678162
    SPORTS : 3638640.1428571427
    TRAVEL_AND_LOCAL : 13984077.710144928
    TOOLS : 10695245.286096256
    PERSONALIZATION : 5201482.6122448975
    PRODUCTIVITY : 16787331.344927534
    PARENTING : 542603.6206896552
    WEATHER : 5074486.197183099
    VIDEO_PLAYERS : 24852732.40506329
    NEWS_AND_MAGAZINES : 9549178.467741935
    MAPS_AND_NAVIGATION : 4056941.7741935486


The recommended Google app store profile for profitability is the **COMMUNICATION** section.

# Conclusions

In this project, I analyzed data from the App Store and Google Play mobile apps with the goal of recommending an app profile that can be profitable for both markets. 

In this occassion "Navigation" apps seemed to be more profitable in the Apple store while "Communication" apps in the Android stores. 

ipynb code license: [MPL 2.0](https://www.mozilla.org/en-US/MPL/2.0/)
