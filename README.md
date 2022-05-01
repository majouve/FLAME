## OVERVIEW

PURPOSE: This program was made to identify possible duplicate coin finds (CFs) in FLAME's database so that they can be addressed.

INPUT: This program takes as inputs dataframes of the CFs, coin groups (CGs), and a key that describes which coin denominations correspond to specific metals. These three data sets can be found in this repository. A user can also input a spread sheet of coin finds that are not duplicates, and the program will disregard them. 

OUTPUT 1: The program will output a spread sheet of the cfIDs of possible duplicates (CF A and CF B) for someone to manually check. There will be a blank YES/NO column for someone to input whether the pair are duplicates are not. In the column "match score," each possible duplicate pair will be assigned a number that quantifies the degree to which the two match based on metrics which will be discussed below. The spread sheet will be sorted by the match score

The spreadsheet will be formatted like this:

match score | cf ID A | cf ID B | YES/NO | NOTES

OUTPUT 2: You can also get a spreadsheet called PossibleDuplicates.xlsx that gives you the geographical cluster that every coin find is attached to. 


## Detailed Description

Now, I will walk through the program using some example coin finds. The code is seperated into chunks, which I have numbered to make it easier to refer to. I will using the following coin finds, which have been idenfied as possible duplicates, as an example case while explaining the code: 

| cfID  |        name                                | x-coord  | y-coord  | excavation start | excavation end | start year | end year | num coins found | 
| ----- | ------------------------------------------ | -------- | -------- | ---------------- | -------------- | ---------- | -------- | --------------- | 
| 13293 | Le trésor byzantin de Nikerta              | 35.42224 | 36.40251 |     1968         |      1969      |    582     |     685  | 534             |
| 9239  | Finds from Belgian excavation of Apamea    | 35.42224 | 36.40251 |       2005       |    2010        |     -300   |   750    | 87              |

| cfID  | Total Gold | Total Silver | Total Bronze | Total Lead |
| ----- | ---------- | ------------ | ------------ | ---------- |
| 13293 | 533        |              |   1          |            |
| 9239  |            |              |   87         |            |

***The date ranges for coins in the coin find data frame sometimes have inconsistent dates. I will soon be correcting this by reconstructing the correct date from the coin group data frame. 

### (ONE) Replacing NAs and Striking Coin Finds
There are a few coin finds that have NAs listed for the number of coins found. To rememdy this, I totaled the coin amounts listed for each coin group and replaced the NA with this total. 

Then, for coin finds/coin groups that meet the following criteria, they are struck from the data set. 
- If any cf_num_coins_found==NA or 0 after an attempt is made to reconstruct this number with Coin Group info. 
- cg_num_coins_found==NA or 0 
- Coin Groups where metal == NA
- Any coin finds with THS in their name, as they follow a data entry convention incompatible with this program.

Therefore, we start with ~5,200 coin finds and reduce down to ~3,800


### (TWO) Location Filter (This block takes the longest to run)
Coin finds are grouped into geographical clusters that set which coin finds get compared to each other throughout the program. 

First, the user sets a radius for the geographical size of the cluster desired. This is currently set to 1 km. Using the latitude and longitude coordinates assigned to every coin find, the program uses the sf package to create spatial objects for each coin find. 

The program iterates through the CFs, setting in each turn to be a central coin find. Let's say this is 13293. The program will check which CFs are within 1 km radius of 13293. By default, 13293 itself will be included in the radius. 

This step will idenitfy the CF, 9239, as being within a 1 km radius of 13293 (referencing the data provide above confirms this). An observation from working with this data is that many coin finds have counterparts that are within small fractions of a km from each other. 

For every central CF, a list of CFs (including itself) are stored in the coin finds data frame (Cut_CoinFinds) in the column "in.radius"
For our example, this will look like c(13293, 9239).


### (THREE) Quantity Filter
The amount of coins for each find is compared between members of a geographical cluster. 

The user can set a tolerance for how large the difference between coin amounts has to be before the program will read them as being different. This is currently set to 5 (this means that if cfA has 5 coins and cfB has 7 coins, they will be marked as having the same amount of coins). 

For each coin find in the "in.radius" list (13293, 9239), the program will compare the central coin find to each coin amount value. 

The first cfID in the list is 13293, which has 534 coins. This number will be compared to 13293 (itself) and will ouput TRUE
The second cfID in the list is 9239, which has 87 coins. This number will be compared to the central coin find, 13293 and will output FALSE. 

This section outputs a list of TRUES/FALSES in the same order as the coin finds are originally listed in the "in.radius" list. This list goes into the "are.num.coins.same" column. The output for our example would be c(TRUE, FALSE). 

There is also a column (dif.num.coins) for the list of the differences between the central coin find and the radius coin finds (central cf - radius cf = 13293 - 9239 = 534 - 87 = 447)
The final output in the column will be c(0, 447)

***Hereafter, I will not do the 13293 to 13293 comparison. This understood comparison will always yield a TRUE as the first term of the output lists. 


### (FOUR) Metal Filter
The amount of coins of each metal for each find is compared between members of a cluster, producing a seperate TRUE/FALSE list for each metal. 

First, for each coin find, all the coin amounts for the coin groups of a specific metal are added together to produce total amounts. Values for our example coin finds are listed in the tables at the top of the document. 

