#salesforcemarketingcloud #sfmc #lookup #dataextension #lookuprows #formatlookup

This is how to format a date when it's inserted into an email template using AMPScript but the data 
is in a different data extension than the one the email uses in a journey automation.

The AMPScript is placed at the top of an email template and can be placed before the first email 
body block. What you should do is have your code saved as a code snippet in SFMC and just add it
in each time an email template needs it.

(top of email)

~~~
%%[ 
VAR @psps1, @Outage_Start_Earliest_Date, @Outage_Start_Earliest_Time, @psps, @Restore_Earliest_Date, @Restore_Earliest_Time, @Restore_Latest_Date, @Restore_Latest_Time, @Conditions
SET @psps = AttributeValue('psps') 
SET @psps1 = AttributeValue('PSPSName')  
SET @Conditions = lookup('PSPS_Zone_time','Conditions','zone',@psps)
SET @Outage_Start_Earliest_Date = lookup('PSPS_Zone_time','Outage_Start_Earliest_Date','zone',@psps)
SET @Outage_Start_Earliest_Time = lookup('PSPS_Zone_time','Outage_Start_Earliest_Time','zone',@psps) 
SET @Outage_Start_Latest_Date = lookup('PSPS_Zone_time','Outage_Start_Latest_Date','zone',@psps) 
SET @Outage_Start_Latest_Time = lookup('PSPS_Zone_time','Outage_Start_Latest_Time','zone',@psps) 
SET @Restore_Earliest_Date = lookup('PSPS_Zone_time','Restore_Earliest_Date','zone',@psps) 
SET @Restore_Earliest_Time = lookup('PSPS_Zone_time','Restore_Earliest_Time','zone',@psps) 
SET @Restore_Latest_Date = lookup('PSPS_Zone_time','Restore_Latest_Date','zone',@psps) 
SET @Restore_Latest_Time = lookup('PSPS_Zone_time','Restore_Latest_Time','zone',@psps) 
]%%
~~~

(use dynamic placement in email body)

~~~
%%=Format(@Outage_Start_Earliest_Date,"MMM d")=%%
~~~

That's it.
