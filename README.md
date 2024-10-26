## Data Mining Project 1
## Charlotte's Neighborhood Crime Over Time

### Introduction
Crime is a concern for many urban areas in the United States, and Charlotte is no exception. It is important to understand crime patterns and statistics to help communities and law enforcement agencies develop plans, allocate resources, and engage with the community to improve public safety. The Charlotte-Mecklenburg Police Department (CMPD) regularly publishes detailed crime reports. As of July 22, 2024, overall crime in Charlotte has seen a slight increase of 1% compared to the previous year. This includes various types of crimes, categorized broadly into violent crimes and property crimes.

For this project, I plan to explore how crime has evolved over time in different communities and how does the location (such as open field, department store, hotel/motel, etc.) of the incident within neighborhoods effective crime? To answer this question, I will use the CMPD_PATROL_DIVISION and the PLACE_DETAIL-DESCRIPTION columns to examine these trends over time. By determining communities that are experiencing higher than normal levels of crime and specific location, the city, local law enforcement, and community can help allocated resources, develop plans and support community outreach to support all neighborhoods.
https://data.charlottenc.gov/datasets/charlotte::cmpd-incidents-1/about

###Dataset

The dataset is available from the city of Charlotte's open data portal. Data is available in various formats including CSV and contains both criminal and non-criminal incident reports from 2017 through 2024. It contains 688,973 observations and 29 features.
These features are 'X', 'Y', 'YEAR', 'INCIDENT_REPORT_ID', 'LOCATION', 'CITY', 'STATE', 'ZIP', 'X_COORD_PUBLIC', 'Y_COORD_PUBLIC', 'LATITUDE_PUBLIC', 'LONGITUDE_PUBLIC', 'DIVISION_ID', 'CMPD_PATROL_DIVISION', 'NPA', 'DATE_REPORTED', 'DATE_INCIDENT_BEGAN', 'DATE_INCIDENT_END', 'ADDRESS_DESCRIPTION', 'LOCATION_TYPE_DESCRIPTION', 'PLACE_TYPE_DESCRIPTION', 'PLACE_DETAIL_DESCRIPTION', 'CLEARANCE_STATUS', 'CLEARANCE_DETAIL_STATUS', 'CLEARANCE_DATE', 'HIGHEST_NIBRS_CODE', 'HIGHEST_NIBRS_DESCRIPTION', 'OBJECTID', and 'GlobalID'.
The website does not contain detailed metadata to describe. So, an attempt was made to define features based on the dataset. Below are descriptions of the less obvious features:

•	'X', 'Y' are unknown decimal values.

•	'INCIDENT_REPORT_ID' is the case number associated with the incident.

•	'LOCATION' is the physical address of the incident.

•	'X_COORD_PUBLIC', 'Y_COORD_PUBLIC' are unknown integer values.

•	'CMPD_PATROL_DIVISION' is the name of the division. It corresponds to numeric 'DIVISION_ID'.

•	'ADDRESS_DESCRIPTION' is a higher-level description of where the incident took place. Field mainly contained 'Location of occurrence' or 'Location where officer took report'

•	'NPA' is the Neighborhood Profile Area ID, a unique number that is assigned to different neighborhoods in Charlotte. It replaced the previous method of using the name of the community.

•	'PLACE_TYPE_DESCRIPTION' is a detailed description of 'LOCATION_TYPE_DESCRIPTION' which indicates if private resident, Gas station, etc.

•	'CLEARANCE_DETAIL_STATUS' is detailed description of 'CLEARANCE_STATUS' which provides how a case was cleared.

•	'HIGHEST_NIBRS_CODE' is the highest offense id number for the incident as defined by the FBI's National Incident-Based Reporting System (NIBRS)

•	'OBJECTID' is the index.

•	'GlobalID' is an unknown alpha-numeric value.

Mainly, I used CMPD_PATROL_DIVISION and the PLACE_DETAIL-DESCRIPTION columns for this project. The CMPD_PATROL_DIVISION column provides the names of the patrol division for a particular area of the city such as "North" and "Steele Creek" divisions. These divisions will represent the neighborhoods. The PLACE_DETAIL-DESCRIPTION provides the location where a crime took place. For example, "Private Resident", "Open Field", and "Air/Bus/Train Terminal". These columns will allow us to compare neighborhoods and look within neighborhoods to see if certain locations have experienced above average crime incidents.

