Last updated: 2025-05  

Created this for an SMS send that happens twice per year. Got tired of setting up a unique SMS journey for English and Spanish customers where I had a season start SMS for winter and summer; or four SMS templates with static content.

I updated the AMPScript to bring in a specific value if the months were between certain calendar months, also added a "Welcome" path to the journey to send an SMS Welcome message to anyone not opted into the short code.  

Journey flow looks like this:  

[data extension data source] --> Primary Language -->

ENG --> OptInStatus Null --> SMS Welcome ENG  
ENG --> OptInStatus Not Null --> SMS Season Start ENG  
                                                          
SPN --> OptInStatus Null     --> SMS Welcome SPN  
SPN --> OptInStatus Not Null --> SMS Season Start SPN  

**Welcome + Season Start ENG**  
~~~
%%[var @contentBlockID set @contentBlockID = 1383709]%%%%=ContentBlockByID(@contentBlockID)=%%PGE: Welcome to Peak Time Rebates texts! Approx. 6-10 msgs/month Nov-Feb & Jun-Sept. Reply STOP to stop and HELP for help. Msg & Data Rates May Apply.

Peak Time Rebates %%=v(@season)=%% season begins %%=v(@seasonDate)=%%. Visit portlandgeneral.com/ptrshift to help keep energy reliable and save.
~~~

**Season Start ENG**  
~~~
%%[var @contentBlockID set @contentBlockID = 1383709]%%%%=ContentBlockByID(@contentBlockID)=%%PGE: Peak Time Rebates %%=v(@season)=%% season begins %%=v(@seasonDate)=%%. Visit portlandgeneral.com/ptrshift to help keep energy reliable and save. Reply STOP to opt out.
~~~

**Welcome + Season Start SPN**  
~~~
%%[var @contentBlockID set @contentBlockID = 1383775]%%%%=ContentBlockByID(@contentBlockID)=%%PGE: ¡Bienvenido a los textos de Peak Time Rebates! 6 a 10 mensajes/mes (nov. a feb./jun. a sep). Responda STOP (cancelar) y HELP (ayuda). Puede tener costo.

La temporada de %%=v(@season)=%% de Peak Time Rebates empieza el %%=v(@seasonDate)=%%. Ingrese a portlandgeneral.com/ptrcambio.
~~~

**Season Start SPN only**  
~~~
%%[var @contentBlockID set @contentBlockID = 1383775]%%%%=ContentBlockByID(@contentBlockID)=%%PGE: La temporada de %%=v(@season)=%% de Peak Time Rebates empieza el %%=v(@seasonDate)=%%. Ingrese a portlandgeneral.com/ptrcambio. Responda STOP para cancelar la suscripción.
~~~

**SMS preview:**  

PGE: Peak Time Rebates summer season begins 6/1. Visit portlandgeneral.com/ptrshift to help keep energy reliable and save. Reply STOP to opt out.  
PGE: La temporada de verano de Peak Time Rebates empieza el 6/1. Ingrese a portlandgeneral.com/ptrcambio. Responda STOP para cancelar la suscripción.  

As you can see, there are different ContentBlockID for ENG and SPN, that is because the season name is different.  

**ContentBlockID = 1383709**  
~~~
%%[VAR @currentDate, @month, @day, @season, @seasonDate
SET @currentDate = Now()
SET @month = FormatDate(@currentDate, "M") IF (@month > 9 and @month < 3) THEN SET @season = "winter" SET @seasonDate = "11/1" ELSE SET @season = "summer" SET @seasonDate = "6/1" ENDIF]%%
~~~

**ContentBlockID = 1383775**  
~~~
%%[VAR @currentDate, @month, @day, @season, @seasonDate
SET @currentDate = Now()
SET @month = FormatDate(@currentDate, "M") IF (@month > 9 and @month < 3) THEN SET @season = "invierno" SET @seasonDate = "11/1" ELSE SET @season = "verano" SET @seasonDate = "6/1" ENDIF]%%
~~~
