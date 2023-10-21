# Spotify End-to-End Data Engineering Project

### Introduction
In this project, we will build an ETL pipeline using Spotify API on AWS. The pipeline will retrieve data from Spotify API, transform it to a desired format, and load it into an AWS data store.

### Architecture
![Architecture Diagram](https://github.com/PHenriquesReps/spotify-end-to-end-dataengineering-project/blob/main/Architecture.png)

### About Dataset/API
This API contains information about music artists, albums, and songs. For this project, we retrieve data from the Top 50 Global playlist.

[Spotify API](https://open.spotify.com/playlist/37i9dQZEVXbNG2KDcFcKOF)

### Services Used

* **Amazon S3 (Simple Storage Service):** is a highly scalable object storage service that can store and retrieve any data anywhere on the web. It is commonly used to store and distribute large media files, data backups, and static website files.

* **AWS Lambda:** is a serverless computing service that lets you run your code without managing servers. You can use Lambda to run code responding to events like changes in S3, DynamoDB, or other AWS services.

* **Amazon CloudWatch:** is a serverless computing service that lets you run your code without managing servers. You can use Lambda to run code responding to events like changes in S3, DynamoDB, or other AWS services.

* **AWS Glue Crawler:** is a fully managed service that automatically crawls your data sources, identifies data formats, and infers schemas to create an AWS Glue Data Catalog.

* **AWS Glue Data Catalog:** is a fully managed metadata repository that makes it easy to discover and manage data in AWS. You can use the Glue Data Catalog with other AWS services, such as Athena.

* **Amazon Athena:** is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL. You can use Athena to analyze data in your Glue Data Catalog or in other S3 buckets.


### Project Execution Flow
Crate the Bucket and the folder structure -> Extract Data From Sporify API (spotify_api_data_extarct) -> Store the Raw Data on S3 -> Transform the Data and load it to S3 (spotify_transformation_load_function) -> Add Lambda Trigger to the Extract Function (to run weekly) -> Add Lambda Trigger to the Transform Function (run based on existence of files on the to_process folder) -> Activate Crawler to get the metadata of the Data and create Glue Data Catalog -> Query the data using Athena

1- create the bucket and the respective folder structure on S3.
##### Bucket Structure
The bucket was divided into 2 folders:

* raw_data
  - to_process
  - processed
* transformed_data
  - album_data
  - song_data
  - artist_data

The raw_data folder contains 2 folders inside, to_process and processed. 
The to_process contains the raw extracted data that was not processed yet. The processed contain the raw extracted data that was already processed.

Inside the transformed_data folder we have 3 folders, representing the 3 intances we are pulling data too.


2- Create the request and extract the data from the Spotify API, and store it in a raw format on the to_process folder. 
3- Create the transfomation and load function to clean, structure and retrive only the data we wanted to for further analysis.
The treated and transformed data will be stored on the transformed_data folder under it's respective instance.
After this the data will be moved to the processed folder being deleted from the to_process, so that it wont get processed again.

4- The 2 triggers were added to the pipeline, automating the pipeline run, based on the required schedule (weekly), and on the existence of new data on the to_process folder.
**Note**: To complete this step we need two different permissions: *AmazonS3FullAccess* and  *AWSLambdaRole*

5- Create the Crawler to read the data, teh data formats, and infer schemas. 

6- Create the tables on the Data Catalog using the crawler information.

7- Query the data using the AWS Athena. 


### Install Packages
```
import json
import os
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials
import boto3 
from datetime import datetime
from io import StringIO 
```

On Amazon Lambda the **Pandas** and **Spotipy** packages will require a layer creation to enable them to be used.
- AWS has its own Pandas layer, *AWSSDKPandas-Python310*, you just need to add it to the function.

- For Spotipy we need to provide the file with the package to AWS. [Spotipy Package](https://https://github.com/PHenriquesReps/spotify-end-to-end-dataengineering-project/blob/main/spotipy_layer.zip)

