# Using Sentiment Analysis, Machine Learning in D365 to predict Win/Loss of opportunity

Welcome to the BA-SA-ST204 wiki! This page will show you all the steps that are required to complete the lab. Let's get started. 

Here are the steps you need to follow for this lab:

## 1. Dynamics 365 CE Customization
We will set up things required in Dynamics 365 CE in this section. Nothing major, we'll just a couple of fields for storing Text Sentiment Analysis data for each opportunity.

* Logic to Dynamics 365 CE using test credentials provided.
* Navigate to Settings -> Solutions
* Click on import, and navigate to Opportunities_SentimentAnalysis_1_0_0_0.zip
* Click on Next
* Click on Import
* After import is completed successfully, click on Publish All Customizations.
* Verify that Opportunity entity has the new fields created and added to its main form: Customer Need Sentiment Score, Current Situation Sentiment Score, Proposed Solution Sentiment Score, Notes Average Sentiment Analysis Score

![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(317).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(318).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(319).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(320).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(314).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(315).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(313).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(316).png)


## 2. Text Analytics service creation in Azure.
We will not create Text Sentiment Analytics API in Azure now. 

* Login to Azure Portal and click on Create new Resource on top left.
* Search for text analytics in the new resource search bar.
* Select Text Analytics from the results.
* Click on Create
* Fill in name, Location as West US, Pricing as F0.
* For resource group, Click on Create new and fill in the Resource Group name in this format - BA-SA-ST204-YourAlias
* Click on Create

![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(321).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(322).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(323).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(324).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(325).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(326).png)


## 3. Logic App creation in Azure, to calculate Text Sentiment Analytics of fields on Opportunity Entity and related notes, and update the same back in Dynamics 365 CE.
We will now create a Logic App that will use the Text Sentiment Analytics API to calculate Sentiment Score of required fields of Opportunity entity and also of the related notes for each opportunity.

* Login to Azure Portal and click on Create new Resource on top left.
* Search for Logic App in the new resource search bar.
* Select Logic App from the results.
* Click on Create
* Fill in name, Location as West Central US.
* For resource group, select the resource group you created earlier - BA-SA-ST204-YourAlias
* Click on Create
* Navigate to Logic App Designer from left pane, and then select Recurrence.
* Set Interval to 1, and Frequency to Day.
* Create variable with name Opportunity of type Object.
* Create variable with name NoteText of type String. 
* Create variable with name Opportunity of type Object.
* Create variable with name SumOfSentimentScore of type Float, and initiative it with 0.
* Create variable with name Count of type Integer, and initiative it with 0.
* Create variable with name SumOfSentimentScore of type Float, and initiative it with 0.
* Create variable with name Average Sentiment Score of type Float, and initiative it with 0.
* Add a new step, of type Dynamics 365 and then select List Records (Preview) from Actions.
* Select your Dynamics 365 CE instance in Organization Name, and select Opportunities entity in the Entity name field.
* Add a new step and select Foreach from Control Built-In section.
* Select value from dynamic content popup for the output from previous step field in foreach action.
* Add a new action inside foreach, and select text analytics -> Detect Sentiment (Preview). If your connection is not already populated in the Connection dropdown, click on Add new Connection, and give a name and your Account Key from the Text Analytics service.
* Once you are connected, select Current Situation field from Dynamic Content popup.
* Add a new Text Analytics action, and select Customer Need field from from Dynamic Content popup.
* Add a new Text Analytics action, and select Proposed Solution field from from Dynamic Content popup.
* Add a new Set Variable step, and paste below JSON in the value field.

`{
  "CurrentSituation": "",
  "CustomerNeed": "",
  "OpportunityId": "",
  "ProposedSolution": ""
}`

* For the value of CurrentSituation field in JSON, set the Score output from Detect Sentiment step of Current Situation field.
* For the value of CustomerNeed field in JSON, set the Score output from Detect Sentiment step of Customer Need field.
* For the value of OpportunityId field in JSON, set the OpportunityId field from Dynamic Content popup.
* For the value of ProposedSolution field in JSON, set the Score output from Detect Sentiment step of Proposed Solution field.
* Add a new step and select Foreach from Control Built-In section. Select Opportunities variable from dynamic content popup for the output from previous step field in foreach action.
* Add a new step, of type Dynamics 365 and then select List Records (Preview) from Actions.
* Select your Dynamics 365 CE instance in Organization Name, and select Notes entity in the Entity name field. Add a filter query for filtering by _objectid_value to be equal to OpportunityId of the current item in loop.
* Add a new step and select Foreach from Control Built-In section.
* Select value from dynamic content popup for the output from previous step field in foreach action.
* Click on three dots at right end of Foreach and go to settings.
* Enable Concurrency Control and set Degree of Parallelism to 1. Then click on Done.
* Set variable NoteText with Description field from Note Entity record.
* Add a Text Analytics step,  and pass the variable NoteText to it as input.
* Increment SumOfSentimentScore variable with score output of the NoteText Detect Sentiment step.
* Increment Count variable by 1.
* Outside the Notes Foreach loop, add a condition to check whether count variable is greater than 1 or not.
* Now inside the True section of the condition step, add a set variable action, and set the Average Sentiment Score variable to this expression:

