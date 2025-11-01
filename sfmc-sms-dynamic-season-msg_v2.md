2025-10, Looks like it took five months to come up with this revision but there was a code freeze on SMS creation during an active "season". In reality, just a few hours to change how the season start SMS messaging handles a customer's primary language and current season. Instead of having eight (8) SMS notification templates, I have reduced the need to two (2) SMS templates. The first is for a Welcome message for customers who have newly enrolled into a program or updated their mobile number; and the second message is for customers that are already enrolled and opted into a short code. Before AMPScript static SMS notification messages was used for each season. There were:  

* ENG summer start  
* ENG summer welcome + start  
* SPN summer start  
* SPN summer welcome + start  
* ENG winter start  
* ENG winter welcome + start  
* SPN winter start  
* SPN winter welcome + start

It is cumbersome for a marketer to manually setup summer and winter season start journeys for each program event season. This AMPScript reduces the number of journey SMS to two notification messages:

* ENG/SPN season start
* ENG/SPN season welcome + start

It relies on the source data extension to have a field for PrimaryLanguage and the values being "ENG" or "SPN". Other than that, as long as the data is prepared correctly, these SMS messages shouldn't have an issue when sending.

**Season Welcome + Start AMPScript:**  

~~~
%%[
VAR @currentDate, @season, @seasonDate, @primaryLanguage, @seasonLabel, @seasonMessage
/* Get primary language from data */
SET @primaryLanguage = AttributeValue("primaryLanguage")
/* Get current date */
SET @currentDate = Now()
/* Determine season */
IF DatePart(@currentDate, "M") >= 10 OR DatePart(@currentDate, "M") <= 2 THEN
  SET @season = "winter"
  SET @seasonDate = "11/1"
ELSEIF DatePart(@currentDate, "M") >= 3 AND DatePart(@currentDate, "M") <= 9 THEN
  SET @season = "summer"
  SET @seasonDate = "6/1"
ELSE
  SET @season = "spring"
  SET @seasonDate = "3/1"
ENDIF
/* Translate season label */
IF @primaryLanguage == "SPN" THEN
  IF @season == "summer" THEN
    SET @seasonLabel = "verano"
  ELSEIF @season == "winter" THEN
    SET @seasonLabel = "invierno"
  ELSE
    SET @seasonLabel = @season
  ENDIF
ELSE
  SET @seasonLabel = @season /* default to English */
ENDIF
/* Generate final message based on language */
IF @primaryLanguage == "SPN" THEN
  SET @WelcomeMessage = "PGE: ¡Bienvenido a los textos de Peak Time Rebates! 6 a 10 mensajes/mes (nov. a feb./jun. a sep). Responda STOP (cancelar) y HELP (ayuda). Puede tener costo.

"
  SET @seasonMessage = Concat(@WelcomeMessage, "La temporada de ", @seasonLabel, " de Peak Time Rebates empieza el ", @seasonDate, ". Ingrese a portlandgeneral.com/ptrcambio. Responda STOP para cancelar la suscripción.")
ELSE
  SET @WelcomeMessage = "PGE: Welcome to Peak Time Rebates texts! Approx. 6-10 msgs/month Nov-Feb & Jun-Sept. Reply STOP to stop and HELP for help. Msg & Data Rates May Apply.

"
  SET @seasonMessage = Concat(@WelcomeMessage, "Peak Time Rebates ", @seasonLabel, " season begins ", @seasonDate, ". Visit portlandgeneral.com/ptrshift to help keep energy reliable and save.")
ENDIF
]%%
~~~

**Season Start AMPScript:**  
~~~
%%[
VAR @currentDate, @season, @seasonDate, @primaryLanguage, @seasonLabel, @seasonMessage
/* Get primary language from data */
SET @primaryLanguage = AttributeValue("primaryLanguage")
/* Get current date */
SET @currentDate = Now()
/* Determine season */
IF DatePart(@currentDate, "M") >= 10 OR DatePart(@currentDate, "M") <= 2 THEN
  SET @season = "winter"
  SET @seasonDate = "11/1"
ELSEIF DatePart(@currentDate, "M") >= 3 AND DatePart(@currentDate, "M") <= 9 THEN
  SET @season = "summer"
  SET @seasonDate = "6/1"
ELSE
  SET @season = "spring"
  SET @seasonDate = "3/1"
ENDIF
/* Translate season label */
IF @primaryLanguage == "SPN" THEN
  IF @season == "summer" THEN
    SET @seasonLabel = "verano"
  ELSEIF @season == "winter" THEN
    SET @seasonLabel = "invierno"
  ELSE
    SET @seasonLabel = @season
  ENDIF
ELSE
  SET @seasonLabel = @season /* default to English */
ENDIF
/* Generate final message based on language */
IF @primaryLanguage == "SPN" THEN
  SET @seasonMessage = Concat("PGE: La temporada de ", @seasonLabel, " de Peak Time Rebates empieza el ", @seasonDate, ". Ingrese a portlandgeneral.com/ptrcambio. Responda STOP para cancelar la suscripción.")
ELSE
  SET @seasonMessage = Concat("PTR: Peak Time Rebates ", @seasonLabel, " season begins ", @seasonDate, ". Visit portlandgeneral.com/ptrshift to help keep energy reliable and save.")
ENDIF
]%%
~~~

The SMS body looks like this:  

**Not opted in (This receives the Season Welcome + Start SMS)**  
~~~
%%[var @contentBlockID set @contentBlockID = 1445429]%%%%=ContentBlockByID(@contentBlockID)=%%%%=v(@seasonMessage)=%%
~~~

**Opt In Not Null (This receives the Season Start SMS)**  
~~~
%%[var @contentBlockID set @contentBlockID = 1443472]%%%%=ContentBlockByID(@contentBlockID)=%%%%=v(@seasonMessage)=%%
~~~

Here is how the SMS templates preview:  

**SMS preview - ENG Welcome**  

~~~
PGE: Welcome to Peak Time Rebates texts! Approx. 6-10 msgs/month Nov-Feb & Jun-Sept. Reply STOP to stop and HELP for help. Msg & Data Rates May Apply.

Peak Time Rebates winter season begins 11/1. Visit portlandgeneral.com/ptrshift to help keep energy reliable and save.
~~~

**SMS preview - SPN Welcome**

~~~
PGE: ¡Bienvenido a los textos de Peak Time Rebates! 6 a 10 mensajes/mes (nov. a feb./jun. a sep). Responda STOP (cancelar) y HELP (ayuda). Puede tener costo.

La temporada de invierno de Peak Time Rebates empieza el 11/1. Ingrese a portlandgeneral.com/ptrcambio. Responda STOP para cancelar la suscripción.
~~~

**SMS preview - ENG season start**

~~~
PTR: Peak Time Rebates winter season begins 11/1. Visit portlandgeneral.com/ptrshift to help keep energy reliable and save.
~~~

**SMS preview - SPN season start**

~~~
PGE: La temporada de invierno de Peak Time Rebates empieza el 11/1. Ingrese a portlandgeneral.com/ptrcambio. Responda STOP para cancelar la suscripción.
~~~
