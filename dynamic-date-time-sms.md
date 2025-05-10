Posted: 2025-05-07  

The evolution of this AMPScript came about from a need to shift from manual SMS template updating to a less manual and more automated template. In previous seasons for flex programs (get customers to use less electricity), we created a number of AM and PM messages for winter and summer messaging. When the program manager called a Flex event where the time range of the event varied from 4-7pm, 5-8pm, 7-10am or 8-11am, these changes had to be manually made to update the time range in each channel type notification.

To address this, a standalone data extension was created to store the event date, event start time, event end time, event theme, and event status.

> [!NOTE]
> A SFMC user should be able to simply update fields in the Event_Dates data extension as the only input for updating the AMPScript-driven SMS or Email templates.

**A standalone editable data extension (Event_Dates) with the following setup:**

| Event_ID | EventType | EventTheme | EventStart | EventEnd | EventStatus |  
| --- | --- | --- | --- | --- | --- |  
| 1 | PTR | Standard | 2025-08-01 17:00:00.000 |	2025-08-01 20:00:00.000	| Frozen |  
| 2 | PTR | HighHeat | 2025-08-01 16:00:00.000 |	2025-08-01 19:00:00.000	| Active |  
| 3 | PTR | ExtremeCold | 2025-08-01 08:00:00.000 |	2025-08-01 11:00:00.000	| Frozen |  
| 4 | PTR | Standard | 2025-07-01 17:00:00.000 |	2025-08-01 20:00:00.000	| Active |  
| 5 | PTR | Standard | 2025-07-03 17:00:00.000 |	2025-08-01 20:00:00.000	| Scheduled |  
| 6 | PTR | HighHeat | 2025-07-06 16:00:00.000 |	2025-08-01 19:00:00.000	| Scheduled |  

Event_ID is a primary key field. Without a unique ID field, these rows would not be editable.

For an active message to display in an SMS, both the EventType and EventStatus row field needs to have an "Active" EventStatus.

**This example displays the date & time range based on the EventType and "Active" EventStatus in an SMS:**

~~~
%%[VAR @rows, @row, @eventStart, @eventEnd, @SMSformattedDate, @ActiveStartHour, @ActiveEndHour, @ActiveEventStartHour, @ActiveEventEndHour, @ActivetimeRange
/* Lookup the row where eventtheme is "standard" and eventstatus is "active" */
SET @rows = LookupRows("event_dates", "eventtheme", "standard", "eventstatus", "active")
/* Retrieve the first row from the results */
IF RowCount(@rows) > 0 THEN
    SET @row = Row(@rows, 1)

    /* Retrieve the eventstart and eventend fields */
    SET @eventStart = Field(@row, "eventstart")
    SET @eventEnd = Field(@row, "eventend")
    /* Format the eventstart date */
    SET @SMSformattedDate = FormatDate(@eventStart, "m/d")
    /* Extract and format the hours for eventstart and eventend */
    SET @ActiveStartHour = FormatDate(@eventStart, "h tt")
    SET @ActiveEndHour = FormatDate(@eventEnd, "h tt")
    /* Combine the hours for the time range */
    SET @ActivetimeRange = Concat(@ActiveStartHour, " to ", @ActiveEndHour)
ENDIF
]%%Get ready! Tomorrow, %%=v(@SMSformattedDate)=%% is a Peak Time Rebates event from %%=v(@ActivetimeRange)=%%. Earn a rebate on your bill by reducing energy use. Reply STOP to opt out.
~~~

The SMS message displays to the customer as:

> Get ready! Tomorrow, 7/1 is a Peak Time Rebates event from 5 PM to 8 PM. Earn a rebate on your bill by reducing energy use. Reply STOP to opt out.

Our current cadence is to send a Day Before message a day before the Day Of message. 

**This example displays the date & time range plus scheduled events in an email:**

