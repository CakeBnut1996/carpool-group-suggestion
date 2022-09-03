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

Figure 4. Cluster result of dbscan:

![image](https://user-images.githubusercontent.com/46463367/188283697-eeeea70a-1ed8-4423-8f31-8cf88117e37e.png)

2.	Kmeans

a.	Goal

To divide the area into clusters, in which the distance are within walking distance (0.5 mi).

b.	Number of clusters n

Elbow chart is drawn to find the best number of clusters. The optimal number is three in Tucson. But that will cause the same problem as dbscan, which is too big clusters. Therefore, number of clusters are selected manually. The value is set to be N = 120 at first.

Figure 5. Elbow chart of K-means:

<img src="https://user-images.githubusercontent.com/46463367/188283708-b4ed2737-00fd-44cf-b216-67ecc218afea.png" width="400" height="400" />

c.	Result

The clustering result is shown below. Same color might be used for different clusters. But clusters that are adjacent are colored differently. 
The clusters are divided exhaustively and all the clusters are relative small. 

Figure 6. Cluster result of K-means:

<img src="https://user-images.githubusercontent.com/46463367/188283726-e9eb6566-a428-49a1-bddf-6728e96fcb89.png" width="400" height="400" />

d.	Iteration
The number of clusters could be changed many times to see if the max distance within each group is small enough (best within walking distance)

3.	Hierachical clustering

a.	Goal

The number of the best clusters could be selected automatically. And the each group should be relative small. 

b.	Algorithm

Agglomerative clustering is really a suite of algorithms all based on the same idea. The fundamental idea is that you start with each point in it’s own cluster and then, for each cluster, use some criterion to choose another cluster to merge with. Do this repeatedly until you have only one cluster and you get get a hierarchy, or binary tree, of clusters branching down to the last layer which has a leaf for each point in the dataset.(https://hdbscan.readthedocs.io/en/latest/comparing_clustering_algorithms.html) 

The following linkage methods are used to compute the distance d(s,t) between two clusters s and t. The algorithm begins with a forest of clusters that have yet to be used in the hierarchy being formed. When two clusters s and t from this forest are combined into a single cluster u, s and t are removed from the forest, and u is added to the forest. When only one cluster remains in the forest, the algorithm stops, and this cluster becomes the root. A distance matrix is maintained at each iteration. The d[i,j] entry corresponds to the distance between cluster i and j in the original forest.
method=’complete’ assigns

d(u,v)=max(dist(u[i],v[j]))

for all points i in cluster u and j in cluster v. This is also known by the Farthest Point Algorithm or Voor Hees Algorithm. (https://docs.scipy.org/doc/scipy/reference/generated/scipy.cluster.hierarchy.linkage.html) 

Draw dendrogram of 1189 important locations of all users. The y axis represents the distance (km) between each group. We specify the distance to be 1.5 km.

Figure 7. Dendrogram of GPS points in Tucson:

![image](https://user-images.githubusercontent.com/46463367/188283751-1649cda2-7de3-48cf-98ea-1b16ab24019b.png)

c.	Result

By setting the maximum radius of the group, there are 261 groups generated by this method.

Figure 8. Cluster result of Hierachical Clustering:

![image](https://user-images.githubusercontent.com/46463367/188283762-760598e8-3392-4024-8f51-61590f009f62.png)

4.	Other 
https://en.wikipedia.org/wiki/Fuzzy_clustering#Fuzzy_C-means_clustering 

![image](https://user-images.githubusercontent.com/46463367/188283788-e792c0f2-57e0-4a3a-8403-6e44439a5f66.png)

![image](https://user-images.githubusercontent.com/46463367/188283796-cb37e8c1-8e1c-43ce-aac7-6c13530ee219.png)


## Evaluation
Comparison among algorithms  

1.	Dbscan

Distance matrix of each group are used to evaluate the algorithm.

The table below shows the descriptive statistics of distance between each point in each group. The average maximum distance in each group is about 15 kilometers. This means people that are 15 kilometers far from each other could be grouped together. 

Table 4. Descriptive statistics of within-group distance matrix by dbscan

![image](https://user-images.githubusercontent.com/46463367/188283812-ef47727d-661f-40b4-b592-db83e8509c60.png)

DBSCAN's parameter of maximum radius is not actually the maximum distance because multiple steps are joined transitively. Clusters can become much larger than mr. If the goal is the have relative big groups so that every possible people that could be matched for car pooling, dbscan could be used and get good result. 

2.	K-means
While K-Means is easy to understand and implement in practice, the algorithm does not take care of outliers, so all points are assigned to a cluster even if they do not belong in any. In the domain of anomaly detection, this causes problems as anomalous points will be assigned to the same cluster as “normal” data points. The anomalous points pull the cluster centroid towards them, making it harder to classify them as anomalous points.

To identify if the distance within each group is small enough, the distance matrix in one group is calculated as an example. This group has the most locations. The distance from all the points else in the group to point 21, 550,… to the points else are all less than 1.5 km. This means that the group is small enough.  It could be seen that people in group could reach each other within 1.5 km. 

Table 5. Descriptive Statistics of within-group distance matrix by K-means

![image](https://user-images.githubusercontent.com/46463367/188283827-6fb8c284-9122-4450-b1dc-d06000084cfe.png)

3.	Hierachical Clustering 
The advantage of this approach is that clusters can grow ‘following the underlying manifold’ rather than being presumed to be globular. (https://hdbscan.readthedocs.io/en/latest/comparing_clustering_algorithms.html)  Use the distance matrix of one group to see if the max distance within that group is small enough.
The maximum group contain 27 important locations. We could find that maximum and average distance within the group are less than that generated by K-means.

Table 6. Descriptive Statistics of within-group distance matrix by Hierachical Clustering

![image](https://user-images.githubusercontent.com/46463367/188283842-ea02bd32-82d7-4c76-a13e-8a0890a7a575.png)

