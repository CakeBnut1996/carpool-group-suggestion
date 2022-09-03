# user spatiotemporal clustering for carpool group suggestion

## Problem Description
Users are clustered based on spatial and temporal trip pattern. Users in the same group will be matched and car-pooling suggestions could be offered. Based on their previous trip histories, groups could be generated and promoted to users for them to join. Users could select their own groups as well. Groups could be updated automatically by analyzing their latest travel pattern or manually by users. 

Users are clustered based on the geometrical location and departure time of their important locations. First, important locations that are within walking distance to each other are grouped together using hierarchical clustering. Based on users’ departure time distribution of different important locations. Several groups could be generated for each user. For example, if a user usually leaves community A around 7 AM and leaves community B around 4 PM, he or she will be suggested a group that is labeled “6 - 8 AM, community A” and “4 – 6 PM, community B”. Before starting the trip, he or she could look for people who leave similar time at the same place (not necessarily end at the same place  ) to car pool

## Descriptive Statistics
1.	Sample data

Trip history (user and trip id, origin label, destination label, trip start and end time, city) and important locations (location id, user id, location center longitude and latitude, city) for each user are collected for clustering. Sample data are showed below.

Table 1. Sample Data of Trip history
![image](https://user-images.githubusercontent.com/46463367/188283564-905c20e5-d987-4817-b8b2-9057bd324990.png)

Table 2. Sample Data of Important Locations 
![image](https://user-images.githubusercontent.com/46463367/188283567-ed9aacbf-5e6e-421d-850a-44d3a578e8d3.png)

2.	Dataset summary

5000 important locations and 37005 records of trips from 3/12/2018 to 10/22/2018 are used here. 1320 users and 1189 important locations are included in the dataset. Among all the user, 310 of which from Tucson, 839 from El Paso, 280 from Austin. 

3.	Departure time distribution 

The following figure show the trip departure time in the three cities. Austin is an hour earlier than Tucson. El Paso is in the same time zone with Tucson. Departure time in Tucson is around 7,8AM and 4, 5 PM. Departure time in Austin is around 6, 7 AM and 4 – 6 PM. Departure time in El Paso is around 7 AM and 5 PM.

Figure 1. Start time distribution in Tucson, Austin and El Paso:

![image](https://user-images.githubusercontent.com/46463367/188283582-94d1af54-99a2-4c5d-b745-ba89209e657e.png)
![image](https://user-images.githubusercontent.com/46463367/188283587-8d6a8e9f-3552-46aa-a772-65bdc5775595.png)
![image](https://user-images.githubusercontent.com/46463367/188283591-9d1faf7b-2ae7-4138-a284-3352ac11d29a.png)


4.	Important locations distribution

The important locations are distributed in three cities, as is shown in the figure below. 

Figure 2. Spatial distribution of the important locations:

![image](https://user-images.githubusercontent.com/46463367/188283608-050b3fc0-a046-4666-a15c-adc286eda645.png)

Most of the users have less than 5 important locations. The distributions of number of important locations in the three cities are showed in the following figure.

Figure 3. Distribution of number of important locations:

![image](https://user-images.githubusercontent.com/46463367/188283623-5012f3b8-c015-46c4-9216-321e69961cdb.png)
![image](https://user-images.githubusercontent.com/46463367/188283624-b7b73308-0ab5-4993-a1d0-996715464124.png)
![image](https://user-images.githubusercontent.com/46463367/188283626-4015429e-bdaa-402e-bac2-b553f89a0e71.png)


## Data Preprocessing 

Longitude and latitude of each important locations are converted to points on Cartesian coordinate system of the city for the convenience of calculation. The original point is the minimum longitude and minimum latitude of all the GPS points in that city. The longitude is converted to distance to minimum longitude in kilometers. The latitude is converted into distance to minimum latitude in kilometers. 

Considering some important locations of users are not frequent locations. For example, a user goes to every football game in the season, which is not a daily trip like commute trip. Weekend groups may usually involve trip to theatres, sports fields or restaurants. Therefore, as long as we know a user always goes for a certain activity, a group could be created by the name of the event, which do not need to have a specific departure time. Here, we assume weekday trips always involve commute trips, which should be clustered by location and time and weekend trips always involve entertainment trips, which could be clustered only by location/event. Therefore, trip history database is divided based on weekend and weekdays. There are 5125 trips on weekends and 31880 trips on weekdays. 

Distance matrix is calculated for the following algorithms. It contains the distance between each location to another. The unit is in kilometers. The size is of the matrix is 1189×1189.  The sample data are in the figure below. 

Table 3. Sample data of distance matrix
![image](https://user-images.githubusercontent.com/46463367/188283667-21437771-f551-4be8-9327-d485476d35cb.png)

## Algorithms used and rationale

Several algorithms are tried to find out the best way to divide users based on their important locations.

1.	Density-based spatial clustering of applications with noise (DBSCAN)

a.	Goal

To find clusters that minimum locations and maximum radius could be specified. 

b.	Algorithm

DBSCAN is a density based algorithm – it assumes clusters for dense regions. It only finds clusters at or above the density based on the following parameters. The input of dbscan include distance matrix, minimum points mp and maximum radius mr of a group. At least mp people will be in a group, and the maximum radius of the group is mr. A location is a core point if at least mp locations are within distance mr. Those locations are said to be directly reachable from the core location. All the locations with the circle whose radius is the core point and radius mr are in the same group.  A location q is reachable from core p if there is a path consist of core points that are directly reachable to each other. Outliers are also identified, from which all points are not reachable.

c.	Result

mp = 2 (locations), mr = 0.8 km
119 groups of important locations are identified. The largest groups contain 456 locations. This is not reasonable since the group is too big for users to find the appropriate match. 

