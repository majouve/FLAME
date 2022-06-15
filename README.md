## OVERVIEW

PURPOSE: This program was made to identify possible duplicate coin finds (CFs) in FLAME's database so that they can be addressed.

INPUT: This program takes as inputs dataframes of the CFs, coin groups (CGs), and a key that describes which coin denominations correspond to specific metals. These three data sets can be found in this repository. A user can also input a spread sheet of coin finds that are not duplicates, and the program will disregard them. 

OUTPUT : The program will output a spread sheet of the cfID pairs that are possible duplicates (CF A and CF B) for someone to manually check. There will be a blank YES/NO column for someone to input whether the pair are duplicates are not. In the column "match score," each possible duplicate pair will be assigned a number that quantifies the degree to which the two matches. The spread sheet will be sorted by the match score so that highest matches scores are at the top.

## Detailed Description

Now, I will walk through the program using some example coin finds. The code is seperated into numbered sections that I will explain piece by piece. Below are the example coin finds:


INPUT: 
| cfID  |        name                                | x-coord  | y-coord  | excavation start | excavation end | start year | end year | num coins found | 
| ----- | ------------------------------------------ | -------- | -------- | ---------------- | -------------- | ---------- | -------- | --------------- | 
| 13293 | Le trésor byzantin de Nikerta              | 35.42224 | 36.40251 |     1968         |      1969      |    582     |     685  | 534             |
| 9239  | Finds from Belgian excavation of Apamea    | 35.42224 | 36.40251 |       2005       |    2010        |     325    |   668    | 87              |

| cfID  | Total Gold | Total Silver | Total Bronze | Total Lead |
| ----- | ---------- | ------------ | ------------ | ---------- |
| 13293 | 533        |              |   1          |            |
| 9239  |            |              |   87         |            |


OUTPUT: 

VerifyDuplicates.xlsx
| central_cf | central_name                     | central_bib | central_auth | radius_cf |   radius_name                      | radius_bib | radius_auth | 
| ---------- | -------------------------------- | ----------- | ------------ | --------- | ---------------------------------  | ---------- | ----------- |
|   13293    | Le trésor byzantin  de Nikertai  | Morrisson...| hsubeh19...  |   9239    | Finds from Belgian excav of Apamea | Finds...   |  pyzmark... |

| distribution | G | S |   B   |   SD  |   ED   |   EXS  |   EXE  | sequential | single | match.score | is.it.a.match |
| ------------ | - | - | ----- | ----- | ------ | ------ | ------ | ---------- | ------ | ----------- | ------------- |
|     0        | 1 | 0 | .9885 | .791  |  .025  |  .018  |  .020  |       0    | .0924  |    -.348    |    1 or 0     |

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

Currently commented out right now is the option to input an old version of VerifyDuplicates with is.it.a.match filled out. This file will allow the program (section 12) to take out all pairs that have already been checked in subsequent runs.

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

### (12) (COMMENTED OUT) Removing Coin Find Pairs Already Addressed
One you have taken the code from section 2 out of comments, you may use this section. Section 2 reads in a spread sheet of VerifyDuplicates.xlsx with is.it.a.match already filled out, which this code chunk uses to disconsider coin finds that have already been addressed. 

If is.it.a.match has been marked TRUE or FALSE in the old VerifyDuplicates spreadsheet, then this pair is taken out of the new spreadsheet being developed. If is.it.a.match is empty, then the pair will remain in the new VerifyDuplicates spreadsheet. 

### (13) Histogram Matching
In this section, histograms visualizing the coins minted per year within a coin find are compared within pairs to see how similar they are (x axis: years; y axis: number of coins minted per year). This is the same histogram found in FLAME's circulation application. I standardize the date ranges to all be 1 year to make the histograms comparable. The program uses a Kolmogorov–Smirnov test to compare the histograms. 

For our example cf pair, here is the histogram for 13293 (central coin find):

<img width="805" alt="Screen Shot 2022-06-14 at 6 26 54 PM" src="https://user-images.githubusercontent.com/98130904/173709279-e527f69b-1fac-41a7-a025-55df7e7e4e83.png">

Here is the histogram for 9239:

