# Introduction

We are a yacht manufacturing company and need to supply pieces for building yachts. Below the as-is supplier selection process can be seen.

<img src="C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\modelling\Supplier selection as-is_200326.svg" style="zoom:100%;" />

One can see that nearly all tasks are user tasks, hence a person is involved in each step.

When digitalizing the process our goal was to automate process steps and reduce user and manual tasks to the minimum. When deciding about ways to do this we focused on using all the required technologies but also on keeping a pragmatic approach while implement things as realistically as possible. This was sometimes challenging. To not blow up the scope, we excluded the proof-of concept phase in the to-be process.



The final to-be process looks as following:

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\modelling\Supplier selection to-be_200521.svg)



The process is split into 3 parts because the second process is is executed for each supplier response, while the first and the third part are only executed once per purchasing request.

For an overview, the image below shows all Integromat scenarios which we used in the process:

![image-20200521155356033](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521155356033.png)

# Process description

## Conversational AI as entry point

The process starts with a chatbot. 

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\Dialogflow PurchaseRequisisitionService.png)



The user is being asked for the product he wants to order, the requirements, the budget and the preferred delivery date. After the chatbot has received all required information, the data is stored in a Google sheet.

In the screenshot below, we the training phrases of the chatbot are visible:

![image-20200521141934442](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521141934442.png)

The following image shows the parameters used:

![image-20200521142106018](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521142106018.png)

Now each step in the Camunda process is explained in the following sub-chapters. The titles refer to the task in the Camunda process.

## Purchasing requirement arised (1.1 in Integromat)

The webhook in Integromat receives the data from Dialogflow and inserts it into the google sheet "PurchaseRequisition" into the columns

- ComponentCategory (@ComponentCategory)
- Requirements (@sys.any)
- Budget (@sys.number)
- PreferredDeliveryDate (@sys.date)

Then Integromat creates a business key variable, which is also stored in a google sheet. Finally, via an HTTP request, the key-value pairs are posted to Camunda, which initiates the process.

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\1.1.PNG)

## Check if preferred supplier is available

This task checks, whether there is an existing preferred supplier for the specified product category, received in the previous step. The variable componentCategory is injected into the decision table.  

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\DMN supplier available.png)

If the component category is for example, sails, mast or wood, it results in true, meaning we already have a preferred supplier. This leads to the next task "Use existing supplier". If there is no existing preferred supplier, for example for wheels or everything else, the process goes on to "Identify supplier".

## Use existing supplier

To be done!

## Identify suppliers

Via a common Google search, the strategic purchaser looks for potential suppliers. She/he enters the email addresses to contact the suppliers into a Google sheet named "supplierEmail". 

## Send request for information (1.2 in Integromat)

The following Integromat scenario is triggered by Camunda. It gets the email address from the sheet "supplierEmail" and the component category, requirements and preferred delivery date from the sheet "ProductRequirements". This information is placed into an email and which is sent to the potential supplier. Our strategic purchaser is called Laurin and he has a Gmail account with the address buyer.laurin@gmail.com. The emails are being sent from this account.

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\1.2.PNG)



This is the email being sent:

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\Email invite.png)

## Simulated supplier response (2.1 in Integromat)

As this is not a real-life process with actual suppliers, our suppliers responses need to be simulated. For this purpose we created another Integromat scenario, where our virtual suppliers Fritz, Hans and Lisa make an offer via email. The scenario needs to be executed manually. Again, strategic purchaser Laurin receives the responses.

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\2.1.PNG)

So that it is possible to further process the responses by machine, the responses all have the same structure:

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\Email answer.png)

## Check industry experience of potential suppliers

To evaluate, whether the supplier is suitable for us, we first want to check the industry experience. This is important because we want our suppliers have experience in delivering high quality and safe products which a yacht needs. We demand a minimum experience of 3 years.

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\DMN shortlist.png)

If the supplier has less than 3 years experience, he is automatically rejected in the step "Reject supplier via email"

## Reject supplier via email (Integromat 2.4)



![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\2.4.PNG)



## Show responses (Integromat 2.3)

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\documentation\pictures\2.3.PNG)