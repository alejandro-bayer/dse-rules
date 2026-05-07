[Image description: Bayer corporate logo/header banner at the top of the document, likely featuring the Bayer cross logo in corporate colors as a title page header.]

## Model Playbook

[Image description: A decorative graphic or banner associated with the "Model Playbook" heading, possibly featuring an icon or illustration representing machine learning models, data science, or predictive analytics (such as neural networks, charts, or AI-related imagery).]

## What is a Model?

A model is tool for making predictions or decisions based on data. A model is created using the Model Development Lifecycle, which all can be executed through the DSE.

## DSE supports all phases of Model Lifecycle

- Model Development: Data Collection, Data Cleaning, Exploratory Data Analysis (EDA), Model Training & Validation
- Model Deployment and Invoking: Model Registration, Model Versioning, Model Deploying, Model Invoking, Model Managing (Monitoring & Maintenance)

## What types of Models does DSE support?

- Python
- R

The DSE Starter Kit, a CLI based tool that can be used locally or within DSE, comes with production-ready implementations of 16 common models written in both Python and R.

The rest of this documentation assumes that you have gone through the Onboarding Journey for DSE and know how to launch a DSE environment, launch your space in JupyterLab or RStudio, and connect to GitHub. If you need a refresher on basic DSE functionality and setup before starting model development, please click here .

## I Want To...

- [x]  Develop a Model

- [x]  Register a Model

 Deploy a Registered Model

- [x]  Connect Microsoft Entra to a Model

- [x]  Invoke a Deployed Model

 Manage a Model

- [x]  Review Model FAQ

- [ ]  View a Full Model Deployment Demo

## Model Development

Model development happens in the DSE Sandbox Environment. In a DSE Sandbox Environment you can read data from CSW or AWS S3, clean the data, do exploratory data analysis (EDA), and train and validate models. You'll find a demonstration in the recording on the right.

## Build and Evaluate Your Model

1. Load all model requirements.
- a. Particularly important is the DSE package, which connects to CSW and is important for model registry.

import dse_sdk or dse_sdk <-import("dse_sdk")

2. Ingest data, for example dse.read_csw() or dse.read_s3()

3. Explore the data for model evaluation.
4. Build and train your model and choose the model you'd like to utilize.

If you'd like to use a template, review the templates found in the DS Starter Kit .

## Write DSE Code for Registering Model

Now that you have established the model you'd like to use, it must be adapted to work within DSE. In short, a file should be created in the repo that uses your chosen model in one of the 2 pre-determined named functions. Tips for implementing successful code:

1. Must be written in a main.py or main.R file
2. Should have at least the following definitions:
- a. def train( ) or train ← function()
- i. Should be able to save trained model
- b. def model( )or model ← function()
- i. Should be able to load the saved model and do predictions
3. Optionally, but recommended include a requirements.txt

- a. Should house all of the versions or necessary packages required to ensure the model runs effectively.
4. Optionally, but recommended include input.jsd and output.jsd schema files so others can easily use your model

If you need assistance creating a schema, review How-To Create a Schema .

## Register a Model

## Quick registration steps

1. Register the Model in DSE once (creates the model's record).
2. Register the Model's first version .
3. After that, register new versions when you update the code.

## Lifecycle

- One-time setup
- Develop
- Register Model
- Test / troubleshoot
- Maintenance
- Update: register new version and replace deployment
- Remove Model (if needed)

## Recommended development workflow

- Test changes in dev or locally
- Make code changes on side/feature branches (for example feature/... or dev/...), not directly on main .
- Submit a pull request and get a trusted reviewer's approval before merging to main, to avoid introducing breaking changes.

- This minimizes regressions and deployment issues.

## Model Registration basics

1. Develop anywhere — You can build and test your Model in a local IDE or inside the DSE SageMaker space.
2. Register in SageMaker — You must be in the SageMaker space to register the Model with DSE; development and testing can happen elsewhere.
3. Repository layout — Your repo must have a main.py or main.R file and contain either def train() or def model() or model ←function() or train ← function().
4. Git Commit & Push Changes — Your workspace must be clean. Commit or stash any changes before attempting to register.
5. Increase Version on Change — Increment the version for every new release.

## Registering a Model Steps

The first two steps occur within a Jupyter notebook in your Sandbox environment:

1. Register the model with register_model()

Use appropriate metadata, including:

A. Display Name: You can only name a model once, any updates to the model should result in an update to the version, as outlined in step 2.

- B. Description: Anyone looking at your model should understand what it does and why it's there.

C. Visibility: choose between PUBLIC or PRIVATE

2. To register a model version, run register_model_version():

A. You are creating a new version and will see an error if you do not update the version_id.

- B. You can also set a subdirectory if the model's file is not in the root folder
3. Validate successful registration of the model.

A. In that Notebook, check the response status (see image on right).

- B. Check the Model section of the DSE Portal to see your model.
- a. Find your Model name.
- b. Click "Model Profile".
- c. Ensure that the version number on the right, under "Model Version(s)" is as expected.

C. In that Notebook, run dse_sdk.list_models() to get a list of models for this tenant

## Deploying a Registered Model

## Quick Deploy & Invoke steps

1. Deploy the Model in DSE, either in Non-Production or Production.
2. Create a connection between the Entra App and the Model
3. Invoke the Model in DSE

The DSE allows users to manage entitlements and costs associated with the discovery of a model and the deployment of a model separately. This is enabled by having separate environments designated for Sandbox and Model Deployment.

Below are useful resources for deploying models including the Model Deployment UI.

Refer to this example model registration and deployment repo with relevant scripts for further support. Reference the DSE SDK document for up-to-date functionality.

Click the image to the right for a demonstration of model deployment and invocation.

## How to Deploy a Registered Model

In the Models section of the DSE Portal, click on the Model Profile of the Model that you wish to deploy.

Deploy your model to Non-prod or Prod environments.

Best Practice: deployment to Prod environments should be reserved for models that have undergone appropriate review following the steps in the Decision Science Life Cycle.

After you hit the deploy button a popup will appear. Select the appropriate Target Environment you'd like the model deployed to and the appropriate model version you'd like deployed. Click "Request Deployment".

Your model will now display "Awaiting review" .

Wait for an individual who has approval entitlements in the PAPI Group associated with the model you wish to deploy to approve your deployment request.

You can monitor the status of your request in the Deployments section of the DSE Portal .

Once your model is successfully built and approved, it will be deployed and available for use within the DSE.

To manage the deployed model, go to the Models section of the DSE Portal and select the model by clicking on the Model Profile. There, you will be able to remove a deployed model, replace a deployed model with an alternative version, visit the model GitHub repository, identify PAPI entitlements associated with the model, and access the CREST Application that was created with the deployed model.

Every deployed model creates a unique CREST Application that is a Bayer-internal client authorization platform. It uses Microsoft Entra IDs to manage entitlements for model deployments.

## Establishing Connection Between Microsoft Entra Application and Your Model

To invoke a model, you must to establish the appropriate connection between a Microsoft Entra Application and the model you are trying to invoke. Strategize with departmental subject matter experts on the Microsoft Entra Application you will utilize for DSE Model Deployment purposes. For help finding the model you wish to invoke, reference How to Search and Interact with Models .

Follow the steps below:

## 1. Ensure you have a Microsoft Entra Application and have access to the Secret ID and Client ID .

If you don't have a Microsoft Entra Application, follow instructions here . (Business Owner, System Owner, or System Owner Delegate of the BEAT ID you ' re using for the Microsoft Entra Application must manage / create the application.)

