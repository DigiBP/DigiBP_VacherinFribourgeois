# Introduction

This documentation describes the digitalization of a fictional supplier selection process of a yacht manufacturing company.

Below the as-is supplier selection process can be seen.

![](https://raw.githubusercontent.com/DigiBP/DigiBP_VacherinFribourgeois/79a97cec5bead50bf80db247e18f1ce9683b97e8/documentation/pictures/Supplier%20selection%20as-is_200326.svg)

One can see that nearly all tasks are user tasks, hence a person is involved in each step.

When digitalizing the process our goal was to automate process steps and reduce user and manual tasks to the minimum. When deciding about ways to do this we focused on using all the required technologies but also on keeping a pragmatic approach while implement things as realistically as possible. This was sometimes challenging. To not blow up the scope, we excluded the proof-of concept phase in the to-be process.

The final to-be process looks as following:

![](https://raw.githubusercontent.com/DigiBP/DigiBP_VacherinFribourgeois/f9fdd23439cee49da05be3f62ebc365375c7e332/documentation/pictures/Supplier%20selection%20to-be_200528.svg)



The process is split into 2 parts because the second process is is executed for each supplier response, while the first part is only executed once per purchasing request. Additionally, this approach represents the microservices approach, which allows processes to be reused in other processes. 

For an overview, the image below shows all Integromat scenarios which we used in the process:

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Integromat%20Overview.png?raw=true)

# Process description

## Conversational AI as entry point

The process starts with a chatbot. 

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/videos/screen-video-chatbot.gif?raw=true">



The user is being asked for the product he wants to order, the requirements, the budget and the preferred delivery date. After the chatbot has received all required information, the data is stored in a Google sheet.

In the screenshot below, the training phrases of the chatbot are visible:

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Training%20phrases.png?raw=true">

The following image shows the parameters used:

<img src="https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/Dialogflow%20parameters.png?raw=true">

Now each step in the Camunda process is explained in the following sub-chapters. The titles refer to the task in the Camunda process.

## Purchasing requirement arised (Integromat 1.1)

- Step 1: The webhook in Integromat receives the data from Dialogflow
- Step 2: the data is inserted it into the Google sheet "PurchaseRequisition" into the columns
  - ComponentCategory (@ComponentCategory)
  - Requirements (@sys.any)
  - Budget (@sys.number)
  - PreferredDeliveryDate (@sys.date)
- Step 3: Integromat creates a business key variable
- Step 4: Business key variable is stored in a Google sheet
- Step 5: Via an HTTP request, the key-value pairs are posted to Camunda, which initiates the process

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/1.1.PNG?raw=true">

## Check if preferred supplier is available

This task checks, whether there is an existing preferred supplier for the specified product category, received in the previous step. The variable componentCategory is injected into the decision table.  

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/DMN%20supplier%20available.png?raw=true)

If the component category is for example mast or wood, it results in true, meaning we already have a preferred supplier. This leads to the next task "Show preferred supplier". If there is no existing preferred supplier, for example for wheels or everything else, the process goes on to "Identify supplier".

## Show preferred supplier 

In this process step, the preferred supplier from the decision table output and the component category from the chatbot entry are displayed. The use of the preferred supplier leads to a separate process "use existing supplier", which is not in the scope of this project.  

![]()

## Identify suppliers

Via a common Google search, the strategic purchaser looks for potential suppliers. She/he enters the email addresses to contact the suppliers into a Google sheet named "supplierEmail". For simplification purpose for the testing, this step was already done including 3 fictious email addresses - from suppliers Fritz, Hans and Lisa. Hereby the first process is finished with "Supplier defined". 

## Send request for information (Integromat 1.2)

The following Integromat scenario is triggered by Camunda. It gets the email address from the sheet "supplierEmail" and the component category, requirements and preferred delivery date from the sheet "PurchaseRequisition". This information is placed into an email and which is sent to the potential suppliers. Our strategic purchaser is called Laurin and he has a Gmail account with the address buyer.laurin@gmail.com. The emails are being sent from this account to the suppliers Fritz, Hans and Lisa. 

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/1.2.PNG?raw=true)



This is the email being sent:

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/1.2%20message.png?raw=true">

Hereby the first process is finished with "Supplier identified". 

## Simulated supplier response (Integromat 2.1 and 2.2)

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
- --> To be extended!!!

## Check industry experience of potential suppliers

To evaluate, whether the supplier is suitable for us, we first want to check the industry experience. This is important because we want our suppliers to have experience in delivering the high quality and safe products  a yacht needs. We demand a minimum experience of 3 years.

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/DMN%20shortlist.png?raw=true)



## Reject supplier via email (Integromat 2.4)

If the supplier has less than 3 years experience, he is automatically rejected. The process accesses the following Integromat scenario, where an email with rejection notice is sent to the supplier.

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.4.PNG?raw=true)



The supplier is rejected with the following message:

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.4%20message.PNG?raw=true)

Hereby, the second process is completed with "Supplier rejected". 

## List responses (Integromat 2.3)

If the supplier has 3 or more years of industry experience, then **email address**, **price** and **industry experience** are extracted from the message. The **business key** is taken from the respective Google sheet and all variables are put into the Google sheet "supplierResponse", second sheet "Shortlist". Additionally, the **time stamp**, is stored in an additional column. 




![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.3.PNG?raw=true)

The data is filled in the Google sheet as follows:

<img src = "https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/2.3%20excel.PNG?raw=true" align="center">

Hereby, the second process is completed with "Supplier responses collected". 

## Select best supplier

As the pricing varies from tender to tender and the best supplier selection depends on the **price** and **industry  experience** combination, the best supplier selection remains a user task. The user chooses the best combination of **price** and **industry experience** from the Google sheet "supplierResponse", sheet "Shortlist". The selected best supplier email address is inserted to the form field "selectedEmail" in Camunda. 

--> insert Heroku-image

## Invite supplier for contract negotiation via email and reject other suppliers (Integromat 3.1)

If the supplier was selected as best supplier in the previous step, the invitation for contract negotiation is sent via Integromat. If the supplier was not awarded best supplier, an offer rejected email is being generated via Integromat. 

![](https://raw.githubusercontent.com/DigiBP/DigiBP_VacherinFribourgeois/master/documentation/pictures/3.1.PNG)

## Negotiate contract terms

The contract terms are being negotiated with the best selected supplier in a manual task. This is the final task of the supplier selection process. 

The process finishes with "Supplier selected". 

# User Management in Camunda

In Camunda we created a group of two strategic purchasers "strategicPurchasing", which includes the users Max Meier and Lihong Wang. All user tasks in our process can be claimed by the group "StrategicPurchasing".

![](https://github.com/DigiBP/DigiBP_VacherinFribourgeois/blob/master/documentation/pictures/UserTasks.PNG?raw=true)