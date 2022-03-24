# astra-cdc-demo

This is a step by step guide to enabling the CDC feature of Astra DB. It connectes an existing table with the database with an Astra Streaming topic, which we then connect to a sink that writes changes messages back to another database.

## Prerequisites

- Active Astra paid account (the CDC feature is not available to accounts with no saved payment methods)

## Creating a single table database and enabling CDC

1. Create a new serverless database
  ![image](https://user-images.githubusercontent.com/16946028/159967778-0be9572f-0fa5-45b4-ad9a-8e7f31313f80.png)

1. With a status of "Active", load restaurant sample data
  ![image](https://user-images.githubusercontent.com/16946028/159969713-758c1735-6227-4252-87fc-34de2a606b4b.png)
  ![image](https://user-images.githubusercontent.com/16946028/159970840-daa952d4-f23c-47d5-832a-21cdf2131958.png)
  
1. 
