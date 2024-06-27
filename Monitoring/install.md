# Installing Prometheus and Grafana in EC2 Instance:
----------------------------------------------------
* Prometheus and Grafana are most popular open-source tools for monotoring especially for kubernetes. Prometheus is the logs and metrics collector and Grafana will give us dashboards to visualize the details collected by Prometheus in the form graphs, tables and charts etc.

### What is Prometheus:
-----------------------
* Prometheus is an open-source monitoring and alerting toolkit designed for reliability, scalability, and extensibility.
* It focuses on collecting time-series data metrics from various targets, storing them, and enabling querying and alerting based on these metrics.

[Know More About Prometheus](https://prometheus.io/)

* **Key Features:**
    * Multi-dimensional Data Model:
    ------------------------------- 
      * Metrics are stored with labels, allowing for flexible querying and aggregation.
 
    * PromQL:
    --------- 
      * Prometheus Query Language enables powerful querying of metrics.
    
    * Service Discovery:
    --------------------
      * Built-in support for discovering targets dynamically, including integration with Kubernetes and other service discovery mechanisms.
  
    * Alerting:
    ----------- 
      * Supports alerting based on predefined rules and conditions.
 
    * Scalability:
    -------------- 
      * Designed to be horizontally scalable to handle large-scale deployments.


### What is Grafana:
--------------------
* Grafana is an open-source platform for data visualization and monitoring.
* It allows users to create, explore, and share dashboards with data from various sources, including Prometheus.

[Know More About Grafana](https://grafana.com/)
[Know More About Grafana Dashboards](https://grafana.com/grafana/dashboards/)

* **Key Features:**
    * Visualization:
    ---------------- 
      * Rich visualizations including graphs, charts, and tables.
    
    * Dashboarding:
    --------------- 
      * Create and customize dashboards to display metrics and logs.
    
    * Alerting:
    -----------
      * Define alert rules and receive notifications based on data thresholds.
    
    * Data Source Integration:
    --------------------------
      * Grafana supports integration with numerous data sources, including Prometheus, to visualize and analyze metrics and logs.
    
    * Plugins and Extensions:
    ------------------------- 
      * Extensible with plugins and extensions for additional features and integrations.


**Integration and Use:**
------------------------
  * Prometheus and Grafana Integration:
    ------------------------------------- 
      * Prometheus metrics can be easily visualized and analyzed in Grafana dashboards. Grafana supports Prometheus as a data source, allowing users to query Prometheus metrics using PromQL and create rich visualizations.
  
  * Common Use Cases:
  -------------------- 
      * Together, Prometheus and Grafana are commonly used for monitoring cloud-native applications, microservices architectures, containerized environments (like Kubernetes), and traditional infrastructure. They provide deep insights into system performance, resource utilization, and application health.



#### Installing Prometheus:
---------------------------

##### Prerequisites:
--------------------

* EC2 Instance:
----------------
  * Ensure you have an EC2 instance running with appropriate permissions to install software.
  * I have created ubuntu-22.04 server

* SSH Access:
-------------- 
  * Connect to your EC2 instance via SSH.
  * Logged into ubuntu server via ssh using `ssh <server-username>@<publicIP-of-the-server>`


* Steps:
--------
* After Login to the server we need to update the packages using `sudo apt update` command.

* Then we need to download the stable or latest version of prometheus from the official page of promethues follow [Download Promethus](https://prometheus.io/download/) for downloading the Promethus tar file.

* Here, I have used `prometheus-2.43.0.linux-amd64.tar.gz` for the download.
 
**Download Prometheus:**
------------------------
`wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz`

* The above command will download the Prometheus tar file in the home path of the server.


**Extracting the downloaded Promethues tar file:**
---------------------------------------------------
* After Downloading the tar file we need to extract the tar file to get all the binaries required to install the Prometheus.

`sudo tar xvfz prometheus-2.43.0.linux-amd64.tar.gz`

* This command will extract the tar file in the home path, after the extraction is done we will get a folder named `prometheus-2.43.0.linux-amd64` which containts all the binaries and dependencies required to install the Prometheus.


**Installation:**
-----------------
* As part of Installation process we need to move the Prometheus related filed and binaried to the appropriate directories in-order run the Prometheus.
* Follow the below steps for the Installation

```bash
sudo mv prometheus-2.43.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.43.0.linux-amd64/promtool /usr/local/bin/
sudo mkdir /etc/prometheus
sudo mv prometheus-2.43.0.linux-amd64/consoles /etc/prometheus
sudo mv prometheus-2.43.0.linux-amd64/console_libraries /etc/prometheus
sudo mv prometheus-2.43.0.linux-amd64/prometheus.yml /etc/prometheus/
```

* The above commands will move the files to bin folder/PATH variable to let the PATH variable know the default location of the Prometheus.

* Because Executables are accessible system-wide by placing them in `/usr/local/bin/`.

* The consoles and `console_libraries` directories contain web console templates and libraries that Prometheus uses for its web UI.

* Configuration and related files are organized in `/etc/prometheus/` for easy management and adherence to standard Linux filesystem hierarchy conventions.

* Prometheus has access to necessary resources like console templates and its configuration file.


**Creating User for Prometheus:**
----------------------------------
* We need to create a dedicated user for Prometheus so that service follows best practices for Unix/Linux system administration.

* It helps in tracking and managing resources used by specific services and simplifies troubleshooting.

* So we need create a user and change the ownership and permissions for the user accordingly.

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

* The above steps will creates a user names `prometheus` and gives the `prometheus` user ownership for the directories which were required.


**Create Promethues Service:**
------------------------------
* Now we need to create the service for Prometheus.

* Creating a new systemd service file for Prometheus is essential for managing Prometheus as a background service on a Linux system.

`sudo nano /etc/systemd/system/prometheus.service` (or) `sudo vi /etc/systemd/system/prometheus.service`

* This above command will create a service file for prometheus in `/etc/systemd/system` path with the service name as `prometheus` (Generally any service file will present in this location in linux ).

* In the service we need to add the below content will is essential to run prometheus as a service, this file file containts the location of promethues and other details.

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

* After adding the above content in `prometheus.service` file we need to save the file.

* save the file using `:wq!` in `vi` editor and with `ctrl+X and Y` in `nano` editor.

* Then we need to run commands which will restart the daemon and start and enables the prometheus service.

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

* After starting the we need to verify for the prometheus service status.

* Use `sudo systemctl status prometheus` command for the status verification of prometheus service.

* If our service is up and `running` we can now able to access the prometheus UI in the browser using the publicIP of the server on port `9090`. `http://<public-ip>:9090`.


**Configuring Prometheus:**
---------------------------
* Prometheus configuration files will be present in `/etc/prometheus` location.

* To configure prometheus we need to edit the `prometheus.yml` file which we be present in `/etc/prometheus`.



#### Installing Grafana:
------------------------
* We can install Grafana in the same instance where we have installed Prometheus.

* Steps:
--------

**Download Grafana:**
---------------------
`wget https://dl.grafana.com/oss/release/grafana-9.3.6.linux-amd64.tar.gz`

* Download the Grafana tar file using the above command or use `wget` and URL of the choice from official documentation.

[Download Grafana](https://grafana.com/grafana/download)

* Here, I have used `grafana-9.3.6.linux-amd64.tar.gz` to download grafana.


**Extracting the downloaded Grafana tar file:**
-----------------------------------------------
* After Downloading the tar file we need to extract the tar file to get all the binaries required to install the Grafana.

`sudo tar xvfz tar -zxvf grafana-9.3.6.linux-amd64.tar.gz`

* This command will extract the tar file in the home path, after the extraction is done we will get a folder named `grafana-9.3.6.linux-amd64` which containts all the binaries and dependencies required to install the Grafana.


**Move the Extracted Files:**
-----------------------------
* Move the extracted Grafana directory to `/usr/local`

`sudo mv grafana-9.3.6 /usr/local/grafana`


**Create a Symbolic Link:**
---------------------------

* Create a symbolic link to make it easier to access Grafana.

```bash
sudo ln -s /usr/local/grafana/bin/grafana-server /usr/local/bin/grafana-server
sudo ln -s /usr/local/grafana/bin/grafana-cli /usr/local/bin/grafana-cli
```


**Create a Grafana User:**
--------------------------
* Create a new system user for running Grafana

* We need to create a dedicated user for grafana so that service follows best practices for Unix/Linux system administration.

* It helps in tracking and managing resources used by specific services and simplifies troubleshooting.

* So we need create a user and change the ownership and permissions for the user accordingly.

```bash
sudo useradd --no-create-home --shell /bin/false grafana
sudo chown -R grafana:grafana /usr/local/grafana
```


**Create Grafana Service:**
---------------------------
* Now we need to create the service for Grafana.

* Creating a new systemd service file for Grafana is essential for managing Grafana as a background service on a Linux system.

`sudo nano /etc/systemd/system/grafana.service` (or) `sudo nano /etc/systemd/system/grafana.service`

* This above command will create a service file for prometheus in `/etc/systemd/system` path with the service name as `grafana` (Generally any service file will present in this location in linux ).

* In the service we need to add the below content will is essential to run grafana as a service, this file file containts the location of promethues and other details.

```ini
[Unit]
Description=Grafana instance
After=network.target

[Service]
User=grafana
Group=grafana
Type=simple
ExecStart=/usr/local/grafana/bin/grafana-server --config=/usr/local/grafana/conf/defaults.ini --homepath=/usr/local/grafana

[Install]
WantedBy=multi-user.target
```

* After adding the above content in `grafana.service` file we need to save the file.

* Save the file using `:wq!` in `vi` editor and with `ctrl+X and Y` in `nano` editor.

* Then we need to run commands which will restart the daemon and start and enables the grafana service.

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana
sudo systemctl enable grafana
```

* After starting the we need to verify for the grafana service status.

* Use `sudo systemctl status grafana` command for the status verification of grafana service.

* If our service is up and `running` we can now able to access the grafana UI in the browser using the publicIP of the server on port `3000`. `http://<public-ip>:3000`.


**Access Grafana:**
-------------------
* Login to Grafana using the default credentials then we need to update the password.

```text
default username: admin
default password: admin
```