### Pre-Processing

Pre-processing is one of the most important steps. By thoroughly cleaning the data, we will improve the accuracy of our model and save time by removing errors in advance.
The pre-processing begins with importing the csv dataset. Initially, I received a “ParserError: Error tokenizing data. C error” which is caused when python process senses some rows have more data than expected. I added on_bad_lines ='skip' to the import statement which would skip any bad rows of data. After the import, there were no rows removed. Its shape is (688973, 29).
```
CLT_crime = pd.read_csv(r"C:\Users\SPRQ679X2\Downloads\CMPD_Incidents.csv", on_bad_lines = 'skip')
pd.set_option('display.max_columns', None)
```

#### Irrelevant and Missing Values
'X' and 'X_COORD_PUBLIC' contained the same value. The only difference was 'X' was in decimal format and 'X_COORD_PUBLIC' was in integer format. This was the same situation for 'Y' and 'Y_COORD_PUBLIC'. Metadata could not be located on the website to determine the purpose of 'X_COORD_PUBLIC' and 'Y_COORD_PUBLIC'. All four features were removed, since they would not be used for analysis in this project.
For this project, there is no use for 'LATITUDE_PUBLIC' and ' LONGITUDE_PUBLIC'. Both were removed from the analysis.
'INCIDENT_REPORT_ID' represents the unique report number associated with each incident. This ID does not offer beneficial information for the project.
'GlobalID' is an alphanumeric variable. Its purpose could not be determined. So, I decided to remove the column.
'OBJECTID' seems to be an index for the dataset. I decided to remove this column and create an index after pre-processing the dataset.

Checked for missing values by using CLT_crime.info(). We can see six columns were missing values: STATE, ZIP, CMPD_PATROL-DIVISION, DATE_INCIDENT_END, ADDRESS_DESCRIPTION, and CLEARANCE DATE.
LOCATION (physical address of the incident), CITY, STATE, and ZIP can be replaced by the DIVISION_ID OR CMPD_PATROL-DIVISION which are better suited in describing where the incident took place. Those four features were removed for this project.
ADDRESS_DESCRIPTION can also be replaced by the DIVISION_ID OR CMPD_PATROL-DIVISION.
CMPD_PATROL_DIVISION is a more descriptive version of DIVISION_ID, so we can impute the missing information based on the current values in DIVISION_ID.
```
# fillna to address missing values in CMPD_PATROL_DIVISION
CLT_crime['CMPD_PATROL_DIVISION'] = CLT_crime['CMPD_PATROL_DIVISION'].fillna(CLT_crime['DIVISION_ID'].map(division_mapping))

# drop rows with Na in the CMPD_PATROL_DIVISION column
CLT_crime = CLT_crime.dropna(subset=['CMPD_PATROL_DIVISION'])
```

DATE_INCIDENT_END indicates the date that the incident or cases was resolved. I will impute those missing dates with today’s current date. This will provide an accurate measure of the number of days that a case has been open, if I decide to create a column in the future. For the same reason we will retain the CLEARANCE DATE and impute it with the current date.
```
# impute replace values with current date
from datetime import datetime

current_date = datetime.now().date()

CLT_crime['CLEARANCE_DATE'].fillna(current_date, inplace=True)
CLT_crime['DATE_INCIDENT_END'].fillna(current_date, inplace=True)
```
LOCATION_TYPE_DESCRIPTION provides a high-level location for the incident. This will be replaced with a better feature.
Data Types Checked
The data types were checked using CLT_crime.info(). All variables were of type object, except for YEAR, X_COORD_PUBLIC, Y_COORD_PUBLIC, LATITUDE_PUBLIC, LATITUDE_PUBLIC, and NPA. Those data types are fine.
Index
Lastly, the data set was indexed using CLT_crime.index = [x for x in range(1, len(CLT_crime.values)+1)] Normally, when data is imported, python automatically creates an index; however, the first row is index as 0 rather than 1. Now, the first row is indexed at 1 and the column is named ID.
```
CLT_crime.index = [x for x in range(1, len(CLT_crime.values)+1)]

# add index field name
CLT_crime.index.name = 'id'
CLT_crime.head(4)
```




