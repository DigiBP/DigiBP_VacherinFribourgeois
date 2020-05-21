# Introduction

We are a yacht manufacturing company and need to supply pieces for building yachts. Below the as-is supplier selection process can be seen.

<img src="C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\modelling\Supplier selection as-is_200326.svg" style="zoom:100%;" />

One can see that nearly all tasks are user tasks, hence a person is involved in each step.

When digitalizing the process our goal was to automate process steps and reduce user and manual tasks to the minimum. When deciding about ways to do this we focused on using all the required technologies but also on keeping a pragmatic approach while implement things as realistically as possible. This was sometimes challenging. To not blow up the scope, we excluded the proof-of concept phase in the to-be process.



The final to-be process looks as following:

![](C:\Users\celia\OneDrive\Dokumente\01_FHNW\Semester 4\DigiBP\Repository Group Work\DigiBP_VacherinFribourgeois\modelling\Supplier selection to-be_200521.svg)



The process is split into 3 parts because the second process is is executed for each supplier response, while the first and the third part are only executed once per purchasing request.

For an overview, the image below shows all Integromat scenarios which we used in the process:

![image-20200521144822453](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521144822453.png)

# Process description

## Conversational AI as entry point

The process starts with a chatbot. 

![image-20200521144352346](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521144352346.png)



The user is being asked for the product he wants to order, the requirements, the budget and the preferred delivery date. After the chatbot has received all required information, the data is stored in a Google sheet.

In the screenshot below, we the training phrases of the chatbot are visible:

![image-20200521141934442](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521141934442.png)

The following image shows the parameters used:

![image-20200521142106018](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521142106018.png)

Now each step in the Camunda process is explained in the following sub-chapters. The titles refer to the task in the Camunda process.

## Purchasing requirement arised (1.1 in Integromat)

The webhook in Integromat receives the data from Dialogflow and inserts it into the google sheet "PurchaseRequisition" into the columns

- ComponentCategory (@CompoentCategory)
- Requirements (@sys.any)
- Budget (@sys.number)
- PreferredDeliveryDate (@sys.date)

Then Integromat creates a business key variable, which is also stored in a google sheet. Finally, via an HTTP request, the key-value pairs are posted to Camunda, which initiates the process.

![image-20200521151730726](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521151730726.png)

## Check if preferred supplier is available

This task checks, whether there is an existing preferred supplier for the specified product category, received in the previous step. This is done with the help of a decision table. 

![image-20200521152240353](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521152240353.png)

If the component category is for example, sails, mast or wood, it results in true, meaning we already have a preferred supplier. This leads to the next task "Use existing supplier". If there is no existing preferred supplier, for example for wheels or everything else, the process goes on to "Identify supplier".

## Use existing supplier

To be done!

## Identify suppliers

Via a common Google search, the strategic purchaser looks for potential suppliers. She/he enters the email addresses to contact the suppliers into a Google sheet named "supplierEmail". 

## Send request for information (1.2 in Integromat)

The following Integromat scenario gets the email address from the sheet "supplierEmail" and the requirements from the sheet "ProductRequirements". This information is placed into an email and which is sent to the potential supplier.

![image-20200521154445112](C:\Users\celia\AppData\Roaming\Typora\typora-user-images\image-20200521154445112.png)

## 