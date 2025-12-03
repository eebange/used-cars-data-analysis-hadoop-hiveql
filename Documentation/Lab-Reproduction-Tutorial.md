# Reproduction Lab Tutorial for CarGurus Dataset Project

Reference this file as a step by step reproduction guide for this and similar projects

# Objectives

In this hands-on lab, you will learn how to:

**Download and Prepare Large Datasets on a Local Computer** 

Handle multi-GB CSV files and avoid issues with Excel/OneDrive file locks.

**Connect to an Apache Hadoop Cluster using SSH (Secure Shell)**

Use Linux commands, navigate directories, and maintain active sessions.

**Upload Large Files to the Hadoop Cluster**

Transfer 10GB+ datasets using scp and stage them in /tmp/<user> before loading to HDFS.

**Create, Navigate, Rename, and Delete Directories on the Hadoop Linux Node**


Manage personal workspace on the cluster (/home/<user> and /tmp/<user>).

**Load Data into HDFS (Hadoop Distributed File System)**


Create HDFS directories and upload local files into distributed storage.

**Register Raw CSV Files as Hive External Tables Using OpenCSVSerde**
Ensure correct parsing of CSV files that contain commas, quotes, or complex values.

**Inspect and Validate Raw Data Directly in Hive**
Use SELECT, LIMIT, and DESCRIBE to confirm correct column alignment.

**Identify and Remove Corrupted or Misaligned Records**
Use HiveQL filtering and regular expressions to clean the dataset and enforce data integrity rules.

**Create Clean Sample Tables for Analysis (Sampling Techniques)**
Use RAND() <= x to reduce dataset size for visualization and local analysis.

**Export Cleaned Tables from Hive to HDFS in Safe Formats**
Use INSERT OVERWRITE with tab-delimited (\t) output to avoid comma-related corruption.

**Download Cleaned Data from HDFS to Local Machine**
Use hdfs dfs -get and scp to move data from the cluster to your PC.

**Prepare the Dataset for Tableau and Excel Analysis**
Handle delimiter issues, remove units (e.g., "in", "gal", "seats"), convert string fields into numeric fields, and import into BI tools.

**Perform Full Column-By-Column Validation**
Verify VIN integrity, numeric fields, booleans, coordinates, year ranges, and dimension formats using HiveQL regular expressions.

## Hadoop System Specifications
- **Hadoop Version:** Hadoop 3.1.2
- **Hive Version:** Hive 3.1.2
- **HDFS Capacity:** 390.17 GB Configured for Usage
- **# of DataNodes (WorkerNodes):** 3, Report permission denied, but provided information by Professor Jungwook Woo
- **CPU Model:** AMD EPYC 7J13 64-Core Processor
- **CPU Speed:** 2445.406 MHz
- **CPU Cores:** 6
- **Memory Total:** 31 G
- **Cluster Information:** Oracle Cloud Infrastructure (OCI)
BigDAimn0.hadoop.iscu.ac.kr

