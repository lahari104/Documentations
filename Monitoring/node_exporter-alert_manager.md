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

* The aboce command will download the node exporter tar file.


**Extracting the downloaded Node Exporter tar file:**
-----------------------------------------------------
* After Downloading the tar file we need to extract the tar file to get all the binaries required to install the Node Exporter.

`sudo tar xvfz node_exporter-1.3.1.linux-amd64.tar.gz`

* This command will extract the tar file in the home path, after the extraction is done we will get a folder named `node_exporter-1.3.1.linux-amd64` which containts all the binaries and dependencies required to install the Node Exporter.


**Move the Extracted Files:**
-----------------------------
* Move the extracted Grafana directory to `/usr/local/bin`

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
* Open Grafana in web browser (default port is 3000, e.g., http://localhost:3000).

* Go to Configuration > Data Sources > Add data source.

* Select Prometheus and enter the URL of your Prometheus server (e.g., http://localhost:9090). Then save and test the configuration, if it returns success then prometheus data source is added in grafana.

* Then we need to import the dashboard in grafana to visualize the metrics.


* Import a Node Exporter Dashboard:
-----------------------------------
* Go to Create > Import.

* You can find pre-built Node Exporter dashboards on Grafana's dashboard repository.

* Enter the dashboard ID and import it, then we can able to see the EC2 instance metrics visually from graphs in the imported grafana dashboard.



### Installing and Configuring Alert Manager:
---------------------------------------------

#### What is Alert Manager:
----------------------------
* Alertmanager is a component of the Prometheus ecosystem that handles alerts sent by Prometheus server. 

* It manages the alerts, including silencing, inhibition, aggregation, and sending notifications via methods like email, Slack, PagerDuty, etc.