~~~
<!--BEGIN HEADER-->
%%[
/* Declare all variables */
VAR @rows, @row, @eventStart, @eventEnd, @formattedDate
VAR @ActiveStartHour, @ActiveEndHour, @ActiveEventStartHour, @ActiveEventEndHour
VAR @timeRange, @contentBlockID
VAR @scheduledRows, @scheduledRowCount, @scheduledList
VAR @scheduledEventStart, @scheduledEventEnd, @scheduledStartHour, @scheduledEndHour, @scheduledTimeRange
VAR @formattedScheduledDate, @i

/* Lookup the row where eventtheme is "standard" and eventstatus is "active" */
SET @rows = LookupRows("event_dates", "eventtheme", "standard", "eventstatus", "active")

/* Retrieve the first row from the results */
IF RowCount(@rows) > 0 THEN
  SET @row = Row(@rows, 1)

  /* Retrieve the eventstart and eventend fields */
  SET @eventStart = Field(@row, "eventstart")
  SET @eventEnd = Field(@row, "eventend")

  /* Format the eventstart date for subject line and email body */
  SET @formattedDate = FormatDate(@eventStart, "ddddd, MMMM d, yyyy")

  /* Extract and format the hours for eventstart and eventend */
  SET @ActiveStartHour = FormatDate(@eventStart, "h tt")
  SET @ActiveEndHour = FormatDate(@eventEnd, "h tt")

  /* Replace "AM" and "PM" with custom format */
  SET @ActiveEventStartHour = Replace(@ActiveStartHour, "AM", "a.m.")
  SET @ActiveEventStartHour = Replace(@ActiveEventStartHour, "PM", "p.m.")
  SET @ActiveEventEndHour = Replace(@ActiveEndHour, "AM", "a.m.")
  SET @ActiveEventEndHour = Replace(@ActiveEventEndHour, "PM", "p.m.")

  /* Combine the hours for the time range */
  SET @timeRange = Concat(@ActiveEventStartHour, " to ", @ActiveEventEndHour)


ENDIF

/* Lookup all rows where eventstatus is "Scheduled" */
SET @scheduledRows = LookupRows("event_dates", "eventstatus", "Scheduled")

/* Get the number of scheduled rows */
SET @scheduledRowCount = RowCount(@scheduledRows)

/* Initialize the scheduled events list */
SET @scheduledList = ""

IF @scheduledRowCount > 0 THEN
  FOR @i = 1 TO @scheduledRowCount DO
    SET @row = Row(@scheduledRows, @i)

    /* Retrieve scheduled eventstart and eventend fields */
    SET @scheduledEventStart = Field(@row, "eventstart")
    SET @scheduledEventEnd = Field(@row, "eventend")

    /* Format the date and time */
    SET @formattedScheduledDate = FormatDate(@scheduledEventStart, "MMMM d, yyyy")
    SET @scheduledStartHour = FormatDate(@scheduledEventStart, "h tt")
    SET @scheduledEndHour = FormatDate(@scheduledEventEnd, "h tt")

    /* Customize AM/PM format */
    SET @scheduledStartHour = Replace(@scheduledStartHour, "AM", "a.m.")
    SET @scheduledStartHour = Replace(@scheduledStartHour, "PM", "p.m.")
    SET @scheduledEndHour = Replace(@scheduledEndHour, "AM", "a.m.")
    SET @scheduledEndHour = Replace(@scheduledEndHour, "PM", "p.m.")

    /* Combine time range */
    SET @scheduledTimeRange = Concat(@scheduledStartHour, " to ", @scheduledEndHour)

    /* Add to the list */
    SET @scheduledList = Concat(@scheduledList, "", @formattedScheduledDate, " | ", @scheduledTimeRange, "<br>")
  NEXT @i
ENDIF

/* Add custom header block */
SET @contentBlockID = 1367180 /* Content Builder PTR Event Generic Day Of Header ENG */

]%%
~~~

In the email body, the AMPScript is formatted like this:

> **Event**
> 
> %%=v(@formattedDate)=%% %%=v(@timeRange)=%%  
> 
> %%[IF @scheduledRowCount > 0 THEN]%%  
> 
> **Upcoming Events**  
>
> %%=v(@scheduledList)=%%  
> %%[ENDIF]%%

What is displayed to the customer is this:  

> **Event**
> 
> Tuesday, July 1, 2025
> 5 p.m. to 8 p.m.
> 
> **Upcoming Events**
> 
> July 3, 2025 | 5 p.m. to 8 p.m.\
> July 6, 2025 | 4 p.m. to 7 p.m.

If you are wondering why the email version displays the time as a.m./p.m. while the SMS displays AM/PM, the former is a brand format, the latter is to save on the character count to be under the 160 characters for an US-sms message.
