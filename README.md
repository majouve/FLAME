## OVERVIEW

PURPOSE: This program was made to identify possible duplicate coin finds (CFs) in FLAME's database so that they can be addressed.

INPUT: This program takes as inputs dataframes of the CFs, coin groups (CGs), and a key that describes which coin denominations correspond to specific metals. These three data sets can be found in this repository. A user can also input a spread sheet of coin finds that are not duplicates, and the program will disregard them. 

OUTPUT : The program will output a spread sheet of the cfIDs of possible duplicates (CF A and CF B) for someone to manually check. There will be a blank YES/NO column for someone to input whether the pair are duplicates are not. In the column "match score," each possible duplicate pair will be assigned a number that quantifies the degree to which the two matches. The spread sheet will be sorted by the match score so that highest matches scores are at the top.

## Detailed Description

Now, I will walk through the program using some example coin finds. The code is seperated into numbered sections that I will explain piece by piece. Below are the example coin finds:


INPUT: 
| cfID  |        name                                | x-coord  | y-coord  | excavation start | excavation end | start year | end year | num coins found | 
| ----- | ------------------------------------------ | -------- | -------- | ---------------- | -------------- | ---------- | -------- | --------------- | 
| 13293 | Le trésor byzantin de Nikerta              | 35.42224 | 36.40251 |     1968         |      1969      |    582     |     685  | 534             |
| 9239  | Finds from Belgian excavation of Apamea    | 35.42224 | 36.40251 |       2005       |    2010        |     325    |   688    | 87              |

| cfID  | Total Gold | Total Silver | Total Bronze | Total Lead |
| ----- | ---------- | ------------ | ------------ | ---------- |
| 13293 | 533        |              |   1          |            |
| 9239  |            |              |   87         |            |


OUTPUT: 

VerifyDuplicates.xlsx
| central_cf | central_name           | radius_cf |   radius_name    | G | S | B | SD | ED | EXS | EXE | sequent | single | match.score | is.it.a.match |
| ---------- | ---------------------- | --------  | ---------------- | - | - | - | -- | -- | --- | --- | ------- | ------ | ----------- | ------------- |
|   13293    | Le trésor byzantin..   |  9239     | Belgian excav... | 0 | 1 | 0 | 0  | 1  |  0  |  0  |     0   |  XXX   |  ---------- |    1 or 0     |

G = Gold
S = Silver 
B = Bronze
SD = Start Date (date range on coins)
ED = End Date (date range on coins)
EXS = Excavation Start Date
EXE = Excavation End Date

### (1 and 2) Packages and Loading Data
The most important package loaded is "sf," as it does the location matching portion of the program. 

The second section takes the program's inputs. CoinFinds and CoinGroups are the product of downloading all coin data from the Circulation Application with no filters. Coin_Info is a key that defines which denominations correspond to each metal.

Currently commented out right now is the option to input an old version of VerifyDuplicates with is.it.a.match filled out. This file will allow the program to take out all pairs that have already been checked in subsequent runs.

### (3 and 4) Formatting Coin Denomination/Metals 
In these sections, I reformat the coin denomination and metal information from Coin_Info and use it to create a new column in CoinGroups with the metal of each coin group. 

### (5) Replacing NAs and Striking Coin Finds
There are a few coin finds that have NAs listed for the number of coins found. To rememdy this, I totaled the coin amounts inputted in the coin group data frame (df) and replaced the NAs with this total. 

Coin finds that meet the following criteria are struck from the data set: 
- If any cf_num_coins_found==NA or 0 after an attempt is made to reconstruct this number with Coin Group df info. 
- Any coin finds with THS in their name, as they follow a data entry convention incompatible with this program.

Coin groups that meet the following criteria are struck from the data set. When coin groups are struck, some coin finds no longer have coin groups listed in the system. These are struck as well, which amounts to 60:
- cg_num_coins_found==NA or 0 
- Coin Groups where metal == NA

Therefore, the CF dataframe starts with ~5,200 coin finds and is reduced to ~3,800 through the above cuts.

