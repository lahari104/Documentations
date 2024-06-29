### Install Node_Exporter and Alert_Manager in Prometheus and Grafana server:
-----------------------------------------------------------------------------

#### What is Node Exporter:
---------------------------
* Node Exporter is a monitoring tool used to collect system metrics from Linux and Unix-based systems.

* Node Exporter is a powerful tool for collecting system metrics, which can be scraped by Prometheus and visualized in Grafana

* It acts as an exporter that exposes these metrics in a format that Prometheus can scrape and store.

* Grafana is often used in conjunction with Prometheus to visualize the metrics collected by Node Exporter.

* Here, I wanted to monitor an EC2 instance so for that we need to install node exporter to expose the metrics of the EC2 instance to prometheus and grafana.

* Node Exporter collects a variety of system metrics, such as:
    * CPU usage
    * Memory usage
    * Disk I/O
    * Network statistics
    * Filesystem statistics
    * System load

* These metrics are made available via an HTTP endpoint which Prometheus scrapes.


#### Installation and Configuration:**
--------------------------------------

**Download Node Exporter:**
---------------------------
* Download the tar file of node exporter from the official docs.

[Download Node Exporter](https://github.com/prometheus/node_exporter/releases)

`wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz`

* The above command will download the node exporter tar file.


**Extracting the downloaded Node Exporter tar file:**
-----------------------------------------------------
* After Downloading the tar file we need to extract the tar file to get all the binaries required to install the Node Exporter.

`sudo tar xvfz node_exporter-1.3.1.linux-amd64.tar.gz`

* This command will extract the tar file in the home path, after the extraction is done we will get a folder named `node_exporter-1.3.1.linux-amd64` which containts all the binaries and dependencies required to install the Node Exporter.


**Move the Extracted Files:**
-----------------------------
* Move the extracted Node Exporter directory to `/usr/local/bin`

`sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/`

* The above commands will move the files to bin folder/PATH variable to let the PATH variable know the default location of the Node Exporter.

* Because Executables are accessible system-wide by placing them in `/usr/local/bin/`.


**Create a User for Node Exporter:**
------------------------------------
* Create a new system user for running Node Exporter

* We need to create a dedicated user for Node Exporter so that service follows best practices for Unix/Linux system administration.

* It helps in tracking and managing resources used by specific services and simplifies troubleshooting.

* So we need create a user and change the ownership and permissions for the user accordingly.

`sudo useradd -rs /bin/false node_exporter`


**Create Service for Node Exporter:**
-------------------------------------
* Now we need to create the service for Node Exporter.

* Creating a new systemd service file for Node Exporter is essential for managing Node Exporter as a background service on a Linux system.

`sudo nano /etc/systemd/system/node_exporter.service` (or) `sudo vi /etc/systemd/system/node_exporter.service`

* This above command will create a service file for Node Exporter in `/etc/systemd/system` path with the service name as `node_exporter` (Generally any service file will present in this location in linux ).

* In the service we need to add the below content will is essential to run Node Exporter as a service, this file file containts the location of Node Exporter and other details.

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```

* After adding the above content in `node_exporter.service` file we need to save the file.

* Save the file using `:wq!` in `vi` editor and with `ctrl+X and Y` in `nano` editor.

* Then we need to run commands which will restart the daemon and start and enables the node_exporter service.

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

* After starting the we need to verify for the node_exporter service status.

* Use `sudo systemctl status node_exporter` command for the status verification of node_exporter service.

* If our service is up and `running` we can now able to access the node_exporter UI in the browser using the publicIP of the server on port `9100`. `http://<public-ip>:9100`.


**Configure Prometheus to Scrape Node Exporter:**
-------------------------------------------------

* After the successful installation we need to configure Node Exporter with prometheus to scrape metrics to prometheus by editing the `prometheus.yml` file which will be present in `/etc/prometheus` path.

* Edit the Prometheus Configuration File (prometheus.yml) as below and add the content to scrape metrics to prometheus.

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

Note:
----
* Replace 'localhost:9100' with the appropriate address if Node Exporter is running on a different machine.

* As we have changed prometheus configuration we need to restart the prometheus service using `sudo systemctl restart prometheus`

* Then we can able to Visualize Metrics in Grafana, for this we need to add `prometheus data source` in grafana, follow the below steps for this.


* Add Prometheus as a Data Source in Grafana:
---------------------------------------------
* Open Grafana in web browser `http://grafana-server-ip:3000`

* Go to Configuration > Data Sources > Add data source.

* Select Prometheus and enter the URL of the Prometheus server `http://prometheus-server-ip:9090`. Then save and test the configuration, if it returns success then prometheus data source is added in grafana.

* Then we need to import the dashboard in grafana to visualize the metrics.


* Import a Node Exporter Dashboard:
-----------------------------------
* Go to Create > Import.

* You can find pre-built Node Exporter dashboards on Grafana's dashboard repository.

* Enter the dashboard ID and import it, then we can able to see the EC2 instance metrics visually from graphs in the imported grafana dashboard.



### Installing and Configuring AlertManager:
---------------------------------------------

#### What is AlertManager:
----------------------------
* Alertmanager is a component of the Prometheus ecosystem that handles alerts sent by Prometheus server. 

* It manages the alerts, including silencing, inhibition, aggregation, and sending notifications via methods like email, Slack, PagerDuty, etc.


**Download Alertmanager:**
--------------------------
* Download the tar file of Alertmanager from the official docs.

[Download Alertmanager](https://github.com/prometheus/alertmanager/releases)

`wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz`

* The above command will download the Alertmanager tar file.


**Extracting the downloaded Alertmanager tar file:**
-----------------------------------------------------
* After Downloading the tar file we need to extract the tar file to get all the binaries required to install the Alertmanager.

`sudo tar xvfz alertmanager-0.25.0.linux-amd64.tar.gz`

* This command will extract the tar file in the home path, after the extraction is done we will get a folder named `alertmanager-0.25.0.linux-amd64` which containts all the binaries and dependencies required to install the Alertmanager.


**Move the Extracted Files:**
-----------------------------
* Move the extracted AlertManager directory to `/usr/local/bin`

```bash
sudo mv alertmanager-0.25.0.linux-amd64/alertmanager /usr/local/bin/
sudo mv alertmanager-0.25.0.linux-amd64/amtool /usr/local/bin/
```

* The above commands will move the files to bin folder/PATH variable to let the PATH variable know the default location of the AlertManager.

* Because Executables are accessible system-wide by placing them in `/usr/local/bin/`.


**Create Directories for Configuration and Data:**
--------------------------------------------------

```bash
sudo mkdir /etc/alertmanager
sudo mkdir /var/lib/alertmanager
```

`/etc/alertmanager` This directory will store the configuration file (alertmanager.yml).

`/var/lib/alertmanager` This directory will store persistent data for Alertmanager.


Note:
-----
Creating dedicated directories for configuration and data is a best practice for organizing, securing, and managing the Alertmanager installation effectively. It separates concerns, enhances security, and ensures persistent storage for critical application data.


#### Create Alertmanager Configuration and Notifications:**
-----------------------------------------------------------
* After the successful Installation we need to configure the AlertManager to send the alerts for the events.


**Set Up Alerting Rules in Prometheus:**
----------------------------------------
* Create an alerting rules file (e.g., alert_rules.yml) in `/etc/prometheus` that defines the conditions under which alerts are triggered.

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

rule_files:
  - 'alert_rules.yml'
```

* Ensure localhost:9093 is replaced with the correct address if Alertmanager is running on a different machine or port.


* Create the `alert_rules.yml` file and add the below context to get the alerts for instance down event.

```yaml
groups:
  - name: example_alerts
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
```

Note:
-----
* Ensure the rule_files section in `prometheus.yml` includes the path to your alerting rules file as shown above.


**Visualize Alerts in Grafana:**
--------------------------------

* Add Prometheus as a Data Source in Grafana:
---------------------------------------------
* Open Grafana in web browser `http://grafana-server-ip:3000`

* Go to Configuration > Data Sources > Add data source.

* Select Prometheus and enter the URL of the Prometheus server `http://prometheus-server-ip:9090`. Then save and test the configuration, if it returns success then prometheus data source is added in grafana.


* Import a AlertManager Dashboard:
-----------------------------------
* Go to Create > Import.

* You can find pre-built AlertManager dashboards on Grafana's dashboard repository.

* Enter the dashboard ID and import it, then we can able to see the alerts visually from graphs in the imported grafana dashboard.


**Create a Configuration File to send email notifications for alerts:**
-----------------------------------------------------------------------
* Create the `alertmanager.yml` configuration file in `/etc/alertmanager`

`sudo nano /etc/alertmanager/alertmanager.yml` (or) `sudo vi /etc/alertmanager/alertmanager.yml`

* After create the above file add the below context to the file to send the notifications to the mail for alerts based on events.

```yaml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alertmanager@yourdomain.com'
  smtp_auth_username: 'your-email@gmail.com'
  smtp_auth_password: 'your-app-password'
  smtp_require_tls: true

route:
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'destination-email@example.com'
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

* Adjust the email configuration settings as necessary for SMTP server and notification preferences.


**Create a User for AlertManager:**
-----------------------------------
* Create a new system user for running AlertManager

* We need to create a dedicated user for AlertManager so that service follows best practices for Unix/Linux system administration.

* It helps in tracking and managing resources used by specific services and simplifies troubleshooting.

* So we need create a user and change the ownership and permissions for the user accordingly.

`sudo useradd -rs /bin/false alertmanager`


**Create Service for AlertManager:**
-------------------------------------
* Now we need to create the service for AlertManager.

* Creating a new systemd service file for AlertManager is essential for managing AlertManager as a background service on a Linux system.

`sudo nano /etc/systemd/system/alertmanager.service` (or) `sudo vi /etc/systemd/system/alertmanager.service`

* This above command will create a service file for AlertManager in `/etc/systemd/system` path with the service name as `alertmanager` (Generally any service file will present in this location in linux ).

* In the service we need to add the below content will is essential to run alertmanager as a service, this file file containts the location of alertmanager and other details.

```ini
[Unit]
Description=Alertmanager
Documentation=https://prometheus.io/docs/alerting/latest/alertmanager/
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --storage.path=/var/lib/alertmanager
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

* After adding the above content in `alertmanager.service` file we need to save the file.

* Save the file using `:wq!` in `vi` editor and with `ctrl+X and Y` in `nano` editor.

* Then we need to run commands which will restart the daemon and start and enables the alertmanager service.

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

* After starting the we need to verify for the alertmanager service status.

* Use `sudo systemctl status alertmanager` command for the status verification of alertmanager service.

* If our service is up and `running` we can now able to access the alertmanager UI in the browser using the publicIP of the server on port `9093`. `http://<public-ip>:9093`.


**Configure Prometheus to Use Alertmanager:**
---------------------------------------------
* Now we need to configure Alertmanager with prometheus to send the alerts.

* Edit the Prometheus Configuration File `prometheus.yml`

* Add an alerting section to specify the Alertmanager instances as below.

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 'localhost:9093'
```

Note:
-----
Replace `localhost:9093` with the appropriate address if Alertmanager is running on a different machine.


* As we have changed the prometheus configuration we need to restart the prometheus using `sudo systemctl restart prometheus` command.


* Once all the required tools installation and configuration is done check the status of each of the tool.


```bash
sudo systemctl status prometheus
sudo systemctl status grafana
sudo systemctl status node_exporter
sudo systemctl status alertmanager
```

* If every tool is up and running then we are all set to monitor the resources from prometheus and grafana with alerts and notifications.


Note:
----
* For trouble shooting use the below command `sudo journalctl -u service-name`
