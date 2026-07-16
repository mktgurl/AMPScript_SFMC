This is a technique that I use with a few critical and program email sends in order to capture 
dyanmic field values to a standalone data extension. Because the SubscriberKey and Email address
values are picked up, the data can be used to preview text in an email template that has these
dynamic fields in the email body. In the SFMC link expiration configuration at this utility company,
SFMC links persist for 730 days (2 years). Any user can copy/paste the view_email_url hyperlink
into a browser window and view a fully rendered email that was sent to a customer. There are many 
system table fields that can be pulled into the data extension, including the Journey Name. 

Here is one example of how it works:

The standalone data extension for the AMPScript field value capture needs to have field names that
are of an appropriate field format and have matching field names called by the AMPScript.

An preview email test send should show up in the logging data extension as "NotFromJourneyBuilder"
in the JourneyName field, along with the customer's dynamic data used for the preview.

The data in the source data extension (used for sending the emails) comes from Snowflake. Most of
the data is pre-calculated in Snowflake. Some fields represented here are calculated using SFMC
SQL, such as the variable text field "moreorless" where the customer is being told that they are
saving "more than", "less than", or "the same as". Other post-processing calculated values are
the bar chart bar lengths "actualwidth1" and 'actualwidth2".

Sample **logging data extension** field names (comma delimited):
~~~
PER_ID,Email,SA_ID,Total_Previous_Months_Bill_TOD,Total_Basic_Cost_of_Service,On_Peak_kWh,Mid_Peak_kWh,Off_Peak_kWh,Delta_TOD_vs_Basic_Cost_of_Service,Basic_Service_Savings,Overall_Savings_on_TOD_since_Enrollment,Monthly_Percent_of_On_Peak_Usage,Monthly_Percent_of_On_Peak_Usage_Prior_Month,Last_Bill_Date,Percent_of_Savings_for_TOD,CurrentBillDelta,totalkwh,midoffpeakkwh,actualwidth1,actualwidth2,moreorless,ACCT_ID,view_email_url,SentDate,Template,JobID,SendType,JourneyName,IsTestSend
~~~

Data extension fields of the **sending data extension** (comma delimited):
~~~
PER_ID,Email,SA_ID,Total_Previous_Months_Bill_TOD,Total_Basic_Cost_of_Service,On_Peak_kWh,Mid_Peak_kWh,Off_Peak_kWh,Delta_TOD_vs_Basic_Cost_of_Service,Basic_Service_Savings,Overall_Savings_on_TOD_since_Enrollment,Monthly_Percent_of_On_Peak_Usage,Monthly_Percent_of_On_Peak_Usage_Prior_Month,Last_Bill_Date,Percent_of_Savings_for_TOD,CurrentBillDelta,totalkwh,midoffpeakkwh,actualwidth1,actualwidth2,moreorless,SQL_LastRunDate,SP_ID,ACCT_ID
~~~

