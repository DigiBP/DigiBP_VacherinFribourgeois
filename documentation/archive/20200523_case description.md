# Introduction

This documentation describes the digitalization of a fictional supplier selection process of a yacht manufacturing company.

Below the as-is supplier selection process can be seen.



![]()

One can see that nearly all tasks are user tasks, hence a person is involved in each step.

When digitalizing the process our goal was to automate process steps and reduce user and manual tasks to the minimum. When deciding about ways to do this we focused on using all the required technologies but also on keeping a pragmatic approach while implement things as realistically as possible. This was sometimes challenging. To not blow up the scope, we excluded the proof-of concept phase in the to-be process.

The final to-be process looks as following:

![]()



The process is split into 3 parts because the second process is is executed for each supplier response, while the first and the third part are only executed once per purchasing request.

For an overview, the image below shows all Integromat scenarios which we used in the process:

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Integromat%20Overview.png?raw=true)

# Process description

## Conversational AI as entry point

The process starts with a chatbot. 

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Dialogflow%20PurchaseRequisisitionService.png?raw=true">



The user is being asked for the product he wants to order, the requirements, the budget and the preferred delivery date. After the chatbot has received all required information, the data is stored in a Google sheet.

In the screenshot below, the training phrases of the chatbot are visible:

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Training%20phrases.png?raw=true">

The following image shows the parameters used:

<img src="https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Dialogflow%20parameters.png?raw=true">

Now each step in the Camunda process is explained in the following sub-chapters. The titles refer to the task in the Camunda process.

## Purchasing requirement arised (1.1 in Integromat)

- Step 1: The webhook in Integromat receives the data from Dialogflow
- Step 2: the data is inserted it into the google sheet "PurchaseRequisition" into the columns
  - ComponentCategory (@ComponentCategory)
  - Requirements (@sys.any)
  - Budget (@sys.number)
  - PreferredDeliveryDate (@sys.date)
- Step 3: Then Integromat creates a business key variable
- Step 4: Business key variable is stored in a google sheet
- Step 5: Via an HTTP request, the key-value pairs are posted to Camunda, which initiates the process

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/1.1.PNG?raw=true">

## Check if preferred supplier is available

This task checks, whether there is an existing preferred supplier for the specified product category, received in the previous step. The variable componentCategory is injected into the decision table.  

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/DMN%20supplier%20available.png?raw=true)

If the component category is for example, sails, mast or wood, it results in true, meaning we already have a preferred supplier. This leads to the next task "Use existing supplier". If there is no existing preferred supplier, for example for wheels or everything else, the process goes on to "Identify supplier".

## Use existing supplier

To be done!

## Identify suppliers

Via a common Google search, the strategic purchaser looks for potential suppliers. She/he enters the email addresses to contact the suppliers into a Google sheet named "supplierEmail". For simplification purpose for the testing, this step was already done including 3 fictious email addresses - from suppliers Fritz, Hans and Lisa. 

## Send request for information (1.2 in Integromat)

The following Integromat scenario is triggered by Camunda. It gets the email address from the sheet "supplierEmail" and the component category, requirements and preferred delivery date from the sheet "ProductRequirements". This information is placed into an email and which is sent to the potential supplier. Our strategic purchaser is called Laurin and he has a Gmail account with the address buyer.laurin@gmail.com. The emails are being sent from this account.

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/1.2.PNG?raw=true)



This is the email being sent:

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/1.2%20message.png?raw=true">

## Simulated supplier response (2.1 and 2.2 in Integromat)

As this is not a real-life process with actual suppliers, our suppliers responses need to be simulated. For this purpose we created another Integromat scenario, where our virtual suppliers Fritz, Hans and Lisa make an offer via email. The scenario needs to be executed manually. Again, strategic purchaser Laurin receives the responses.

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.1.PNG?raw=true)

So that it is possible to further process the responses by machine, the responses all have the same structure:

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.1%20message.png?raw=true)



This could also be achieved in an actual case by providing a template for the RfX response. Afterwards, the needed information is extracted from the email in the following scenario. It runs every 15 minutes when turned on.



![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.2.PNG?raw=true)

Let us explain this step-by step. Every incoming supplier response goes through this process.

- Step 1: Email is being fetched and marked as read in the inbox
- *Step 2: Reset a value (Never) ??*
- Step 3-6: By the means of Regex expressions we extract **price** and **experience**
- Step 7-8: **price** and **experience** are put into variables.
- Step 9: The **business key** is being read from the corresponding Google sheet
- Step 10: email address, price, experience and business key are sent to the Camunda process via an HTTP POST request 

## Check industry experience of potential suppliers

To evaluate, whether the supplier is suitable for us, we first want to check the industry experience. This is important because we want our suppliers have experience in delivering the high quality and safe products  a yacht needs. We demand a minimum experience of 3 years.

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/DMN%20shortlist.png?raw=true)



## Reject supplier via email (Integromat 2.4)

If the supplier has less than 3 years experience, he is automatically rejected. The process accesses the following Integromat scenario, where an email with rejection notice is sent to the supplier.

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.4.PNG?raw=true)



The supplier is reject with the following message:



![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.4%20message.PNG?raw=true)



## Show responses (Integromat 2.3)

If the supplier has 3 or more years of industry experience, then **email address**, **price** and **industry experience** are extracted from the message and put into the Google sheet "SupplierResponse". Additionally, the time stamp is stored in another column.



![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.3.PNG?raw=true)

The data is filled in the excel as follows:



<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.3%20excel.PNG?raw=true" align="center">



## Select best supplier

As the pricing varies from tender to tender and the rating of supplier depends on the **price** and **industry  experience** combination, the best supplier selection remains a user task. The user chooses the best combination of **price** and **industry experience**. The selected best supplier email address is inserted to the form selectedEmail in Camunda. 

--> How to include DB selected supplier responses?

--> How to insert images? 

## Invite supplier for contract negotiation via email and reject other suppliers (Integromat 3.1)



If the supplier was selected as best supplier in the previous step, the Invitation for contract negotiation is sent via Integromat. If the supplier was not awarded best supplier, an Offer rejected email is being generated via Integromat. 

--> What exactly does the router in Integromat? 

--> How to insert images? 

## Negotiate contract terms

The contract terms are being negotiated with the best selected supplier in a manual task. This is the final task of the supplier selection process. 

--> How to insert images? 