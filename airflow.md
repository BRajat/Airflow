# AirFlow - 


## Why AirFlow Needed?

Data Pipeline to trigger at scheduled time.

```
Downloading Data > Processing Data > Storing Data
```
----

## AirFlow:

Programmitically orchestrate data pipline. Orchestration - automating data pipelines, adding schedule and triggers tasks, alert users when certain events happen, to handling scaling of application.

**AirFlow Components**

1. Web Server - Flask Server with Gunicorn serving the UI
2. Scheduler - to Schedule workflows.
3. Metastore - Database where task related metadata is stored.
4. Executor - Class defining how tasks should be executed. (sequential or parallel)
5. Worker - process/sub process executing your task.

----
**AirFlow DAG**

**DAG** - Directed Acyclic Graph

Sequential dependency of tasks, no circular dependency

DAG is graph with nodes as tasks/operators and task dependency defined between them as edges

So, Data Pipeline Workflow is DAG in Apache Airflow.

----
**How it works**

**Single Node Architecture**

- In single node, queue is part of executor. 
- The data base components send and recieves data from other airflow components and also store data. 
- Webserver serves the client using AirFlow. 
- The scheduler triggers the executor which add the tasks in the queue for worker to work upon.
- The queue stores the task details in FIFO form.
- The executor and worker together completes the task and task history is stored in metastore.

Running - 

1. We will have a DAG Folder which has Python AirFlow DAG files in it - what is in DAG file(operators and task dependency).
2. First, the webserver and scheduler parses the DAG file from the folder.
3. The Scheduler creates a DAG run object that is stored in Metastore.
4. The first task to execute is scheduled. At right time it is triggered by scheduler, by creating a task instance object that it shares with executor.
5. Executor runs the task instance and updates its status, which will reflect in metastore.
6. If task is run successfully, the final status of task is updated in metastore.
7. The webserver updates the user.


**Multi Node Architectuer**

- Queue is external to executor.
- Worker fetch the task from queue to work on them.

