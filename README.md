# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Azure Resource | Service Tier | Monthly Cost |
| ------------ | ------------ | ------------ |
| *Azure Postgres Database* |Basic - Single Server |$65.08   |
| *Azure Service Bus*   |Basic Service Plan - B1| $13.14       |
| *App Service*         |Basic                  | $0.05        |
| *Azure Functions*     |Consumption Tier       | ...          |
| *Storage Accounts*    |Storage (general purpose v1)|...      |

## Architecture Explanation
1. **Azure Postgres Database:**
   - **Service Tier:** Basic - Single Server
   - **Monthly Cost:** $65.08
   - **Explanation:**
     - The Basic tier is chosen for cost efficiency while meeting the database requirements.
     - The Single Server option is suitable for smaller applications, providing a dedicated PostgreSQL server.
     - This tier offers a balance between performance and cost for your database needs.

2. **App Service:**
   - **Service Tier:** Basic
   - **Monthly Cost:** $0.05
   - **Explanation:**
     - The Basic tier is selected to keep costs low while meeting the needs of hosting the Flask-based web application.
     - This tier provides sufficient resources for small to medium-sized applications.
     - App Service offers easy deployment and scaling based on demand.

3. **Azure Functions:**
   - **Service Tier:** Consumption Tier
   - **Monthly Cost:** (Not specified in the table)
   - **Explanation:**
     - The Consumption tier is chosen for Azure Functions to leverage the serverless architecture.
     - This tier incurs costs based on actual consumption, making it cost-effective for sporadic workloads.
     - Azure Functions handle background processing, such as notification tasks, in an event-driven manner.

### Additional Considerations:

- **Azure Service Bus:**
  - **Service Tier:** Basic Service Plan - B1
  - **Monthly Cost:** $13.14
  - **Explanation:**
    - The Basic Service Plan provides sufficient features for message queueing.
    - B1 is selected for cost efficiency while meeting the needs of the application.
    - Azure Service Bus facilitates asynchronous communication between the web app and Azure Functions.

- **Storage Accounts:**
  - **Service Tier:** Storage (general purpose v1)
  - **Monthly Cost:** (Not specified in the table)
  - **Explanation:**
    - Storage Accounts are likely used for storing static assets, logs, or other data.
    - General-purpose v1 is a standard storage tier suitable for various storage needs.
    - The cost depends on the amount of data stored and operations performed.

This architecture enables efficient handling of user interactions, background processing, and communication between components, ensuring a cost-effective and scalable solution for the application.
