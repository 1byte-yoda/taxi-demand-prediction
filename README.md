# Taxi Demand Prediction

### Problem Statement:
There's a company who owns a taxi booking mobile app like "Uber" who wants to achieve the following:
1. Define a Mathematical model that will predict how many taxis were needed in a given location at Metro Manila, on a particular time T and coordinates C. The problem was that, taxi drivers randomly (following their gut) chooses the place where to wait for a customer booking. That said, gas usage wasn't optimized and hence, lesser gross income for the taxi drivers. With this solution, taxi drivers shall be able to pool a customer booking at the right location and to have an efficient gas usage.

2. Be able to create analytics dashboard that will monitor taxi drivers' performance, and that will answer some of the business questions below:
A. Who are the top 20 performing taxi drivers?
B. How much was the total taxi income per City?
C. How many taxi drivers are there per City?
D. What's the average speed of taxis per City?
E. What's the current traffic status across Metro Manila?


#### Sources of data:
1. The Taxi booking app, we are only interested with the data from the taxi driver's perspective.
All data attributes below respects this constraint: Data from their first trip in that day up to current time
(Number of trips made, Their average speed, Total trip distance, Total Fare, Current Longitude & Latitude they were in)
2. Twitter API, we are only interested with the latest tweets from the Government Traffic Authority.
3. Traffic API, an outsourced API which provides the current volume of vehicles in a particular location.

### Data Flows:
[![Pipeline-architecture][architecture-screenshot]](https://github.com/1byte-yoda/taxi-demand-prediction/blob/main/docs/images/architecture.png)


#### Data Pipeline Flow for Predictive Analytics Use Case (Streaming):

1. "Stream Processor" will load a trained ML model from the data lake (the latest model version, and the model metadata), via the model packaging layer.
2. Then it will consume stream messages from 1 topic (Taxi App).
3. Consumed data will then go through the ML API which contains data preparation and feature engineering phase, such features will be derive with the help of the feature metadata from the data lake.
4. ML API will use the data from the previous step to make predictions, and expose the result into the Mobile App.

#### Data Pipeline Flow for Predictive Analytics Use Case (Batch):

1. "Batch Processor" will load a trained ML model from the data lake (the latest model version, and the model metadata).

2. Then it will parse from the data lake the latest hourly data (data version) for batch incremental learning/training, which will also perform data preparation and feature engineering.

3. The trained model will then be persisted into the data lake with the latest model version/metadata.

4. The latest model will then replace the current model used in the streaming pipeline, and be able to solve the problem with late data.

#### Data Preparation:
1. Outlier Removal (Trip times, Speed, Trip Distance, Total Fare, Coordinates)
2. Filling missing values
3. Text normalization for twitter data (stop words removal, special character removal, lemmatization)

#### Feature Engineering:
1. Clustering (Which cluster or location will a taxi gets assigned?)
2. Fourier Transformation (Trip times, Trip Distance, Total Fare)

#### Data Pipeline Flow for Analytics Dashboard Use Case:

1. Batch Pipeline will consume the data from 3 topics (Twitter data, Traffic data, Taxi App), and then store it into a data lake.
2. Data warehouse will then refine the data from the data lake for the analytics dashboard.
3. Data are now available for the dashboard use cases.


#### Open Source Technology:
1. Streaming Broker - Apache Kafka (It supports my use case, batch and stream processing. Also horizontally scalable in case data stream grows)
2. Data Processing Framework - Apache Spark (Distributed Processing Framework which supports my low latency requirements (streaming) and also batch processing for Analytics and Batch Learning. It also has support for Machine Learning (MLLib))
3. Data Lake - Local File System (I'll use this for simplicity, but the idea here is that I will need to store raw data with different file formats, and hence I'll be using a distributed file system, azure blob storage for example)
4. Data Warehouse - Apache Hive (Supports SQL querying and column oriented storage. The core idea here is to use a Database which supports column oriented storage since I'll store denormalized data here. Might as well SQL querying support for Data Analysts)
5. API - Flask (I chose Flask because I will use python and its also easy to develop APIs in Flask, hence, easier to maintain)
6. Dashboard - Redash (I choose Redash since it supports various data sources to create dashboards, ie. Hive. And also, I don't need complex visualizations, like in D3.js)
7. Logging & Monitoring - Prometheus and Grafana
   
<!-- MARKDOWN LINKS & IMAGES -->
[architecture-screenshot]: docs/images/architecture.png