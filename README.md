# astra-cdc-demo

This is a step by step guide to enabling the CDC for Astra DB. It connectes an existing table in Astra DB to an Astra Streaming topic, which we then connect to a sink that writes changes messages back to another table.

## Prerequisites

- Active Astra paid account (the CDC feature is not available to accounts with no saved payment methods)

## Create database, tables, and streaming tenant

1. Choose a region that is supported by CDC for Astra DB by visiting the Streaming documentation's ["Regions" page](https://docs.datastax.com/en/astra-streaming/docs/astream-regions.html) (we'll be using **us-central1** in GCP for this example)

    ![image18](https://user-images.githubusercontent.com/16946028/160468872-d04619f4-8c7f-4315-b673-5d0eb425d47e.png)

1. Login to your Astra account

    ![Screenshot 2022-03-28 152026](https://user-images.githubusercontent.com/16946028/160471085-86edcc64-639f-4395-83b2-9098f2be7d00.png)

3. Choose "Create Database" and create a serverless database with the following details:

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

1. Create a table that will hold information from the CDC sink

    ```
    create table our_product.cdc_accounts (id uuid primary key, full_name text, email text);
    ```

    ![image10](https://user-images.githubusercontent.com/16946028/160469403-64d0f820-74d7-4d4d-831e-5bd5b5b1a6d5.png)

1. Choose "Create Streaming" and create a new tenant with the following details;

    Tenant Name: `my-company-streams`
    Provider and Region: `Google Cloud > uscentral1`

    ![image9](https://user-images.githubusercontent.com/16946028/160469465-829a24cb-312a-4248-963d-ea3b22116add.png)

The new tenant will be ready to go very quickly and your view will automatically refresh to its “Quickstart” tab. CDC will automatically create a namespace and topic within the tenant.

## Enable CDC and initialize the streaming topic

1. Navigate to the database’s "CDC" tab and choose "Enable CDC"

    ![image2](https://user-images.githubusercontent.com/16946028/160469556-940760e0-9ebf-43ae-baba-b18e262a6594.png)

1. Enable CDC with the following details:

    Tenant: `pulsar-gcp-uscentral1 / my-company-streams`
    Keyspace: `our_product`
    Table name: `all_accounts`

    ![image13](https://user-images.githubusercontent.com/16946028/160469581-47f9772d-d624-4495-91e9-39edffb05cea.png)

    Wait for the new CDC process to have a status of "Running"

    ![image6](https://user-images.githubusercontent.com/16946028/160469636-0330efbd-708d-4f0e-bac3-af89cfe3a2c9.png)

1. Navigate back to the database CQL console 

    ![image14](https://user-images.githubusercontent.com/16946028/160469251-eb7bd2eb-c2a9-495c-a2bc-2d57c212316f.png)

1. Add a new record to the all_accounts table, to initialize the streaming objects

    ```
    insert into our_product.all_accounts (id, full_name, email) values (85540e16-aca8-11ec-b909-0242ac120002, 'Joe Smith', 'joesmith@domain.com');
    ```

    ![image3](https://user-images.githubusercontent.com/16946028/160469791-b528c9d4-8c47-4f25-900d-d356ac554c92.png)

## Create a streaming sink

1. Navigate to your organization settings

    ![image15](https://user-images.githubusercontent.com/16946028/160469843-45a812ee-e336-472f-8e42-733b24552f8d.png)

1. Generate a new token with the "Organization Administrator" role

    ![image7](https://user-images.githubusercontent.com/16946028/160470699-808ae7d4-0a1e-47a6-9ea6-a14ab5e18527.png)

1. Click the clipboard ![image4](https://user-images.githubusercontent.com/16946028/160470766-ef040ba0-e437-438f-8e37-be64b72c57c1.png) icon next to the “Token” value to add that string to your clipboard

    ![image5](https://user-images.githubusercontent.com/16946028/160469936-be3c5f31-293a-4050-925e-c8eb4a38c6b5.png)

1. Navigate to my-company-streams "Sinks" page and choose "Create Sink"

    ![image19](https://user-images.githubusercontent.com/16946028/160469996-0acccca4-d6d1-46a9-bb8f-ab765702cba8.png)

1. Create a new sink with the following details:

    The Basics
    - Namespace: `astracdc`
    - Sink Type: Astra DB
    - Name: `astra-account-sink`

    Connect Topics
    - Topic: data-&lt;UUID>-our_product.all_accounts

    Sink-Specific Configuration
    - Database: `my_company`
    - Keyspace: `our_product`
    - Table name: `cdc_accounts`
    - Token: &lt;paste from clipboard>
    - Mapping: `id=key.id, full_name=value.full_name, email=value.email`

    > !! The default "Mapping" value is incorrect for our schema. Make sure to overwrite with the above value.


    ![image1](https://user-images.githubusercontent.com/16946028/160470132-141d5fab-db5e-45c7-b6a9-c1f83f3bbcbe.png)

	Your view will go back to the "Sinks" page where you can see the "Status". Once it says "Running" the sink is complete (you may need to refresh the browser window).

    ![image12](https://user-images.githubusercontent.com/16946028/160470291-41cc67ba-0f60-4d91-bfa8-4838e2b14330.png)

## Insert data to observe CDC in action

1. Navigate back to the database CQL console 

    ![image14](https://user-images.githubusercontent.com/16946028/160469251-eb7bd2eb-c2a9-495c-a2bc-2d57c212316f.png)

1. Add a new record to the all_accounts table

    `insert into our_product.all_accounts (id, full_name, email) values (5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Suzie Shoe', 'suzie_s@acoolplace.com');`

1. Observe the new record in cdc_accounts table by selecting all rows

    `select * from our_product.cdc_accounts;`

    ![image11](https://user-images.githubusercontent.com/16946028/160470454-7b143f63-f59f-4385-9028-013ea796923b.png)

## Summary

That was cool
