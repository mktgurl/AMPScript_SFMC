Created this process in December 2021 and have finally gotten around to writing up how it works. A stakeholder needed to capture transactional dynamic content that was used on an email message sent to customers. This method allows capture of the "view as a web page" (vawp) link to be upserted into a standalone data extension. Links stay active according to what your SFMC account settings are. For this company, it's 730 days or 2 years from the time a link is generated.

This set of fields is common to all email templates using the script:

* Email
* vawp
* SubscriberKey
* EmailsentDate

This set of fields is unique to the company and to the program:

* service_address_id
* EmailTemplateName (this is hard coded into the AMPScript)
* Program (also hard coded into the AMPScript)

In total, seven (7) fields are captured to a data extension.

The field names are case sensitive, so don't forget what's capitalized and what isn't between how the AMPScript is setup and what the fields are called in your collection data extension.

Each time the email template is sent to a customer, the EmailSentDate is populated with the system date for the send. The SubscriberKey is whatever value is being used for the subscriber in the data extension setup. Because this uses an Update Contact activity, the vawp data extension is also configured for 'Used for Sending' and SubscriberKey relates to subscriber on Subscriber Key. If you're not using a numeric or text based ID key for your SubscriberKey and are using a customer's email address as their SubscriberKey then you don't need to have both in the data extension nor AMPScript. In the example below, we have a field called person_id that is a numeric custom key that relates to subscriber on SubscriberKey.

```
%%[

var @person_id, @vawp, @program, @emailtemplatename, @sentdate
set @per_id = AttributeValue("person_id")
set @emailtemplatename = AttributeValue("emailtemplatename")
set @program = "ProgNameGoesHere"
set @vawp = view_email_url
set @sentdate = now()

InsertDE ("TOD_vawplog",
     "EMAIL", Email,
     "EmailTemplateName", emailname_,
     "person_id", _subscriberkey,
     "vawp", @vawp,
     "Program", @program,
     "SentDate", @sentdate)

]%%
```

One of the drawbacks to using this logging method is that when a user runs a preview test using a subscriber's dynamic content, that test send gets picked up by this script in the data extension. The source data extension fields in the journey and the data fields used by the vawp data extension need to be named the same. And, there appears to be a limit on the number of dynamic fields that can be picked up by the AMPScript. Though, I have not fully tested to see how many dynamic fields I can upsert into a table from an AMPScript.

What is unique about this AMPScript or the vawp link that is captured is that a) no one had to write SQL to get it, b) each vawp link is unique to the customer receiving it, and c) when the link is copied and pasted into a browser search bar, what the customer saw at the time of the email is shown in the browser and this includes any and all dynamic content that was present at the time of the send.

Sort of using SQL to grab and store dynamic content that comes from an external source to the SFTP, I am unaware of any other out-of-the-box methods to store and retain dynamic content used in an email template.

Who uses this vawp data? Customer service when a customer calls in to inquire about an email they just received.
