# Time Zones in Power Apps
In 2023, More and More companies are taking advantage of the Distributed workforce and being able to hire in almost all 50 States of the US. This represents a problem in Power Apps when you have multiple users using a app in different time zones. This is not inherintely a problem in Power Apps as it has the ability to automatically adjust the time from UTC to whatever the user's local settings are. But it becomes a problem when the Business requires that the App and and Date Time Value are saved in a Specific Time Zone. 

## How do you Dynamically save Date Time Values to a Specific Time Zone?
This problem sent me down about three days of investigation, troubleshooting and developing to develop an internal solution to this problem. 

**Business Requirements**

 1. Date Time Values Need to be Stored in PST Regardless of TimeZone
		*We have users in multiple Time Zones, PST,MST, CST and EST*
 2. Date Time Values Need to be displayed in PST Regardless of TimeZone
 3. Calendar events need to be created in PST


**Problem Statements**
 1. Power Apps Automatically Converts DateTimeValues to user's locale Settings.
 2. Power Apps will Save Any Input Time as the User's locale Settings Converted to UTC. 
 3. You cannot select a Specific Locale Setting for the user or set default. 
 
## Solution
When I first began work
