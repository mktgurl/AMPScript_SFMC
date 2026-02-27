2026-02-26, I updated the Season End SMS so that when the SMS is previewed before a season ends, it'll show that season's end date. Example, today is two days before the last day of the season for a winter program. It still looks at the source data extension field for a customer's preferred language to show either an English or Spanish message. I am using a Journey Builder Decision Split criteria to determine a customer's mobile number OptInStatus.

Also, I do not have this giant wall of AMPScript in the SMS, I am using AMPScript in the SMS template to call another AMPScript code snippet:

~~~
%%[var @contentBlockID set @contentBlockID = #######]%%%%=ContentBlockByID(@contentBlockID)=%%%%=v(@msg)=%%
~~~

**Updated AMPSCript:**
~~~
%%[
VAR @currentDate, @primaryLanguage
VAR @seasonlabel1_eng, @seasonlabel2_eng, @seasonlabel1_spn, @seasonlabel2_spn
VAR @seasonenddate, @seasonstartdate
VAR @winterEnd, @summerEnd
VAR @msg

SET @primaryLanguage = AttributeValue("PrimaryLanguage")
SET @currentDate = Now()

/* Define season end dates for current year */
SET @winterEnd = DateParse(Concat(FormatDate(@currentDate,"yyyy"),"-02-28"))
SET @summerEnd = DateParse(Concat(FormatDate(@currentDate,"yyyy"),"-09-30"))

/* Determine which season messaging to show */
IF @currentDate <= @winterEnd THEN

  /* Winter season ended */
  SET @seasonlabel1_eng = "winter "
  SET @seasonlabel2_eng = "summer "
  SET @seasonlabel1_spn = "invierno "
  SET @seasonlabel2_spn = "verano "
  SET @seasonenddate = "2/28"
  SET @seasonstartdate = "6/1"

ELSEIF @currentDate <= @summerEnd THEN

  /* Summer season ended */
  SET @seasonlabel1_eng = "summer "
  SET @seasonlabel2_eng = "winter "
  SET @seasonlabel1_spn = "verano "
  SET @seasonlabel2_spn = "invierno "
  SET @seasonenddate = "9/30"
  SET @seasonstartdate = "11/1"

ELSE

  /* After 9/30 (Oct–Dec) */
  SET @seasonlabel1_eng = "summer "
  SET @seasonlabel2_eng = "winter "
  SET @seasonlabel1_spn = "verano "
  SET @seasonlabel2_spn = "invierno "
  SET @seasonenddate = "9/30"
  SET @seasonstartdate = "11/1"

ENDIF


/* Build the message */
IF @primaryLanguage == "SPN" THEN

  SET @msg = Concat(
    "PGE: La temporada de Peak Time Rebates de ",
    @seasonlabel1_spn,
    "terminó ",
    @seasonenddate,
    ". Visite portlandgeneral.com/ptrcambio para prepararse para la temporada de ",
    @seasonlabel2_spn,
    "que inicia ",
    @seasonstartdate,
    "."
  )

ELSE

  SET @msg = Concat(
    "PGE: Peak Time Rebates ",
    @seasonlabel1_eng,
    "season ended ",
    @seasonenddate,
    ". Visit portlandgeneral.com/ptr to get ready for the ",
    @seasonlabel2_eng,
    "season that starts ",
    @seasonstartdate,
    "."
  )

ENDIF
]%%
~~~


**Season End SMS (when previewed on 2/26):** 
~~~
PGE: Peak Time Rebates winter season ended 2/28. Visit portlandgeneral.com/ptr to get ready for the summer season that starts 6/1.

PGE: La temporada de Peak Time Rebates de invierno terminó 2/28. Visite portlandgeneral.com/ptrcambio para prepararse para la temporada de verano que inicia 6/1.
~~~

**This is its companion SMS for customers whose mobile numbers are new to this program's short code:**
~~~
%%[
VAR @currentDate, @primaryLanguage
VAR @WelcomeMessage, @seasonlabel1_eng, @seasonlabel2_eng, @seasonlabel1_spn, @seasonlabel2_spn
VAR @seasonenddate, @seasonstartdate
VAR @winterEnd, @summerEnd
VAR @msg

SET @primaryLanguage = AttributeValue("PrimaryLanguage")
SET @currentDate = Now()

/* Define season end dates for current year */
SET @winterEnd = DateParse(Concat(FormatDate(@currentDate,"yyyy"),"-02-28"))
SET @summerEnd = DateParse(Concat(FormatDate(@currentDate,"yyyy"),"-09-30"))

/* Determine which season messaging to show */
IF @currentDate <= @winterEnd THEN

  /* Winter season ended */
  SET @seasonlabel1_eng = "winter "
  SET @seasonlabel2_eng = "summer "
  SET @seasonlabel1_spn = "invierno "
  SET @seasonlabel2_spn = "verano "
  SET @seasonenddate = "2/28"
  SET @seasonstartdate = "6/1"

ELSEIF @currentDate <= @summerEnd THEN

  /* Summer season ended */
  SET @seasonlabel1_eng = "summer "
  SET @seasonlabel2_eng = "winter "
  SET @seasonlabel1_spn = "verano "
  SET @seasonlabel2_spn = "invierno "
  SET @seasonenddate = "9/30"
  SET @seasonstartdate = "11/1"

ELSE

  /* After Sept 30 (Oct–Dec) */
  SET @seasonlabel1_eng = "summer "
  SET @seasonlabel2_eng = "winter "
  SET @seasonlabel1_spn = "verano "
  SET @seasonlabel2_spn = "invierno "
  SET @seasonenddate = "9/30"
  SET @seasonstartdate = "11/1"

ENDIF


/* Language-specific message */
IF @primaryLanguage == "SPN" THEN

  SET @WelcomeMessage = "PGE: ¡Bienvenido a los textos de Peak Time Rebates! 6 a 10 mensajes/mes (nov. a feb./jun. a sep). Responda STOP (cancelar) y HELP (ayuda). Puede tener costo.

"

  SET @msg = Concat(
    @WelcomeMessage,
    "La temporada de Peak Time Rebates de ",
    @seasonlabel1_spn,
    "terminó ",
    @seasonenddate,
    ". Visite portlandgeneral.com/ptrcambio para prepararse para la temporada de ",
    @seasonlabel2_spn,
    "que inicia ",
    @seasonstartdate,
    "."
  )

ELSE

  SET @WelcomeMessage = "PGE: Welcome to Peak Time Rebates texts! 6-10 msgs/mo (Nov-Feb/Jun-Sep). Reply STOP to cancel, HELP for help. Msg&Data rates may apply.

"

  SET @msg = Concat(
    @WelcomeMessage,
    "Peak Time Rebates ",
    @seasonlabel1_eng,
    "season ended ",
    @seasonenddate,
    ". Visit portlandgeneral.com/ptr to get ready for the ",
    @seasonlabel2_eng,
    "season that starts ",
    @seasonstartdate,
    "."
  )

ENDIF
]%%
~~~

**What customers see for their preferred language Welcome SMS + Season End message:**
~~~
PGE: Welcome to Peak Time Rebates texts! 6-10 msgs/mo (Nov-Feb/Jun-Sep). Reply STOP to cancel, HELP for help. Msg&Data rates may apply.

Peak Time Rebates winter season ended 2/28. Visit portlandgeneral.com/ptr to get ready for the summer season that starts 6/1.

-or-

PGE: ¡Bienvenido a los textos de Peak Time Rebates! 6 a 10 mensajes/mes (nov. a feb./jun. a sep). Responda STOP (cancelar) y HELP (ayuda). Puede tener costo.

La temporada de Peak Time Rebates de invierno terminó 2/28. Visite portlandgeneral.com/ptrcambio para prepararse para la temporada de verano que inicia 6/1.
~~~