The tolerance for this cluster works differently than the others. Within a coin find, for each coin group of a specific metal that is added together, the tolerance is increased by 1. For example, the tolerance for the bronze in 13293 is 1 because there is 1 bronze coin group. The tolerance for the gold in 13293 is 9 because there are 9 bronze coin groups. For 9239, the tolerance for bronze would be 58 because there are 58 bronze coin groups. 

Now for the comparisons, 13293 is still the central find, and 9239 is the radius find:

13293 has 533 gold coins and 9239 has 0 gold coins. (FALSE)
13293 has 0 silver coins and 9239 has 0 silver coins. (TRUE)
13293 has 1 bronze coin and 9239 has 87 bronze coins. (FALSE)
13293 has 0 lead coins and 9239 has 0 lead coins. (TRUE)

In this module, I also calculate the difference between the amount of coins for each metal between the central and radius coin finds. Here's what this looks like with our example (13293 - 9239 = central cf - radius cf): 

Gold: 533 - 0 = 533
Silver: 0 - 0 = 0
Bronze: 1 - 87 = -86
Lead: 0 - 0 = 0

This module produces the following columns with the following example outputs:
is.bronze.same: c(TRUE, FALSE)
is.gold.same: c(TRUE, FALSE)
is.silver.same: c(TRUE, TRUE)
is.lead.same: c(TRUE, TRUE)

dif.silver = c(0, 533)
dif.gold = c(0,0)
dif.bronze = c(0, -86)
dif.lead = c(0,0)

These are the column names for the total metal columns:
total.gold = 
total.silver = 
total.bronze = 
total.silver = 


### (FIVE) Excavation Filter
The dates of the excavation start and end years are compared seperately between the central coin find and each radius coin find. 
The same comparison occurs here as well for the variable excav.start.year and excav.end.year for each coin find in the cluster. The tolerance for this comparison is currently set to 2.

13293's excavation started in 2005, and 9239's started in 1969. (FALSE)
13293's excavation ended in 1968, and 9239's ended in 2010. (FALSE)

Therefore, this module produces two lists
are.excav.start.same = c(TRUE, FALSE)
are.excav.end.same = c(TRUE, FALSE)

NOTE: Alternative outputs are:
NA: This occurs when the central coin find has NA for the start year or the end year
NA as the component of a list (ex. c(0, 67, NA)): This occurs when one of the radius years has an NA listed for the start year or end year


### (SIX) Date Range Filter
The dates of the start and end years of a coin find are compared seperately between the central coin find and each radius coin find. 
The same comparison occurs here as well for the variable is.cf.start.year.same and is.cf.end.year.same for each coin find in the cluster. The tolerance for this comparison is 2.

13293's excavation started in 582, and 9239's started in -300. (FALSE)
13293's excavation ended in 685, and 9239's ended in 750. (FALSE)

Therefore, this module produces two lists
is.cf.start.year.same = c(TRUE, FALSE)
is.cf.end.year.same = c(TRUE, FALSE)

Then, it also produces to difference lists:
start.date.dif = c(0, 882)
end.date.dife = c(0, -65)

NOTE: Alternative outputs are:
NA: This occurs when the central coin find has NA for the start year or the end year
NA as the component of a list (ex. c(0, 97, NA)): This occurs when one of the radius years has an NA listed for the start year or end year


### (SEVEN) Cut out Finds with Nothing in the Radius
In the Cut_CoinFinds dataframe (main dataframe where all the columns have been added), there are many coin finds with nothing in their radius. 
I subset these from the data set, saving the smaller data set of coins with possible duplicates into TEST_CoinFinds. This has ~2,000 cfs. 

The coin finds that have no geographical matches are also saved into a dataframe. This dataframe can be inputted into the program so that these coin finds are disconsidered when the program is run in the future.


### (EIGHT) Create Cluster IDs
Then I created a cluster ID for each geographical cluster. It is composed of the cfIDs in the cluster seperated by ".". For our example, the cluster ID would be 13293.9239


### (NINE & TEN & ELEVEN) Create Cluster Comparison Data Frame (FULL_clusters)
Now, I created a new data frame (FULL_clusters) with new metrics to help make comparisons between the clusters better. I include the following information in this data frame:

-cfID, name (long name assigned to find), author (person who inputted the find)
-cluster ID
-radius = the radius the geographical cluster is in (set to 1 for all)

After this indentifying material, I cycle through the same three types of columns for each metric:
-total.gold / silver / bronze / lead
-start/end.year
-excav.start/end.year

I have columns for three metric:
-I list the value itself (ex. total.gold, excavation start year)

-avg.X.dif = sum(central find - each radius find)
ex. avg.bronze.dif = (0-86)/(2-1) =  -86
NOTE = the bronze difference list is c(0, -86). By subtracting one from the denominator, we don't consider the 0 from 13293 being subtracted from itself.

-percent.X.same = percent of the values in the TRUE/FALSE list that are TRUE (excluding the value that is true because the central coin find is compared to itself)
ex. For avg.bronze.dif of 13293 = (1/(2-1)) = 0% 
NOTE = the bronze TRUE/FALSE list is c(TRUE,FALSE). By subtracting one from the denominator, we don't consider the TRUE from 13293 being compared to itself.

### (TWELVE) Match Score
Then I create the match score to help us prioritize which coin clusters to compare for duplicates first. The match score will use the metrics calculated in the FULL_clusters data frame. In the final output data frame, the match score will be used to sort coin finds that are the most likely to be duplicates to the top.

