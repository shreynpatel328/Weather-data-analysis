0. Username

	asava003
	spate067
	node: z5

1. Project Description

	- In the project, we have two datasets, one contains the actual data of the weather across the United States and one file contains the metadata about the data (i.e. which station has collected the data, station name, place, latitude, longitude etc ). 

	- So in order to combine the two files we performed a map-side join. We have made one method named setup where the reading of the metadata fields takes place and extract the file in such a way that we get a key, value where key is the station id of the weather station and the value is the state to which that weather station id belongs. We have also taken care of the weather stations where it does not have which state in the United States it belongs to. We have assigned the value as ‘ZZ’ for the states with no state.
	
	- Now in the map phase we read the data file and split the data file with the white spaces and separate the station id, the month, the temperature and the count of how many times the temperature was recorded for that station for a particular day. Now we merge the month, temperature and count in a single string and merge it in to a single value.

	- After that step, we compare the station id and check it belongs to which state. And then output of the map phase which contains the key value pairs where key is the state name and the value is merge of month, temperature and count.

	- In the reduce phase, we are extracting the value outputed in the map phase by dividing into month, temperature and count and then we multiply the temperatures with the count so that we can find the weighted average and add that value to the an array where the index of the array will be the month. We keep adding the weighted temperature as well as the count for each month, for each station id. Then we find the average by dividing the temperature with the count and then find the minimum and maximum temperature for a year and store in which month it had the highest temperature and which had the lowest.

	- We are using the second job just to sort the output from the first job. We need this job because we can not take the output in sorted format form the reducer. Reducer does not sort its data.

2. Job Description

	- Here we have made two jobs one for map side join with extracted output and the second job for sorting the tempreature difference.

	1. First job is doing map side join in using setup method. We are loading the first locations file in distributed cache, so that we can use them while reading the data from recordings file. So in mapper we have used hashmap to store the intermediate result form distributed cache, so the we can use them quickly. While in map method we are doing map side join with wrriting output as below. It will sort the output data, so that it will take less time for processing in the reducer.
	outputKey: stateName
	outputValue: month, temperature, count and precipitation

	We are taking that in reducer as input. And for a particular key i.e. stateName, we are calculating the weighted average temperature and precipitation for a particular month. And on that we are finding the month with highest and lowest temperature, then we are writting output as below
	outputKey: stateName
	outputValue: month, minimum and maximum average temperature, difference of them(min-max) and precipitation

	Computation time: 30 seconds

	2. The second job is just for the sorting of the difference of the temperature. It is taking the output of the first job as a input. It is reading the the output file of the first job. And in that map will write the output as below.
	outputKey: Difference(min-max)
	outputValue: stateName, months with minimum & maximum temperature, precipitaion 

	Now we will take this output as a input in reducer. But here this mapper will sort the records according to the output key. So our records will be sortd with respect to temperature difference. And in this reducer we just chande the key and value pair as below.
	outputKey: stateName
	outputValue: months with minimum & maximum temperature, precipitaion, difference
	Computation time: 6-8 seconds


3. 

	- We have chosen a Map-side join. As we have two files, one is for locations and second is for readings. We need a station locations with the second file, so that we can do a computation for a particular station in reduce function.
	
	- To implement this we take a Hash Map to store a station locations with the key as weather station id and value as state name. Now we can use this hash map into Map function.We will read records from the recording files by key as a weather station id in the map. Using this key we can fetch the state name from the Hashmap. And using this state name as a key, we are writting the output in the MapOutput. So we will get the records joined by weather station id having state name on each.


4.
	- Our over all program is running in less than 36 seconds which is very optimized. It was because of map side join. That helps us in getting the join in very efficient way.
	
	- We have added average precipitation details successfully for two months having average highest and lowest temperature.
	- We are trying to add the locations which do not have a state name, with the country name as "US".
		We can take lattitude and longitude to dicide the location of a particaular station without state name. We have parted the stations into two parts Altantic and Pacific according to their longitude and latitude. 





Sample output::

