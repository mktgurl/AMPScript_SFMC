This script example looks at a data extension where there are multiple records 
for an email address. Let's say you're service provider and you have customers 
that have both business and residential street addresses. But, you want your
email update to say that your new policy change affects all the addresses 
associated with a unique customer email address. This is one way how it can 
be done.

In this example, two data extensions of the same audience segment are used to
create the desired output of 1 customer email adddress plus multiple service
addresses. In the email journey, the data source data extension contains a
list of customers by unique email addresses. The data extension in the email
template uses its companion data extension which has all the addresses and
duplicate emails. Whenever the journey sends an email, it grabs the address
list that matches the unique email address of the journey source data
extension.

Here's how the AMPScript looks like in the email body:

~~~
%%[ 
  var @rows, @row, @rowCount, @numRowsToReturn, @email,@i, @template 
  set @email = AttributeValue("email") /* value from attribute or DE column in send context */ 
  set @template = AttributeValue("template") 
  set @numRowsToReturn = 0 /* 0 means all, max 2000 */ 
  set @rows = LookupOrderedRows("DataExtensionName", @numRowsToReturn, "Email, ServStreet,ServCity, Template", "Email", @email, "Template", @template) 
  set @rowCount = rowcount(@rows) set @rebatesum = 0 if @rowCount > 0 then 
]%%
<ul>
%%[ 
  for @i = 1 to @rowCount do 
]%% 
%%[ 
  set @row = row(@rows, @i) /* get row based on counter */ 
  set @email2 = field(@row,"email") 
  set @servstreet = field(@row,"servstreet") 
  set @pcservstreet = ProperCase(@servstreet) 
  set @servcity = field(@row,"servcity") 
  set @pcservcity = ProperCase(@servcity) 
]%%
  <li style="color:#252525; font-weight:normal; font-family:arial, helvetica, sans-serif; line-height:24px; font-size:14px;">%%=v(@pcservstreet)=%%, %%=v(@pcservcity)=%%</li>
%%[ 
next @i 
]%%
</ul> 
%%[ 
endif 
]%%
~~~
