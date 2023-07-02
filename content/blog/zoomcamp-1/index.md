---
title: DE Zoomcamp 2023
date: "2023-01-17 17:50:01 -0800"
description: "DE Zoomcamp Week 1 notes"
---
## Table of contents

- [Link to future weeks](#link-to-future-weeks)
- [Brainstorm: Practices I'm pulling in from past software projects](#brainstorm-practices-im-pulling-in-from-past-software-projects)
  - [Take notes in Markdown](#take-notes-in-markdown)
  - [“Try three before Slack”](#try-three-before-slack)
  - [Ask the best possible question](#ask-the-best-possible-question)
  - [Have a growth mindset](#have-a-growth-mindset)
- [Notes: Docker and Postgres](#notes-docker-and-postgres)
  - [Introduction to Docker](#introduction-to-docker)
    - [Why Docker?](#why-docker)
  - [Our first Dockerfile and Python pipeline](#our-first-dockerfile-and-python-pipeline)
    - [What is a Dockerfile?](#what-is-a-dockerfile)
    - [Dockerfile Syntax](#dockerfile-syntax)
    - [Our first Dockerfile](#our-first-dockerfile)
    - [Understanding each instruction](#understanding-each-instruction)
    - [Our simple Python pipeline (pipeline.py)](#our-simple-python-pipeline-pipelinepy)
  - [Ingesting NY Taxi Data to Postgres](#ingesting-ny-taxi-data-to-postgres)
    - [Key packages to install](#key-packages-to-install)
    - [Useful documentation](#useful-documentation)
    - [Docker commands to spin up database](#docker-commands-to-spin-up-database)
    - [Using pgcli to connect to postgres](#using-pgcli-to-connect-to-postgres)
    - [CSV link](#csv-link)
    - [Jupyter Notebook (upload_date.ipynb)](#jupyter-notebook-upload_dateipynb)
  - [Connecting pgAdmin and Postgres](#connecting-pgadmin-and-postgres)
    - [Docker Networks](#docker-networks)
    - [Create a network](#create-a-network)
    - [Run Postgres](#run-postgres)
    - [Run pgAdmin](#run-pgadmin)
  - [Putting the ingestion script into Docker](#putting-the-ingestion-script-into-docker)
    - [Converting the Jupyter notebook to a Python script](#converting-the-jupyter-notebook-to-a-python-script)
    - [Parametrizing the script with argparse](#parametrizing-the-script-with-argparse)

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

-   Look through the
    [FAQ](https://docs.google.com/document/d/19bnYs80DwuUimHM65UV3sylsCn2j1vziPOwzBwQrebw/edit)
-   Search the DataTalksClub DE Zoomcamp Slack channel
-   Copy the error to my clipboard and Google it
    -   Google the error with terms like “reddit” or “stackoverflow” to find
        results from these particular sites
-   Re-read materials or re-watch videos

### Ask the best possible question

The good thing about “trying three before Slack” is that you’ll gain
clarity on the best possible question to ask.

Here is how I ask questions in StackOverflow and Slack to provide context and reduce back-and-forths:

“I am trying to accomplish **\_\_\_** (very specific step). I expected **\_\_\_** but got **\_\_\_** (error message
or issue). I tried to resolve the error by **\_\_\_**, **\_\_\_**, and **\_\_\_**
(write a quick list of the three steps you took to solve your problem). Can someone explain how to **\_\_\_** (if necessary, explain the
information you need to move forward)."

### Have a growth mindset

In my experience, schools and businesses incentivize getting things
“right” right away. People who take on challenging work or actually
take time to grapple with concepts are not rewarded. This actually goes
against all research we have about learning. In order to learn, you need
to engage in tasks that provide a bit of struggle. As you struggle, new
connections form in your brain - this is how we learn.

* * *

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

```dockerfile
FROM python:3.9

RUN pip install pandas

WORKDIR /app
COPY pipeline.py pipeline.py

ENTRYPOINT [ "python", "pipeline.py" ]
```

#### Understanding each instruction

| INSTRUCTION | argument                | What’s happening?                                                              |
| :---------- | :---------------------- | :----------------------------------------------------------------------------- |
| FROM        | python:3.9              | Establishes a base image                                                       |
| RUN         | pip install pandas      | Runs PIP to install Pandas                                                     |
| WORKDIR     | /app                    | Changes the current director to /app                                           |
| COPY        | pipeline.py pipeline.py | Copies pipeline.py from our local host to app/pipeline.py in the container     |
| ENTRYPOINT  | “python”, “pipeline.py” | Runs the `python` command to start the Python interpreter and runs pipeline.py |

#### Our simple Python pipeline (pipeline.py)

```python
import sys
import pandas as pandas

print(sys.argv)

day = sys.argv[1]

# Fancy stuff with pandas

print(f'Job finished successfully for day = {day}')
```

* * *

### Ingesting NY Taxi Data to Postgres

#### Key packages to install

```python
# You might need to use pip3 instead of pip (depends on your environment)
pip install pgcli
pip install notebook
pip install SQLAlchemy
pip install psycopg2-binary # You might not need this - I had to install it
```

#### Useful documentation

-   [Pandas
    Dataframes](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html)
-   [Jupyter Notebook](https://docs.jupyter.org/en/latest/)
-   [SQLAlchemy
    Engine](https://docs.sqlalchemy.org/en/20/core/engines.html)
-   [Iterators and Iterables - Corey
    Schafer](https://www.youtube.com/watch?v=jTYiNjvnHZY)

#### Docker commands to spin up database

```python
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v /home/candace/Desktop/Code/zoomcamp/docker_test/ny_taxi_postgres_data \
  -p 5432:5432 \
  postgres:13
```

#### Using pgcli to connect to postgres

```bash
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

#### CSV link

Link to CSV backup:


#### Jupyter Notebook (upload_date.ipynb)

```python
# Import Pandas with the alias pd
import pandas as pd

# Check Pandas version
pd.__version__
```

```python
# Create a Pandas dataframe with the first 100 rows of the csv
# See Pandas documentation or YouTube videos for info about dataframes
df = pd.read_csv('yellow_tripdata_2021-01.csv', nrows=100)
```

```python
# Show dataframe
df
```

```python
# Print schema
print(pd.io.sql.get_schema(df, name='yellow_taxi_data'))
```

```python
# Change schema so that the pickup and dropoff datetimes are modeled as DATETIME instead of TEXT in the database schema
df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
```

```python
# Import create_engine from SQLalchemy
# I had to import psycopg2 but you might not have to
from sqlalchemy import create_engine
import psycopg2
```

```python
# Create a SQLalchemy engine using the local Docker container server we have running at port 5432
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')
```

```python
# Connect to engine
engine.connect()
```

```python
# Create a dataframe iterator that will iterate over the csv in chunks of 100,000
# See Corey Schafer video above for info about iterators
df_iter = pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100_000)
```

```python
# Import time module so we can calculate how long each iteration takes
from time import time
```

```python
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

* * *

### Connecting pgAdmin and Postgres

Note: This section was particularly hard for me because I did not have the right argument behind the -v flag. I consulted the FAQ and used a volume name instead of a path:

```dockerfile
-v ny_taxi_postgres_data:/var/lib/postgresql/data
```

#### Docker Networks

We need to use a Docker network so that one container that is running pgAdmin can communicate with another container that has the database (with a mounted volume).

#### Create a network

```bash
docker network create pg-network
```

#### Run Postgres

```python
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

```python
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin2 \
  dpage/pgadmin4
```

* * *

### Putting the ingestion script into Docker

#### Converting the Jupyter notebook to a Python script

Jupyter notebooks are useful because they help us:

-   Create a [computational narrative](https://odsc.medium.com/why-you-should-be-using-jupyter-notebooks-ea2e568c59f2) that others can understand and modify independently
-   Document our actions and learning with both code snippets and text
-   Run pieces of a script or a program line by line (this helps with troubleshooting)

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

Since this script is a Python program, we had to use 'if **name** == "**main**'. Check out Corey Schafer's [YouTube video about 'if **name** == "**main**'](https://www.youtube.com/watch?v=sugvnHA7ElY) for more information about it.

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