<img width="790" alt="Screen Shot 2022-06-14 at 6 27 57 PM" src="https://user-images.githubusercontent.com/98130904/173709395-120c1943-98e7-4ede-bc53-e2a54ed7d8e1.png">

Clearly, these distributions are not similar. Correspondingly, the ks.test reports a p value of <.05.
 
Finally, I add the "distribution" column to the Verify_Duplicates dataframe. If the histograms match, then the pair receives a 1 for the column. If they do not match, the pair receives a 0. With our example of the pair (13293, 9239), they would receive a 0 in this column.

NOTE: When creating the dataframe, I originally though that making it the length of the year span (325-750) of FLAME's data would be sufficient, that is 426 entries. However, some coin groups have year spans fall outside of the 325-750 range. Therefore, the histogram dataframe is longer that it should be.

### (14) Filters Part I
In this section, specific metrics of the coin find pairs are compared and metrics are added to the Verify_Duplicates data frame. The following attributes use percent difference calculations between the central and radius values in the Verify_Duplicates df as their metric: 
-metal composition (how many coins of each metal are in a coin find)
-start and end date (what is the date of the earliest date coin in a cf and what is the latest)
-excavation start and end years 

All use the following equation: abs((central - radius)/radius). However, if the radius value is 0, I flip it to avoid an undefined value: abs((radius - central)/central). If both the radius and central values are 0, 0 is reported. 

## (A) Metal Filter
The amount of coins of each metal (bronze/silver/gold) is compared between the cf pair. 
For the example pair (13293, 9239), this is the calculation: 

Gold (Remember, since the radius value is 0, the equation is flipped): abs(0-533) / 533 = 1
Silver: Zero is reported because both values are 0. 
Bronze: abs(1-87) / 87 = .9885

## (C) Excavation Filter (Commented Out)
This section is commented out because many values do not have excavation years listed, making it a less valuable metric.
The dates of the excavation start and end years are compared seperately amongst the coin find pair. Here are the calculations for our example:

Start year: abs(1968-2005) / 2005 = .018
End year: abs(1969-2010) / 2010 = .020

## (D) Date Range Filter
The dates of the start and end years of a coin find are compared seperately between pairs. Here are the calculations for our example:

Early date: abs(582-325) / 325 = .791
Late date: abs(685-668) / 668 = .025

### (15) Filters Part II

There are few more metrics used to compare the pair of cfs: 

## (B) Sequential

If the IDs of the cfs are sequential (ex. 5473 and 5474), they are given a 1 in the "sequential" column of the data frame and a 0 if not. 
For our pair, the value in the "sequential" column would be 0. 

## (C) Singelton: Total Coin Find Amount Comparison

Coin find pairs that are both singelton coin finds have a lower chance of being duplicates. This metric compares the total amount of coins within a pair to the other paris, rating those on the lower end (ie. a total of 2) lower and those on the upper end (The max is 6701) higher. 

First the amount of coins total in a coin find pair is totaled up (radius + central). Then I compare this total to a distribution all the totals Verify_Dupes to see what percent of values is less than this total. 

For our example pair, 13293 has a total of 534 coins. 9239 has a total of 87 coins. The total between them in 621. In a distribution of totals ranging from 2 to 6701, it is larger than 9.23% of values. 

The value .0923 is entered into the singelton column.

### (16) Computing Composites

The above metrics are combined into a single score so that the pairs more likely to be duplicates can be sorted to the top. Before doing the calculation, I use the scale() function to normalize the percent different calculations. Some percent difference values are so large (~2,000) that they would overshadow the sequnetial and singleton metrics, which can only be as high as 1. 

The higher the match score, the more likely a coin find pair is to a match. The variables distribution and singleton are added together for this metric because as they increase, the likelihood of a match increases. However, sequential, silver, bronze, gold, start date, and end date are subtracted in the equation because as the percent difference scores increase, the liklihood of a match decreases. 

This is the equation used to find the composite score:

Distribution * .1 + Singleton * .1 - Sequential * .1 - Gold * .14 - Silver * .14 - Bronze * .14 - Start Date * .14 - End Date * .14 

For our examples pair this produces a score of: 

(0 * 1) + (.0924 * .1) - (0 * .1) - (1 * .14) - (0 * .14) - (.9885 * .14) - (.791 * .14) - (.025 * .14) = -.348








