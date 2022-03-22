Input: 
-CoinFinds
-CoinGroups
-Coin_Info = contains information on the type of metal that corresponds to each denomination
-disconsidered = any coin finds that are definetly not duplicates. This portion of the code is currently commented out

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

Date Range Filter: T
-filter currently set to +/- 25

Output: This program creates four output spread sheets:

Not_A_Duplicate.xlsx
- if coins do not have any others in their cluster by location, then they will be placed in this group

Possible_Duplicates.xlsx


At_Least_One_Match.xlsx


Full_Analysis
- I'm supplying this spread sheet to show the comparisons made for each coin find even if they end up being disconsidered in the end. 
Here is a description of what each column means that I have added to the original data frame inputted:

