### (6) Reconstructing Metrics in CoinFinds from CoinGroups
Here, I create columns in CoinFinds that break up the total amount of coins in a cf by metal, using the information from the CoinGroups df. 

Then I total up the amount of coins for each metal and replace the number of coins found listed in CoinFinds with this value. This remedies an error in the CoinFinds data, where an incorrect total coin amount was listed that didn't match the CoinGroups data.

### (7) Location Filter 
Coin finds are grouped into geographical clusters that set which CFs get compared to each other throughout the program. 

First, the user sets a radius for the geographical size of the cluster desired. This is currently set to 1 km. Using the latitude and longitude coordinates assigned to every coin find, the program uses the sf package to create spatial objects for each coin find. 

The program iterates through the CFs, setting each in turn to be a central coin find. 13293 will be the central coin find for the purpose of the example. The program will check which CFs are within a 1 km radius of 13293. By default, 13293 itself will be included in the radius. 

This step will idenitfy the CF, 9239, as being within a 1 km radius of 13293. An observation from working with this data is that many coin finds have counterparts that are within small fractions of a km from each other. 

For every central CF, a list of CFs (including itself) are stored in the coin finds data frame (Cut_CoinFinds) in the column "in.radius"
For our example, this will look like c(13293, 9239).

### (9) Remove Finds not in a Cluster
In this section, I remove all coin finds that do not fall into a geographical cluster from consideration of being a duplicate. I save this smaller group in a new df called Dupe_CoinFinds It has 1,186 entries. 

### (10) Create Verify_Dupes
Then I start to construct the output df. I break down the coin find clusters into pairs, and each pair forms a row in the Verify_Dupes df. I create a column for each metric that I will be comparing between the two finds (Gold, Silver, Bronze, Start Date, End Date, Excavation Start Date, Excavation End Date). These columns will be filled with a 1 or 0, indicating whether the metrics match (within a tolerance) or not.

With our example, the program would create the entry listed at the beginning of this file under output. 

### (11) Delete Redundant Pairs/Those Not to be Considered
Coin finds that are both within a database that FLAME imported are likely not duplicates. Therefore, if coins within a pair both were labeled with CHRE or PAS or they were entered by the user "PeterPhilips," that pair is struck.

Section 10 created rows with some cf pairs having the same cf (ex. A and A), which I strike in this section. Some pairs are also permutations of each other (ex. A and B ; B and A), so I remove one of these permutations.

In all, there are 1,459 coin find pairs.

### (12) Removing Coin Find Pairs Already Addressed

### (13) Histogram Matching
In this 


### (14) Filters Part I
In this section, specific metrics of the coin find pairs are compared. Currently, there are two different version of this section. 

The other version calculates the percent difference between the central and radius metrics. It also does not use excavation start and end dates because these metrics have more missingness. 

VERSION 1:

VERSION 2: 

### (15) Filters Part II

Are the cfIDs sequential?


Where does the total amount of coins in the coin find pair fall in comparison to the other pairs?



### (16) Computing Composites


### (THREE) Quantity Filter
The amount of coins for each find is compared between members of a geographical cluster. 

The user can set a tolerance for how large the difference between coin amounts has to be before the program will read them as being different. This is currently set to 5 (this means that if cfA has 5 coins and cfB has 7 coins, they will be marked as having the same amount of coins). 

For each coin find in the "in.radius" list (13293, 9239), the program will compare the central coin find to each coin amount value. 

The first cfID in the list is 13293, which has 534 coins. This number will be compared to 13293 (itself) and will ouput TRUE
The second cfID in the list is 9239, which has 87 coins. This number will be compared to the central coin find, 13293 and will output FALSE. 

This section outputs a list of TRUES/FALSES in the same order as the coin finds are originally listed in the "in.radius" list. This list goes into the "are.num.coins.same" column. The output for our example would be c(TRUE, FALSE). 

