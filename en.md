# Stateless data processing with BigQuery continuous query

## Overview

BigQuery continuous queries enable customers addresses to simplify real-time pipelines, unlock real-time AI use cases, streamline reverse ETL, provide scalability and performance.

### Learning objectives

1. Setup a BigQuery remote connection.
2. Create an Application Integration trigger.
3. Create a BigQuery continuous query.

## Setup and requirements

![[/fragments/startqwiklab]]

![[/fragments/gcpconsole]]

![[/fragments/cloudshell]]

## Task 1. Setting up the project

1. Open a new Cloud Shell terminal by clicking the Cloud Shell icon in the top right corner of the Google Cloud Console.

2.  In your Cloud Shell terminal, use `gcloud` to enable the services used in the lab:

<ql-code-block noWrap>
gcloud services enable \
  vertex.googleapis.com \
  pubsub.googleapis.com \
  iamcredentials.googleapis.com \
  aiplatform.googleapis.com \
  bigquery.googleapis.com \
  cloudbuild.googleapis.com 
</ql-code-block>

3. In the Google Cloud Console, select __Navigation menu__ > __BigQuery__.

The __Welcome to BigQuery in the Cloud Console__ message box opens. This message box provides a link to the quickstart guide and the release notes.

4. Click __Done__.

The BigQuery console opens. Next, create a new dataset titled __Continuous_Queries_demo__ in BigQuery to store your table.

5. In the left pane, click on the name of your BigQuery project (`qwiklabs-gcp-xxxx`).

6. Click on the three dots next to your project name, then select __Create dataset__.

The __Create dataset__ dialog opens.

7. Set the **Dataset ID** to `Continuous_Queries_demo`, leave all other options at their default values.

8. Click **Create dataset**.

9. In the BigQuery Studio Welcome panel, click on __SQL QUERY__ to create a new query, and then paste the query below to create a new table:
```
CREATE TABLE `Continuous_Queries_Demo.abandoned_carts`(
  customer_name string,
  customer_email string,
  last_updated timestamp default current_timestamp,
  products string);
```
10. Click __RUN__ to create the table.


## Task 2. Create a BigQuery remote connection and BigQuery ML remote model

1. In BigQuery console, click __+ Add__, and then click __Connections to external data sources__.

2. In the __Connection type__ list, select _Vertex AI remote models, remote functions and BigLake (Cloud Resource)_.

3. In the __Connection ID__ field, enter a name for your connection.

4. Click __Create connection__.

5. Click __Go to connection__. In the Connection info pane, copy the service account ID for use in the next steps.

6. In the Google Cloud console, on the __Navigation menu__ (![Navigation menu icon](https://storage.googleapis.com/cloud-training/images/menu.png)), select __IAM & Admin__ > __IAM__.

7. Click **VIEW BY PRINCIPALS**. Next, click **GRANT ACCESS**

8. In the **New principals** field, enter the service account ID you copied earlier.

9. In the **Select a role** drop-down list, select the **Vertex AI User** role.

10. Click **Save**.

11. In the Google Cloud Console, select __Navigation menu__ > __BigQuery__.

12. In the query editor, paste the following to create a remote model, and then click __RUN__
```
#Creates a BigQuery ML remote model named gemini_1_5_pro
CREATE MODEL `Continuous_Queries_Demo.gemini_1_5_pro`
REMOTE WITH CONNECTION `us.continuous-queries-connection`
OPTIONS(endpoint = 'gemini-1.5-pro');
```

## Task 3. Create a pub/sub topic and grant necessary permissions
1. In the Google Cloud console, go to the Pub/Sub Topics page and click __Create topic__.

2. In the Topic ID field, enter _recapture_customer_.

3. Check the option __Add a default subscription__. Leave other options as default.

4. Click __Create topic__.



## Task 4. Setup an Application Integration trigger

1. In the Google Cloud Console, type _application integration_ in the search bar, and then click on __Application Integration__ in the dropdown list.
2. In the Application Integration page, select {{{ project_0.default_region | REGION }}} as your region.
3. Click __QUICK SETUP__. This setup enables the neccessary APIs.
4. Once setup is complete, click the __CREATE INTEGRATION__ button and name your integration `abandoned-shopping-carts-integration`.
5. Click __CREATE__.
6. Next, click __TRIGGERS__ at the top, select __Cloud Pub/Sub__ from the list, and add your Pub/Sub trigger onto the canvas.
7. In the input form that loads on the right, add the name of your Pub/Sub topic for _Trigger Input_, and the "bq-continuous-query-sa" service account you previously created.
8. If you see a warning that says "Grant the necessary roles", click GRANT.
9. Click __TASKS__ at the top, search for __Data Mapping__, and add the Data Mapping item to your canvas.
10. On your canvas, connect the Cloud Pub/Sub Trigger to the Data Mapping item.
11. Click the __Data Mapping__ item on your canvas, and click the button that says "OPEN DATA MAPPING EDITOR". This will open the _Variables_ page.
12. You'll create 4 Input variables, each of the type `CloudPubSubMessage.data`. Under Input, click on __Variable or Value__, and for the _Variable_ field, select __CloudPubSubMessage.data__ and click __SAVE__.


## Task 3. Create a custom service account

1. In your Cloud Shell terminal, run the following commands to create a custom service account for Vertex AI custom training with Tensorboard:

<ql-code-block noWrap>
SERVICE_ACCOUNT_ID=vertex-custom-training-sa
gcloud iam service-accounts create $SERVICE_ACCOUNT_ID  \
    --description="A custom service account for Vertex custom training with Tensorboard" \
    --display-name="Vertex AI Custom Training"
</ql-code-block>


2. Grant it access to Cloud Storage for writing and retrieving Tensorboard logs:

<ql-code-block noWrap>
PROJECT_ID=$(gcloud config get-value core/project)
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/storage.admin"
</ql-code-block>

3. Grant it access to your BigQuery data source to read data into your TensorFlow model:

<ql-code-block noWrap>
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/bigquery.admin"
</ql-code-block>

4. Grant it access to Vertex AI for running model training, deployment, and explanation jobs:

<ql-code-block noWrap>
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/aiplatform.user"
</ql-code-block>


## Setup

![[/fragments/start-qwiklab]]


## Task 1. Create a Dataform repository

1. In the Console, expand the Navigation menu, then select __BigQuery__.
