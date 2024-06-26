# A Systemwide Assessment of Wait Times for the New York City Subway (2024 ENGR 1050 Final Project)

## How to Run: 
*Option 1*

Download the .zip file “ENGR 1050 Final Project” and upload it to Google Drive. On Colab, run the first and second blocks of code. This will unzip this zip file. Change endLoc in the second block of code as desired.

You will need to change realTimeData (block 3 line 8), stops (block 3 line 9), and bullet (line 143) in the main code file depending on your file directory arrangement. 

Note that this program takes around 10-13 minutes to complete. Google Colab has a tendency to disconnect the runtime at random, so if this occurs while running the program, you may need to rerun it.

## Summary
For this project, we analyzed data provided by the MTA for the New York City Subway. The MTA provides real time data, as to the positions of trains in the system, and various sources collect this data. We were able to gain access to the data collected for all the trains in the system on Monday, February 19th, 2024 (which we can treat as a “typical” weekday). With this data, we set out to explore three main ideas. Firstly, we wanted to visually understand the paths of trains as they traveled through the system during morning and evening rush hours; this could lead to qualitative insights as to where trains tend to terminate, whether trains run express vs. local on a certain line, or where choke points tend to occur that could lead to cascading delays. Secondly, we sought to explore the distribution of interval times for each line (we define an interval time to be the time between trains of a certain line at a certain station—to take the distribution of these interval times for a specific line, we combine all the interval times for all the stations on that line). We wanted to explore how different lines have similar or different interval time distributions using statistical testing. Finally, using the median of interval time distributions as a loose quantitative measure of line performance, we wanted to rank train lines based on the median of their interval time distributions. We also wanted to come up with the average wait time for each line (which is found by averaging the interval time distribution if it is assumed that subway lines behave as a Poisson distribution).

For the first part of my program, we imported two CSV files as pandas dataframes. One contained the stop times for each trip a train took. This dataframe displayed the train ID, when that train stopped at a given station, when it left that station, and so on for all the stations in that trip. It collected this data for all train trips of all the NYC Subway lines on this arbitrarily chosen date. The other CSV file contained the stop ID for all stations in the NYC subway system and data relating to that stop ID, including the street name of that stop. We set the directories referring to these files as global variables. For each trunk line (color in the subway system), we then ran trainPlot to gain a plot of the paths of trains as they traveled north and south through the system during morning and evening rush hours. After this, we created histograms displaying the train interval distribution for all the number lines (known as the IRT) and all of the letter lines (known as the IND or BMT) separately. We ran statistical tests on the distributions at a given rush hour time and with a given direction (north or south) for each line and obtained a dictionary of Shapiro-Wilk test p-values, an ANOVA test p-value, a boolean indicating whether the set of data we provided was all normal or not, and a dataframe containing the pairwise two sample t-test or Mann-Whitney U test p-val (depending on whether all the data are normally distributed or not). We inputted this dataframe into the makeHeatmap function in order to create a logarithmically-scaled heatmap of pairwise p-values. Finally, we printed the mean wait times for each line as a dictionary and ran boxAndWhisker to generate a box and whisker plot for each line ordered from smallest median interval time to largest median interval time.
	
We fulfilled computation through extensive filtering and manipulations of the imported CSV file (which involved filtering by time of day, querying by line, the conversion of HH:MM:SS time into time in just seconds), scaling of my plots to ensure that they all had the same size regardless of the number of plots on each figure, creating a distribution that collected the seconds between trains at a station during a specified time of day, logarithmic scaling of the heatmap of pairwise p-values (to make it easier to read), and the calculation of the mean and five number summary for each line’s time interval distribution. 

For statistics, we performed various analyses, including a Shapiro-Wilk Test to determine normality, an ANOVA Test, and pairwise Mann-Whitney U Test/Two Sample t Test depending on the normality of the distributions. We also collected a five number summary for each line’s time interval distribution and calculated a mean.

