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

Not_A_Duplicate.xlsx
-all coin finds that have nothing in their cluster
*The first half of the columns are what was inputted into the program*
in.radius = list of the cfIDs that are in a geographical cluster with the central cf
num.in.radius = lists how many coin finds are in a cluster
are.X.same  = For each coin find in a cluster, TRUE if the value of a given attribute is the same as the central find (within a tolerance) and FALSE if the value is not the same
dif.X = This is a list of the differences the central find value for a given attribute - the given attribute for each coin find in the radius
one.in.radius = TRUE if a coin find has an empty cluster and FALSE otherwise

Possible_Duplicates.xlsx
- this data frame contains all location clusters (length = ~2,300)

cfID = coin find ID of central coin find
name = name of central coin find
clusterID = all the IDs of the coin finds in a cluster seperated by periods
radius = size of radius in km
percent.num.coins.same = percent of coin finds in a cluster that have the same amount of coins as the central ID (+/- 5)
avg.coin.num.dif = average(number of coins of central cf - number of coins in each 
*All other percent and average columns follow the same idea*
is.gold/bronze/silver.zero = Some of the metals (esp gold and silver) will be absent from many coin finds. This means that coins finds with an equal amount of gold coins, for example, will look the same in the spread sheet (percent.gold.same = 0 and avg.gold.dif = 0) as coin finds all with gold zero coins. I added this column that will be TRUE when all coins in a cf cluster have zero of one type of metal and that will be FALSE otherwise. 
how.many.matches = this counts how many attributes have a percent match > 0


At_Least_One_Match.xlsx
- this data frame contains only location clusters that also match on one other attribute (amount of coins, data range, etc.) (length = ~2,200)
- the column descriptions from Possible_Duplicates apply here

Short_Clusters.xlsx
- In Possible_Duplicates and At_Leat_One_Match, there is a repeat of clusters in the data frames because many coin finds will be in the same cluster together. Therefore, I created this shorter data frame (~500 entries) that keeps only one of the coin find entries for each cluster. 
















