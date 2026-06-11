2026-06-11, built this for a marketing user who had 4 email templates that 1 email template could achieve. Basically, Group B had a different start and end time range, 5 pm to 8 pm, and Group C had a different start and end time range, 10 pm to 12 am. There are three content areas in the email handled by dynamic AMPScript.

The Content Blocks were created and saved as HTML content blocks.

Here is the AMPScript:

~~~
%%[
SET @eventGroup = Trim(AttributeValue("Event Group"))

/* Determine content block */
IF IndexOf(@eventGroup, "WeaveGrid") > 0 THEN
  SET @contentBlockId = ######6
ELSEIF IndexOf(@eventGroup, "EVSE") > 0 THEN
  SET @contentBlockId = ######9
ENDIF

/* Determine times */
IF Substring(@eventGroup, 1, 7) == "Group B" THEN
  SET @startTime = "5 p.m."
  SET @endTime = "8 p.m."

ELSEIF Substring(@eventGroup, 1, 7) == "Group C" THEN
  SET @startTime = "10 p.m."
  SET @endTime = "12 a.m."
ENDIF

/* Participation bullets */
IF IndexOf(@eventGroup, "WeaveGrid") > 0 THEN
  SET @bulletBlockId = ######8
ELSEIF IndexOf(@eventGroup, "EVSE") > 0 THEN
  SET @bulletBlockId = #####89
ENDIF
]%%
~~~

ChatGPT suggested using a different name for the ContentBlockID for the second block set. This works great. These are the possible groups:

Group B - EVSE  
Group B - WeaveGrid  
Group C - EVSE  
Group C - WeaveGrid  

**For WeaveGrid customers, the intro paragraph displayed is:**

~~~
Here’s your friendly reminder that we’re continuing to call Smart Charging Events this month. At the beginning of each event, your car’s charging will pause and then resume at the end of the event depending on your managed charging schedule with WeaveGrid.
~~~

**For EVSE customers, the intro paragraph is:**

~~~
Here’s your friendly reminder that we’re continuing to call Smart Charging Events this month. At the beginning of each event, your car’s charging will pause and then resume at the end of the event.
~~~

**Group B's start time and end time:**  
Start time: Every weekday at 5 p.m. Pacific Time  
End time: Every weekday at 8 p.m. Pacific Time  

**Group C's start time and end time:**  
Start time: Every weekday at 10 p.m. Pacific Time  
End time: Every weekday at 12 a.m. Pacific Time  

**WeaveGrid Bulleted List:**

~~~
	• Participate in at least 3 Smart Charge events
	• Charge your car at least 13 times during this season
~~~

**EVSE Bulleted List:**

~~~
	• Participate in at least 3 Smart Charging Events.
	• Keep your charger connected to the internet 50% of the time.
	• Charge your car at least 13 times during this season.
~~~

These items are called into the email body using this AMPScript:

**The first content block (intro paragraph):** %%=ContentBlockByID(@contentBlockId)=%%

**Start time:** Every weekday at %%=v(@startTime)=%% Pacific Time  
**End time:**  Every weekday at %%=v(@endTime)=%% Pacific Time  

**The bulleted list content block:** %%=ContentBlockByID(@bulletBlockId)=%%