For data visualization, we created a line plot for each line, time of day, and direction to show the path train lines took as they traveled from their origin to their terminus. In this plot, we added the image of each train bullet to the title as well as the stop names on the y-axis instead of the internal values we assigned to them. We also created various histograms displaying time interval distributions as well as a boxplot ranking train lines from least to greatest median time interval distribution and giving a better comparison of time interval distributions between lines. In this latter visualization, we was able to color each boxplot by the color corresponding to the train line.

## Function Documentation
| Function                         | Purpose                                                                                                         | Contribution to Overall Project                                                                                                                                                                         |
|----------------------------------|---------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `readData(link)`                 | Imports CSV as a dataframe with the first column of the spreadsheet set to row indices                        | Allows for access of CSV data downloaded from online source                                                                                                                                             |
| `extract(link, endLoc)`          | Extracts all files from zip file at location link and sends them to the folder specified by endLoc             | Used to unzip the file downloaded from online source                                                                                                                                                    |
| `subwayLineQuery(df, line, direction)` | Takes in dataframe with realtime subway data, string representing subway line, and string representing direction. Returns dataframe with rows that correspond to weekdays, the line specified by the user, and the direction specified by the user | Filters the data for the day by line, direction, and time of day, which allows for the paths of trains going in that direction during that time of day to be visualized                                 |
| `getSec(time)`                   | Get total seconds from time of the form HH:MM:SS                                                               | Convert HH:MM:SS time (which is the time on the imported CSV file) into just seconds to allow for easier comparison between times and easier filtering by time of day                                    |
| `addSeconds(df, colList)`        | Given dataframe with specified list of columns that have time in format HH:MM:SS, adds that many columns with the same time in seconds format | Adds the time in seconds as a column in the imported CSV file (now converted to dataframe) to allow for easier comparison between times and easier filtering by time of day                             |
| `subwayTimeQuery(df, col1, col2, time1, time2)` | Given dataframe, two columns, and two time values, creates filtered view of dataframe that only contains rows between these times | Allows to filter by a certain time of day so plots can be made for paths of trains during morning and evening rush hour                                                                                 |
| `readBullet(line)`               | Given the specified train line, loads image of train bullet with OpenCV, switching the R and B color channels | Imports image of a train line’s bullet (or sign) so that it can be added to the graph showing the paths trains take in a certain direction at a certain time of day                                     |
| `stopExtractor(secondsDf, line, direction)` | Creates dictionary that associates each stop ID with a number based on order in a line’s trip                | Creates an internal stop value for each stop in order for a plot between stop value and time of day to be created later. This will also ultimately allow for the stop’s street name to be plotted on the y-axis |
| `stopIDtoName(stopsDf, stopIDlist)` | Takes in list of stop IDs, returns dictionary with stop IDs as keys and stop names as values                  | Associating stop ID with stop name creates an implicit association between stop value and stop name via stop ID, which allows for the replacement of stop values with stop street names in the y-axis of the plot of train paths |
| `partOne(line, direction, rushHourType)` | Given a line, a direction, and the time of day (morning rush or evening rush), returns stopDict, stopNames, timeList, time1, time2, and filteredSecondsDf | Provides all data needed to `trainPlot(lines)`, the function that creates the plot of train paths at a certain time of day going in a certain direction                                                 |
| `trainPlot(lines)`               | Create plot of all the trains that run in both directions during morning and evening rush. Return nothing      | Allows for visualization of train paths for a set of lines during both morning and evening rush hour and going both northbound and southbound                                                           |
| `distributionFinder(line, direction, rushHourType)` | Finds distribution of intervals between trains for a line given time and direction. Returns distribution as a numpy array | Creates array of time of arrivals, which can then be used to create a time interval distribution that can be plotted                                                                                    |
| `noExpressDifferentiatedQuery(df, line, direction)` | Takes in dataframe with realtime subway data, string representing subway line, and string representing direction. Returns dataframe with rows that correspond to weekdays, the line specified by the user, and the direction specified by the user. Does not differentiate by express vs. local variants for 6, 7, and F trains | This function was created so that the 6 and 6X and the 7 and 7X were treated as the same train as both essentially operate under the same internal designation. Previously, this was not done in order to visualize the train paths of the local and express variants separately |
| `pairwiseDifferences(df, columnName)` | Given dataframe and a specified column name, returns numpy array of all pairwise differences in that column  | Creates the time interval distribution that will then be plotted                                                                                                                                         |
| `lineDistribution(line, direction, rushHourType)` | Finds the time interval distribution for all lines going in a certain direction at a specified rush hour time (morning or evening) | Creates the data that will then be used to create histogram plots                                                                                                                                       |
| `trainIntervals(system)`         | Based on a given system (IRT or IND), plots all of the histograms representing distributions for time in between trains based on time of day and type of rush hour | Plots the letter line distributions and number line time interval distributions separately in order to prevent overcrowding. Statistical analysis will then be done on these distributions              |
| `shapirowilk(dataval)`           | Takes in list or 1D array of data values and returns p statistic from test                                     | Used to determine whether the distribution is normal or not                                                                                                                                             |
| `anovatest(distDict)`            | Takes in dictionary associating lines with their distributions. Perform ANOVA test on these distributions and returns associated p-val | Used to determine whether there is at least one pair for which the difference in means is statistically significant                                                                                     |
| `mannwhitneypairwise(type1, type2)` | Takes in two lists/1D arrays (type1 and type2) and returns p val                                              | Used to determine if two samples come from the same population or if there is sufficient evidence that they do not. Used if not normally distributed                                                    |
| `twosamplet(type1, type2)`         | Takes in two lists or arrays of data and returns p val                                                           | Used to determine if two samples come from the same population or if there is sufficient evidence that they do not. Used if normally distributed                                                        |
| `trainComparison(lines, direction, rushHourType)` | Takes in a set of lines, a common direction, and rushHourType. Performs ANOVA test and returns p-value. Runs Shapiro Wilk Normality test on all data and returns dataframe of p-values and whether all data is normal or not as a boolean. If all data are normal, performs pairwise t-tests on all data. If all data are not normal, performs pairwise Mann Whitney U tests on all data. Returns matrix of pairwise p-values | Gets all statistical results that will be printed and be used to create logarithmically scaled heatmap                                                                                                   |
| `makeHeatmap(statDf, normal, direction, rushHourType)` | Using the statDf that contains p-values from pairwise t-tests/MWU tests, creates a heatmap                      | Creates heatmap of p-values for pairwise tests and logarithmically scales them for easier visualization. This is done for each direction and each rush hour type (morning/evening)                      |
| `combineDistributions(lines)`     | Combines the four distributions created for a line (N and morning, N and evening, S and morning, S and evening) into one distribution for comparison between lines | This combining of distributions is done in order to create a boxplot that does not differentiate based on direction and based on time of day (to get a general idea of where lines stand)              |
| `boxAndWhisker(lines)`            | Creates a box and whisker plot for each line, highlighting 5 number summary. Outliers excluded. Boxplots are ordered from lowest median to highest median in order to get a picture of what trains come most frequently and what trains the least | Gives visual of what trains come most frequently and what trains come least frequently; gives some insight into the “best” most frequent trains and the “worst” least frequent trains                  |
| `averageWaitTime(lines)`          | Given list of lines, returns their average wait times in a dataframe as given by the Poisson distribution. Takes mean of distribution and converts it from seconds to minutes. At an arbitrary time of rush hour, gives expected amount of time you will wait for a particular line assuming the last train just left the station | Provides an expected average wait time for each line during rush hour, which can be used for planning and analysis                                                                                      |
| `main()`                          | Runs all aspects of the program. Expected output is train plots for each trunk line, train interval graphs for IRT and BMT, statistical testing for all lines north/south and morning/evening rush, four heatmaps for each of these times showing pairwise p-values, a boxplot of time interval distributions and an expected wait time given that the last train just left the station | Integrates all functions to produce comprehensive analysis and visualizations of train data, interval distributions, statistical test results, and graphical representations for practical insights       |

