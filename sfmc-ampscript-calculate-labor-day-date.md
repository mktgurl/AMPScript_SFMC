--- 2025-10-30, This is part of a set of AMPScript that I am using to build a list of upcoming holidays in a customer email.  
--- The AMPScript for Labor Day, the first Monday in September, begins with Sept 1 and looks for the next occurrence of Monday  
--- if Sept 1 is not a Monday.

~~~
%%[
/* Get current year */
SET @today = Now()
SET @currentYear = DatePart(@today, "Y")
SET @nextYear = Add(@currentYear,1)

/* Start from September 1 of current year */
SET @sep_first = DateParse(Concat(@currentYear, "-09-01"))

/* Get weekday name */
SET @weekdayName = FormatDate(@sep_first, "dddddd")

/* Convert weekday name to number (1=Sunday ... 7=Saturday) */
IF @weekdayName == "Sunday" THEN
  SET @sep_first_weekday = 1
ELSEIF @weekdayName == "Monday" THEN
  SET @sep_first_weekday = 2
ELSEIF @weekdayName == "Tuesday" THEN
  SET @sep_first_weekday = 3
ELSEIF @weekdayName == "Wednesday" THEN
  SET @sep_first_weekday = 4
ELSEIF @weekdayName == "Thursday" THEN
  SET @sep_first_weekday = 5
ELSEIF @weekdayName == "Friday" THEN
  SET @sep_first_weekday = 6
ELSEIF @weekdayName == "Saturday" THEN
  SET @sep_first_weekday = 7
ENDIF

/* Days to add until Monday */
IF @sep_first_weekday == 2 THEN
  SET @daysToAdd = 0
ELSEIF @sep_first_weekday == 1 THEN
  SET @daysToAdd = 1
ELSE
  SET @daysToAdd = 9 - @sep_first_weekday
ENDIF

/* First Monday in September */
SET @LaborDay = DateAdd(@sep_first, @daysToAdd, "D")

/* Final output */
SET @labordayOutput = Concat(formatdate(@laborday, "dddddd"), ", Labor Day (", formatdate(@laborday, "M/d"), ")")

]%%
~~~

This outputs:

Monday, Labor Day (9/1)