Store the Secret ID and credentials for the application in a Bayer-approved secure location .

2. Contact the team who owns the target Model and ask them to add your Microsoft Entra Client ID to their Model's CREST Application invoke Restriction.

The owning team can find the link to their Application on their Model Profile Page in the DSE Portal.

Users who want to grant access to invoke the deployed Model must know the Entra Client ID of the caller, add that ID to the "invoke"-alike Entitlement Restriction on that CREST Application, then click "SAVE".

3. Assign the Microsoft Entra Application with invoke rights to the sandbox where you are attempting to invoke the model.
2. To add your credentials to your Sandbox, you must run a command on the command-line within your Sandbox, as there is not a DSE SDK helper function for it. DO NOT write the following code in your own Model code's repository; write it directly on the commandline -- otherwise you might end up pushing secret information to GitHub, which is very bad!
3. The command you need to run is:

dse-cli store-entra-credentials --clientid <YOUR_CLIENT_ID> --client-secret <YOUR_CLIENT_SECRET>

(NOTE: This only needs to be run once per Sandbox. If you have multiple team

members using the same Sandbox, they do not need to run this command again.)

Once you get the success message, you are ready to invoke the model.

## Invoking a Deployed Model

Invoking a model within the DSE means using a deployed model within a DSE Sandbox Environment to discover, develop, and generate insights on Bayer-related data assets.

Once you've established the appropriate connection between a Microsoft Entra Application and the model you are trying to invoke, gather information to use the invoke_model()function of the DSE SDK Model Deployment release.

The DSE SDK invoke_model() function (dse_sdk.invoke_model()for python, or dse_sdk$invoke_model()for R) requires four pieces of information:

- A. Model ID
- B. Model Version
- C. Environment Type
- D. Payload (i.e., new input data)

Your Payload (i.e., new input data), must be:

- A valid JSON string as expected by the target Model.
- Already converted to JSON before passing it to this function.

To understand the types of Payload that the model was trained to receive, you

## Model Management

Model Management enables Data Science teams to better collaborate across the DS Community, amplifying impact and efficiency by connecting projects' objectives across team lines to leverage work completed by any team. By centralizing our model catalog across Bayer Crop Science, we encourage teams to work in ways that fit standards and best practices.

## Best Practices

can use dse_sdk.fetch_model_schemas() and pass the model name and version and it will output the types of data the input is expecting.

Now run the function with the appropriate information to invoke the model.

- When a model is created, via registering a version, it zips up the workspace in the latest commit and creates a PAPI application.
- Utilize the "model repository" link on the Model UI (see image to right) to view the model repo.
- Be mindful of model name - once a model is registered, this cannot be changed.
- If you want new model name, you need to create a new repo and follow the Model registration steps.
- Each model version is tied to a specific commit, so the links under "Model Version(s)" will link you to the specific commit.
- Updates to the model code require updates to version number (name never changes).
- Follow good versioning practices .
- Owners can adjust private entitlements via the PAPI Application link on the Model UI.
- Be mindful about access/visibility – once the PAPI application is created, access/visibility cannot be changed.
- If the model is marked public, anyone can view it.
- If the model is private, only individuals within the PAPI groups that have entitlements for read/write will be able to do so for the model in question.
- Owners of the model must also be added to a given PAPI group to be included in the entitlements granted to that group.
- Follow repo DSE template best practices - choose easy-to-follow repo organization structures to facilitate collaboration and ease of understanding. Use the DS Starter Kit .
- Don't leave unnecessary/large git files in your repo; doing so can add to cost. Store data where it should be stored in the Crop Science Warehouse .
- Maintain a clear metadata trail: make sure you're updating your readme files to facilitate collaboration and understanding.

Additional Materials for DSE Models

-  DSE SDK: Model Section
-  Demo Model Repo: Predictive Forecasting of Weather Data
-  DSE Dev Docs: Model Registration and Deployment
-  Demo Model Repo: Deep Learning of Weather Data
