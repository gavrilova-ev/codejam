# SAP Leonardo Machine Learning Foundation and the SAP S/4HANA Cloud SDK
Here, we provide the instructions to proceed with the code jam "SAP Leonardo Machine Learning and the SAP S/4HANA Cloud SDK". Below, you find the following information:
* [Technical prerequisites](#prerequisites): setup required to execute the steps described in this documentation. This information was provided before the workshop, so, we assume that those prerequisites are already fulfilled. Nevertheless, you can use this description to double check.
* [Task 0: Preparation steps](#task0)
* [Task 1: Retrieve SAP S/4HANA data using the SAP S/4HANA Cloud SDK virtual data model](#task1)
* [Task 2: Set up the Continuous Delivery Toolkit and the CI/CD pipeline of the S/4HANA Cloud SDK in GKE](#task2)
* [Task 3: Integrate SAP Leonardo Machine Learning service to provide translations](#task3)
* [Bonus, Task 4: Write data back to SAP S/4HANA using the SAP S/4HANA Cloud SDK virtual data model](#task4)

So, let us get started!

## <a name="prerequisites">Technical prerequisites</a>
You do not need any additional software installed locally for this code jam, as we will use the full stack WebIDE for development. All the steps of this code jam also are described considering that you re using the WebIDE.
```diff
- TODO: remove this part if OP WebIDE is used
```
### Set up Web IDE
To get access to WebIDE, you can use your SAP Cloud Platform, Neo trial account:
![Access SAP Cloud Platform, Neo account](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.55.20.png)
Then, navigate to Services and choose SAP Web IDE Full-Stack:
![SAP Web IDE Full-Stack service](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.56.46.png)
Make sure that the service is enabled or enable it, at the end you should see the status "Enabled":
![SAP Web IDE is enabled](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.57.13.png)
After that, choose "Go to Service" in the section "Take Action":
![Go to service Web IDE](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.57.36.png)
After the Web IDE loads, you should see the workspace, where we will create a new project in the next steps:
![Web IDE workspace](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.58.45.png)

### Set up SAP Cloud Platform account
We will deploy the application in SAP Cloud Platform, Cloud Foundry. For that purpose, you would require your own trial account. [Here](https://cloudplatform.sap.com/try.html), you can find information on how to get your trial account in SAP Cloud Platform, Cloud Foundry.
You can choose different regions and infrastructure provider for your subaccounts in the menu section "Regions" after you have created your trial account. In the Regions section choose US Central (IA) on the GCP infrastructure:
![SAP Cloud Platform account on Google Cloud Platform](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/SCPaccount/Screenshot%202019-02-25%20at%2013.15.43.png)
If you already have a trial account and want to reuse it, please make sure to have sufficient entitlements for the following services in the US Central (IA) region:
Application runtime: 3 or more
Destination: 1
![Manage entitlements](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/SCPaccount/Screenshot%202019-02-25%20at%2013.19.22.png)

### SAP S/4HANA
For this workshop, we provide a ready to use SAP S/4HANA system that you can integrate into your SAP Cloud Platform application using the APIs. You can address the system via the URL:
```diff
- TODO: provide URL to CAL system
```
Please, reach out to your instructors to get your cfredentials to this system.

We will share our source code via [GitHub](https://github.com/). So, you will require your own GitHub account for this CodeJam. If you do not already have one, please, make sure to Sign Up for GitHub, as proposed in the [initial page](https://github.com/).

## <a name="task0">Task 0: Preparation steps</a>
Before, we get started with the actual implementation, we need to perform some preparation steps and familiarize ourselves with the project structure.
* We provide you with the GitHub repository for this codejam that includes the skeleton of the project you will extend with the S/4HANA and Leonardo ML Foundation integration capabilities. All you need to do is to fork this repository. All the steps in this code jam will be executed in your own fork of this repository.
```diff
- TODO: add screenshot -> forking the repo (after the repo is published)
```
* Open and login into WebIDE
* Now, we will clone our new forked repository into our WebIDE workspace. In the WebIDE, choose the "Development" perspective, right click on "Workspace" -> Git -> Clone repository -> paste the URL of your fork of the code jam and choose "Clone". This will create a local copy of your project in WebIDE. After the request is completed successfully, we can start investigating our predefined project structure.
* Investigate your project structure:
  * **.che** is a technical folder created automatically by WebIDE and is required to correctly interpret the projecgt structure and to build and run project
  * Artifacts **cx-server**, **Jenkinsfile**, **pipeline_config.yml** help to set up and customize CI/CD server and the pipeline for your SDK based solutions. Those resources will be used to set up CI/CD pipeline for our application later. We also highly encourage you to check out [the related resources after the workshop](https://blogs.sap.com/2017/09/20/continuous-integration-and-delivery/)
  * **srv** folder and its artifacts:
    ** **application** folder contains the business logic that we will extend in this code jam. It also contains the JS based frontend components in the **webapp** subfolder. We will only focus on backend components, though.
    ** **integration-tests** and **unit-tests** folders include integration and unit tests. We have already prepared the integration tests for your application, they do not pass yet, though, and therefore are ignored for now.
    ** **pom.xml** is a [maven configuration file](https://maven.apache.org/pom.html)
  * **.gitignore** file is used to exclude certain files in your working directory from your Git history
  * **mta.yaml** is a build and deployment descriptor to be able to build and deploy the application in SAP Cloud Platform, Cloud Foundry.
  * **solutions** folder contains the solutions for the coding tasks of this code jam. Use it wisely :)


Before we get started with the development, let us familiarize ourselves on how to build and execute tests of the application in WebIDE without deploying it in SAP Cloud Platform. That will help us to quickly test changes in the application that we will perform in the next steps.

While building the application, we will execute integration tests. For the integration tests, you need to provide the URL and credentials of your SAP S/4HANA system.
* Open the file `srv/integration-tests/src/test/resources/systems.yml`.
As we will be using the parameters of the provided SAP S/4HANA system in this code jam, please make sure that the file contains the following content. 
```diff
- TODO: Put correct S4 URL
```

```
---
erp:
  default: "S4HANA"
  systems:
    - alias: "S4HANA"
      uri: "https://odata-mock-server-shy-sitatunga.cfapps.eu10.hana.ondemand.com/"

```
* In the same directory, create a `credentials.yml` file used during tests with the following content and make sure to put the correct name and password provided by the code jam instructor.
```
---
credentials:
- alias: "S4HANA"
  username: "(username)"
  password: "(password)"
```

After this, right clink on the "srv" folder and choose Build -> Build and Run Tests. Wait till the build finishes and make sure that you get "SUCCESS" for all the executed steps: Root, Application, Unit Tests, Integration Tests.

Now, after we got familiar with the local testing of the application, let us start with the first step: integrating SAP S/4HANA into this application using the SAP S/4HANA Cloud SDK.

## <a name="task1">Task 1: Retrieve SAP S/4HANA data using the SAP S/4HANA Cloud SDK virtual data model</a>
In this step, we will investigate two queries to SAP S/4HANA to retrieve business partner data. Firstly, we will retrieve the list of business partners for the list view in  the application. Secondly, we will take a look at the query retrieving detailed data of a single business partner by ID.

Start the development of queries by looking into the class BusinessPartnerServlet, which is the servlet exposing the business partner APIs. 
We could use any API framework here, such as JAX-RS or Spring. However, we use a servlet here for simplicity. Looking into the servlet, we can see that the main functionality is moved out into the commands GetAllBusinessPartnersCommand and GetSingleBusinessPartnerByIdCommand. Open and implement the command *GetAllBusinessPartnersCommand* as explained below.

The *GetAllBusinessPartnersCommand* should return a list of available business partners in the ERP system. The class was already created. We just have to implement the execute method:
* The instance of the class *BusinessPartnerService* already provides a method to retrieve all business partners. Type *service* to see a list of all available methods. Use the method *getAllBusinessPartner* to fetch multiple business partner entities.
* We only want to return the properties first name, last name and id. Thus, select only these properties by using the *select* method on the result from step 1. Luckily, we do not have to know the exact names of these properties in the public API of S/4HANA. They are codified as static member of the class *BusinessPartner*. We can select the business partner id by using *BusinessPartner.BUSINESS_PARTNER*. Please, do the same for the first name and last name.
* There are multiple categories of business partners. In this session, we only want to retrieve persons. The category is identified by a number, which is stored in the static class variable called *CATEGORY_PERSON*. The method to filter is called *filter* and can be executed on the result from the previous step.
The property *BusinessPartner.BUSINESS_PARTNER_CATEGORY* should equal *CATEGORY_PERSON*. To express that use the methods provided by the object *BusinessPartner.BUSINESS_PARTNER_CATEGORY*.
* All the previous steps did not execute any requests, but just defined the request. With the method *execute* you finally execute the query and retrieve the result.

Hint: Try to solve it on your own. However, the solution can also be found in the solution folder in the session material.

Now, also take a look at he command *GetSingleBusinessPartnerByIdCommand*. It was already implemented for you. Based on this source code, can you find out how the OData "expand" method can be implemented using the Virtual Data Model of the SAP S/4HANA Cloud SDK? Hint: addresses of business partners are retrieved using *expand*.

To check whether the queries are implemented correctly, go to the integration-tests folder and remove the *@Ignore* annotation for the following test: *BusinessPartnerServletTest.testGetAll()*.
Now, build and test the application as described in [Task 0](#task0) and make sure that the tests ran successfully. 

If the unignored test do not show errors, congratulations! You have successfully integrated SAP S/4HANA with your application.
In this case, let us push the local changes into our remote GitHub repository. 
For that, choose "Git Pane" on the right, choose the files that you have modified, stage and commit and push:
![Push to GitHub from Web IDE](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/git/Screenshot%202019-02-25%20at%2013.30.25.png)

Now, it is time to think about how we can continuously deliver this application.

## <a name="task2">Task 2: Set up the Continuous Delivery Toolkit and the CI/CD pipeline of the S/4HANA Cloud SDK in GKE</a>
Generally, you can use several ways to deploy your applications in SAP Cloud Platform. The recommended way for productive applications is to use the [Continuous Delivery Toolkit](https://github.com/SAP/cloud-s4-sdk-pipeline), which also ensures that our source code is properly tested and checked before being deployed. 

In this task, we will setup the Continuous Delivery Toolkit on GKE and SAP Cloud Platform account - everything we need to successfully deploy our application in an automated manner in SAP Cloud Platform account.

### Prepare SAP Cloud Platform Account
Here, we describe how to deploy your application manually using the SAP Cloud Platform, Cloud Foundry Cockpit. Necessary preliminary steps are also described here.
* Create service instances for S/4HANA connectivity
* Create destination endpoints

Firstly, create an instance of the destination service to connect to SAP S/4HANA (mock) system. For that, in the cloud platform cockpit on the level of your development space choose Services -> Service Marketplace and choose the destination service from the catalog.
Instantiate the service with all the default parameters. Give the name my-destination to your instance.
![Destination service in the Service Marketplace](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/destination.PNG)

Secondly, create an instance of the Authorization and Trust Management service. In the Service Marketplace, choose the Authorization and Trust Management service and instantiate it with the default parameters. Give the name my-xsuaa to your service instance.
![Authorization and Trust Management](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/uaa.PNG)

Next, we will create a destination endpoint to connect to the provided SAP S/4HANA system.
You can find the configuration of the destination endpoints on the level of your subaccount by choosing Connectivity -> Destinations.
Then, you can create a new destination endpoint by choosing "New Destination".

For the S/4HANA connectivity, create the destination with the following parameters. Make sure to put user credentials provided by the instructor. Those are the same credentials that you have used for testing in WebIDE in [Task 0](#task0) <br>
```diff
- TODO: 
add correct S4 URL
```
Name: ErpQueryEndpoint <br>
Type: HTTP <br>
URL: https://odata-mock-server-shy-sitatunga.cfapps.eu10.hana.ondemand.com/ <br>
Proxy type: Internet <br>
Authentication: BasicAuthentication <br>
User: (username) <br>
Password: (password) <br>

Now, we are ready to deploy our application in SAP Cloud Platform, Cloud Foundry. We will do it using the Continuous Delivery Toolkit that is also testing the application and ensures the code quality in the next step.

### Set up Continuous Delivery Toolkit in GKE
```diff
- TODO: 
modify the linked Google next tutorial removing google next-specific points
and adding the Jenkins configurations: 
test credentials
deployment credentials
```
[GKE CI/CD Tutorial](https://github.com/SAPDocuments/Tutorials/blob/73cd62ac35d1ecd0df14c2b1cac60eccac20107c/tutorials/s4sdk-continuous-delivery-toolkit-setup/s4sdk-continuous-delivery-toolkit-setup.md)

If you still have time, continue with the next task.
In the next step, we will see how to integrate one of the SAP Leonardo Machine Learning services into your application.

## <a name="task3">Task 3: Integrate SAP Leonardo Machine Learning services</a>
```diff
- TODO: 
consider introducing the OCR use case (business cards)
```

In this step, we will integrate SAP Leonardo Machine Learning services into an application using an example of the translation service, which is a part of the set of functional services.

There are several steps involved to make the integration with SAP Leonardo ML services work, those steps are described in details below:
* Implement the integration with ML services in Java Backend
* Create service instance for Leonardo ML integration
* Deploy the application using the CI/CD pipeline prepared in the previous step

### Implement the integration with ML services in Java Backend 
To implement the integration with ML services, we will leverage the SAP S/4HANA Cloud SDK component that simplifies the integration and handles the boilerplate code for you, such as OAuth 2.0 authentication against ML services.

To implement the integration, find the package *machinelearning* in your project, where you will find the *TranslateServlet* class. This class contains the method *translate()*, which delegates the translation logic to the commands *MlLanguageDetectionCommand* for the language detection and *MlTranslationCommand* for the translation.

Navigate to the class *MlTranslationCommand* and investigate its methods. Here, in the method *executeRequest*, you will find the next task. 
In this method, we already provide the logic for the execution of the translation request using the instance of *LeonardoMlService* class and retrive the resulting payload. The rest is left for you. To make the translation work in integration with your application, add the following steps into the *executeRequest* method:

* In WebIDE, navigate to the LeonardoMlService, which is a part of the *machinelearning* package of the SAP S/4HANA Cloud SDK and investigate its methods. Also, looks through the other classes and methods provided in this library. You may also use the [Javadoc for those classes](https://help.sap.com/http.svc/rc/76ceac609c19443099fca151cf9c9e21/1.0/en-US/com/sap/cloud/sdk/services/scp/machinelearning/package-summary.html) to get more information.
* Instantiate the *LeonardoMlService* class, which is a part of the SAP S/4HANA Cloud SDK component for ML services integration. Consider that you use trial beta as Cloud Foundry Leonardo ML service type and Translation as a Leonardo ML service type.
* Create an object request of type *HttpPost*
* Create an object body of type *HttpEntity* (*StringEntity*). Use *requestJson* and *ContentType.APPLICATION_JSON* to instantiate the object.
* Add the created body to the request using the method *setEntity*.

If you experience difficulties, you can compare you solution with the one provided in the [folder solutions](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/solutions/application/src/main/java/com/sap/cloud/s4hana/examples/addressmgr/machinelearning/commands/MlTranslationCommand.java).

Build and test this version in WebIDE. If everything compiles successfully, push your changes into GitHub repository, as was explained in [Task 1](#task1).

### Create service instances for Leonardo ML integration
In the cloud platform cockpit on the level of your development space choose Services -> Service Marketplace and choose ml-foundation-trial-beta. Instantiate the service with the defailt parameters and give it the name my-ml.
![SAP Leonardo Machine Learning](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/ml.PNG)

### Deploy the application using the Continuous Delivery Toolkit
Finally, we will deploy the application in your development space in SAP Cloud Platform, Cloud Foundry, as it was done in the previous task.

Firstly, go to your project *mta.yaml* file and add the service my-ml into the section "requires" and "resources" to make sure that the newly create machine learning service will be bound to our application. Please, see in the example below:

```diff
- TODO: 
add the example of the correct mta.yaml with bound ml service 
```
Push your changes into GitHub repository, as was explained in [Task 1](#task1).

Now, go to your Jenkins and start the build for your project again.
```diff
- TODO: 
add the screenshot
```

When the application is deployed, you can drill down into the application, choose the link for the application and append "/address-manager" to it. You should be able to see the business partner coming back from the mock server and you should be able to translate their professions by clicking on them.

![Result of the deployment](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/deploymentResult.PNG)

![Business Partner Address Manager with the integrated translation service](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/Translation.PNG)

Congratulations! You have just finished the main steps in this code jam:
* Firstly, we have integrated SAP S/4HANA Business Partner APIs to read the list of business partners and to read the detailed information
* Secondly, we have deployed the Continuous Delivery Toolkit of the S/4HANA Cloud SDK in our GKE cluster to continuously and automatically test, check, and deliver our applications.
* Thirdly, we have integrated SAP Leonardo Machine Learning functional service, using the Translation as an example.

Continue to the next steps in case you have time. 

## <a name="task4">Bonus, Task 4: Write data back to SAP S/4HANA using the SAP S/4HANA Cloud SDK virtual data model</a>
Here, we will further investigate the capabilities of the SAP S/4HANA Cloud SDK virtual data model to integrate SAP S/4HANA now also for update, and delete operations.

*	The class BusinessPartnerService already offers methods to create, update or delete address. The input values, such as the addresses or IDs to delete are member variables of the commands. They are passed into the command from the servlet.
*	Implement the run methods in the commands UpdateAddressCommand and DeleteAddressCommand. 
*	For the delete method we first have to create a business partner address instance, which has the IDs for the business partner and the address specified. The class BusinessPartnerAddress offers the method builder to create a builder and can be used as follows:

```
BusinessPartnerAddress addressToDelete = BusinessPartnerAddress.builder()
        .businessPartner(businessPartnerId)
        .addressID(addressId)
        .build();
```

*	Two commands expect an integer to be returned as result. These integers should correspond to the status code returned from SAP S/4HANA as result of the modification. You can simple call getHttpStatusCode to get the status code back.

Try to implement the queries by yourself. Feel free to check out the solution folder that we have prepared for you in case you are experiencing difficulties.

To test your logic, we have already prepared the tests. Go the the class AddressServletTest, which resides in the integration-tests module and remove all @Ignore annotations. Run the tests in this class and make sure that all tests are green. If not, get back to your commands and fix the issues.

If the tests are successful, you can now push your changes to GitHub again and start the Jenkins build and deployment
