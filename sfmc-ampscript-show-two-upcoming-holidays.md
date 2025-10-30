--- 2025-10-30, this AMPScript has taken three days to write and troubleshoot, but it finally works. The purpose of this AMPScript  
--- is to provide a bulleted list of upcoming holidays. The holiday list includes three fixed (New Year's Day, Independence Day,  
--- and Christmas Day) and three floating holidays (Memorial Day, Labor Day, and Thanksgiving). The AMPScript was written in  
--- parts and then combined. It is added to the email body as a code snippet. It is quite long, so I'll use a ContentBlockID  
--- so that marketing users don't have to see all the AMPScript used to generate the list of holidays.  

~~~
%%[var @contentBlockID set @contentBlockID = #######]%%%%=ContentBlockByID(@contentBlockID)=%%
~~~

Here is the AMPScript:

~~~
%%[
/* ============================================
   Setup
   ============================================ */
SET @today = Now()
SET @currentYear = DatePart(@today,"Y")
SET @nextYear = Add(@currentYear,1)
SET @holidays = ""

/* ============================================
   Build holiday list for this year and next
   ============================================ */
FOR @y = @currentYear TO @nextYear DO

  /* Fixed holidays */
  SET @newYears = DateParse(Concat(@y,"-01-01"))
  SET @independence = DateParse(Concat(@y,"-07-04"))
  SET @christmas = DateParse(Concat(@y,"-12-25"))

  /* Memorial Day: last Monday in May */
  SET @jun_first = DateParse(Concat(@y,"-06-01"))
  SET @weekdayName = FormatDate(@jun_first,"dddddd")
  IF @weekdayName == "Sunday" THEN SET @jun_first_weekday = 1
  ELSEIF @weekdayName == "Monday" THEN SET @jun_first_weekday = 2
  ELSEIF @weekdayName == "Tuesday" THEN SET @jun_first_weekday = 3
  ELSEIF @weekdayName == "Wednesday" THEN SET @jun_first_weekday = 4
  ELSEIF @weekdayName == "Thursday" THEN SET @jun_first_weekday = 5
  ELSEIF @weekdayName == "Friday" THEN SET @jun_first_weekday = 6
  ELSE SET @jun_first_weekday = 7 ENDIF

  IF @jun_first_weekday == 2 THEN
    SET @MemorialDay = DateAdd(@jun_first,-7,"D")
  ELSEIF @jun_first_weekday == 1 THEN
    SET @MemorialDay = DateAdd(@jun_first,-6,"D")
  ELSEIF @jun_first_weekday == 3 THEN
    SET @MemorialDay = DateAdd(@jun_first,-1,"D")
  ELSEIF @jun_first_weekday == 4 THEN
    SET @MemorialDay = DateAdd(@jun_first,-2,"D")
  ELSEIF @jun_first_weekday == 5 THEN
    SET @MemorialDay = DateAdd(@jun_first,-3,"D")
  ELSEIF @jun_first_weekday == 6 THEN
    SET @MemorialDay = DateAdd(@jun_first,-4,"D")
  ELSE
    SET @MemorialDay = DateAdd(@jun_first,-5,"D")
  ENDIF

  /* Labor Day: first Monday in September */
  SET @sep_first = DateParse(Concat(@y,"-09-01"))
  SET @weekdayName = FormatDate(@sep_first,"dddddd")
  IF @weekdayName == "Sunday" THEN SET @sep_first_weekday = 1
  ELSEIF @weekdayName == "Monday" THEN SET @sep_first_weekday = 2
  ELSEIF @weekdayName == "Tuesday" THEN SET @sep_first_weekday = 3
  ELSEIF @weekdayName == "Wednesday" THEN SET @sep_first_weekday = 4
  ELSEIF @weekdayName == "Thursday" THEN SET @sep_first_weekday = 5
  ELSEIF @weekdayName == "Friday" THEN SET @sep_first_weekday = 6
  ELSE SET @sep_first_weekday = 7 ENDIF

  IF @sep_first_weekday == 2 THEN
    SET @daysToAdd = 0
  ELSEIF @sep_first_weekday == 1 THEN
    SET @daysToAdd = 1
  ELSE
    SET @daysToAdd = 9 - @sep_first_weekday
  ENDIF
  SET @LaborDay = DateAdd(@sep_first,@daysToAdd,"D")

  /* Thanksgiving: fourth Thursday in November */
  SET @dec_first = DateParse(Concat(@y,"-12-01"))
  SET @weekdayName = FormatDate(@dec_first,"dddddd")
  IF @weekdayName == "Sunday" THEN SET @dec_first_weekday = 1
  ELSEIF @weekdayName == "Monday" THEN SET @dec_first_weekday = 2
  ELSEIF @weekdayName == "Tuesday" THEN SET @dec_first_weekday = 3
  ELSEIF @weekdayName == "Wednesday" THEN SET @dec_first_weekday = 4
  ELSEIF @weekdayName == "Thursday" THEN SET @dec_first_weekday = 5
  ELSEIF @weekdayName == "Friday" THEN SET @dec_first_weekday = 6
  ELSE SET @dec_first_weekday = 7 ENDIF

  IF @dec_first_weekday == 1 THEN
    SET @thanksgiving = DateAdd(@dec_first,-3,"D")
  ELSEIF @dec_first_weekday == 2 THEN
    SET @thanksgiving = DateAdd(@dec_first,-4,"D")
  ELSEIF @dec_first_weekday == 3 THEN
    SET @thanksgiving = DateAdd(@dec_first,-5,"D")
  ELSEIF @dec_first_weekday == 4 THEN
    SET @thanksgiving = DateAdd(@dec_first,-6,"D")
  ELSEIF @dec_first_weekday == 5 THEN
    SET @thanksgiving = DateAdd(@dec_first,-7,"D")
  ELSEIF @dec_first_weekday == 6 THEN
    SET @thanksgiving = DateAdd(@dec_first,-1,"D")
  ELSE
    SET @thanksgiving = DateAdd(@dec_first,-2,"D")
  ENDIF

  /* Add all holidays for this year */
  SET @holidays = Concat(@holidays,
    FormatDate(@newYears,"yyyy-MM-dd"),"|New Year's Day~",
    FormatDate(@MemorialDay,"yyyy-MM-dd"),"|Memorial Day~",
    FormatDate(@independence,"yyyy-MM-dd"),"|Independence Day~",
    FormatDate(@LaborDay,"yyyy-MM-dd"),"|Labor Day~",
    FormatDate(@thanksgiving,"yyyy-MM-dd"),"|Thanksgiving~",
    FormatDate(@christmas,"yyyy-MM-dd"),"|Christmas~"
  )

NEXT @y

/* ============================================
   Identify the next two upcoming holidays
   ============================================ */
SET @holidayRows = BuildRowsetFromString(@holidays,"~")
SET @next1Date = ""
SET @next2Date = ""
SET @next1Name = ""
SET @next2Name = ""

FOR @i = 1 TO RowCount(@holidayRows) DO
  SET @row = Row(@holidayRows,@i)
  SET @raw = Trim(Field(@row,1))

  IF Length(@raw) > 0 AND IndexOf(@raw,"|") > 0 THEN
    SET @fields = BuildRowsetFromString(@raw,"|")

    IF RowCount(@fields) >= 2 THEN
      SET @dateStr = Field(Row(@fields,1),1)
      SET @holi_name = Field(Row(@fields,2),1)
      IF Length(@dateStr) > 0 AND Length(@holi_name) > 0 THEN
        SET @holi_date = DateParse(@dateStr)

        /* only future holidays (after today) */
        IF @holi_date > @today THEN
          /* pick earliest two */
          IF Empty(@next1Date) THEN
            SET @next1Date = @holi_date
            SET @next1Name = @holi_name
          ELSEIF @holi_date < @next1Date THEN
            SET @next2Date = @next1Date
            SET @next2Name = @next1Name
            SET @next1Date = @holi_date
            SET @next1Name = @holi_name
          ELSEIF Empty(@next2Date) OR @holi_date < @next2Date THEN
            SET @next2Date = @holi_date
            SET @next2Name = @holi_name
          ENDIF
        ENDIF
      ENDIF
    ENDIF
  ENDIF
NEXT @i

/* ============================================
   Output next two upcoming holidays
   ============================================ */
SET @upcomingHolidays = ""

IF NOT Empty(@next1Date) THEN
  SET @weekday1 = FormatDate(@next1Date,"dddddd")
  IF @weekday1 == "Saturday" THEN
    SET @next1Display = Concat("Friday (Observed), ",@next1Name)
  ELSEIF @weekday1 == "Sunday" THEN
    SET @next1Display = Concat("Monday (Observed), ",@next1Name)
  ELSE
    SET @next1Display = Concat(@weekday1,", ",@next1Name)
  ENDIF
  SET @upcomingHolidays = Concat(@upcomingHolidays,
    "<li>",@next1Display," (",FormatDate(@next1Date,"M/d"),")</li>")
ENDIF

IF NOT Empty(@next2Date) THEN
  SET @weekday2 = FormatDate(@next2Date,"dddddd")
  IF @weekday2 == "Saturday" THEN
    SET @next2Display = Concat("Friday (Observed), ",@next2Name)
  ELSEIF @weekday2 == "Sunday" THEN
    SET @next2Display = Concat("Monday (Observed), ",@next2Name)
  ELSE
    SET @next2Display = Concat(@weekday2,", ",@next2Name)
  ENDIF
  SET @upcomingHolidays = Concat(@upcomingHolidays,
    "<li>",@next2Display," (",FormatDate(@next2Date,"M/d"),")</li>")
ENDIF
]%%

<ul>
%%=v(@upcomingHolidays)=%%
</ul>
~~~

This outputs:

* Thursday, Thanksgiving (11/27)  
* Thursday, Christmas (12/25)  
