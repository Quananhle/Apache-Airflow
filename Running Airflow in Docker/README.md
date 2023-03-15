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

![image](https://user-images.githubusercontent.com/35042430/225348812-2091e3b9-d502-400d-a0a9-8f0f538aa5df.png)

![image](https://user-images.githubusercontent.com/35042430/225348970-736b3efc-31d2-4030-82f3-338ae5436422.png)

10. Create an ```Airflow``` user

```Shell
airflow users create --username jmaciel --firstname Jorge --lastname Maciel --role Admin --email jorge.maciel@fii-na.com --password fiiadmin
```

![image](https://user-images.githubusercontent.com/35042430/225349394-514474c0-8b9b-497b-9a39-6f86732861a3.png)

11. Setup ```Nginx```

```Bash
sudo systemctl enable nginx
```

![image](https://user-images.githubusercontent.com/35042430/225349461-d4a09e90-a26b-45e4-a967-6d326175d811.png)

```Bash
# Create available site
sudo nano /etc/nginx/sites-available/airflow 
```

```Vim
server {
    listen 81;
    server_name localhost;
location / {
    proxy_pass [http://localhost:8081](http://localhost:8081);
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    }
}
```

```Bash
# Link available and enable nginx sites
sudo ln -s /etc/nginx/sites-available/airflow /etc/nginx/sites-enabled/airflow
```

```Bash
sudo nano /etc/nginx/nginx.conf
```

```Bash
##
# Basic Settings
##

sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 65;
types_hash_max_size 2048;
```
![image](https://user-images.githubusercontent.com/35042430/225350487-82550f08-882e-42bd-b1e8-cd760abf53a9.png)

```
# Run to check that nginx configs are correct
sudo nginx -t
```

![image](https://user-images.githubusercontent.com/35042430/225350654-3c517548-7ba6-4559-a55f-98a1464360f7.png)

12. Create Webserver Service with ```Systemd```

```Bash
# Webserver Service
sudo curl -o /etc/systemd/system/airflow-webserver.service
"https://raw.githubusercontent.com/apache/airflow/master/scripts/systemd/airflow-webserver.service"
"https://raw.githubusercontent.com/apache/airflow/master/scripts/systemd/airflow-webserver.service"
```

![image](https://user-images.githubusercontent.com/35042430/225351838-ed3ac702-7ac5-4b9d-bf67-95d1959336aa.png)

```Bash
# Modifying Parameters
sudo nano /etc/systemd/system/airflow-webserver.service
# EnvironmentFile=/etc/sysconfig/airflow <--[Comment out this line]
Environment="PATH=/home/airflow/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" <--[Add this line]
ExecStart=/home/airflow/.local/bin/airflow webserver -p 8081 -w 2 --pid /home/airflow/airflow-webserver.pid
```

![image](https://user-images.githubusercontent.com/35042430/225352737-f1916ff2-4e0a-4e2e-b971-6ab684cd2374.png)

```Bash
# Scheduler Service
sudo curl -o /etc/systemd/system/airflow-scheduler.service "https://raw.githubusercontent.com/apache/airflow/master/scripts/systemd/airflow-scheduler.service"
```

![image](https://user-images.githubusercontent.com/35042430/225352947-ced19a2f-0215-4e1c-8064-bf1c393a9bae.png)

```Bash
# Modifying Parameters
sudo nano /etc/systemd/system/airflow-scheduler.service
# EnvironmentFile=/etc/sysconfig/airflow <--[Comment out this line]
Environment="PATH=/home/airflow/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
ExecStart=/home/airflow/.local/bin/airflow scheduler
```

![image](https://user-images.githubusercontent.com/35042430/225352967-0f96f397-8f24-4181-a95a-7f1f36311d47.png)


