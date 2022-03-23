## Installation of airflow


1. First install virtualbox latest version, .deb file can be directly installed on linux. 

2. Download the Airflow.ovf file, which has the vm configuration shared in the course.

3. Install Remote-host extension on VS code, and with ssh service we will start the vs code on the remote host (vm)

- unix command - **CTRL + P**
	
- enter - `>remote-host`
	
- select `Add New SSH Host`
	
- enter - `ssh -p 2222 airflow@localhost`
	
- ssh-config file will open, check details and close
	
- again use **CTRL + P**
	
- enter - `>remote-host`
	
- select `Connect to host`
	
- select `localhost`
	
	This will start the vs code on the remote vm. after entering password as `airflow` wait for some time
	
- start the vs code terminal of the remote:localhost
	
- create a python virtualenv with command `python3 -m venv sandbox`
	
- enter command - `source sandbox/bin/activate`
	
	Now the python virtual env, is started with name `sandbox`
	
- enter command - `pip install wheel`
	
- enter command - `pip3 install apache-airflow=2.1.0 --constraint [RAW_CONSTRAINT_FILE_HTTP_LINK]`, check github gist for latest version.
	
- enter command - `airflow db init` to initialize airflow metastore.
	
- enter command - `ls` - list file contents
	
- enter command - `cd airflow/` - change prompt to airflow folder, to check different airflow installation generated files.
	
- enter command - `airflow webserver` to start webserver
	
- run airflow webapp on `localhost:8080`

4. Start the web app - now to login to airflow, we can't since we don't have username and password.

- create username and password using airflow cli
	
- enter command - `airflow -h`

- enter command - `airflow db -h`, to explore commands

- To create user

- enter command - `airflow users create -u admin -p admin -f Rajat -l B -r Admin -e rajat.bombale@konverge.ai`, this creates a admin user.

- Now login from web app to access user interface.

**OTHER AIRFLOW COMMANDS**

1. `airflow db init` - only once execute

2. `airflow db upgrade` - to upgrade airflow version

3. `airflow db reset` 

4. `airflow webserver`

5. `airflow scheduler`

6. `airflow list dags`

7. `airflow tasks list DAG_NAME`

8. `airflow tasks trigger -e 2022-03-23 DAG_NAME`

------

**AIRFLOW UI**

------
**CREATING FIRST DATA PIPELINE IN AIRFLOW**

1. Make sure we are connected to VM and connect the localhost on vs code with the `>remote-host` .

2. Enter into sandox environment with `source sandbox/bin/activate`

3. Open the airflow folder in vs code, enter password when prompted.

4. Now when the airflow folder is open check if **dags** folder is present, if not create a empty **dags** folder. The **dags** folder will store all the python dags code that we write.


**OUR DATA PIPE LINE**

Create sql table >> is_api_available >> extracting_users >> processing_users >> storing_users



**Best Practice to test single tasks individually before creating more tasks**

command : `airflow tasks test user_processing creating_table 2022-03-22`

---------
There are around 700 airflow operators for running different tasks.

`airflow providers list` - to list installed operators.

`pip install apache-airflow-providers-http==2.0.0` - to install specific operator listed in airflow documentation.
-----

1. FIRST TASK: Creating sqlite users table with SqliteOperator

2. SECOND TASK: Checking if API is available (sensor to check if endpoint is available) with HttpSensor

3. THIRD TASK : Getting the data from http request to the API (Action operator) with SimpleHttpOperator

4. FOURTH TASK : Extracting the data from http response to csv file with PythonOperator

5. FIFTH TASK : Loading the csv data into sqlite users table with bash operator

Finally, define task dependencies with bitwise notation.

Schedule the DAG for future runs.

**Scheduling the DAG**

Two important parameters, when it comes to scheduling -

1. Start_date : Date when the DAG should execute

2. Schedule_Interval : Increments on the execution date to know next DAG Run.


**Catchup and Backfilling**

1. The Airflow DAG's start on start date and the execution date is updated with each successful DAG Run.

2. When catchup is True, Airflow will trigger all tasks from last execution run which were paused. In case the Last execution was not there, all tasks upto next execution time will trigger.

