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

in order to mitigate this, we will first upload the file to our Hadoop Linux Cluster (a server we connect to), and then upload it to the HDFS from there. 

**Move the file from your local PC to the temporary (tmp) directory in your Hadoop cluster, which has over 10 GB of dedicated storage, but will routinely delete all files in this directory, so make sure not to leave a file inside for more than 3 days a time to be safe:**

## First, check if your personal folder exists within the directory:

**ls /tmp/username**

## If you get no error code, that means your folder already exists and you are good to continue. If you get an error code or directory does not exist, use the following to create your personal directory within /tmp:

**mkdir /tmp/username** -tells CLI to create your personal folder within the tmp storage system. 

**Upload dataset from Local PC to the Hadoop Linux Cluster /tmp Path**

**scp "path/to/used_cars_data.csv" username@cluster_ip:/tmp/username/** 

-scp means secure copy, used to copy files across computers, followed by the path of file you are going to upload, think of it as directions the computer will use to get to that file, then you use your username@ip_address:file_path within the cluster, telling the computers where to copy the file to

**example in this case: scp C:\Users\erik\Downloads\used_cars_data.csv eebange@161.153.21.139:/tmp/eebange/**

## Confirm the file was uploaded successfully:

**ls -lh /tmp/username**

# Step 5: Creating HDFS Directories and Moving Data Into HDFS

Before we can start using the data to create tables in HIVE, we must first upload the file to the HDFS:

## create a folder inside HDFS so that we can upload our file there from the server (linux cluster):

**hdfs dfs -mkdir /user/username/used_cars_raw**

_note: hdfs and dfs are used so that the CLI specifies to create the folder in HDFS_

## Move the files from cluster to HDFS:

**hdfs dfs -put /tmp/username/used_cars_data.csv /user/username/used_cars_raw/**

## Confirm file was successfully moved to HDFS:

**hdfs dfs -ls -h /user/username/used_cars_raw**

# Step 6: Creating the Raw Hive Table (with OpenCSVSerde)

Although the raw dataset came separated by commas, making it a CSV, the records themselves had commas, quotes, list text, and symbols in them. Because of this, HIVE would automatically misalign the data (having strings in columns that should be int, and vice versa)

In order to mitigate this, we create the table in HIVE using the raw dataset






















