`div(variables('SumOfSentimentScore'), variables('Count'))`

* Add a set variable step, and reset the County variable to 0.
* Now add a Update Dynamics 365 record action, and set Entity name to Opportunities, and record identifier to OpportunityId of the current loop item.
* Update fields Current Situation Sentiment Score, Customer Need Sentiment Score and Proposed Solution Sentiment Score  with CurrentSituation, CustomerNeed, ProposedSolution value of the current loop item.
* Update field Notes Average Sentiment Analysis Score with value of SumOfSentimentScore variable.
* Reset SumOfSentimentScore variable value to 0 in the next step.
* Now inside the False section of the condition step, we need to add a Update Dynamics 365 record action, and there we need to Update only fields Current Situation Sentiment Score, Customer Need Sentiment Score and Proposed Solution Sentiment Score in CRM. We dont have to set field Notes Average Sentiment Analysis Score in this section.
* Save the Logic App.


![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(321).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(331).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(332).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(333).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(334).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(341).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(335).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(338).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(339).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(352).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(340).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(353).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(342).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(344).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(345).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(343).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(346).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(347).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(348).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(349).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(350).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(351).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(354).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(456).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(455).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(356).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(357).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(358).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(359).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(360).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(361).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(362).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(365).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(370).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(371).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(372).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(369).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(373).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(374).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(375).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(376).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(377).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(378).png)


## 4. Azure Machine Learning Studio Workspace creation in Azure.
Now we will create Azure Machine Learning Studio workspace in Azure.

* Login to Azure Portal and click on Create new Resource on top left.
* Search for machine learning in the new resource search bar.
* Select Machine Learning Studio Workspace from the results.
* Click on Create
* Fill in Workspace name, Storage Account Name, Location as South Central US, Workspace Pricing Tier as Standard, and Web Service plan pricing tier as S1 Standard.
* For resource group, select the resource group you created earlier - BA-SA-ST204-YourAlias
* Click on Create

![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(321).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(327).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(328).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(329).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(330).png)

## 5. Machine Learning Experiment creation in Azure ML Studio and train it using sample data
Now we will create Machine Learning Experiment Studio workspace in Azure.

* Login to Azure Portal and go to your Machine Learning Studio Workspace. Click on Lauch Machine Learning Studio.
* Click on Sign In, it should automatically log you into ML Studio.
* Go to datasets from left navigation bar, and click on New
* Select From File, navigate to the file ClosedOpportunitiesData.csv and click on Ok button. Your csv file will be imported and will be available under Datasets.
*  Navigate to Experiments from left navigation, and click on New.
* Select Blank Experiment
* Add ClosedOpportunitiesData.csv from saved dataset into your experiment.
* Add clean missing data component from Data Transformation -> Manipulation -> Clean Missing Data
* Save your experiment and Run it.
* Add edit metadata component from Data Transformation -> Manipulation -> Edit Metadata
* Select Edit Metadata component that you added to experiment, click on Launch column selector.  Add Potential Customer, Account, Decide Go/No-Go, Rating and Status to Selected Columns to right from list of available columns and click on OK button. Once the popup window closes, from right panel, select Make Categorical option from Categorical dropdown.
* Add split data component from Data Transformartion -> Sample and Split -> Split Data
* Add Two-Class Bayes Point Machine and Two-Class Boosted Decision Tree classification models from Machine Learning -> Classification
* Add two Train Model components from Machine Learning -> Train -> Train Model, just below each classification model components added in previous step.
* For both of the Train Model components, select them, click on Launch Column Selector on right and select Status in the list of Selected Columns in the popup. Click on OK button.
* Add two Score Model components from Machine Learning -> Score -> Score Model. 
* Add a final Evaluate Model component from Machine Learning -> Evaluate -> Evaluate Model.
* Connect the components as shown in the screenshots below.
* Save and Run the Experiment.
* Right Click on Evaluate Model Component, go to Evaluation Results -> Visualize

![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(379).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(380).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(385).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(386).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(388).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(389).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(381).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(382).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(391).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(392).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(396).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(398).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(400).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(401).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(402).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(404).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(415).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(416).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(417).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(418).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(419).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(420).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(423).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(424).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(425).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(426).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(427).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(428).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(429).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(430).png)


## 6. Convert the predictive machine learning experiment to Web Service, Deploy and test the same.
Now we will convert the training experiment created in previous section to a Predictive Experiment, then deploy it as a Web Service and finally we will also test the same.

* Go to your training experiment, and run it.
*  There's a button on bottom of page, Set up Web Service. Hover on it, and click on Predictive Web Service (Recommended).
* You will get a popup asking you to select a train model. This is because we have more than 1 train model components in out training experiment.
* Select the train model associated to Two-Class boosted decision tree component, and then click on Set up web Service -> Predictive Web Service (Recommended).
* ML Studio will automatically convert your training experiment into predictive experiment.
* Save and Run the Predictive experiment.
* Click on Deploy Web Service -> Deploy Web Service (Classic) button from bottom of page.
* ML Studio will deploy your predictive experiment as a web service, and open the Web Service Dashboard for you.
* Click on Test (Preview) next to Request/Response.
* ML Studio will open the test page for your web service with input fields. You will notice that the input fields are currently empty, if you want to populate them with sample data, click on Enable from Sample Data section.
* Click on Test Request Response at bottom of page. You will get web service response just right next to the request fields.