VI	JUL 83.276 PRCP:0.166	FEB 77.546 PRCP:0.066	Diff: 5.729
PR	AUG 82.381 PRCP:0.000	FEB 76.161 PRCP:0.000	Diff: 6.22
HI	AUG 78.120 PRCP:0.073	FEB 70.217 PRCP:0.209	Diff: 7.903
Pacific	SEP 53.911 PRCP:0.000	MAR 43.934 PRCP:0.000	Diff: 9.977
FL	AUG 82.540 PRCP:0.521	JAN 61.215 PRCP:0.201	Diff: 21.325
Altantic	JUL 64.549 PRCP:0.146	FEB 41.558 PRCP:0.278	Diff: 22.991
CA	JUL 73.384 PRCP:0.056	DEC 47.856 PRCP:0.194	Diff: 25.528
LA	AUG 82.513 PRCP:0.374	JAN 52.008 PRCP:0.300	Diff: 30.506
OR	JUL 67.301 PRCP:0.068	DEC 36.684 PRCP:0.226	Diff: 30.616
WA	JUL 66.510 PRCP:0.084	DEC 35.551 PRCP:0.232	Diff: 30.96
GA	AUG 80.820 PRCP:0.431	JAN 48.314 PRCP:0.245	Diff: 32.507
TX	AUG 82.803 PRCP:0.382	JAN 49.325 PRCP:0.179	Diff: 33.478
MS	AUG 81.661 PRCP:0.343	JAN 48.112 PRCP:0.341	Diff: 33.549
AL	AUG 81.148 PRCP:0.420	JAN 47.467 PRCP:0.296	Diff: 33.68
SC	AUG 80.852 PRCP:0.366	JAN 46.688 PRCP:0.181	Diff: 34.164
NC	AUG 78.926 PRCP:0.333	JAN 44.276 PRCP:0.181	Diff: 34.65
VA	AUG 77.079 PRCP:0.350	FEB 39.286 PRCP:0.150	Diff: 37.793
NM	JUL 74.949 PRCP:0.218	JAN 35.273 PRCP:0.080	Diff: 39.676
TN	AUG 80.125 PRCP:0.347	JAN 40.180 PRCP:0.284	Diff: 39.946
WV	AUG 72.732 PRCP:0.291	FEB 32.684 PRCP:0.119	Diff: 40.048
DE	JUL 76.525 PRCP:0.296	FEB 36.389 PRCP:0.144	Diff: 40.136
AR	AUG 80.886 PRCP:0.385	JAN 40.683 PRCP:0.278	Diff: 40.203
MD	JUL 76.595 PRCP:0.254	FEB 36.255 PRCP:0.172	Diff: 40.34
AZ	JUL 84.707 PRCP:0.206	DEC 44.282 PRCP:0.153	Diff: 40.425
RI	JUL 72.825 PRCP:0.311	FEB 31.992 PRCP:0.220	Diff: 40.833
KY	AUG 78.079 PRCP:0.346	FEB 36.459 PRCP:0.201	Diff: 41.62
NJ	JUL 75.340 PRCP:0.355	FEB 33.392 PRCP:0.189	Diff: 41.949
MA	JUL 71.463 PRCP:0.295	FEB 29.321 PRCP:0.245	Diff: 42.142
CT	JUL 72.865 PRCP:0.319	FEB 30.632 PRCP:0.253	Diff: 42.233
OK	AUG 82.268 PRCP:0.352	JAN 39.977 PRCP:0.142	Diff: 42.291
PA	JUL 72.566 PRCP:0.276	FEB 29.188 PRCP:0.157	Diff: 43.378
NY	JUL 71.360 PRCP:0.295	FEB 26.999 PRCP:0.146	Diff: 44.361
OH	AUG 73.040 PRCP:0.287	FEB 28.044 PRCP:0.157	Diff: 44.996
MO	AUG 78.262 PRCP:0.308	JAN 33.153 PRCP:0.169	Diff: 45.109
ME	JUL 66.197 PRCP:0.300	JAN 21.017 PRCP:0.173	Diff: 45.18
DC	JUL 77.440 PRCP:0.000	JAN 32.229 PRCP:0.000	Diff: 45.211
NH	JUL 67.890 PRCP:0.344	FEB 22.366 PRCP:0.227	Diff: 45.523
IN	AUG 73.887 PRCP:0.288	FEB 28.347 PRCP:0.196	Diff: 45.54
CO	JUL 68.565 PRCP:0.158	JAN 23.011 PRCP:0.074	Diff: 45.555
AK	JUL 54.231 PRCP:0.148	JAN 8.462 PRCP:0.164	Diff: 45.769
IL	AUG 74.671 PRCP:0.292	FEB 28.644 PRCP:0.209	Diff: 46.026
MI	JUL 68.577 PRCP:0.234	FEB 22.024 PRCP:0.098	Diff: 46.553
KS	JUL 79.211 PRCP:0.322	DEC 32.569 PRCP:0.178	Diff: 46.642
VT	JUL 68.281 PRCP:0.337	JAN 21.097 PRCP:0.129	Diff: 47.185
ID	JUL 72.191 PRCP:0.088	JAN 24.225 PRCP:0.111	Diff: 47.966
NV	JUL 83.312 PRCP:0.110	DEC 34.107 PRCP:0.069	Diff: 49.206
WY	JUL 71.499 PRCP:0.154	DEC 20.928 PRCP:0.059	Diff: 50.571
NE	JUL 76.358 PRCP:0.211	DEC 25.621 PRCP:0.119	Diff: 50.737
MT	JUL 73.344 PRCP:0.086	DEC 21.946 PRCP:0.061	Diff: 51.398
IA	JUL 74.452 PRCP:0.320	JAN 22.998 PRCP:0.068	Diff: 51.454
WI	JUL 70.045 PRCP:0.295	FEB 18.492 PRCP:0.101	Diff: 51.553
UT	JUL 79.177 PRCP:0.082	JAN 25.017 PRCP:0.123	Diff: 54.159
SD	JUL 75.425 PRCP:0.175	FEB 20.323 PRCP:0.059	Diff: 55.102
MN	JUL 71.116 PRCP:0.250	FEB 13.834 PRCP:0.063	Diff: 57.282
ND	JUL 72.366 PRCP:0.222	FEB 11.906 PRCP:0.039	Diff: 60.459
