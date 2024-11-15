## **Charlotte's Neighborhood Crime Over Time Through Clustering**

### **Overview**
For Project 4, the goal is to work with clustering.  Writing is critical here, as the main goal will be to discuss your process, why you take certain steps (e.g., what preprocessing steps and why), and tell a story around your data and insights gained through both modeling (working with the clustering algorithms we learned about) and visualizations. Thus, your writing should portray your critical thinking about the data, the process, and what knowledge you find.

##  **Introduction**
Crime is a concern for many urban areas in the United States, and Charlotte is no exception. It is important to understand crime patterns and statistics to help communities and law enforcement agencies develop plans, allocate resources, and engage with the community to improve public safety. The Charlotte-Mecklenburg Police Department (CMPD) regularly publishes detailed crime reports. As of July 22, 2024, overall crime in Charlotte has seen a slight increase of 1% compared to the previous year. This includes various types of crimes, categorized broadly into violent crimes and property crimes.
Previously, [Project 1 ](https://pmb-7684.github.io/Data_Mining_Project_1/) Charlotte's Neighborhood Crime Over Time explored the data through exploratory data analysis. It focused on how much crime changed over time based on the different neghborhoods or divisions.  

For this project, the data will be explored through clustering analysis.  Again, the data explored how crime has evolved over time in different communities and how does location (such as open field, department store, hotel/motel, etc.) of the incident  within neighborhoods effective crime.  The difference is clustering will be used to explore if certain neighborhood belong to the same cluster and types of crimes likely to happen in those clusters.  

 By determining communitites that are experiencing higher than normal levels of crime and specific location, the city, local law enforcement, and community can help allocated resources, develop plans and support community outreach to support all neighborhoods.

CMPD Data [Portal](https://data.charlottenc.gov/datasets/charlotte::cmpd-incidents-1/about)

City of Charlotte[ GIS](https://maps.mecknc.gov/openmapping/data.html)


### **What is clustering and how does it work?**
`Explain what clustering is and how it works (e.g., k-means and/or agglomerative that we have gone over in class).`
In general, clustering is the process of grouping items with common characteristics into a group.

**k-Means** is an unsupervised algorithm used for clustering. The algorithm works by dividing a group of observations into a predetermined number of clusters. This number of clusters (k) is determined by the user. The user can either randomly select a number or use a method such as "elbow method" to calculate the number of clusters. Then k data are randomly selected to be centroid(center of the cluster). Each observation is assigned to a centroid based on distance to the closest centroid. Once all observations are assigned, recalculate the centroid by taking the average of the points assignment to the cluster. then determine the distance and reassign each observation. The algorithm is repeated until the centroids no longer move, no data points change clusters or the maximum number of iterations is reached.

**Agglomerative Hierarchical Clustering** works by splitting and merging the closest pairs of clusters until all the observations belong to a single cluster.

### **The Dataset**

The dataset is available from the city of Charlotte's open data portal. Data is available in various formats including CSV and contains both criminal and non-criminal incident reports from 2017 through 2024. It contains 688,973 observations and 29 features.


###Some Domain and Variable Notes:

•	`X`, `Y` are unknown decimal values.

•	`INCIDENT_REPORT_ID` is the case number associated with the incident.

•	`LOCATION` is the physical address of the incident.

•	`X_COORD_PUBLIC`, `Y_COORD_PUBLIC` are unknown integer values.

•	`CMPD_PATROL_DIVISION` is the name of the division. It corresponds to numeric 'DIVISION_ID'.

•	`ADDRESS_DESCRIPTION` is a higher-level description of where the incident took place. Field mainly contained 'Location of occurrence' or 'Location where officer took report'

•	`NPA` is the Neighborhood Profile Area ID, a unique number that is assigned to different neighborhoods in Charlotte. It replaced the previous method of using the name of the community.

•	`PLACE_TYPE_DESCRIPTION` is a detailed description of 'LOCATION_TYPE_DESCRIPTION' which indicates if private resident, Gas station, etc.

•	`CLEARANCE_DETAIL_STATUS` is detailed description of 'CLEARANCE_STATUS' which provides how a case was cleared.

•	`HIGHEST_NIBRS_CODE` is the highest offense id number for the incident as defined by the FBI's National Incident-Based Reporting System (NIBRS)

•	`OBJECTID` is the index.

•	`GlobalID` is an unknown alpha-numeric value.

The shape of the orginal dataset is (688973, 29).

**How does this step relate to your modeling?**


### **Pre-Processing**

By thoroughly cleaning the data, we will improve the accuracy of our model and save time by removing errors in advance.  The same pre-processing were completed on this project as completed on project one. Since, we are looking for clusters between the different neighborhoods (or districts) in the city, the number of features will be considerably smaller. Since most of the pre-processing was completed for project 1, below is a summary of task completed.  The following features were not used in the modeling or analysis and removed.

##### **Irrelevant**

`X` and `X_COORD_PUBLIC` contained the same value. The only differece was `X` was in decimal format and  `X_COORD_PUBLIC` was in integer format.  This was the same situation for '`Y` and `Y_COORD_PUBLIC`. 

For this project, there is no use for 'LATITUDE_PUBLIC' and '	LONGITUDE_PUBLIC'. 

`INCIDENT_REPORT_ID` represents the unique report number associated with each incident. 

`GlobalID` is an alphanumeric variable. It's purpose could not be determined. 

`OBJECTID` seems to be an index for the dataset. This column was removed and a seperate index was created during the pre-processing.

`LOCATION` (physical address of the incident), `CITY`, `STATE`, and `ZIP` can be replaced by the DIVISION_ID OR CMPD_PATROL-DIVISION which are better suited in describing where the incident took place. Those four features were removed.

##### **Missing Values**

After checking for missing data, there were six columns were missing values: STATE, ZIP, CMPD_PATROL-DIVISION, DATE_INCIDENT_END, ADDRESS_DESCRIPTION, and CLEARANCE DATE.  STATE and ZIP as stated above were not used and removed.

`CMPD_PATROL_DIVISION` is a more descriptive version of DIVISION_ID, so we can impute the missing information based on the current values in DIVISION_ID.  

`DATE_INCIDENT_END` indicates the date that the incident or cases was resolved.  I will impute those missing dates with today’s current date.  This will provide an accurate measure of the number of days that a case has been open, if I decide to create a column in the future.  For the same reason we will retain CLEARANCE DATE and impute it with the current date.

`ADDRESS_DESCRIPTION` can also be replaced by the CMPD_PATROL-DIVISION.

`LOCATION_TYPE_DESCRIPTION` provides a high-level location for the incident. This will be replace with a better feature.

#####**Data Types Checked**

All variables were of type `object`, except for YEAR, X_COORD_PUBLIC, Y_COORD_PUBLIC, LATITUDE_PUBLIC, LATITUDE_PUBLIC, and NPA.  Those data types are numeric.

##### **Index**

Normally, when data is imported, python automatically creates an index; however, the first row is index as 0 rather than 1. Now, the first row is indexed at 1 and the column is named ID.

The data set contains 14 features and 688,345 observations.

### **Data Understanding/Visualization**
`Use methods to try to further understand and visualize the data. Make sure to remember your initial problems/questions when completing this step.
While exploring, does anything else stand out to you (perhaps any surprising insights?)`
When working with k-mean clustering it is important that all features are on the same scale. Recall with k-means algorithm it calculates the distance between points. If we failed to scale, then points in a higher range skew the calculations and likely those calculated would be inaccurate.

The x-labels were removed to reduce the amount of clutter on the axis since there are so many features. We can visually see the importance of standardization. Before standardization, our features ranged from 0 to over 1200. After standardization, all features are between -3 and 4. The standardized data will perform much better.

<img src="images/Scaled_comparison.png?raw=true"/>


<img src="images/correl_map.png?raw=true"/>

### **Modeling(Clustering)**
`What model(s) do you use to try to solve your problem? Why do you choose those model(s)?

For example, why choose k-means over agglomerative, or vice versa? Or perhaps experiment with both and discuss the pros/cons of each? You may also try experimenting with other methods of clustering not discussed in class.`


### **Storytelling (Clustering Analysis)**
Use this section to further analyze your clusters.

What information or insights does it tell you? What have you learned? Were you able to answer your initial problems/questions (if so, discuss that)?


### **Impact Section**
Discuss the possible impact of your project. This can be socially, ethically, etc. It cannot be something like "our project has no impact" or "our project has no negative impact." Even the most well-intentioned projects *could* have a negative impact. We will not be checking for "right" or "wrong" answers, but showing your critical thinking.


### **References**
