
Giving an overview of the data and what generally the program does. 

All assistant knows is if find A is same as find B
second doc:
13292 in one column | second ID | YES or NO
all possible cominations within a cluster must be tested between each other so groups of two of each cluster in rows

Subclustering:

comparing items in the clusters to each other. 
if there are more than zero silver coins, 
-match up whether the silver of that one match the silver to that one. 
-do same for gold


- create a percent match
the matches get sorted to the top of the list 

same number silver gold bronze and have the same date range - 


date range- min in coin groups for start year to max in coin groups for end year

check that you aren't subtracting the same thing from each other

an example from one coin find. 

------
TODO: 

-direct comparison list with quantification (what do manually entered answers look like
  - simple TRUE or FALSE with recommendation of what to do with them and then a maybe
  FIND A | FIND B | SAME? | What do we do with it?
  
-fix date ranges
-do group me file with the larger overivew of logic as well as example mathematically
-input finds that aren't the same back into code so that it discounts

-----

Input: 
-CoinFinds
-CoinGroups
-Coin_Info = contains information on the type of metal that corresponds to each denomination
-disconsidered = any coin finds that are definetly not duplicates. This portion of the code is currently commented out
#######

Which coin finds/coin groups are not tested to see if they are duplicates: 
- If any cf_num_coins_found==NA or 0 after an attempt is made to reconstruct this number with Coin Group info. 
- cg_num_coins_found==NA or 0 after an attempt is made to reconstruct this number with Coin Group info. 
- Coin Groups where metal == NA

Location Filter: For each coin find location, return all coin finds within a certain radius (that you input) in a list, forming a column of CoinFinds (in.radius)
- I currently have this set to 1 km. However, half of all coin finds still form clusters even with this small radius. 

Quantity Filter: compares cf_num_coins_found between the central point and the radius points. Creates a column of TRUE/FALSES in Cut_CoinFinds indicating whether the radius point is the same or different (within a range you input) from the central point. I also calculate the difference between the amount of coins of the central find and that of each radius find. This forms another column of lists. 
-filter currently set to +/-5 coins

Metal Filter: Uses amount of each metal in a coin find and compares this between the central coin find and the radius coin finds. Generates TRUE/FALSE list like above that's added to Cut_CoinFinds. Also generates columns of the difference in metal amounts between the central coin find and each radius coin find.
-filter for each metal = number of coin groups containing that metal for a central point
  - an example to make this more clear: If a central coin find has 5 bronze metal coin groups, then the range used to compare the central coin find bronze amount and the radius bronze amounts will be 5. 

Date Range Filter: I break up start and end date for each coin find. Then I compare the start and end dates between the central coin find and the radius coin finds. Creates a column of TRUE/FALSES in Cut_CoinFinds indicating whether the start and end dates of the radius point is the same or different (within a range you input) from the central point. I also calculate the difference between the start and end dates of the central find and that of each radius find. This forms another column of lists. 
-filter currently set to +/- 25

#########
Output: This program creates four output spread sheets:


Possible_Duplicates.xlsx
- this data frame contains all location clusters (length = 1,200)

cfID = coin find ID of central coin find
name = name of central coin find
clusterID = all the IDs of the coin finds in a cluster seperated by periods
radius = size of radius in km
percent.num.coins.same = percent of coin finds in a cluster that have the same amount of coins as the central ID (+/- 5)
avg.coin.num.dif = average(number of coins of central cf - number of coins in each 


*All other percent and average columns follow the same idea*
is.gold/bronze/silver.zero = Some of the metals (esp gold and silver) will be absent from many coin finds. This means that coins finds with an equal amount of gold coins, for example, will look the same in the spread sheet (percent.gold.same = 0 and avg.gold.dif = 0) as coin finds all with gold zero coins. I added this column that will be TRUE when all coins in a cf cluster have zero of one type of metal and that will be FALSE otherwise. 
how.many.matches = this counts how many attributes have a percent match > 0

















