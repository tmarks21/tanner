---
  title: "Time Zone In Power Apps"
  tags:
    - "Power Apps"
    - "M365 Power Platform"
    - "Development"
  categories:
    - Development
  toc: true
---
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
When I first began working on this issue, I had assumed it was going to be an easy fix to make it dynamic, I was incredibly wrong. For this solution I am building with a Custom Component within a Component Library. For this example I am only solving for U.S Time Zones, but I believe it could be expanded to other time zones. 

### Step 1  - Defining Time Zones
One Issue I ran into right away was that there is no way to truly get a Named Time zone and view other timezones other than your own, so first step was to define the Time Zones. 

In the Component I defined an Output Property with a Record Data Type to host the Time zone Definitions.
This Property is called **Definitions or Def for short**

    {
            Pacific:{
                Standard:8,
                Daylight:7,
                Label:"Pacific"
            },
            Mountain:{
                Standard:7,
                Daylight:6,
                Label:"Mountain"
            },
            Central:{
                Standard:6,
                Daylight:5,
                Label:"Central"
            },
            Eastern:{
                Standard:5,
                Daylight:4,
                Label:"Eastern"
            }
    }

Now that **Definitions** has been defined, we can define a Output Property with a Table Data Type called **Time Zone Tables or TZT  for short**

    With({t:Self.Def},
            Table(
                {
                    Label:t.Pacific.Label,
                    DST: false,
                    Value:t.Pacific.Standard
                },
                {
                    Label:t.Pacific.Label,
                    DST: true,
                    Value:t.Pacific.Daylight
                },
                {
                    Label:t.Mountain.Label,
                    DST: false,
                    Value:t.Mountain.Standard
                },
                {
                    Label:t.Mountain.Label,
                    DST: true,
                    Value:t.Mountain.Daylight
                },
                {
                    Label:t.Central.Label,
                    DST: false,
                    Value:t.Central.Standard
                },
                {
                    Label:t.Central.Label,
                    DST: true,
                    Value:t.Central.Daylight
                },
                {
                    Label:t.Eastern.Label,
                    DST: false,
                    Value:t.Eastern.Standard
                },
                {
                    Label:t.Eastern.Label,
                    DST: true,
                    Value:t.Eastern.Daylight
                }
            )
          )

Now we have a Record Property that we can use as a enum to call specific Time Zones, and we have Table Property that can preform lookups and etc. 

### Step 2 - Helper Functions
Now that we have everything defined, I built a few helper functions that can be called to ensure that everything is consistent. 

 1. IsDayLightSavingsTime()
 2. GetMyTimeZoneName()
 3. GetTimeZoneOffset()
 4. ConvertToUTC()

#### IsDayLightSavingsTime(_DateTimeValue)
This is an Output Function Property with a Boolean Output. The Purpose of this function is to test a DateTimeValue's Current Offset Value to see if it falls within daylight Savings Time or Standard Time. 

    Switch(TimeZoneOffset(_DateTimeValue),
		TimeZoneOffset(Date(Year(_DateTimeValue),1,1)),false,
		TimeZoneOffset(Date(Year(_DateTimeValue),8,1)),true
	)

#### GetMyTimeZoneName()
This is an Output Function Property with Text Output. The Purpose of this function is to return a Text Value that display what time zone they are in in the Display Name format. 

	    If(
			Self.isDST(Now()),
		    //Switch based on the Daylight Savings time Values for UTC Offset
		    Switch(TimeZoneOffset()/60,
			    4,"Eastern",
			    5,"Central",
			    6,"Mountain",
			    7,"Pacific"
		    )
		    &" Daylight Time"
		    ,

		    //Switch Based on Standard time values for UTC Offset
		    Switch(TimeZoneOffset()/60,
			    5,"Eastern",
			    6,"Central",
			    7,"Mountain",
			    8,"Pacific"
		    )
		    &" Standard Time"
	)
#### GetTimeZoneOffset(_TimeZone)
This is a output Function Property with a Number Output. The Purpose of this function is to supply a Text Value that can be lookedup in the TZT to retreive the current Offset based on DST. Optionally can supply the text value "local" to retrieve local time offset. 

    If(
	    _TimeZone = "local",
	    TimeZoneOffset()/60,
	    LookUp(Self.TZT,Label=_TimeZone&&DST = Self.isDST(Now()) ,Value)
	)

#### ConvertToUTC(_DateTimeValue)
This is a output function property with a DateTimeValue Output. The purpose of this function is to get the UTC based on the DateTimeValue Supplied. 

    DateAdd(_DateTimeValue,TimeZoneOffset(_DateTimeValue)/60,TimeUnit.Hours)

### Step 3 - Conversion Function
Now is the time for the the final solution, There exsists two scenarios in which a user may need to convert timezones. 

    DateAdd(
    _DateTimeValue,
    -(_DestinationTimeZone - _SourceTimeZone),
    TimeUnit.Hours
	)

 1. Convert DateTimeValue to a Specific TimeZone. 
 2. Convert a DateTimeValue from Destination Time Zone to Source Time Zone. 

#### Convert DateTimeValue to Specific TimeZone
For This Scenario: Need to Convert a Time That may have been saved in Any Time Zone from Local Time Zone to Specific Destination Time Zone. 

So there are probably easier methods for doing this, but this is what I found for our scenarios. 

	// This would be executed like so:
	// My Time: 11/16 10AM MST // MST = 7
	// Expected Time: 11/16 9AM PST // PST = 6
	// ConvertTimeZone(_DateTimeValue,6,7)
	
#### Convert DateTime Value
For this Scenario: Need to Convert a Time so that it is saved in a specific time zone no matter the local time zone of the user who is saving it. 

    // This would be executed like so: 
    // My Time 11/16 10AM MST // MST = 7
    // Expected Time to Save: 11/16 10AM PST // PST = 6
    // Expected MST Time to Accound for Offset: 11AM MST // MST = 7
    // ConvertTimeZone(_DateTimeValue,7,6)

### Extra : US World Clocks
For this, I thought it would be cool to show a gallery that shows what time it is in each time zone based on a specific time. 

    AddColumns(
	    Filter(
	        Self.TZT,DST = Self.isDST(Coalesce(_DateTimeValue,Now()))
	    ),
	    "convertedTime",
	    Self.ConvertTimeZone(_DateTimeValue,Value,_SourceTimeZone)
	)

## Final Thoughts
Timezones and Time functions in powerapps seems like it would be an easy solve, but in reality this took about 3 days to come to a solution that should have been a little easier. 