# Step 1: Researching Online Datasets
For this project assignment, we were tasked to research, choose, and download a valid dataset that was at least 2GB in size.
The project began by identifying a large real-world dataset suitable for Big Data processing and tempo-spatial analysis. The dataset needed to contain temporal information (dates, timestamps) and spatial information (city, latitude, longitude).
We decided on Kaggle, _a platform for data science and maching learning professionals._ Users can share and collaborate on datasets with other data scientists and engineers.
We decided to use [This Kaggle CarGurus Dataset](https://www.kaggle.com/datasets/ananaymital/us-used-cars-dataset?resource=download) for our project, but feel free to research a dataset of your choosing.

_NOTE: this specific dataset came in csv format, so this project assumes similarily formatted files_

# Step 2: Downloading the files 
[Navigate to this Dataset](https://www.kaggle.com/datasets/ananaymital/us-used-cars-dataset?resource=download)

## Dataset Selected: Used Car Listings (Cargurus 3 Million Rows)
- **Source: Kaggle**
- **CSV Format**
- **Size ~9.98 GB**
- **Description of Dataset:** Large dataset scraped from CarGurus containing 3 million used car listings with attributes like VIN, price, mileage, location, dealer information, vehicle dimensions, engine specs, etc.
## Characteristics of Dataset
- **The dataset uses comma separted values**
- **The raw dataset contained 66 columns**
- **Records in the raw dataset contained data in string format**
- **Raw data is too large for Excel to handle given its limitations, uploaded to Hadoop for storage and HDFS to parse in Hive**
- **Raw file had misaligned data in wrong columns, needed to be cleaned in Hive**

# Step 3: Connecting to Hadoop Cluster 
Once file is appropriately downloaded to local PC, we will upload the file to Hadoop using SSH

The next step is to use a command-line interpreter of your choosing (terminal on mac, shell on windows, or download 3rd party software like GitBash)
## SSH
**Access the a Hadoop file cluster using the SSH protocol:**

Open your command-line interpreter, for this case, we are using **GitBash**

  **ssh username@cluster_ip**
  
  **example: eebange@161.153.21.139**

_note: username will be your assigned personal username for the cluster_
_note: the cluster ip address you connect to will be the one given to you by either your professor or admin. In our case both were provided to us by our professor_

Once the connection is made, in your GitBash command line interface (CLI), we can test some commands:

**examples**

**ls** -tells the CLI to display the directories and files in your current directory. Think of a house, a current directory is the bedroom you are currently in, sub-directories are the storage containers in the rooms, and files are the items in the room

**pwd** -print working directory, think of it as telling the system what room you currently are in

**cd** **directory_name** -change directory, think of telling the system to change you to a different room

**mkdir** **folder_name** -make directory, think of it as telling the CLI to make a new storage container in your current room

_note: we will do the majority of our data ETL on our local PC, hadoop cluster, and HDFS using CLI commands_


# Step 4: Uploading the Dataset to the Cluster
Our raw dataset is over the storage limit allocated for us on our HDFS.
In order to address the memory limitation, first upload the file to the Hadoop Linux Cluster (a server we connect to), and then upload it to HDFS from there. 

Move the file from your local PC to the temporary (tmp) directory in your Hadoop cluster, which has over 10 GB of dedicated storage, but will routinely delete all files in this directory, so make sure not to leave a file inside for more than 3 days a time to be safe:

## First, check if your personal folder exists within the directory:

**ls /tmp/username**

## If you get no error code, that means your folder already exists and you are good to continue. If you get an error code or directory does not exist, use the following to create your personal directory within /tmp:

**mkdir /tmp/username** -tells CLI to create your personal folder within the tmp storage system. 

Upload dataset from Local PC to the Hadoop Linux Cluster /tmp Path

**scp "path/to/used_cars_data.csv" username@cluster_ip:/tmp/username/** 

-scp means secure copy, used to copy files across computers, followed by the path of file you are going to upload, think of it as directions the computer will use to get to that file, then you use your username@ip_address:file_path within the cluster, telling the computers where to copy the file to

**example in this case: scp C:\Users\erik\Downloads\used_cars_data.csv eebange@161.153.21.139:/tmp/eebange/**

## Confirm the file was uploaded successfully:

**ls -lh /tmp/username**

# Step 5: Creating HDFS Directories and Moving Data Into HDFS

Before we can start using the data to create tables in HIVE, we must first upload the file to the HDFS:

## Create a folder in the HDFS so that we can upload and store our large dataset from the hadoop cluster server:

**hdfs dfs -mkdir /user/username/used_cars_raw**

_note: hdfs and dfs are used so that the CLI specifies to create the folder in HDFS_

## Move the files from cluster to HDFS:

**hdfs dfs -put /tmp/username/used_cars_data.csv /user/username/used_cars_raw/**

## Confirm file was successfully moved to HDFS:

**hdfs dfs -ls -h /user/username/used_cars_raw**

# Step 6: Creating the Raw Hive Table (with OpenCSVSerde)

Although the raw dataset came separated by commas, making it a CSV, the records themselves had commas, quotes, list text, and symbols in them. Because of this, HIVE would automatically misalign the data (having strings in columns that should be int, and vice versa)

In order to mitigate this, we are doing 2 important things:
1. Format every column as a **STRING**
2. Register the file in HIVE as an external table in HDFS **/user/eebange/used_cars_raw** to avoid accidentally dropping our tables, which would delete the source data. Doing this adds a level of security for the data
_note: we are not cleaning the data yet, this is simply to upload the dataset and create a raw dataset inside HIVE_

## Use your database
while in beeline, make sure to use your own database so that changes made are reflected on your own profile/databases.

**use username_database;**

**example: use eebange;**

## Create the raw table _used_cars_raw_ , applying the STRING datatype to all records

CREATE EXTERNAL TABLE IF NOT EXISTS used_cars_raw (

  vin                     STRING,
  
  back_legroom            STRING,
  
  bed                     STRING,
  
  bed_height              STRING,
  
  bed_length              STRING,
  
  body_type               STRING,
  
  cabin                   STRING,
  
  city                    STRING,

  city_fuel_economy       STRING,
  
  combine_fuel_economy    STRING,
  
  daysonmarket            STRING,
  
 
  dealer_zip              STRING,
  
  description             STRING,
  
  engine_cylinders        STRING,
  
  engine_displacement     STRING,
  
  engine_type             STRING,
  
  exterior_color          STRING,
  
  fleet                   STRING,
  
  frame_damaged           STRING,
  
  franchise_dealer        STRING,
  
  franchise_make          STRING,
  
  front_legroom           STRING,
  
  fuel_tank_volume        STRING,
  
  fuel_type               STRING,
  
  has_accidents           STRING,
  
  height                  STRING,
  
  highway_fuel_economy    STRING,
  
  horsepower              STRING,
  
  interior_color          STRING,
  
  iscab                   STRING,
  
  is_certified            STRING,
  
  is_cpo                  STRING,
  
  is_new                  STRING,
  
  is_oemcpo               STRING,
  
  latitude                STRING,
  
  length                  STRING,
  
  listed_date             STRING,
  
  listing_color           STRING,
  
  listing_id              STRING,
  
  longitude               STRING,
  
  main_picture_url        STRING,
  
  major_options           STRING,
  
  make_name               STRING,
  
  maximum_seating         STRING,
  
  mileage                 STRING,
  
  model_name              STRING,
  
  owner_count             STRING,
  
  power                   STRING,
  
  price                   STRING,
  
  salvage                 STRING,
  
  savings_amount          STRING,
  
  seller_rating           STRING,
  
  sp_id                   STRING,
  
  sp_name                 STRING,
  
  theft_title             STRING,
  
  torque                  STRING,
  
  transmission            STRING,
  
  transmission_display    STRING,
  
  trimid                  STRING,
  
  trim_name               STRING,
  
  vehicle_damage_category STRING,
  
  wheel_system            STRING,
  
  wheel_system_display    STRING,
  
  wheelbase               STRING,
  
  width                   STRING,
  
  year                    STRING
)


ROW FORMAT DELIMITED

FIELDS TERMINATED BY ','

STORED AS TEXTFILE

LOCATION '/user/eebange/used_cars_raw/'

TBLPROPERTIES ('skip.header.line.count' = '1');


_note: usee all strings in setting up the raw dataset in HIVE so that there is no accidental data misalignments due to commas, symbols, quotes, etc._

## Test table is working properly
To see if the table is working properly, use the following query to select all records from the table, limiting the result to 5 records

**select * from used_cars_raw limit 5;**

# Step 7: Create a Cleaned Hive Table _used_cars_cleaned_
After saving the raw data table _used_cars_raw_ the next step is to build a cleaned up, sample for the analysis

For the cleaned table, omit the following 15 columns or attributes due to high null rates (over 86%), irrelevant or duplicate information (horsepower vs power), or just generally badly parsed columns:

## omitted columns from _used_cars_cleaned_
bed - whether the vehicle had a bed inside or not,  over 95% null

bed_height - height of bed, over 90% null

bed_length - length of bed, over 90% null

cabin - if vehicle had cabin, over 86% null

combine_fuel_economy - combined mileage range, could calculate based on city and highway

description - omitted due to poor structure, little to no pattern or relevance 

engine_type - similar data to engine displacement

fleet - whether car was part of a fleet, mostly null, omitted

is_certified - whether or not vehicle was certified, same as is_cpo

main_picture_url - data type was link to images, challenging to analyze and interpret given current tools

major_options - string columns with little to no cohesion between similar models

power - same information as horsepower

torque - omitted, found unnecessary 

vehicle_damage_category - vehicle damage category, had too many null records

## Cleaned table _used_cars_cleaned_ with a sample of data

use username;

CREATE TABLE used_cars_cleaned AS

SELECT
   
   vin,
   
   back_legroom,
   
   body_type,
   
   city,
   
   city_fuel_economy,
   
   daysonmarket,
   
   dealer_zip,
   
   engine_cylinders,
   
   engine_displacement,
   
   exterior_color,
   
   frame_damaged,
   
   franchise_dealer,
   
   franchise_make,
   
   front_legroom,
   
   fuel_tank_volume,
   
   fuel_type,
   
   has_accidents,
   
   height,
   
   highway_fuel_economy,
   
   horsepower,
   
   interior_color,
   
   iscab,
   
   is_cpo,
   
   is_new,
   
   is_oemcpo,
   
   latitude,
   
   length,
   
   listed_date,
   
   listing_color,
   
   listing_id,
   
   longitude,
   
   make_name,
   
   maximum_seating,
   
   mileage,
   
   model_name,
   
   owner_count,
   
   price,
   
   salvage,
   
   savings_amount,
   
   seller_rating,
   
   sp_id,
   
   sp_name,
   
   theft_title,
   
   transmission,
   
   transmission_display,
   
   trimid,
   
   trim_name,
   
   wheel_system,
   
   wheel_system_display,

   wheelbase,
   
   width,
    
   year
   
FROM used_cars_raw;

## Check that the cleaned table _used_cars_cleaned_ is working properly

the following will return the first 10 records in the dataset

**SELECT * FROM used_cars_cleaned LIMIT 10;**

the following will count all the records in the dataset

**select count(*) from used_cars_cleaned**

# Step 8: Export the Cleaned Dataset to HDFS in TSV Format

After creating _used_cars_cleaned_, the next step is to export the clean dataset for analysis

_note: we used tsv instead of csv because excel was having a hard time parsing the data, we opted to focus on tableau for visualizations, as it handles tsv parsing significantly better_

## Export _used_cars_cleaned_ table to a directory in HDFS 

INSERT OVERWRITE DIRECTORY '/user/username/exports/used_cars_cleaned_tsv'

ROW FORMAT DELIMITED

FIELDS TERMINATED BY '\t'

SELECT * FROM used_cars_cleaned;

## Confirm files exported successfully to HDFS

hdfs dfs -ls -h /user/username/exports/used_cars_cleaned_tsv

## Copy exported files from HDFS to your clusters _/tmp_ directory

hdfs dfs -get /user/username/exports/used_cars_cleaned_tsv /tmp/username/

## Confirm file exported to your _/tmp_ directory

ls /tmp/username

## Download _used_cars_cleaned_tsv_ to your Local PC, using **GitBash**

scp -r user_name@ip_address:/tmp/user_name/used_cars_cleaned_tsv ~/Downloads/

example: scp -r eebange@161.153.21.139:/tmp/eebange/used_cars_cleaned_tsv ~/Downloads/

_note: will ask for your password, in this case, same as username. then a window should appear with a download progress_













