There is also a column (dif.num.coins) for the list of the differences between the central coin find and the radius coin finds (central cf - radius cf = 13293 - 9239 = 534 - 87 = 447)

The final output in the column will be c(0, 447).

***Hereafter, I will not do the 13293 to 13293 comparison. This understood comparison will always yield a TRUE as the first term of the output lists. 


### (FOUR) Metal Filter
The amount of coins of each metal for each find is compared between members of a cluster, producing a seperate TRUE/FALSE list for each metal. 

First, for each coin find, all the coin amounts for the coin groups of a specific metal are added together to produce total amounts. Values for our example coin finds are listed in the tables at the top of the document. 

The tolerance for this cluster works differently than the others. Within a coin find, for each coin group of a specific metal that is added together, the tolerance is increased by 1. For example, the tolerance for the bronze in 13293 is 1 because there is 1 bronze coin group. The tolerance for the gold in 13293 is 9 because there are 9 bronze coin groups. For 9239, the tolerance for bronze would be 58 because there are 58 bronze coin groups. 

Now for the comparisons, 13293 is still the central find, and 9239 is the radius find:

- 13293 has 533 gold coins and 9239 has 0 gold coins. (FALSE)
- 13293 has 0 silver coins and 9239 has 0 silver coins. (TRUE)
- 13293 has 1 bronze coin and 9239 has 87 bronze coins. (FALSE)
- 13293 has 0 lead coins and 9239 has 0 lead coins. (TRUE)

In this module, I also calculate the difference between the amount of coins for each metal between the central and radius coin finds. Here's what this looks like with our example (13293 - 9239 = central cf - radius cf): 

- Gold: 533 - 0 = 533
- Silver: 0 - 0 = 0
- Bronze: 1 - 87 = -86
- Lead: 0 - 0 = 0

This module produces the following columns with the following example outputs:
- is.bronze.same: c(TRUE, FALSE)
- is.gold.same: c(TRUE, FALSE)
- is.silver.same: c(TRUE, TRUE)
- is.lead.same: c(TRUE, TRUE)

- dif.silver = c(0, 533)
- dif.gold = c(0,0)
- dif.bronze = c(0, -86)
- dif.lead = c(0,0)

These are the column names for the total metal columns:
- total.gold = 
- total.silver = 
- total.bronze = 
- total.silver = 


### (FIVE) Excavation Filter
The dates of the excavation start and end years are compared seperately between the central coin find and each radius coin find. 
The same comparison occurs here as well for the variable excav.start.year and excav.end.year for each coin find in the cluster. The tolerance for this comparison is currently set to 2.

- 13293's excavation started in 2005, and 9239's started in 1969. (FALSE)
- 13293's excavation ended in 1968, and 9239's ended in 2010. (FALSE)

Therefore, this module produces two lists
- are.excav.start.same = c(TRUE, FALSE)
- are.excav.end.same = c(TRUE, FALSE)

NOTE: Alternative outputs are:
- NA: This occurs when the central coin find has NA for the start year or the end year.
- NA as the component of a list (ex. c(0, 67, NA)): This occurs when one of the radius years has an NA listed for the start year or end year


### (SIX) Date Range Filter
The dates of the start and end years of a coin find are compared seperately between the central coin find and each radius coin find. 
The same comparison occurs here as well for the variable is.cf.start.year.same and is.cf.end.year.same for each coin find in the cluster. The tolerance for this comparison is 2.

- 13293's excavation started in 582, and 9239's started in -300. (FALSE)
- 13293's excavation ended in 685, and 9239's ended in 750. (FALSE)

Therefore, this module produces two lists
- is.cf.start.year.same = c(TRUE, FALSE)
- is.cf.end.year.same = c(TRUE, FALSE)

Then, it also produces to difference lists:
- start.date.dif = c(0, 882)
- end.date.dife = c(0, -65)

NOTE: Alternative outputs are:
NA: This occurs when the central coin find has NA for the start year or the end year
NA as the component of a list (ex. c(0, 97, NA)): This occurs when one of the radius years has an NA listed for the start year or end year