## Simplifications and Assumptions
One major assumption was that each station should be weighed equally when calculating the time interval distribution. Although this makes sense, it is also important to also take into account the passenger volume at each station; if more people take the train from Times Square compared to Far Rockaway-Mott Av, the weightage of time between trains at Times Square should take precedence over the weightage of time between trains at Far Rockaway. This is especially important because some trains short turn before reaching their terminus, making stations in the middle of their paths have more frequent service than stations at the ends of their paths.
 
This project also assumes that individuals are waiting for a specific line when in reality, various lines can bring an individual from their station of origin to their desired destination; thus, wait times are not necessarily an accurate reflection of how long a passenger need wait to get from their origin to destination. 

Another simplification made was that the time between trains at a station is an accurate proxy of wait time. This assumes that all passengers enter the station right after the previous train leaves. More mathematically intricate calculations will need to be performed to get a better estimate of wait time. 

One exclusion from this project was the shuttle lines, as they contained so few stops that any information that could be gleaned from them was extremely minimal. The Staten Island Railway was also ignored as it functionally serves more as suburban rail compared to rapid transit.

## Results and Observations
The following maps were generated for each line, showing the individual paths of trains that ran during AM and PM rush hours in both directions.
![7Av_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/907fa60d-0ade-4967-b757-2dfb90d1404d)
![LexAv_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/cd66e0c3-7de5-4c94-a3dc-dd9f5ad2b7e0)
![Flushing_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/d870ca4f-92e9-438d-9473-c24a1ad39e74)
![8Av_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/d9f25c9f-9543-4554-a15c-2da11beb9e2a)
![6Av_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/d6d2bfcb-f2b8-41a7-bdf2-0f4331cf3f78)
![Crosstown_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/cfb6f7c5-0fb9-444d-92a0-4524f638ccca)
![Canarsie_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/86288391-1a98-4b31-bd85-2ad1141bd845)
![Nassau_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/5260b145-1bf0-4993-aa06-108154461d22)
![Broadway_RealtimeData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/657a45bd-2261-4073-a033-8938d1c11d14)

