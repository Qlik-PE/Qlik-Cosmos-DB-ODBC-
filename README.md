---
title: Connecting to Cosmos DB SQL API from Qlik Sense using the Cosmos DB ODBC Connector
---

![](media/655ae15571141d257b7e8aa0706f076e.png)

Cosmos DB Overview
==================

Cosmos DB is a fast growing, multi-modal database service in Microsoft Azure,
and offers several API’s. Currently, the SQL API is the most popular and widely
used API. When you create a Cosmos DB Account, you must decide which API you
want to use as the data will get stored in that format (exception being
GremlinAPI which spans multiple APIs).

Cosmos DB Key capabilities
==========================

-   Turnkey global distribution

    -   You can [distribute your
        data](https://docs.microsoft.com/en-us/azure/cosmos-db/distribute-data-globally) to
        any number of [Azure regions](https://azure.microsoft.com/regions/),
        with the [click of a
        button](https://docs.microsoft.com/en-us/azure/cosmos-db/tutorial-global-distribution-sql-api).
        This enables you to put your data where your users are, ensuring the
        lowest possible latency to your customers.

    -   Using Azure Cosmos DB's multi-homing APIs, the app always knows where
        the nearest region is and sends requests to the nearest data center. All
        of this is possible with no config changes. You set your write-region
        and as many read-regions as you want, and the rest is handled for you.

    -   As you add and remove regions to your Azure Cosmos DB database, your
        application does not need to be redeployed and continues to be highly
        available thanks to the multi-homing API capability.

-   Multiple data models and popular APIs for accessing and querying data

    -   The atom-record-sequence (ARS) based data model that Azure Cosmos DB is
        built on natively supports multiple data models, including but not
        limited to document, graph, key-value, table, and column-family data
        models.

    -   APIs for the following data models are supported with SDKs available in
        multiple languages:

        -   [SQL
            API](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-introduction):
            A schema-less JSON database engine with rich SQL querying
            capabilities.

        -   [MongoDB
            API](https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb-introduction):
            A massively scalable *MongoDB-as-a-Service* powered by Azure Cosmos
            DB platform. Compatible with existing MongoDB libraries, drivers,
            tools, and applications.

        -   [Cassandra
            API](https://docs.microsoft.com/en-us/azure/cosmos-db/cassandra-introduction):
            A globally distributed Cassandra-as-a-Service powered by Azure
            Cosmos DB platform. Compatible with existing [Apache
            Cassandra](https://cassandra.apache.org/) libraries, drivers, tools,
            and applications.

        -   [Gremlin
            API](https://docs.microsoft.com/en-us/azure/cosmos-db/graph-introduction):
            A fully managed, horizontally scalable graph database service that
            makes it easy to build and run applications that work with highly
            connected datasets supporting Open Gremlin APIs (based on
            the [Apache TinkerPop specification](http://tinkerpop.apache.org/),
            Apache Gremlin).

        -   [Table
            API](https://docs.microsoft.com/en-us/azure/cosmos-db/table-introduction):
            A key-value database service built to provide premium capabilities
            (for example, automatic indexing, guaranteed low latency, global
            distribution) to existing Azure Table storage applications without
            making any app changes.

The SQL API can be interacted with using ODBC, REST, or native code bases such
as .Net (Core and Standard), Java, Go, Node.js, or Python.

There are many connectivity methods validated with Qlik Partner Engineering:

1.  SQL API using ODBC Connector in Qlik Sense

2.  MongoDB API using the Qlik Sense MongoDB Connector (Beta currently, Oct
    2018)

3.  MongoDB SQL API using REST Connector in Qlik Sense

4.  Mongo DB API using the gRPC connector for Qlik Core

The focus of this document is the details of connecting to the SQL API via the
ODBC Connector.

About Qlik Sense
================

Qlik Sense gives you data superpowers. Easily combine all your data sources, no
matter how large, into a single view. Qlik’s Associative engine indexes every
possible relationship in your data so you can gain immediate insights and
explore in any direction your intuition takes you. Unlike query-based tools,
there’s no pre-aggregated data and predefined queries to hold you back. That
means you can ask new questions and create analytics without having to build new
queries or wait for the experts.

![](media/5d9e7b7c96bb2510c0a98c54cb16d757.png)

**Interactive analysis, without boundaries**

Ask any question and quickly explore across all your data for insight, using
global search and interactive selections. All analytics update instantly with
each click, no matter how deep you go, furthering analysis or pivoting your
thinking in new directions. There’s is no limit to exploration and no data left
behind.

**Simply smarter visualizations**

Innovative visualizations put your data in the right context to answer any
question. Explore the shape of data and pinpoint outliers. Use advanced
analytics integration and geographic calculation to broaden insight. And it's
fully interactive - easily pan, zoom, and make selections to find insights
visually.

**Create and explore on any device**

Explore, create, and collaborate on any device, directly at the point of
decision. Qlik Sense is built from the ground up with responsive mobile design
and touch interaction. Build analytics apps once, and they’ll work everywhere,
on desktop, tablet, or mobile devices.

To learn more, go to <https://www.qlik.com/us/products/qlik-sense>

Cosmos DB Setup
===============

Setting up Cosmos DB is straightforward, once you pick your path. For this
validation exercise, we are testing the SQL API (formerly documentDB) as our
query engine for Qlik.

### Step 1. Sign into the Azure Portal – Navigate to Cosmos DB

![](media/85073b45005d994df301fc6726829181.png)

### Step 2. Add a new Cosmos DB instance

![](media/376fc361c13ec1fba2b2073389b34f99.png)

### Step 3. Complete Project Details

\- For this use case, we are selecting the SQL API.

![](media/3e4ae5b0c606530cbcbcca79a876c1ab.png)

### Step 4. Validate and Create! 

The process will run for a few minutes and we have our database!

![](media/6cc9792c7a914192f6e8f9336c4c6710.png)

Conversation - What is a RU?
----------------------------

Cosmos DB is priced and scaled using a model called a “Request Unit” or RU. The
RU defines how much compute and throughput you can use with the Cosmos DB
instance. There are two modes, **Fixed** or **Unlimited** which govern the
capacity of the system during use. You can learn more about how it works
[here](https://docs.microsoft.com/en-us/azure/cosmos-db/request-units). This
setting will govern the upload and query capacity of the Cosmos DB system, so
consideration must be given to expected workloads during the setup process as
certain element cannot be changed after creation.

Step 5. Create Database and Collection
--------------------------------------

![](media/a4e8c776d993fa029d7ade5db90110ef.png)

Before we can load data, we need to create the repository inside Cosmos DB that
we’ll need to write data into. We start by opening *Data Explorer* in the menu.

We need to create a New Database and a New Collection underneath that database.
Here is where you will set your RU capacity. For this small data set and test,
we’ll go with 1000 RU.

Step 6. Loading Data
--------------------

There are many ways to load data into Cosmos DB, the tool we are going to use
for validation is called the Azure Cosmos DB Data Migration tool. The link to
the utility can be found
[here](https://docs.microsoft.com/en-us/azure/cosmos-db/import-data). The data
set we will use as a test data set is video game sales 1982-2016 and can be
found [here](https://www.kaggle.com/gregorut/videogamesales).

Once we have downloaded both the utility and data, we are ready to proceed.

#### Data Transfer UI 

Navigate to the unzipped folder, and run DTUI.exe. This will spawn the loader
utility UI.

![](media/4767e9d277ea5f328fd664912ec0f89b.png)

#### Source Information

Select CSV File from the drop down in Source information setting… Notice the
utility supports a wide range of data source files and connections as sources.

![](media/fe71f98047406d80e40c636b95581926.png)

#### Target Information

We will need to collect some information from our Cosmos DB system to populate
the required fields.

![](media/186caffeac7327638e7207faf8e8a8f1.png)

*Connection String*: This is the PRIMARY CONNECTION STRING from the Read-Write
Keys section of the Keys menu. NOTE: to get it to work, you have to manually
add: ;database=\<dbname\>

Our connection string looks like:
*AccountEndpoint=https://qlik-cosmos-sql.documents.azure.com:443/;AccountKey=Ay1cxkuONzTOZuRYjI83FIrd99NBaXOjOHb0LifsxeEW3D7jWZLA0uMgHvSt5SuWTJ2Ial00hGkVsO36FRbaUQ==;database=vg*

*Collection*: This is the name of the collection we created earlier.

![](media/40d051c6849abf6e01e4566be23f2f63.png)

Click next a few times, and “Import” to begin the data transfer. This may take a
few minutes…

We can look in our Data Explorer and find that indeed the data has loaded!

![](media/b939bec1b31daab779e673a26e96c423.png)

We have now completed our Cosmos DB setup.

Qlik Sense Configuration
========================

Step 1. Install & Configure Qlik Sense
--------------------------------------

This is not covered in this guide, as we pre-assume a running Qlik Sense system.
If you need to setup Qlik Sense – please refer to this guide ([Install Qlik
Sense on
Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/qlik.qlik-sense))
or download Qlik Sense desktop ([Qlik Sense
Desktop](https://www.qlik.com/us/try-or-buy/download-qlik-sense)).

Step 2. Download, Install & Configure ODBC driver 
--------------------------------------------------

Download the ODBC driver from ([Download ODBC
Driver](https://docs.microsoft.com/en-us/azure/cosmos-db/odbc-driver)). Follow
the setup steps in the ODBC driver link to configure the connection to Cosmos
DB. Note: this is a simple data set that will not require Schema Editing.

### ODBC Concerns

It is important to understand that standard ANSI SQL and the SQL API (formerly
documentDB) are not fully compliant because Cosmos DB is schema-less and well,
SQL is structures. To that end, the SQL issued through the ODBC driver is being
translated into a JSON object query method and that can cause issues. To try and
prevent these translation issues, you might have to apply a “schema” that maps
the conversions and data types. The ODBC driver page gives you suggestions and
methods to help navigate this process. This mapping will have to occur for every
database/collection inside a Cosmos DB instance.

Creating the Qlik Sense App 
============================

### Step 1. Open Qlik Sense and create a new App

![](media/4f4f6898539de1896791cc1d9ad03ac3.png)

### Step 2. Select - Add data from Files and other sources

![](media/22b726ad035163ed79c13412cd5f8589.png)

### Step 3. Select ODBC

![](media/b77d832dae980f8f17f22baedadc744d.png)

### Step 4. Choose the ODBC connection we made earlier and name it…

![](media/ce2572f13f27ae8e9b1ea619662cb82a.png)

### Step 5. Choose the database and collection we need from that connection

![](media/7fe9a5b4f9f1e277bc06614d29085cd0.png)

### Step 6. Add data and Generate Insights…

![](media/c1c022d6f7a1ea58aa8627bc125b4e5b.png)

### Step 7. Explore!

By either using insights or directly building on the canvas, we can build our
app exploring video games sales!

![](media/dd26b2df5dc776617e51c27233051740.png)

Summary & Conclusion
====================

This document shows how to use Qlik Sense with the Cosmos DB SQL API
configuration with a step-by-step tutorial. The ODBC connector is the easiest
driver to get going, but can be difficult with a complex document as the schema
configuration and translation to SQL does add overhead RU cost.

