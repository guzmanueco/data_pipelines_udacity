# Project: Data Pipelines with Apache Airflow
Introduction
The goal is to create high grade data pipelines that are dynamic and built from reusable tasks, can be monitored, and allow easy backfills. Tests need to run against their datasets after the ETL steps have been executed to catch any discrepancies in the datasets.

The source data resides in S3 and needs to be processed in Sparkify's data warehouse in Amazon Redshift. The source datasets consist of JSON logs that tell about user activity in the application and JSON metadata about the songs the users listen to.

# Datasets
For this project, there are two datasets. Here are the s3 links for each:

s3://udacity-dend/song_data/
s3://udacity-dend/log_data/


# Configuring the DAG
In the DAG, add default parameters according to these guidelines

The DAG does not have dependencies on past runs
On failure, the task are retried 3 times
Retries happen every 5 minutes
Catchup is turned off
Do not email on retry
In addition, configure the task dependencies so that after the dependencies are set, the graph view follows the flow shown in the image below.


Configure the task dependencies

start_operator  \
    >> create_trips_table \
    >> [stage_events_to_redshift, stage_songs_to_redshift] \
    >> load_songplays_table \
    >> [ load_songs_table, load_artists_table, load_time_table, load_users_table] \
    >> run_quality_checks \
    >> end_operator

# Project Files
This project workspace includes 2 folders: dags

The spakify_dag.py includes all the imports, tasks and task dependencies
The operators folder includes 4 user defined operators that will stage the data, transform the data, fill the data warehouse, and run checks on data quality.
A helper class for the SQL transformations

# Building the operators
## Stage Operator
The stage operator is expected to be able to load any JSON formatted files from S3 to Amazon Redshift. The operator creates and runs a SQL COPY statement based on the parameters provided. The operator's parameters should specify where in S3 the file is loaded and what is the target table.

The parameters should be used to distinguish between JSON file. Another important requirement of the stage operator is containing a templated field that allows it to load timestamped files from S3 based on the execution time and run backfills.

## Fact and Dimension Operators
With dimension and fact operators, you can utilize the provided SQL helper class to run data transformations. Most of the logic is within the SQL transformations and the operator is expected to take as input a SQL statement and target database on which to run the query against. You can also define a target table that will contain the results of the transformation.

Dimension loads are often done with the truncate-insert pattern where the target table is emptied before the load. Thus, you could also have a parameter that allows switching between insert modes when loading dimensions. Fact tables are usually so massive that they should only allow append type functionality.

## Data Quality Operator
The final operator to create is the data quality operator, which is used to run checks on the data itself. The operator's main functionality is to receive one or more SQL based test cases along with the expected results and execute the tests. For each the test, the test result and expected result needs to be checked and if there is no match, the operator should raise an exception and the task should retry and fail eventually.

# Add Airflow Connections to AWS
Use Airflow's UI to configure your AWS credentials and connection to Redshift.

- Click on the Admin tab and select Connections.
- Under Connections, select Create
- On the create connection page, enter the following values:

    Conn Id: Enter aws_credentials.
    Conn Type: Enter Amazon Web Services.
    Login: Enter your Access key ID from the IAM User credentials you downloaded earlier.
    Password: Enter your Secret access key from the IAM User credentials you downloaded earlier. Once you've entered these values, select Save and Add Another. connection-aws-credentials!
    On the next create connection page, enter the following values:
    Conn Id: Enter redshift.
    Conn Type: Enter Postgres.
    Host: Enter the endpoint of your Redshift cluster, excluding the port at the end. You can find this by selecting your cluster in the Clusters page of the Amazon Redshift console. See where this is located in the screenshot below. IMPORTANT: Make sure to NOT include the port at the end of the Redshift endpoint string.
    Schema: Enter dev. This is the Redshift database you want to connect to.
    Login: Enter awsuser.
    Password: Enter the password you created when launching your Redshift cluster.
    Port: Enter 5439

Once you've entered these values, select Save.