![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(431).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(432).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(433).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(436).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(437).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(438).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(439).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(440).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(441).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(442).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(443).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(444).png)


## 7. Creating final Logic App that will run weekly for all Open Opportunities and update their prediction back in Dynamics 365 CE.
This is the last section of this Lab, here we will be developing a Logic App, which will go through all the OPEN opportunities in Dynamics 365 CE, and use their data to do prediction of Outcome using ML Web Service, and finally update back the prediction in Dynamics 365 CE.

* Login to Azure Portal and click on Create new Resource on top left.
* Search for Logic App in the new resource search bar.
* Select Logic App from the results.
* Click on Create
* Fill in name, Location as Central US.
* For resource group, select the resource group you created earlier - BA-SA-ST204-YourAlias
* Click on Create
* Navigate to Logic App Designer from left pane, and then select Recurrence.
* Set Interval to 1, and Frequency to Day.
* Create variable with name ML Outcome of type String.
* Add a new step, of type Dynamics 365 and then select List Records (Preview) from Actions. Select your Dynamics 365 CE instance in Organization Name, and select Opportunities entity in the Entity name field. Click on Show Advanced Options in the List Records action, and then add this text into Filter Query field - 

`statecode eq 0 and name ne null and actualvalue ne null and budgetamount ne null and new_currentsituationsentimentscore ne null and new_proposedsolutionsentimentscore ne null and new_notesaveragesentimentanalysisscore ne null and new_customerneedsentimentscore ne null and opportunityratingcode ne null`

* Add a new step and select Foreach from Control Built-In section. Select value from dynamic content popup for the output from previous step field in foreach action.
* Add a new step and select HTTP Request from actions.
* In url, add the API URL for your ML Web Service - Request Response API.
* Add following 3 headers as per the key:value pairs mentioned below:

`Accept:application/json`

`Content-Type:application/json`

`Authorization:Bearer Your-ML-Web-Service-API-Key`

* Add the following JSON in Body field, and in the Values array, add CRM fields according to ColumnNames array.
e.g. Topic should be the first field added to Values array, Potential Customer seconed, and so on.

`{
  "GlobalParameters": {},
  "Inputs": {
    "input1": {
      "ColumnNames": [
        "Topic",
        "Potential Customer",
        "Account",
        "Est. Revenue",
        "Actual Revenue",
        "Budget Amount",
        "Current Situation Sentiment Score",
        "Customer Need Sentiment Score",
        "Proposed Solution Sentiment Score",
        "Notes Sentiment Score",
        "Decide Go/No-Go",
        "Rating",
        "Status"
      ],
      "Values": [
        [
          "",
          "",
          "",
          "",
          "",
          "",
          "",
          "",
          "",
          "",
          "",
          "",
          ""
        ]
      ]
    }
  }
}`

* Add a new Parse JSON action in the next step. In content, pass the Body from response of previous HTTP step. In schema field, paste this JSON:

`{
    "Results": {
        "output1": {
            "type": "table",
            "value": {
                "ColumnNames": [
                    "Topic",
                    "Potential Customer",
                    "Account",
                    "Est. Revenue",
                    "Actual Revenue",
                    "Budget Amount",
                    "Current Situation Sentiment Score",
                    "Customer Need Sentiment Score",
                    "Proposed Solution Sentiment Score",
                    "Notes Sentiment Score",
                    "Decide Go/No-Go",
                    "Rating",
                    "Status",
                    "Scored Labels",
                    "Scored Probabilities"
                ],
                "ColumnTypes": [
                    "String",
                    "String",
                    "String",
                    "Int32",
                    "Int32",
                    "Int32",
                    "Double",
                    "Double",
                    "Double",
                    "Double",
                    "String",
                    "String",
                    "String",
                    "String",
                    "Double"
                ],
                "Values": [
                    [
                        "value",
                        "value",
                        "value",
                        "0",
                        "0",
                        "0",
                        "0",
                        "0",
                        "0",
                        "0",
                        "value",
                        "value",
                        "value",
                        "Lost",
                        "0.102679125964642"
                    ]
                ]
            }
        }
    }
}`

* Set variable ML Outcome with this expression - 

`body('Parse_JSON')?['Results']?['output1']?['value']?['Values'][0][13]`

* Add a new Update Record action from Dynamics 365 in next step. Select your instance name in Organization field, select opportunities in Entity Name field. For Record Identifier field, pass the OpportunityId of the current loop item.
* Update the ML Outcome Prediction field with value of ML Outcome variable.
* Save the logic app.


![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(445).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(446).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(447).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(448).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(449).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(450).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(451).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(452).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(453).png)
![](https://github.com/crazyLearning/BA-SA-ST204/blob/master/images/Screenshot%20(454).png)



