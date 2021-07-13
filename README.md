# Monitoring-ec2-using-prometheus-grafana

Monitoring using Prometheus and Grafana on AWS EC2
# Prerequisite:

    >>  Prometheus EC2 instance t2.micro 
    >>  Node EC2 instances to monitor
    >>  Security Groups Configured properly

# Security Groups Configured on EC2 Instances
    >>  Port 9090 — Prometheus Server

    >>  Port 9100 — Prometheus Node Exporter

    >>  Port 3000 — Grafana

# Prometheus EC2 Instance
  Configure the security group on EC2 Instance where Prometheus Server is installed as shown below :
  
 ![image](https://user-images.githubusercontent.com/84037413/125496703-c4810f49-9314-400e-be1c-ffe98a929927.png)



# Node EC2 Instances
  Configure the security group on EC2 Instance which you want to monitor and where you will install Node Exporter as shown below:
![image](https://user-images.githubusercontent.com/84037413/125496739-c257b4f5-a246-43df-9c32-043c1f4e8173.png)

  One entry is from your IP and one entry is the IP of the EC2 instance where Prometheus server is installed so that Prometheus server can read the metrics which is exposed on the Node server.


# Prometheus Architecture
  # Prometheus is an open source tool for monitoring and alerting applications
  # a multi-dimensional data model with time series data identified by metric name and key/value pairs
  # Uses PromQL ( Prometheus Query Language)
  # time series collection happens via a pull model over HTTP
  # Targets System which you want to monitor can be identified using Service Discovery or by static configuration in the yaml file
Below is the diagram of Prometheus architecture and its components

![image](https://user-images.githubusercontent.com/84037413/125496782-57484707-b211-4e1c-a2ec-36088aa364ce.png)


@ Prometheus Server: This component is the central component that collects the metrics from multiple nodes. Prometheus uses the concept of scraping, where target systems’ metric endpoints are contacted to fetch data at regular intervals.

@ Node Exporter: This is called a monitoring agent which we installed on all the target machines so that Prometheus can fetch the data from all the metrics endpoints
Push Gateway: Push Gateway is used for scraping metrics from applications and passing on the data to Prometheus. Push Gateway captures the data and then transforms it into the Prometheus data format before pushing.

@ Alert Manager: Alert Manager is used to send the various alerts based upon the metrics data collected in Prometheus.

@ Web UI: The web UI layer of Prometheus provides the end user with an interface to visualize data collected by Prometheus. In this, we will use Grafana to visualize the data.

Install Prometheus
Now we will install the Prometheus on one of the EC2 Instance.

You can download the latest version from this link https://prometheus.io/download/

  >>  Download the version
  >>  Run the install-prometheus.sh script
  >>  This script will install everything and configured it. You can change the version as per your project.

#This script will do the below steps:

  >>  Create a new user and add new directories
 ```
  sudo useradd --no-create-home prometheus
  sudo mkdir /etc/prometheus
  sudo mkdir /var/lib/prometheus
 ```
  >>  Download the Prometheus, extract it and put it in /usr/local/bin folder and finally delete the software
  
```
  wget  https://github.com/prometheus/prometheus/releases/download/v2.23.0/prometheus-2.23.0.linux-amd64.tar.gz
  tar -xvf prometheus-2.23.0.linux-amd64.tar.gz
  sudo cp prometheus-2.23.0.linux-amd64/prometheus /usr/local/bin
  sudo cp prometheus-2.23.0.linux-amd64/promtool /usr/local/bin
  sudo cp -r prometheus-2.23.0.linux-amd64/consoles /etc/prometheus/
  sudo cp -r prometheus-2.23.0.linux-amd64/console_libraries /etc/prometheus
  sudo cp prometheus-2.23.0.linux-amd64/promtool /usr/local/bin/
  rm -rf prometheus-2.23.0.linux-amd64.tar.gz prometheus-2.19.0.linux-amd64
```
  >>  Now we will configure Prometheus to monitor itself using yaml file. Create a prometheus.yml file at /etc/prometheus/prometheus.yml with the below content

```
  global:
    scrape_interval: 15s
    external_labels:
      monitor: 'prometheus'
  scrape_configs:
    - job_name: 'prometheus'
      static_configs:
        - targets: ['localhost:9090']
```
  >>  Now we want to run the Prometheus as a Service so that in case of server restart service will come automatically.
```
[Unit]
Description=Prometheus
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
  >>  Let’s create a file /etc/systemd/system/prometheus.service with the below content:
  >>  Change the ownership of all folders and files which we have created to the user which we have created in the first step
```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
  >>  Now we will configure the service and start it
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

# Now open it on the browser using below url:

![image](https://user-images.githubusercontent.com/84037413/125498275-25e6f46d-3335-4f35-95a5-3f90b5d0b1af.png)

# If you are not able to access it then make sure your security group is configured for port 9090 and its open from your IP.
![image](https://user-images.githubusercontent.com/84037413/125498357-0dde9a2d-68b4-438c-bb0b-a5889250a931.png)





