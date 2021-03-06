This is a mixed "code" example using both AMPScript and HTML within the body of an email template.
In the beginning.. declare all your data extension fields and temporary fields with VAR and SET.
The example here is for utility rates, but it can easily be adapted for setting up coupon savings
for customers or showing other types of consumption (e.g., grocery store products, cappucinos
sold, most popular drinks, etc.)

At the top of the email template (and, add this as a code snipppet within SFMC):

~~~
%%[ 

VAR @Total_Previous_Months_Bill_TOD, @Total_Basic_Cost_of_Service, @On_Peak_kWh, @On_Peak_Charges, @Mid_Peak_kWh, @Mid_Peak_Charges, @Off_Peak_kWh, @Off_Peak_Charges, @Delta_TOD_vs_Basic_Cost_of_Service, @Total_Savings_for_TOD, @Percent_of_Savings_for_TOD, @Basic_Service_Savings, @Percentage_of_Basic_Service_Savings, @Monthly_Percentage_of_On_Peak_Usage, @Month_to_Month_Savings, @Last_Month_Percentage_of_On_Peak_Usage, @totalkwh, @midoffpeakkwh, @onpeakpctlast, @actualwidth1, @actualwidth2

SET @Mid_Peak_kWh = AttributeValue('Mid Peak kWh')
SET @Off_Peak_kWh = AttributeValue('Off Peak kWh')
SET @On_Peak_kWh = AttributeValue('On Peak kWh')
SET @Month_to_Month_Savings = AttributeValue('Month to Month Savings')
SET @Total_Savings_for_TOD = AttributeValue('Total Savings for TOD')
SET @Monthly_Percentage_of_On_Peak_Usage = AttributeValue('Monthly Percentage of On Peak Usage')
SET @Last_Month_Percentage_of_On_Peak_Usage = AttributeValue('Last Month Percentage of On Peak Usage')


SET @midoffpeakkwh = add(@Mid_Peak_kWh, @Off_Peak_kWh) 
SET @midoffpeakpct = subtract(1,@Monthly_Percentage_of_On_Peak_Usage)
SET @onpeakpctlast = @Last_Month_Percentage_of_On_Peak_Usage
SET @totalkwh = add(@On_Peak_kWh, @midoffpeakkwh)
 
/* On Peak Pct bar chart */
SET @actualwidth1 = multiply(divide(@On_Peak_kWh, @totalkwh),100) 

/* Mid and Off Peak pct bar chart */
SET @actualwidth2 = multiply(divide(@midoffpeakkwh, @totalkwh),100)
 
]%%
~~~

Yes, this could have been written cleaner and pre-calculated with an automation process upon data file
import into SFMC.

The html portion of the graph looks like this:

...
<table border="0" cellpadding="5" cellspacing="5">
         
          <tr>
           <td nowrap="nowrap">
            <span style="font-size:12px;">Your on-peak usage</span></td><td width="200">
            <table height="50" width="100%">
             
              <tr>
               <td height="50" style="background-color:#FFE552;" width="%%=v(@actualwidth1)=%%%">
               </td><td width="100%">
               </td></tr></table></td><td nowrap="nowrap">
            <span style="font-size:12px;">%%=v(@Monthly_Percentage_of_On_Peak_Usage)=%% kWh</span></td></tr><tr>
           <td nowrap="nowrap">
            <span style="font-size:12px;">Your mid- and off-peak usage</span></td><td width="200">
            <table height="50" width="100%">
             
              <tr>
               <td height="50" style="background-color:#00AED0;" width="%%=v(@actualwidth2)=%%%">
               </td><td width="100%">
               </td></tr></table></td><td nowrap="nowrap">
            <span style="font-size:12px;">%%=v(@midoffpeakkwh)=%% kWh</span></td></tr></table></td></tr></table
 ...
 
And, the email preview cranks out like this:
 
 ![AmpScript Bar Chart](https://github.com/mktgurl/AMPScript_SFMC/blob/3e07f8f22b670c1695c2d3e4355b2d0464562f0a/AmpscriptBarChart.png)

The bar portion of the chart is calculated off a percentage of electricity used, while the quantitative value to its right is the actual kwh used.
