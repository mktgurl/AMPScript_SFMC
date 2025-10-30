--- 2025-10-30, After much struggle with floating holidays in general. The logic for Memorial Day, Labor Day, and Thanksgiving  
--- are variations of a similar format for AMPScript. This AMPScript is for calculating Memorial Day, the last Monday in May. 
--- In this example, the AMPScript counts backwards from June 1 to find the preceding Thursday.

~~~
%%[
/* Current year */
SET @today = Now()
SET @currentYear = DatePart(@today, "Y")

/* Start from June 1 of the current year */
SET @jun_first = DateParse(Concat(@currentYear, "-06-01"))

/* Weekday name for June 1 */
SET @weekdayName = FormatDate(@jun_first, "dddddd")

/* Convert weekday name to number (1=Sunday ... 7=Saturday) */
IF @weekdayName == "Sunday" THEN
  SET @jun_first_weekday = 1
ELSEIF @weekdayName == "Monday" THEN
  SET @jun_first_weekday = 2
ELSEIF @weekdayName == "Tuesday" THEN
  SET @jun_first_weekday = 3
ELSEIF @weekdayName == "Wednesday" THEN
  SET @jun_first_weekday = 4
ELSEIF @weekdayName == "Thursday" THEN
  SET @jun_first_weekday = 5
ELSEIF @weekdayName == "Friday" THEN
  SET @jun_first_weekday = 6
ELSEIF @weekdayName == "Saturday" THEN
  SET @jun_first_weekday = 7
ENDIF

/* Map weekday to negative days to pass into DateAdd (so DateAdd(@jun_first, <neg>, "D") yields last Monday in May) */
IF @jun_first_weekday == 2 THEN
  /* If June 1 is Monday, last Monday in May is 7 days earlier */
  SET @MemorialDay = DateAdd(@jun_first, -7, "D")
ELSEIF @jun_first_weekday == 1 THEN
  /* If June 1 is Sunday, last Monday = 6 days earlier (May 26 when June 1 is Sunday) */
  SET @MemorialDay = DateAdd(@jun_first, -6, "D")
ELSEIF @jun_first_weekday == 3 THEN
  SET @MemorialDay = DateAdd(@jun_first, -1, "D")
ELSEIF @jun_first_weekday == 4 THEN
  SET @MemorialDay = DateAdd(@jun_first, -2, "D")
ELSEIF @jun_first_weekday == 5 THEN
  SET @MemorialDay = DateAdd(@jun_first, -3, "D")
ELSEIF @jun_first_weekday == 6 THEN
  SET @MemorialDay = DateAdd(@jun_first, -4, "D")
ELSEIF @jun_first_weekday == 7 THEN
  SET @MemorialDay = DateAdd(@jun_first, -5, "D")
ENDIF

/* Format output "Monday, Memorial Day (M/d)" */

SET @memorialdayOutput = Concat(formatdate(@memorialday, "dddddd"), ", Memorial Day (", formatdate(@memorialday, "M/d"), ")")
]%%

~~~

This outputs:

Monday, Memorial Day (5/26)
