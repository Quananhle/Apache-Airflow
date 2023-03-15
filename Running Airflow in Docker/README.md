## Airflow Installation

__OS Required__:
  - Linux/Ubuntu 20.04 LTS
  - Airflow Version 2.5.1
  - Python 3.10

__OS Setup__

1. Update Packages and disable auto-update

```Bash
sudo apt-get update 
```

![image](https://user-images.githubusercontent.com/35042430/225344067-d1fd50a2-b940-4d2c-a96a-f0aa76036c46.png)

```Bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

```Bash
APT::Periodic::Update-Package-Lists "0";
```

__Airflow Installation__

1. Install ```Airflow``` Dependencies

```Shell
sudo apt-get install -y python3 python3-pip python3-testresources postgresql postgresql-contrib redis nginx
```

![image](https://user-images.githubusercontent.com/35042430/225344205-2044f11b-871f-4151-94cb-85663da156f7.png)

2. Create a non-root user for ```Airflow```

```Bash
sudo adduser airflow sudo --> Password: fiiadmin
su airflow
cd ..
```

3. Setting Variables

```Bash
export AIRFLOW_HOME=~/airflow 
AIRFLOW_VERSION=2.5.1 
PYTHON_VERSION="$(python3 --version | cut -d " " -f 2 | cut -d "." -f 1-2)" 
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt" 
```

4. Install ```Airflow``` with ```pip```

```Python
python3 -m pip install "apache-airflow[postgres,celery,redis]==2.3.4" --constraint "${CONSTRAINT_URL}"
```

![image](https://user-images.githubusercontent.com/35042430/225346218-859f725b-db0a-46fc-99a7-37e4f8e17383.png)

6. Configure ```Postgres```

```Bash
sudo -u postgres psql -c "create database airflow" 
sudo -u postgres psql -c "create user airflow with encrypted password 'airflow'"; 
sudo -u postgres psql -c "grant all privileges on database airflow to airflow";
sudo -u postgres psql -c "alter user airflow createdb;";
```

7. Setup ```Airflow``` executable default directory

```Bash
echo "export PATH=$PATH:$HOME/.local/bin" >> ~/.bashrc
source ~/.bashrc
airflow version
```

8. Configure ```Airflow```

```Bash
sudo usermod -a -G sudo airflow
sudo nano $HOME/airflow/airflow.cfg
```

```Python
[core]
# Class name of the executor
executor = CeleryExecutor
# Connection string to the local Postgres database
sql_alchemy_conn = postgresql://airflow:airflow@localhost:5432/airflow
# Whether to load the DAG examples that ship with Airflow
load_examples = False

[webserver]
# airflow sends to point links to the right web server
# base_url = [http://localhost:8081](http://localhost:8081)
base_url = http://localhost:8081
# The port on which to run the web server
web_server_port = 8081
# Run a single gunicorn process for handling requests.
workers = 1
# Require password authentication to the webserver
authenticate = True
# Use FAB-based webserver with RBAC feature
rbac = True
# Enable werkzeug ``ProxyFix`` middleware for reverse proxy
enable_proxy_fix = True

[celery]
broker_url = redis://localhost:6379/0
result_backend = db+postgresql://airflow:password@localhost:5432/airflow

[smtp] --> Alternatively setup stmp for email
smtp_host = smtp.office365.com
smtp_starttls = False
smtp_user = maciel_mj@hotmail.com
smtp_password = EMAIL_PASSWORD
smtp_port = 587
smtp_mail_from = maciel_mj@hotmail.com
```

9. Initialize the ```Airflow``` metadata database

```Bash
airflow db init
#Display the list of tables that Airflow created
psql -c '\dt'
```

![image](https://user-images.githubusercontent.com/35042430/225348309-b31422b4-a099-4fc0-972a-8b53eb5f288a.png)

