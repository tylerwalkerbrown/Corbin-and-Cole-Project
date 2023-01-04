# Corbin-and-Cole-Project
- Details:
This script is campable of calculating the probability of spin rate and velocity for Corbin Burnes or Gerrit Cole. Throughout this project I had to:
 - Pull data from statcast database
 - Create database for pitchers data to be stored
 - Set up connection using an api from MySQLWorkbench to python 
 - Create dynamic queries based on:
   - Pitch type 
   - Pitcher 
   - Velocity 
   - Spin Rate 
   
 The data had to be pulled from statcast into SQL bench using an api from MySQL. When the database is created I then took the script and started to format queries to get custom reports on the probability of a specific instance using the instance minus the mean to then divide by the standard deviation to get the z-score. With that Z-score you can pass it through scipy.stats.norm.sf() to get the probabilty of the observation. One example of the functional use is that I was able to findout that a spin rate of 2500 had a 8.34% chance for Corbin Burns. I investigated his lowest performing spin rate game and found a vast majority of his pitches had a spin rate P >.05 meaning that there were few pitches that had a major drop in spin rate over the course of the season. This application can also be used to test data against eachother (AB Testing).  
 
 Goal of project:
 - showcase etl process using SQL, Python and Web API
 - probability testing fundamentals 
 - AB testing
 - Documentation
 - Statistics
 
![output_66_1](https://user-images.githubusercontent.com/94020684/209399320-03e2b95d-749f-43c2-b78b-4e82891ad9ee.png)
![output_65_1](https://user-images.githubusercontent.com/94020684/209399321-1e155a6f-cad6-43a1-be1b-89eedcb24aa5.png)
![output_49_1](https://user-images.githubusercontent.com/94020684/209399322-7e64ace9-c74f-46b6-9d59-3d418bf30ede.png)
![output_48_1](https://user-images.githubusercontent.com/94020684/209399323-162e08a5-5888-4226-8453-c09db494b8d9.png)

This series of pitches was tracked from the first to the last pitch thrown in the searson for Corbin Burns. You can see that all the pitches have a somewhat consistent pattern except for a sudden drop in spin rate. I figured that this would potentially cause a loss in velocity as well but it did not seem to be the case. 


![image](https://user-images.githubusercontent.com/94020684/210619929-0a359cad-4bc0-465b-993a-401a1dd0e665.png)

![output_33_1](https://user-images.githubusercontent.com/94020684/209399324-abf59fb6-2c67-49bd-bcbe-e67933aed849.png)