PER_ID, ACCT_ID, and Email values have been sanitized using Excel formulas to mask text in this **logging export sample** of the logging data extension (tab delimited):  
~~~
PER_ID	Email	SA_ID	Total_Previous_Months_Bill_TOD	Total_Basic_Cost_of_Service	On_Peak_kWh	Mid_Peak_kWh	Off_Peak_kWh	Delta_TOD_vs_Basic_Cost_of_Service	Basic_Service_Savings	Overall_Savings_on_TOD_since_Enrollment	Monthly_Percent_of_On_Peak_Usage	Monthly_Percent_of_On_Peak_Usage_Prior_Month	Last_Bill_Date	Percent_of_Savings_for_TOD	CurrentBillDelta	totalkwh	midoffpeakkwh	actualwidth1	actualwidth2	moreorless	ACCT_ID	view_email_url	SentDate	Template	JobID	SendType	IsTestSend
##########	*****@abouaf.com	##########	100	109	55	167	280	9	0	9	11	0	4/7/2026 0:00	8	9	502	447	11	89	more than	##########	https://view.em.portlandgeneral.com/?vawpToken=YVSWNHVJH43EPPBNWS67T3K2MM.10197	4/8/2026 11:04	2026-03-tod-monthly-report-email-rateupdate-saver-tod-tr	31283833	SEND	FALSE
##########	*****@comcast.net	##########	130	154	65	150	496	24	0	166	9	9	4/7/2026 0:00	16	24	711	646	9	91	the same as	##########	https://view.em.portlandgeneral.com/?vawpToken=U6WVXVVBI33EZH6MUHDT5PWXYU.10197	4/8/2026 11:04	2026-03-tod-monthly-report-email-rateupdate-saver-tod-tr	31283833	SEND	FALSE
##########	*****@gmail.com	##########	70	75	39	93	185	5	0	47	12	14	4/7/2026 0:00	7	5	317	278	12	88	less than	##########	https://view.em.portlandgeneral.com/?vawpToken=MGYYXDJNFWPE5HSKAFQTU353BY.10197	4/8/2026 11:04	2026-03-tod-monthly-report-email-rateupdate-saver-tod-tr	31283833	SEND	FALSE
~~~