3. The task is triggered when execution time is elapsed.
	
	@START: EXECUTION_TIME = START_TIME
	NEW_EXECUTION_TIME = PREVIOUS_EXECUTION_TIME + SCHEDULE_INTERVAL

4. To resolve task fails, we pause the dag and check the logs for the failed task. Airflow helps to resolve to us as to what stage the task failed.


-----

XCOM -> to push data between tasks

XCOM - stored in metastore as key-value pair.

To get value of XCOM we use `ti.xcom_pull(task_ids = ['extracting_user'])`, where `ti` is the task_instance objects

-------------
## SECTION-4

**Parallel execution of tasks ** - Task on same level can be executed in parallel, they are independent.
However, the default executor is sequential, so even the independent tasks are sequentially executed.

**The Default configuration**


1. `airflow config get-value core executor` 

2. `airflow config get-value core sql_alchemy_conn`

This are commands to check some default config

`task_1 >> [task_2, task3] >> task_4`

In default task-execution is sequential - task 2 and task 3 do not execute in parallel

-----------
**Configuring tasks in parallel with Local Executor**

1. We need to change database - since sqlite do not allow multiple writes, so we use postgres.

2. Each time a task is run as a sub-process.

**Install postgres and change airflow.cfg file**

`sudo apt update`

`sudo apt install postgresql`

Once postgres is install use commands to change postgres user password

`sudo -u postgres psql` this will start psql

`alter user postgres password 'postgres';`
This will alter the set the password for postgres user

Now modify the airflow.cfg file - 2 fields.

	`sql_alchemy_conn = postgresql+psycopg2://postgres:postgres@localhost/postgres`
	
	`executor = LocalExecutor`
	
Save the configuration file and check if metastore database is working on postgres
`airflow db check` should get connection successful.

Stop the web server and web scheduler

`airflow db init` to initialize the airflow database.

Create new user - `airflow users create -u admin -p admin -f Rajat -l B -r Admin -e rajat.bombale@konverge.ai`, this creates a admin user.

Again start the webserver and webscheduler with commands on separate terminals
`airflow webserver` and `airflow scheduler`

Now you should be able to verify in gantt chart that task 2 and task 3 execute in parallel

`task_1 >> [task_2, task_3] >> task_4` 

-----------
**Scale to infinity with Celery Executor**[Practise]

1. Stop the webserver and scheduler

2. `pip install 'apache-airflow[celery]'` - to install celery executor

3. We also need Redis to be installed, follow commands-
`sudo apt update`

`sudo apt install redis-server`

Now redis is installed, in order to use it as service we need to modify its configuration.

`sudo nano /etc/redis/redis.conf`

change `supervised` value in config file from `no` to `systemd`, save the config file.

`sudo systemctl restart redis.service`

`sudo systemctl status redis.service`

In airflow.cfg file 

1. change the executor to `CeleryExecutor` 

2. modify the `broker_url` change it to `redis://localhost:6379/0`

3. modify `result_backend` and copy the same sqlalchemy_database_value and modify it as 
`db+postgresql://postgres:postgres@localhost/postgres`

`pip install 'apache-airflow[redis]'` - to install redis packages

`airflow celery flower` - and go to localhost:5555 this will help to monitor workers.

To start a new worker - `airflow celery worker`, in our case we are running worker on single machine, but in case of multiple worker, this commands need to be executed at every machine that participates in mutli-node architecture to configure it as celery worker.

We will see a new worker on flower UI, after executing above command

Start webserver and scheduler. Refresh the UI of airflow at localhost:8080
Now, we can scale with as many tasks as we want by adding more machines.

To limit the number of DAG Runs to execute at same time and also the limit the number of tasks on each worker we configure more parameters, which are given in next session.

-------

**Concurrency parameters**

1. parallelism - 32 by default.
change to 1 and restart airflow, you will find the behaviour same as sequential executor.

2. dag_concurrency - 16 by default.
how many dag_runs can run concurrently, a single dag run is one scheduled dag run. Limits number of tasks running accross multiple dag runs, to a set maximum value.

To set this parameter for 1 specific dag - pass is as a DAG object parameter.

3. max_active_runs_per_dag - 16 by default.
set to 1 and observe changes. Only 1 active running dag will be visible. this is useful when the next dag run is dependent on previous dag run. This can be dag specific, so pass in dag object parameter.

--------










	

