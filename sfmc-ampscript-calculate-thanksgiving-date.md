--- 2025-10-30, I finally figured out how to calculate the actual date of a floating holiday. Here is what I used to find the  
--- Thanksgiving Day date. Thanksgiving falls on the last Thursday of November. I am using Dec 1 as the starting point for  
--- the query. It is essentially counting backward.  

~~~
%%[
/* Current year */
SET @today = Now()
SET @currentYear = DatePart(@today, "Y")

/* Start from Dec 1 of the current year */
SET @dec_first = DateParse(Concat(@currentYear, "-12-01"))

/* Weekday name for Dec 1 */
SET @weekdayName = FormatDate(@dec_first, "dddddd")

/* Convert weekday name to number (1=Sunday ... 7=Saturday) */
IF @weekdayName == "Sunday" THEN
  SET @dec_first_weekday = 1
ELSEIF @weekdayName == "Monday" THEN
  SET @dec_first_weekday = 2
ELSEIF @weekdayName == "Tuesday" THEN
  SET @dec_first_weekday = 3
ELSEIF @weekdayName == "Wednesday" THEN
  SET @dec_first_weekday = 4
ELSEIF @weekdayName == "Thursday" THEN
  SET @dec_first_weekday = 5
ELSEIF @weekdayName == "Friday" THEN
  SET @dec_first_weekday = 6
ELSEIF @weekdayName == "Saturday" THEN
  SET @dec_first_weekday = 7
ENDIF

/* Map weekday to negative days to pass into DateAdd (so DateAdd(@jun_first, <neg>, "D") yields last Thursday in November) */
IF @dec_first_weekday == 1 THEN
  SET @thanksgiving = DateAdd(@dec_first, -3, "D")
ELSEIF @dec_first_weekday == 2 THEN
  SET @thanksgiving = DateAdd(@dec_first, -4, "D")
ELSEIF @dec_first_weekday == 3 THEN
  SET @thanksgiving = DateAdd(@dec_first, -5, "D")
ELSEIF @dec_first_weekday == 4 THEN
  SET @thanksgiving = DateAdd(@dec_first, -6, "D")
ELSEIF @dec_first_weekday == 5 THEN
  SET @thanksgiving = DateAdd(@dec_first, -7, "D")
ELSEIF @dec_first_weekday == 6 THEN
  SET @thanksgiving = DateAdd(@dec_first, -1, "D")
ELSEIF @dec_first_weekday == 7 THEN
  SET @thanksgiving = DateAdd(@dec_first, -2, "D")
ENDIF

SET @thanksgivingOutput = Concat(formatdate(@thanksgiving, "dddddd"), ", Thanksgiving Day (", formatdate(@thanksgiving, "M/d"), ")")
]%%
~~~

This outputs:

Thursday, Thanksgiving Day (11/27)