The below histograms show the distribution of times between trains for specific lines (data combines all the stations that line serves).
![IRTIntervalData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/43b2800e-36d5-4d45-8ba0-9fd6d97fef17)
![INDBMTIntervalData_0219](https://github.com/saketpram/nyc-subway/assets/171081824/4d903372-c787-4841-a2af-8a2a72dd0bb7)

Based on these visualizations, we determined that
- Several (1) train trips begin at 238 St (so that they can short turn to the yard between 238 St and 241 St) and 137 St. The (4) train is also seen short turning before its terminus at Woodlawn at Burnside Av and bedford Pk Blvd
- (2) and (5) trains seem to have delays around E 180 St and Rogers Jct (between Franklin Av and President St), both of which are known chokepoints. We know there are delays because the slope for certain trips is much flatter than for other trips.
- The (N) train sometimes runs local, despite its normal route in Manhattan (can be spotted by the one or two yellow lines crossing every other line instead of running roughly parallel). (W) frequency is very low, but is supplemented by (N) train (considered an alternative of the (N)).
- The two (F) peak express trains in Brooklyn can be spotted (crossing over other (F) trains) 
- (M) train frequency has dropped significantly compared to usual as it is not running on Queens Blvd

We also generated the following heatmaps which were generated through pairwise Mann-Whitney U tests for lines.
![AMNorthbound_Heatmap_0219](https://github.com/saketpram/nyc-subway/assets/171081824/7abb4769-f21c-4d08-98c8-e5b1227734cf)
![PMNorthbound_Heatmap_0219](https://github.com/saketpram/nyc-subway/assets/171081824/b3a3520d-c63f-4946-af28-b34433850245)
![AMSouthbound_Heatmap_0219](https://github.com/saketpram/nyc-subway/assets/171081824/88b1a8b6-cb7e-4a1f-8e46-107b16e84c5e)
![PMSouthbound_Heatmap_0219](https://github.com/saketpram/nyc-subway/assets/171081824/500a4505-6aab-4eb4-8887-9c133360f5df)

In the heatmaps, certain p values were extremely small and thus rounded to 0. When performing a logarithmic scaling, 0 maps to negative infinity, which messes up dataframe filtering and visualization, so all squares that had a p value of 0 before logarithmic scaling were just assigned a black square with no label.

Finally, we created a boxplot to display the distribution of time between trains for all lines, as shown below.
![Boxplot_0219](https://github.com/saketpram/nyc-subway/assets/171081824/ef57acc8-81d8-4bdf-9b97-060e5165bd5d)

Lines that have their own dedicated track ((7), (6), (L), and (1)) have the best distribution of time between trains (there are less places for delays to have a domino effect; this is one reason that some advocates believe the NYC subway should be deinterlined). The (7) and (L) also have CBTC signaling (a new generation train signaling mechanism) throughout their entire line, along with the (E) and (F) on Queens Blvd. This allows for greater train frequencies, which may also explain why these lines presumably do better compared to others. Trains that are extensively interlined (particularly the (D), (Q), (N), (B), and (C)) tend to have worse time distributions, possibly due to cascading delays from one line to another. The (N) train has a minimum interval time of 0 seconds because presumably a local (N) and express (N) arrived at the same station at the same time (this can be seen on the (N) train path line graph).

The table below shows the Shapiro-Wilk Test p values for each distribution (given time of day and direction). Based on the normality of these distributions, we used the Mann-Whitney U test (if p is less than 0.05) or a two sample t-test (if p is greater than 0.05).
| Line | Morning Rush Hour (Northbound) | Evening Rush Hour (Northbound) | Morning Rush Hour (Southbound) | Evening Rush Hour (Southbound) |
|------|-------------------------------:|-------------------------------:|-------------------------------:|-------------------------------:|
| 1    | 6.57E-17                       | 1.74E-33                       | 5.30E-30                       | 6.28E-39                       |
| 2    | 5.47E-15                       | 6.04E-18                       | 1.39E-19                       | 1.43E-32                       |
| 3    | 7.18E-20                       | 9.42E-18                       | 4.61E-25                       | 5.45E-22                       |
| 4    | 2.80E-15                       | 8.25E-18                       | 3.24E-20                       | 5.68E-15                       |
| 5    | 1.50E-12                       | 1.07E-10                       | 1.56E-21                       | 3.56E-10                       |
| 6    | 1.32E-34                       | 0                              | 1.60E-36                       | 0                              |
| 7    | 1.65E-41                       | 1.19E-36                       | 2.06E-40                       | 2.88E-36                       |
| A    | 8.48E-30                       | 7.57E-34                       | 1.63E-25                       | 4.63E-34                       |
| C    | 1.18E-12                       | 2.21E-19                       | 4.87E-13                       | 5.50E-10                       |
| E    | 9.02E-25                       | 2.32E-14                       | 2.83E-18                       | 2.35E-23                       |
| B    | 1.42E-16                       | 7.74E-23                       | 9.15E-28                       | 1.51E-25                       |
| D    | 3.31E-22                       | 8.55E-22                       | 5.69E-15                       | 7.19E-16                       |
| F    | 3.99E-19                       | 6.22E-29                       | 3.69E-33                       | 4.66E-34                       |
| M    | 8.17E-15                       | 3.61E-18                       | 5.42E-07                       | 1.24E-08                       |
| G    | 3.41E-12                       | 2.18E-18                       | 1.66E-22                       | 3.17E-17                       |
| L    | 2.54E-30                       | 3.78E-44                       | 2.08E-26                       | 4.06E-42                       |
| J    | 1.50E-20                       | 2.36E-20                       | 6.46E-14                       | 2.51E-26                       |
| N    | 9.95E-09                       | 1.69E-12                       | 2.23E-17                       | 7.51E-09                       |
| Q    | 7.01E-14                       | 2.95E-13                       | 1.31E-14                       | 2.06E-14                       |
| R    | 2.67E-09                       | 3.90E-22                       | 2.42E-32                       | 3.42E-19                       |
| W    | 7.63E-16                       | 7.24E-11                       | 5.08E-23                       | 7.67E-11                       |

Based on the Shapiro-Wilk p values, we assume that it is False that all all the train interval distributions are normally distributed for all lines going northbound in morning rush hour.

Based on the Shapiro-Wilk p values, we assume that it is False that all all the train interval distributions are normally distributed for all lines going northbound in evening rush hour.

Based on the Shapiro-Wilk p values, we assume that it is False that all all the train interval distributions are normally distributed for all lines going southbound in morning rush hour.

Based on the Shapiro-Wilk p values, we assume that it is False that all all the train interval distributions are normally distributed for all lines going southbound in evening rush hour.

The ANOVA Test p value for all lines for morning rush hour and northbound, morning rush hour and southbound, evening rush hour and northbound, and evening rush hour and southbound is 0.0, 0.0, 0.0, and 0.0. We can conclude there is at least one pair of distributions in which the mean difference is statistically different.

The table below shows the mean wait time between trains in minutes for each line.

| Line | Mean Wait Time Between Trains (Min) |
|------|-------------------------------------|
| 7    | 2.91                                |
| L    | 4.06                                |
| 6    | 4.38                                |
| 1    | 4.61                                |
| E    | 5.12                                |
| F    | 5.52                                |
| 4    | 5.64                                |
| 2    | 6.86                                |
| A    | 6.90                                |
| R    | 7.02                                |
| 5    | 7.06                                |
| Q    | 7.21                                |
| 3    | 7.33                                |
| G    | 7.89                                |
| N    | 7.98                                |
| J    | 8.07                                |
| D    | 8.13                                |
| B    | 8.80                                |
| C    | 9.50                                |
| M    | 10.00                               |
| W    | 10.07                               |

One major issue we encountered throughout this project was with the dataset itself. Oftentimes, the dataset was organized intuitively. Stop IDs for a line were not ordered properly, for instance. Some numbers are skipped for no reason (for instance, L09 is not a stop on the L train). Furthermore, because some trains share certain stops, they only go by one stop ID (for instance, G01 is Jamaica Center-Parsons/Archer, which is where the E, J, and Z trains stop). We had to manually go in and create the train paths for each train in cases where the stop IDs did not necessarily correspond to the line of the train.

The IDs for each train also did not make sense at times. One example of this was that all the Queens Blvd Lines have a train ID of the form L0S1… to represent a weekday train while the other train lines have the word “Weekday” explicitly written in the ID. Furthermore, the N and W trains were combined (as internally, they are considered the same, although we wanted to highlight their distinction), so we had to figure out which trips corresponded to the N and which to the W. These minor issues made data validation and filtering somewhat difficult.

Initially, we wanted to focus on waiting time, but because this was not very easy to calculate, we ended up switching over to train time intervals at stations.

We also noticed that when we first created our heatmaps, it was very hard to pick out differences because the p values were so small, so we made the decision to logarithmically scale them. Adding images to the plots was difficult as well: we learned that OpenCV uses BGR, while Matplotlib uses RGB, so we had to convert between the two systems.

We initially also planned to do machine learning, exploring the effect of weather on subway ridership at stations, but we decided to leave this for another time as the current project took up a lot more time than expected.

In the future, we hope to do the same data analysis for a larger period of time; we only collected data for one day, but collecting data across a period of time, say three months, could provide even more accurate insights into train interval times, average waiting time, and the average path of a trip of a given train. It could also provide more insight into where delays tend to occur, and these locations can be highlighted. We also want to take into account the number of individuals that use a certain station or line and take a weighted average rather than naively treating all stations equally, as average waiting time is certainly affected by the number of passengers at one station versus another.
