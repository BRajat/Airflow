## Installation of airflow


1. First install virtualbox latest version, .deb file can be directly installed on linux. 

2. Download the Airflow.ovf file, which has the vm configuration shared in the course.

3. Install Remote-host extension on VS code, and with ssh service we will start the vs code on the remote host (vm)

	-unix command - **CTRL + P**
	-enter - `>remote-host`
	-select `Add New SSH Host`
	-enter - `ssh -p 2222 airflow@localhost`
	-ssh-config file will open, check details and close
	-again use **CTRL + P**
	-enter - `>remote-host`
	-select `Connect to host`
	-select `localhost`
	
	This will start the vs code on the remote vm. after entering password as `airflow` wait for some time
	
	-start the vs code terminal of the remote:localhost
	-create a python virtualenv with command `python3 -m venv sandbox`
	-enter command - `source sandbox/bin/activate`
	Now the python virtual env, is started with name `sandbox`
	-enter command - `pip install wheel`
	-enter command - `pip3 install apache-airflow=2.0.0b3 --constraint [RAW_CONSTRAINT_FILE_SELECT_ALL_CONTENTS] `, version given in github gist.
	-enter command - `airflow db init` to initialize airflow metastore.
	-enter command - `ls` - list file contents
	-enter command - `cd airflow/` - change prompt to airflow folder, to check different airflow installation generated files.
	-enter command - `airflow webserver` to start webserver
	-run airflow webapp on `localhost:8080`
	
	

