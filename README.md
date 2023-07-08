# SQL-Project1

## Introduction
Australia is formally defined by more than 2000 ”Statistical Area Level 2” (SA2) distinct geographical regions, designed to represent communities of between 3000-25000 people ”that interact together socially and economically”

Using 350+ SA2s within Greater Sydney area, we will spatially intergrating several datasets to calculate wel-resourced score of each region.

### Dataset 
| Dataset   | Type  | Source                          	      | About    				|
| :-------: | :---: | :-------------------------------------: | :-------------------------------------: | 
| income    | csv   | Australia Bureau of Statistics  	      | personal income 			|
| polling   | csv   | Australian Electoral Commission         | federal election polling locations	|
| business  | csv   | Australia Bureau of Statistics          | turnover size				|
| stops     | txt   | Transport for NSW                       | public transport stops			|
| catchment | shp   | NSW Department of Education             | school					|
| sa2       | shp   | Australia Bureau of Statistics          | statistical area level 2		|
| toilet    | json  | Department of Health                    | public toilet				|
| crime     | shp   | Bureau of Crime Statistics and Research | domestic assault			|
| employees | csv   | Australia Bureau of Statistics          | employee size 				|

### Cleaning data with Python
The common process is dealing with NaNs, dropping unnecessary rows, columns, and checking data types. We always drop NaN rows in columns like longitude, latitude or geometry. For data sets with longitude and latitude, create new geom column from that. For spatial data sets with geometry column (polygon), convert to new geom column (multi-polygon). We also identify primary columns by comparing the total rows to the total number of unique values that each column holds. If it’s equal, then it’s the primary column.
|income | Replace NaN (“np”) in numerical columns with 0. Because NaN income has same meaning as $0. After that, we convert all these columns to type int64. |

> polling 	-Dropped 140 rows that has NaN in both longitude, latitude, the_geom

> business	-No NaN in the whole dataset and have correct data types.

> population    -No NaN in the whole dataset and have correct data types.

> stops 	  -No NaN in important columns: longitude, latitude

> catchment	_future -Convert numerical year into Y or N so it’s the same as other 2 catchment
	        _secondary & _primary -Drop column PRIORITY so it’s same as catchment_future.

> sa2.shp  	-Filter to “Greater Sydney”. Drop NaN in geometry. Keep: SA2_CODE21, SA2_NAME21, AREASQKM21, geom. Convert SA2_CODE21 to int so it matches sa2_code in other data

> toilet    	-No NaN in columns: longitude, latitude.

> crime	    	-No NaN in the whole dataset

> employees	-Drop NaN (“w”) row in important column: sa2_code and convert to int64. 
            	-In numerical columns, replace NaN with 0 and convert to type int64

### Indexes
We create index either on geom or sa2_code.

### Score analysis
**General formula** : For each score they are divided either by area, or by people. As we only calculate scores for areas with at least 100 total people, these areas and people will be converted to NaN so its final z-score is also NaN. In our pre-check, sa2’s area has no NaN as well as total people and young people. Hence, with a restriction of 100 people, NaN z score should only happen when the region has less than 100 people. As NaN is excluded in calculation of mean and sd, the score won’t be affected. 
The process of transferring score to z-score is the same: **Z = (x - mu)/sd**
Our score analysis: formula > index used in calculation > limitations > interesting findings (if has)
**1. Retail score** : is the number of businesses in retail trade divided by total people in that region (sa2) and multiply by 1000. To calculate score, we join business with sa2 by sa2_code, choose ‘Retail Trade’ then group by sa2_code. Note that sa2 is joined by the population before. There are some limitations in our score. Some areas have a population of less than 100, which we have already excluded these areas from our calculations. Otherwise, some of these scores would be large and affect the z score and visualization.
**2. Health score** : is the number of businesses in the healthcare sector, divided by total people in that region (sa2) and multiply by 1000. It’s the same situation as Retail score, but we choose ‘Healthcare and social assistance’ from business and join it with sa2 by sa2 code. Another limitation with people is that although the score is per 1000 people, regions with less than 1000 people exists. 
**3. Stops score** : is number of public transport stops per area square in km. We identified that stops are formed as points, and the area should contain points, so we use ST_Contains. There are no limitations in calculating stops scores. 
**4. Polling score** : is number of federal election polling locations per area sq km. We identified that polls are formed as points, and the area should contain points, so we also use ST_Contains, and follow the same steps with calculating stops score.
**5. School score** : is the number of schools (catchments) divided by area of each sa2 region. Since school can be shared between 2 areas, we use ST_Overlaps so 1 school can be counted for all areas cutting it. We then get the school score and convert it into z score. The limitation is that raw school data contains duplicated school areas, however most of them are different in catch types so we leave it. It’s interesting that future catchment differs from primary and secondary. 
**6. Toilet score** : is the total counts of public toilets divided by area of sa2 region. We count the number of public toilets that each sa2 region area contains. Then we get the toilet rate per area and convert it into z score.
**7. Crime score** : is crime area divided by area of sa2 region. Raw crime data has some NaN crime areas, that means z score of these areas will also be nan. However, nan z score is only for areas with less than 100 people, hence we replace the crime area of nan with 0, which means that area has no crime. To calculate score, we found overlapping area between crime area and sa2 region area. Then we get the crime rate per area and convert it into z score. 
There are some limitations. We do not know if nan crime data is due to the crime area being super small, super big or no crime because the data isn’t full. However, we are giving a good score (0 crime) for these unknown areas, therefore, the final z score may be biased. Also, the crime score doesn’t reflect the area's whole “crime” because the data set is limited to domestic assaults only, it doesn’t include house breaking, stealing etc… An interesting finding is there’s a weak positive correlation between crime score and median income (0.22), which indicates almost no relationship between crime and income. However, we expect that higher income areas means less crime, since crime can come from financial stress (income, employment), social stress (social standing, inequality).
**8. Non-employing score** : is the number of non employing construction industries per 1000 people, to be more specific is the number of industries which didn’t hire employees. To calculate score, we join employees with sa2 by sa2 code. There are limitations, the nan exists, and  the score is per 1000 people, but the regions with less than 1000 people exist.

The total score is Sigmoid of total z score. A score can be positive (ie: retail business) or negative (ie: crime rate)
**Sc = Sigmoid(Zretail + Zhealth + Zstops + Zpolls + Zschools + Ztoilet – Zcrime – Zemployee)**

The Sigmoid function converts the summary (+ if the score has a positive impact on well_resourced, - if not such as crime) of the z score to a probability value between 0 to 1, which represents the degree of the well-resourced score. Based on the plot and quantile, we can see areas with high well_resourced scores are concentrated in the city of Sydney. (an interactive version of map is under the same directory)






