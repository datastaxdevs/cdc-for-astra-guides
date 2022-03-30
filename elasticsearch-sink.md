# elasticsearch-sink

This is a step by step guide to enabling CDC for Astra DB.The goal is to get updates in Elastic Search when data in an Astra DB table changes. We’ll accomplish this through the change data capture pattern.

## Prerequisites

- Active Astra paid account (the CDC feature is not available to accounts with no saved payment methods)

- Publicly accessible, running instance of ElasticSearch and Kibana (we’ll be using a trial account from [Elastic](https://elastic.co) that has no existing configurations)

## Create database, tables, and streaming tenant

1. Choose a region that is supported by CDC for Astra DB by visiting the Streaming documentation's ["Regions" page](https://docs.datastax.com/en/astra-streaming/docs/astream-regions.html) (we'll be using **us-central1** in GCP for this example)

    ![image18](https://user-images.githubusercontent.com/16946028/160468872-d04619f4-8c7f-4315-b673-5d0eb425d47e.png)

1. Login to your Astra account

    ![Screenshot 2022-03-28 152026](https://user-images.githubusercontent.com/16946028/160471085-86edcc64-639f-4395-83b2-9098f2be7d00.png)

1. Choose "Create Database" and create a serverless database with the following details:

    Database Name: `my_company`
    
    Keyspace Name: `our_product`
    
    Provider and Region: `Google Cloud > North America > us-central1`

    ![image17](https://user-images.githubusercontent.com/16946028/160468967-6b54a8a5-2ac9-4798-95eb-5121e6012ee6.png)

1. Once the database has a status of "Active", navigate to its CQL console (you may need to refresh the browser window)

    ![image14](https://user-images.githubusercontent.com/16946028/160469251-eb7bd2eb-c2a9-495c-a2bc-2d57c212316f.png)

1. Create a table that will hold our test account information

    ```
    create table our_product.all_accounts (id uuid primary key, full_name text, email text);
    ```

    ![image16](https://user-images.githubusercontent.com/16946028/160469384-2ee2128e-2666-46fb-bf69-d54da56690b4.png)

1. Choose "Create Streaming" and create a new tenant with the following details;

    Tenant Name: `my-company-streams`
    Provider and Region: `Google Cloud > uscentral1`

    ![image9](https://user-images.githubusercontent.com/16946028/160469465-829a24cb-312a-4248-963d-ea3b22116add.png)

The new tenant will be ready to go very quickly and your view will automatically refresh to its “Quickstart” tab. CDC will automatically create a namespace and topic within the tenant.

## Enable CDC

1. Navigate to the database’s "CDC" tab and choose "Enable CDC"

    ![image2](https://user-images.githubusercontent.com/16946028/160469556-940760e0-9ebf-43ae-baba-b18e262a6594.png)

1. Enable CDC with the following details:

    Tenant: `pulsar-gcp-uscentral1 / my-company-streams`
    Keyspace: `our_product`
    Table name: `all_accounts`

    ![image13](https://user-images.githubusercontent.com/16946028/160469581-47f9772d-d624-4495-91e9-39edffb05cea.png)

    Wait for the new CDC process to have a status of "Running"

    ![image6](https://user-images.githubusercontent.com/16946028/160469636-0330efbd-708d-4f0e-bac3-af89cfe3a2c9.png)

## Create a streaming sink

1. Navigate to my-company-streams "Sinks" page and choose "Create Sink"

    ![image19](https://user-images.githubusercontent.com/16946028/160469996-0acccca4-d6d1-46a9-bb8f-ab765702cba8.png)

1. Create a new sink with the following details:

    The Basics
    - Namespace: `astracdc`
    - Sink Type: Elastic Search
    - Name: `es-account-sink`

    Connect Topics
    - Topic: data-&lt;UUID>-our_product.all_accounts

    Sink-Specific Configuration
    - Elastic search URL: &lt;endpoint>
    - Index name: `cdc_account_messages`
    - Username: &lt;username> (the default is "elastic")
    - Password: &lt;password>
    - Type name: _doc
    - Ignore record key: false
    - Null value action: Ignore
    - Compatibility mode: AUTO
    - Strip nulls: true
    - Enable schema: true
    - Copy key fields: false

	    Your view will go back to the "Sinks" page where you can see the "Status". Once it says "Running" the sink is complete (you may need to refresh the browser window).

![alt_text](images/image10.png "image_tooltip")

## Insert data to observe CDC in action

1. Navigate back to the database CQL console 

    ![image14](https://user-images.githubusercontent.com/16946028/160469251-eb7bd2eb-c2a9-495c-a2bc-2d57c212316f.png)

1. Add a new record to the all_accounts table

    ```
    insert into our_product.all_accounts (id, full_name, email) values (5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Suzie Shoe', 'suzie_s@acoolplace.com');
    ```

1. In your Elasticsearch deployment choose "Management" > "Stack Management" from the left menu

    ![image12](https://user-images.githubusercontent.com/16946028/160866704-0d6dacfc-c35e-4271-b957-74fa314e37d9.png)

1. Then choose "Kibana" > "Data Views" to get a prompt that you already have data in ElasticSearch. Choose "Create data view".

    ![image5](https://user-images.githubusercontent.com/16946028/160866922-3609aeb4-a1a4-4a87-935e-751cf6d21250.png)

1. Name the data view to match the existing index "cdc_account_messages" and "Create data view"

    ![image14](https://user-images.githubusercontent.com/16946028/160867122-8e83732f-7531-44d9-a321-c266a74091d2.png)

1. Navigate to the "Discover" option of Analytics from the left menu

    ![image6](https://user-images.githubusercontent.com/16946028/160867343-60ec87ac-db5c-461d-8a92-6307e752b14b.png)

1. The new record that was created in AstraDB will be displayed in the view

  ![image1](https://user-images.githubusercontent.com/16946028/160867413-dd1fae82-cc29-428d-b438-6bf0396ac9a5.png)