This is the AMPScript that drives it all:  
~~~
%%[
VAR @PER_ID, @Email, @SA_ID, @Total_Previous_Months_Bill_TOD, @Total_Basic_Cost_of_Service, 
    @On_Peak_kWh, @Mid_Peak_kWh, @Off_Peak_kWh, @Delta_TOD_vs_Basic_Cost_of_Service, 
    @Basic_Service_Savings, @Overall_Savings_on_TOD_since_Enrollment, 
    @Monthly_Percent_of_On_Peak_Usage, @Monthly_Percent_of_On_Peak_Usage_Prior_Month, 
    @Last_Bill_Date, @Percent_of_Savings_for_TOD, @CurrentBillDelta, @totalkwh, 
    @midoffpeakkwh, @actualwidth1, @actualwidth2, @moreorless, @ACCT_ID, 
    @view_email_url, @SentDate, @Template, @JobID, @IsTestSend, @SendType, @JourneyName

/* Get values from the send context (source DE fields) */
SET @PER_ID = AttributeValue("PER_ID")
SET @Email = AttributeValue("Email")
SET @SA_ID = AttributeValue("SA_ID")
SET @Total_Previous_Months_Bill_TOD = AttributeValue("Total_Previous_Months_Bill_TOD")
SET @Total_Basic_Cost_of_Service = AttributeValue("Total_Basic_Cost_of_Service")
SET @On_Peak_kWh = AttributeValue("On_Peak_kWh")
SET @Mid_Peak_kWh = AttributeValue("Mid_Peak_kWh")
SET @Off_Peak_kWh = AttributeValue("Off_Peak_kWh")
SET @Delta_TOD_vs_Basic_Cost_of_Service = AttributeValue("Delta_TOD_vs_Basic_Cost_of_Service")
SET @Basic_Service_Savings = AttributeValue("Basic_Service_Savings")
SET @Overall_Savings_on_TOD_since_Enrollment = AttributeValue("Overall_Savings_on_TOD_since_Enrollment")
SET @Monthly_Percent_of_On_Peak_Usage = AttributeValue("Monthly_Percent_of_On_Peak_Usage")
SET @Monthly_Percent_of_On_Peak_Usage_Prior_Month = AttributeValue("Monthly_Percent_of_On_Peak_Usage_Prior_Month")
SET @Last_Bill_Date = AttributeValue("Last_Bill_Date")
SET @Percent_of_Savings_for_TOD = AttributeValue("Percent_of_Savings_for_TOD")
SET @CurrentBillDelta = AttributeValue("CurrentBillDelta")
SET @totalkwh = AttributeValue("totalkwh")
SET @midoffpeakkwh = AttributeValue("midoffpeakkwh")
SET @actualwidth1 = AttributeValue("actualwidth1")
SET @actualwidth2 = AttributeValue("actualwidth2")
SET @moreorless = AttributeValue("moreorless")
SET @ACCT_ID = AttributeValue("ACCT_ID")

/* System values */
SET @SentDate = FormatDate(Now(), "yyyy-MM-dd hh:mm tt")
SET @Template = emailname_
SET @JobID = JobID
SET @view_email_url = view_email_url
SET @SendType = _messagecontext
SET @IsTestSend = _IsTestSend

/* Get Journey Name */

SET @JourneyName =
Lookup(
    "_Journey",
    "JourneyName",
    "VersionID",
    Lookup(
        "_JourneyActivity",
        "VersionID",
        "JourneyActivityObjectID",
        Lookup(
            "_Job",
            "TriggererSendDefinitionObjectID",
            "JobID",
            JobID
        )
    )
)

IF Empty(@JourneyName) THEN
    SET @JourneyName = "NotFromJourneyBuilder"
ENDIF

    /* Insert record into logging DE */
    InsertDE("TODMonthlyDynamicJourneyLog",
        "PER_ID", @PER_ID,
        "Email", @Email,
        "SA_ID", @SA_ID,
        "Total_Previous_Months_Bill_TOD", @Total_Previous_Months_Bill_TOD,
        "Total_Basic_Cost_of_Service", @Total_Basic_Cost_of_Service,
        "On_Peak_kWh", @On_Peak_kWh,
        "Mid_Peak_kWh", @Mid_Peak_kWh,
        "Off_Peak_kWh", @Off_Peak_kWh,
        "Delta_TOD_vs_Basic_Cost_of_Service", @Delta_TOD_vs_Basic_Cost_of_Service,
        "Basic_Service_Savings", @Basic_Service_Savings,
        "Overall_Savings_on_TOD_since_Enrollment", @Overall_Savings_on_TOD_since_Enrollment,
        "Monthly_Percent_of_On_Peak_Usage", @Monthly_Percent_of_On_Peak_Usage,
        "Monthly_Percent_of_On_Peak_Usage_Prior_Month", @Monthly_Percent_of_On_Peak_Usage_Prior_Month,
        "Last_Bill_Date", @Last_Bill_Date,
        "Percent_of_Savings_for_TOD", @Percent_of_Savings_for_TOD,
        "CurrentBillDelta", @CurrentBillDelta,
        "totalkwh", @totalkwh,
        "midoffpeakkwh", @midoffpeakkwh,
        "actualwidth1", @actualwidth1,
        "actualwidth2", @actualwidth2,
        "moreorless", @moreorless,
        "ACCT_ID", @ACCT_ID,
        "view_email_url", @view_email_url,
        "SentDate", @SentDate,
        "Template", @Template,
        "JobID", @JobID,
        "SendType",@SendType,
        "IsTestSend",@IsTestSend,
        "JourneyName",@JourneyName
    )
]%%
~~~

Instead of display this wall of AMPScript, I am calling it into the email template using this:  
~~~
%%[IF _IsTestSend == false THEN VAR @contentBlockID SET @contentBlockID = ######]%% %%=ContentBlockByID(@contentBlockID)=%% %%[ENDIF]%%
~~~

This specific email series (Time of Day's Monthly Report email for Savers, Cusp savers, and non Savers) also contains the 
Holiday AMPScript that displays the next two upcoming holiday dates for the TOD program.

These are the SFMC system fields that can be added to a "logging" data extension:

| Field Name | System Field |
|:------------|:-------------|
| SubscriberKey | _subscriberkey |
| EmailStatus | Lookup("ENT._Subscribers", "Status", "SubscriberKey", _subscriberkey) |
| SendClassification | AttributeValue("_messageContext") |
| EmailName | emailname_ |
| SubjectLine | _emailSubject |
| Preheader | _emailPreHeader |
| DataSourceName | _DataSourceName |
| SentDate | FormatDate(Now(), "yyyy-MM-dd hh:mm tt") |
| JobID | _JobID |
| IsTestSend | _IsTestSend |

