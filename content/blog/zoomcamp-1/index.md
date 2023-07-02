---
title: DE Zoomcamp 2023
date: "2023-07-02"
description: "DE Zoomcamp Week 1 notes"
---

## Table of Contents

- [Brainstorm: Practices I'm pulling in from past software projects](#brainstorm--practices-i-m-pulling-in-from-past-software-projects)
  * [Take notes in Markdown](#take-notes-in-markdown)
  * [“Try three before Slack”](#-try-three-before-slack-)
  * [Ask the best possible question](#ask-the-best-possible-question)
  * [Have a growth mindset](#have-a-growth-mindset)
- [Notes: Docker and Postgres](#notes--docker-and-postgres)
  * [Introduction to Docker](#introduction-to-docker)
    + [Why Docker?](#why-docker-)
  * [Our first Dockerfile and Python pipeline](#our-first-dockerfile-and-python-pipeline)
    + [What is a Dockerfile?](#what-is-a-dockerfile-)
    + [Dockerfile Syntax](#dockerfile-syntax)
    + [Our first Dockerfile](#our-first-dockerfile)
    + [Understanding each instruction](#understanding-each-instruction)
    + [Our simple Python pipeline (pipeline.py)](#our-simple-python-pipeline--pipelinepy-)
  * [Ingesting NY Taxi Data to Postgres](#ingesting-ny-taxi-data-to-postgres)
    + [Key packages to install](#key-packages-to-install)
    + [Useful documentation](#useful-documentation)
    + [Docker commands to spin up database](#docker-commands-to-spin-up-database)
    + [Using pgcli to connect to postgres](#using-pgcli-to-connect-to-postgres)
    + [CSV link](#csv-link)
    + [Jupyter Notebook (upload_date.ipynb)](#jupyter-notebook--upload-dateipynb-)
  * [Connecting pgAdmin and Postgres](#connecting-pgadmin-and-postgres)
    + [Docker Networks](#docker-networks)
    + [Create a network](#create-a-network)
    + [Run Postgres](#run-postgres)
    + [Run pgAdmin](#run-pgadmin)
  * [Putting the ingestion script into Docker](#putting-the-ingestion-script-into-docker)
    + [Converting the Jupyter notebook to a Python script](#converting-the-jupyter-notebook-to-a-python-script)
    + [Parametrizing the script with argparse](#parametrizing-the-script-with-argparse)
      - [Why argparse?](#why-argparse-)
    + [Dockerizing the ingestion script](#dockerizing-the-ingestion-script)
    + [Docker Compose](#docker-compose)
- [SQL](#sql)
  * [Why?](#why-)
  * [Resources](#resources)
- [GCP](#gcp)
  * [What is GCP?](#what-is-gcp-)
  * [Initial GCP Setup](#initial-gcp-setup)
- [Terraform](#terraform)
  * [What is Terraform?](#what-is-terraform-)
  * [Resources](#resources-1)
  * [Important files](#important-files)
  * [Execution Steps](#execution-steps)
- [Virtual Environment Setup](#virtual-environment-setup)
  * [What is SSH?](#what-is-ssh-)
  * [Generate SSH Keys](#generate-ssh-keys)
  * [Upload Public Key to GCP](#upload-public-key-to-gcp)
  * [Create a VM](#create-a-vm)
  * [SSH into VM](#ssh-into-vm)
  * [Configure the VM](#configure-the-vm)
    + [Download Anaconda](#download-anaconda)
    + [Download Docker](#download-docker)
      - [Download docker-compose](#download-docker-compose)
  * [Install pgcli](#install-pgcli)
  * [SSH with VSCode](#ssh-with-vscode)
  * [Port Forwarding in VSCode](#port-forwarding-in-vscode)
  * [Use pgadmin and Jupyter to run upload-data notebook (ingest data)](#use-pgadmin-and-jupyter-to-run-upload-data-notebook--ingest-data-)
  * [Install Terraform](#install-terraform)
  * [SFTP Google Credentials to VM](#sftp-google-credentials-to-vm)
  * [Run Terraform commands](#run-terraform-commands)
  * [How to start and stop VM (DO NOT DELETE!)](#how-to-start-and-stop-vm--do-not-delete--)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Link to future weeks
[Link to notes](https://github.com/teacherc/de_zoomcamp_candace2023#date-engineering-zoomcamp---candaces-hw-and-notes)


## Brainstorm: Practices I'm pulling in from past software projects

### Take notes in Markdown

After I complete a unit of work, I take notes in a format that allows me
to show code and Markdown text in the same document via a GitHub gist,
[Jupyter Notebooks](https://jupyter.org/try), or Quarto.

[Google Colab](https://colab.research.google.com/) is a great place to take notes because Colab notebooks are free (you can even access free GPUs if necessary and linked to your Google account).


### “Try three before Slack”

When I taught elementary and middle school, I had a rule in my classroom
called “try three before me”. This meant that during work time, students
should try three sources of information before they asked me a question.
I coached students in how to do this before the rule went into effect.

Here is a checklist I use before I ask questions in the DE Slack or
StackOverflow:

- Look through the
  [FAQ](https://docs.google.com/document/d/19bnYs80DwuUimHM65UV3sylsCn2j1vziPOwzBwQrebw/edit)
- Search the DataTalksClub DE Zoomcamp Slack channel
- Copy the error to my clipboard and Google it
  - Google the error with terms like “reddit” or “stackoverflow” to find
    results from these particular sites
- Re-read materials or re-watch videos

### Ask the best possible question

The good thing about “trying three before Slack” is that you’ll gain
clarity on the best possible question to ask.

Here is how I ask questions in StackOverflow and Slack to provide context and reduce back-and-forths:

“I am trying to accomplish _______ (very specific step). I expected _______ but got _______ (error message
or issue). I tried to resolve the error by _______, _______, and _______
(write a quick list of the three steps you took to solve your problem). Can someone explain how to _______ (if necessary, explain the
information you need to move forward)."


### Have a growth mindset

In my experience, schools and businesses incentivize getting things
“right” right away. People who take on challenging work or actually
take time to grapple with concepts are not rewarded. This actually goes
against all research we have about learning. In order to learn, you need
to engage in tasks that provide a bit of struggle. As you struggle, new
connections form in your brain - this is how we learn.

------------------------------------------------------------------------

## Notes: Docker and Postgres

### Introduction to Docker

#### Why Docker?

The [Docker website](https://www.docker.com/resources/what-container/)
says it best:

“A container is a standard unit of software that packages up code and
all its dependencies so the application runs quickly and reliably from
one computing environment to another. A Docker container image is a
lightweight, standalone, executable package of software that includes
everything needed to run an application: code, runtime, system tools,
system libraries and settings.”

Containers are **very useful** to DEs because they allow us to bundle
computing environments, dependencies, and other important elements into
tidy packages that can be run by anyone (no matter which operating
system they are using or packages they have already installed).

Docker containers are also STATELESS. This means that when you start and stop a container, you lose any changes. When we create more sophisticated Docker instructions, we will [mount volumes](https://docs.docker.com/storage/volumes/).

### Our first Dockerfile and Python pipeline

[Video
link](https://www.youtube.com/watch?v=EYNwNlOrpr0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb)

A data pipeline is a process of moving data from one place (the source)
to a destination (such as a data warehouse).

#### What is a Dockerfile?

A `Dockerfile` is a text document that contains all of the commands to
assemble an image. Docker’s [Dockerfile
Reference](https://docs.docker.com/engine/reference/builder/) is a
helpful guide to understanding these files.

#### Dockerfile Syntax

Instructions are written in uppercase and specific arguments are
lowercase.

```dockerfile
INSTRUCTION arguments
```

#### Our first Dockerfile

``` dockerfile
FROM python:3.9

RUN pip install pandas

WORKDIR /app
COPY pipeline.py pipeline.py

ENTRYPOINT [ "python", "pipeline.py" ]
```

#### Understanding each instruction

| INSTRUCTION |  argument | What’s happening? |
|:--- | :--- |:--- |
| FROM        |  python:3.9  | Establishes a base image |
| RUN         |  pip install pandas  | Runs PIP to install Pandas |
| WORKDIR     |  /app  | Changes the current director to /app |
| COPY        |  pipeline.py pipeline.py  | Copies pipeline.py from our local host to app/pipeline.py in the container |
| ENTRYPOINT  |  “python”, “pipeline.py”  | Runs the `python` command to start the Python interpreter and runs pipeline.py |


#### Our simple Python pipeline (pipeline.py)

``` python
import sys
import pandas as pandas

print(sys.argv)

day = sys.argv[1]

# Fancy stuff with pandas

print(f'Job finished successfully for day = {day}')
```
***

### Ingesting NY Taxi Data to Postgres

#### Key packages to install

``` python
# You might need to use pip3 instead of pip (depends on your environment)
pip install pgcli
pip install notebook
pip install SQLAlchemy
pip install psycopg2-binary # You might not need this - I had to install it
```

#### Useful documentation

- [Pandas
  Dataframes](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html)
- [Jupyter Notebook](https://docs.jupyter.org/en/latest/)
- [SQLAlchemy
  Engine](https://docs.sqlalchemy.org/en/20/core/engines.html)
- [Iterators and Iterables - Corey
  Schafer](https://www.youtube.com/watch?v=jTYiNjvnHZY)

#### Docker commands to spin up database

``` python
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v /home/candace/Desktop/Code/zoomcamp/docker_test/ny_taxi_postgres_data \
  -p 5432:5432 \
  postgres:13
```

#### Using pgcli to connect to postgres

``` bash
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

#### CSV link

Link to CSV backup:
<https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz>


#### Jupyter Notebook (upload_date.ipynb)

``` python
# Import Pandas with the alias pd
import pandas as pd

# Check Pandas version
pd.__version__
```

``` python
# Create a Pandas dataframe with the first 100 rows of the csv
# See Pandas documentation or YouTube videos for info about dataframes
df = pd.read_csv('yellow_tripdata_2021-01.csv', nrows=100)
```

``` python
# Show dataframe
df
```

``` python
# Print schema
print(pd.io.sql.get_schema(df, name='yellow_taxi_data'))
```

``` python
# Change schema so that the pickup and dropoff datetimes are modeled as DATETIME instead of TEXT in the database schema
df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
```

``` python
# Import create_engine from SQLalchemy
# I had to import psycopg2 but you might not have to
from sqlalchemy import create_engine
import psycopg2
```

``` python
# Create a SQLalchemy engine using the local Docker container server we have running at port 5432
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')
```

``` python
# Connect to engine
engine.connect()
```

``` python
# Create a dataframe iterator that will iterate over the csv in chunks of 100,000
# See Corey Schafer video above for info about iterators
df_iter = pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100_000)
```

``` python
# Import time module so we can calculate how long each iteration takes
from time import time
```

``` python
# Establish a while loop
while True:
    # Capture the start time
    t_start = time()
    
    # This is an iterable - it's moving to the next iteration
    df = next(df_iter)
    
    # Change schema to reflect datetime (vs. text)
    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
    
    #Use the 'to_sql' method - connect via the engine. If a record already exists, append it
    df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')
    
    # Capture end time of iteration
    t_end = time()
    
    # Print how long that iteration took
    print('inserted another chunk...took %.3f second' % (t_end - t_start))

    # Note - The code will end with an exception
```
***

### Connecting pgAdmin and Postgres

Note: This section was particularly hard for me because I did not have the right argument behind the -v flag. I consulted the FAQ and used a volume name instead of a path:

```dockerfile
-v ny_taxi_postgres_data:/var/lib/postgresql/data
```

#### Docker Networks

We need to use a Docker network so that one container that is running pgAdmin can communicate with another container that has the database (with a mounted volume).

#### Create a network

``` bash
docker network create pg-network
```

#### Run Postgres

``` python
##Network
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v /home/candace/Desktop/Code/zoomcamp/docker_test/ny_taxi_postgres_data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13
```

#### Run pgAdmin

``` python
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin2 \
  dpage/pgadmin4
```
***

### Putting the ingestion script into Docker

#### Converting the Jupyter notebook to a Python script

Jupyter notebooks are useful because they help us:
- Create a [computational narrative](https://odsc.medium.com/why-you-should-be-using-jupyter-notebooks-ea2e568c59f2) that others can understand and modify independently
- Document our actions and learning with both code snippets and text
- Run pieces of a script or a program line by line (this helps with troubleshooting)

We used a Jupyter notebook to write and use a Python script. A script is a set of instructions for an external system (a script can be deleted without changing or disabling the target system). Scripts are an important part of data pipelines because we need sets of instructions to manipulate data (including downloading, ingesting, etc).

Now, we need to take the instructions in the Jupyter notebook, copy-paste them into a python file (ours will be called ingest_data.py) and clean up the instructions so a Python interpreter can read them.

#### Parametrizing the script with argparse

After creating the ingest_data.py file, copying the Jupyter code blocks over, and cleaning up the file (see video), we need to use argparse.

##### Why argparse?

When we created our Jupyter notebook, we were making it for a very specific use case. We used these arguments to specify the name of the file ('yellow_tripdata_2021-01.csv'), postgres-related information for the engine (username, password, port, and name - 'postgresql://root:root@localhost:5432/ny_taxi'), and other information.

To make our script more extendable, we need to make it so that many of these pieces of information are variables that a user can pass through when they run the script. Python's argparse module allows us to communite with the user - the script will use argparse to know which arguments to take in from the user and where to pass them into the script. This is a great [primer](https://towardsdatascience.com/learn-enough-python-to-be-useful-argparse-e482e1764e05) on argparse from the perspective of a data scientist.

This is what our script looks like after pulling it from our Jupyter notebook into a .py file, cleaning it up, and using argparse:

```python
import pandas as pd
from sqlalchemy import create_engine
import psycopg2
from time import time
import argparse
import os


# Main function - ingests csv data to postgres db
def main(params):
    # define parameters
    user = params.user
    password = params.password
    host = params.host
    port = params.port
    db = params.db
    table_name = params.table_name
    url = params.url
    csv_name = 'output.csv.gz'
   
    # download the csv
    os.system(f"wget {url} -O {csv_name}")

    # prepare data for ingestion
    engine = create_engine(f'postgresql://{user}:{password}@{host}:{port}/{db}')

    df_iter = pd.read_csv(csv_name, iterator=True, chunksize=100_000)

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

    df.head(n=0).to_sql(name=table_name, con=engine, if_exists='replace')

    df.to_sql(name=table_name, con=engine, if_exists='append')

    # while loop for data ingestion
    while True:
        t_start = time()
        
        df = next(df_iter)
        
        df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
        df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
        
        df.to_sql(name=table_name, con=engine, if_exists='append')
        
        t_end = time()
        
        print('inserted another chunk...took %.3f second' % (t_end - t_start))

# main function
if __name__ == "__main__":
    # parse arguments - user, password, host, port, database name , table name, url of the csv
    parser = argparse.ArgumentParser()
    parser.add_argument('--user', help='username for postgres')
    parser.add_argument('--password', help='password for postgres')
    parser.add_argument('--host', help='host for postgres')
    parser.add_argument('--port', help='port for postgres')
    parser.add_argument('--db', help='database name for postgres')
    parser.add_argument('--table_name', help='name of the table where we will write the results to')
    parser.add_argument('--url', help='url of the csv file')

    args = parser.parse_args()

    main(args)

```

Since this script is a Python program, we had to use 'if __name__ == "__main__'. Check out Corey Schafer's [YouTube video about 'if __name__ == "__main__'](https://www.youtube.com/watch?v=sugvnHA7ElY) for more information about it.

To test the script, we have to drop our existing table (via pgadmin) and run this in a terminal:

```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}
```

After running this, the table should be visible in pgadmin (after a refresh.)

#### Dockerizing the ingestion script

It would be VERY annoying if we had to run three containers whenever we wanted to ingest and see data in pgadmin. We can do all of this with one yaml file.

The first step is to update the dockerfile:

```dockerfile
FROM python:3.9

RUN apt-get install wget
RUN pip install pandas sqlalchemy psycopg2

WORKDIR /app
COPY ingest_data.py ingest_data.py

ENTRYPOINT [ "python", "ingest_data.py" ]
```

Then, we build the Docker container (don't forget the period at the end!):

```bash
docker build -t taxi_ingest:v001 .
```

Now, our ingestion script can be run via Docker (with a few modifications):

```dockerfile
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```

#### Docker Compose

We make a docker-compose.yaml file that looks like this:

```yaml
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
    ports:
      - "5432:5432"
  pgadmin:
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=root
    ports:
      - "8080:80"
```

Then, we make sure that we kill containers that are already running (CTRL+C in running terminals). Now, we can test it:

```
docker-compose up
```

I like to use 'detached' mode so I can use the terminal as the container is running:

```
docker-compose up -d
```

When the container is running, you should be able to log in to pgadmin, add the server, and see the tables we've created.

***

## SQL

### Why?
SQL allows us to store, manipulate, and query information in relational databases. Each type of SQL databases has a different flavor of SQL (MySQL, PostgresQL, etc).

We are using Postgres so our flavor is PostgreSQL.

### Resources

My favorite SQL links:
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Danny Ma's 8-Week SQL Challenge](https://8weeksqlchallenge.com/case-study-1/)
- [Socratica's SQL lessons on YouTube](https://www.youtube.com/watch?v=nWyyDHhTxYU)

## GCP

### What is GCP?

Google Cloud Platform is a provider of cloud computing and storage services.

Useful resources
- [Containers vs VMs: What's the difference?](https://www.youtube.com/watch?v=cjXI-yxqGTI)
- [GCP video channel](https://www.youtube.com/@googlecloudtech)

### Initial GCP Setup

1. Signup for a free trial (you can create a new email account if you need another trial)
2. Create your first project and note the "Project ID" (for future steps)
3. Create a servie account for this project and give it roles [IAM](https://console.cloud.google.com/iam-admin/iam)
  a. Viewer
  b. Storage Admin
  c. Storage Object Admin
  d. BigQuery Admin
4. Download the service account keys (.json file)
5. Download the [SDK](https://cloud.google.com/sdk/docs/quickstart)
6. Set environment variable to point to your downloaded GCP keys:
```console
export GOOGLE_APPLICATION_CREDENTIALS="<path/to/your/service-account-authkeys>.json"
```
7. Enable APIs
  a. https://console.cloud.google.com/apis/library/iam.googleapis.com
  b. https://console.cloud.google.com/apis/library/iamcredentials.googleapis.com
8. Set the default login and verify auth
```gcloud auth application-default login```

## Terraform

### What is Terraform?
Terraform is an open-source Infrastructure-as-Code (IaC) tool created by Hashicorp. It allows software developers to define and manage their infrastructure via code (instead of using GUIs and other tools). Being able to share, version, and test infrastructure the same way we test code is VERY useful.

### Resources
- [What is Terraform & Infrastructure as Code (IaC)?](https://acloudguru.com/blog/engineering/what-is-terraform-infrastructure-as-code-iac)
- [Terraform GCP Documentation](https://developer.hashicorp.com/terraform/tutorials/gcp-get-started)

### Important files

main.tf
```tf
terraform {
  required_version = ">= 1.0"
  backend "local" {}  # Can change from "local" to "gcs" (for google) or "s3" (for aws), if you would like to preserve your tf-state online
  required_providers {
    google = {
      source  = "hashicorp/google"
    }
  }
}

provider "google" {
  project = var.project
  region = var.region
  // credentials = file(var.credentials)  # Use this if you do not want to set env-var GOOGLE_APPLICATION_CREDENTIALS
}

# Data Lake Bucket
# Ref: https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket
resource "google_storage_bucket" "data-lake-bucket" {
  name          = "${local.data_lake_bucket}_${var.project}" # Concatenating DL bucket & Project name for unique naming
  location      = var.region

  # Optional, but recommended settings:
  storage_class = var.storage_class
  uniform_bucket_level_access = true

  versioning {
    enabled     = true
  }

  lifecycle_rule {
    action {
      type = "Delete"
    }
    condition {
      age = 30  // days
    }
  }

  force_destroy = true
}

# DWH
# Ref: https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/bigquery_dataset
resource "google_bigquery_dataset" "dataset" {
  dataset_id = var.BQ_DATASET
  project    = var.project
  location   = var.region
}
```

variables.tf
```tf
locals {
  data_lake_bucket = "dtc_data_lake"
}

variable "project" {
  description = "Your GCP Project ID"
}

variable "region" {
  description = "Region for GCP resources. Choose as per your location: https://cloud.google.com/about/locations"
  default = "europe-west6"
  type = string
}

variable "storage_class" {
  description = "Storage class type for your bucket. Check official docs for more info."
  default = "STANDARD"
}

variable "BQ_DATASET" {
  description = "BigQuery Dataset that raw data (from GCS) will be written to"
  type = string
  default = "trips_data_all"
}
```

.terraform-version
```tf
1.0.2

```
### Execution Steps

```
# Refresh service-account's auth-token for this session
gcloud auth application-default login

# Initialize state file (.tfstate)
terraform init

# Check changes to new infra plan
terraform plan -var="project=<your-gcp-project-id>"
```

```
# Create new infra
terraform apply -var="project=<your-gcp-project-id>"
```

```
# Delete infra after your work, to avoid costs on any running services
terraform destroy
```

## Virtual Environment Setup

### What is SSH?

[SSH](https://www.ucl.ac.uk/isd/what-ssh-and-how-do-i-use-it) or "Secure Shell" is a communication protocol that allows computers to encrypt, exchange, and decrypt information. 

[SSH keys](https://jumpcloud.com/blog/what-are-ssh-keys) come in pairs. The public key can be widely shared and is used to encrypt information. The private key can decrypt information (make sure your private key is never shared).

### Generate SSH Keys

Resource: [Google Cloud documentation - Create SSH Keys](https://cloud.google.com/compute/docs/connect/create-ssh-keys)

1. Make sure you are in the ```~/.ssh``` directory (typically ```cd /.ssh```)

If you do not have this directory, you should create it. Use the ```ls``` command in your terminal to see which files are already in this directory. Files that end with ```.pub``` are public keys.

2. Use the ```ssh-keygen``` utility 

Use this (or a variation of this) command:
```ssh-keygen -t rsa -f ~/.ssh/KEY_FILENAME -C USERNAME -b 2048```

My example: ```ssh-keygen -t rsa -f ~/.ssh/gcp -C candace -b 2048```

When it prompts you for a passphrase, leave it empty.

3. Use the ```ls``` command to make sure the public AND private keys are there (```gcp``` and ```gcp.pub```)

### Upload Public Key to GCP

4. Navigate to the [VM Metadata](https://console.cloud.google.com/compute/metadata?_ga=2.27862460.1443282184.1674762012-1094820131.1673986597) page and add your PUBLIC ssh keys (ssh tab)

In the terminal window where you are at the ```~/.ssh``` directory, type ```cat gcp.pub``` to have the terminal print the contents of your PUBLIC ssh key. Copy the text of the key and paste it into the ssh section of the Metadata page in Google Cloud Console.

### Create a VM

 Create a new VM instance in the cloud console

Pay particular attention to:
- Name - ```de-zoomcamp```
- Region - You can choose a low CO2 option
- Series - E2
- Machine - E2 - Standard - 4 or higher (watch out for the cost)
- Boot Disk
  - Operating system - Ubuntu
  - Version - 20.04 LTS (AMD64)
  - Boot Disk Type - Balanced Persistent Disk
  - Boot Disk Size - 30 GB

Note the external IP (copy it from the list of VM instances) 

### SSH into VM

In your terminal write ```ssh -i ~/.ssh/gcp USERNAME@VM_EXTERNAL_GCP```

My example: ```ssh -i ~/.ssh/gcp candace@22.222.22.22```

If it asks you to continue, say yes.

You will be logged into the VM (as you can see in the terminal - ```candace@de-zoomcamp```)


### Configure the VM

#### Download Anaconda

Go to [this page](https://www.anaconda.com/products/distribution) and find the latest 64-Bit (x86) Installer. Right-click on the installer link to copy the link.

In your de-zoomcamp terminal do a wget to download the file

```wget LINK_TO_INSTALLER```

My example: ```wget https://repo.anaconda.com/archive/Anaconda3-2022.10-Linux-x86_64.sh```

The last line of the download will give you a link to a file. Copy that to create the next command:

```bash FILENAME```

example: ```bash Anaconda3-2021.11-Linux-x86_64.sh```

Run the installer and accept the terms of service with 'yes'. Let it run ```conda init``` if asked. 

Log out of your ssh session and log back in. You should see the word (base) in front of your console name. Test ```which python``` to make sure it is coming from the Anaconda directory. Type ```source .bashrc``` if you do not want to log in and out.

#### Download Docker
From your ssh terminal:
```sudo apt-get update```
```sudo apt-get install docker.io```

Follow these directions to run Docker without sudo: https://github.com/sindresorhus/guides/blob/main/docker-without-sudo.md

##### Download docker-compose

Head to the docker-compose releases page: https://github.com/docker/compose/releases/tag/v2.15.1

Make a ```bin``` directory:

```console
mkdir bin
cd /bin
```

In the list of releases, find the latest ```Linux-x86-64``` release. Copy-paste the link.

My example: https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-linux-x86_64

Add this to a ```wget``` in the ```/bin``` directory. Specify the output with flags.

Example: ```wget https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-linux-x86_64 -O docker-compose```

Use ```ls``` to in ```/bin``` to make sure the ```docker-compose``` file is there.

Run ```chmod +x docker-compose``` to make the file executable.

The last step to installing docker-compose is to make it executable from any directory.

In your ssh terminal, in the home directory, run ```nano .bashrc``` to open up your bashrc file.

At the end of the file, type: ```export PATH="{HOME}/bin:${PATH}"

Save by hitting ENTER and typing CTRL-X. 

Read the .bashrc file with ```source .bashrc```.

In the home directory, test your work by typing ```which docker-compose```

### Install pgcli

In the ssh terminal, type ```pip install pgcli```

### SSH with VSCode

In a terminal in your home directory (not the ssh terminal), navigate to the ```~/.ssh``` directory.

Use ```touch config``` to create a config file in the ```~/.ssh``` directory.

Open that file in VSCode.

config file
```
Host de-zoomcamp
  HostName 22.222.22.22 #Put the external IP from the GCP VM here
  User candace
  IdentityFile ~/.ssh/gcp # Different on Windows
```

From a terminal in your home directory, try ```ssh de-zoomcamp```. Since you created the config file, this should create a ssh connection to your VM instance.

Download the [Remote - SSH](https://code.visualstudio.com/docs/remote/ssh-tutorial) extension in VSCode.

Open the Command Pallette in Docker and find the ```Remote-SSH Connect to Host...``` option. It should off you ```de-zoomcamp``` right away.

Open a terminal in VSCode - now you have a ssh session in VSCode

### Port Forwarding in VSCode

In the terminal area, click on ```Port Forwarding```. Add the ports 5432, 8888, and 8080 to the list.

### Use pgadmin and Jupyter to run upload-data notebook (ingest data)

Clone the Zoomcamp repo: ```git clone https://github.com/DataTalksClub/data-engineering-zoomcamp.git```

Navigate to ```~data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql``` in your ssh terminal

Use the command ```docker-compose up -d``` to build the docker containers (it'll use the Dockerfile)

Run ```docker ps``` to see if both containers are running. Note the name of the pgdatabase terminal. In my case, it was ```2_docker_sql_pgdatabase-1```

You should be able to access pgadmin in a browser at ```localhost:8080```

Use the command ```pgcli -h localhost -p 5432 -u root -d ny_taxi``` to check out the schema of the database using pgcli

Navigate to ```~data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql``` in your ssh terminal and run the command ```jupyter notebook```

Select the ```upload_data.ipynb``` notebook. Use the process we used earlier to ingest the yellow taxi data, zones, and green taxi data (you'll need to edit the code blocks for each ingestion).

Links:
- Yellow taxi data: https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz 
- Zones: https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
- Green taxi data: https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz

Use pgcli or pgadmin to make sure that the data has been ingested properly.


### Install Terraform

Navigate to the [Terraform downloads page](https://developer.hashicorp.com/terraform/downloads)

Right-click on the Linux AMD64 link (example: https://releases.hashicorp.com/terraform/1.3.7/terraform_1.3.7_linux_amd64.zip)

Navigate to your ```/bin``` folder and run ```wget https://releases.hashicorp.com/terraform/1.3.7/terraform_1.3.7_linux_amd64.zip```

Run ```unzip terraform_1.3.7_linux_amd64.zip``` to unzip the file.

If you do not have unzip installed, run ```sudo apt-get install unzip``` first. 

Run ```rm terraform_1.3.7_linux_amd64.zip``` to remove the zip file from storage.

Since we've already added ```/bin``` to the path folder and the unzipped terraform file is executable, you should be able to run terraform commands.

Navigate to ```~data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform```

The ```main.tf``` and ```variables.tf``` files should be in that directory.


### SFTP Google Credentials to VM

In a local terminal, navigate to where you stored your .json account keys.

From the local terminal, start sftp ```sftp de-zoomcamp```

In the sftp session, make a ```.gc``` directory. ```mkdir .gc```

Enter the ```.gc``` directory - ```cd .gc```

Use the ```put``` command to transfer the files to the ```.gc``` directory.

Example: ```put ny_rides.json

Use one of your ssh terminals to make sure that the ```ny_rides.json``` file is in the ```.gc``` directory.

In the ssh directory, run these commands to authenticate with the ssh file:

```console
export GOOGLE_APPLICATION_CREDENTIALS=~/.gc/ny_rides.json
```

```console 
gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS
```

### Run Terraform commands

Navigate back to ```~data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform``` in a ssh window

The ```main.tf``` and ```variables.tf``` files should be in that directory.

Run these terraform commands:
```terraform init```
```terraform plan``` (make sure you have your GCP Project ID handy)
```terraform apply```

### How to start and stop VM (DO NOT DELETE!)

On the GCP VM page, you can click the three dots next to your running instance. Select STOP to pause the instance (all of your files will be available when you return to the VM). Do not press DELETE unless you want all of your information to be deleted.

When you need to use the VM again, start the VM in GCP (three dots). Take note of the External IP. Edit the config file in the ```~/.ssh``` folder with the new External IP. Save the file and ssh via the terminal or VSCode.
