# 📊 Centralized Log Aggregation and Monitoring using ELK Stack

## 🚀 Project Overview

In modern distributed systems, logs are often scattered across multiple servers, making troubleshooting slow and inefficient. This project implements a centralized logging system using the ELK Stack (Elasticsearch, Logstash, Kibana) to collect, process, visualize, and monitor logs from multiple EC2 instances.

The system enables real-time log analysis, error detection, and alerting to improve system reliability and reduce debugging time.

---

## 🎯 Objective
* Collect logs from multiple EC2 instances
* Parse and structure logs using Logstash
* Store logs in Elasticsearch
* Visualize logs using Kibana dashboards
* Trigger alerts on abnormal error spikes

---

### 🏗️ Architecture

![](./img/architecture%20(2).png)

---

### ⚙️ Tech Stack

*  AWS EC2
* ELK Stack
     * Elasticsearch
     * Logstash
     * Kibana
* Filebeat
* Nginx

---

### 🖥️ Setup Instructions
**1️⃣ Launch ELK Server**
* Create an EC2 instance (Ubuntu recommended)

![](./img/Screenshot%20(408).png)


* Install ELK Stack:

```
# Update system
sudo apt update

# Install Java
sudo apt install openjdk-11-jdk -y

# Install Elasticsearch
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.x.deb
sudo dpkg -i elasticsearch-7.x.deb

# Install Logstash
sudo apt install logstash -y

# Install Kibana
sudo apt install kibana -y
```

![](./img/Screenshot%20(406).png)

* Start services:

```
sudo systemctl start elasticsearch
sudo systemctl start logstash
sudo systemctl start kibana
```
---

### 2️⃣ Setup Application Servers
* Launch 2 EC2 instances
* Install Nginx:

```
sudo apt update
sudo apt install nginx -y
```

![](./img/Screenshot%20update.png)

* Generate logs:

```
curl localhost
```
---

### 3️⃣ Install and Configure Filebeat

```
sudo apt install filebeat -y
```
Edit configuration:

```
sudo nano /etc/filebeat/filebeat.yml
```
Update:

```
filebeat.inputs:
  - type: log
    paths:
      - /var/log/nginx/access.log
      - /var/log/nginx/error.log

output.logstash:
  hosts: ["<ELK-SERVER-IP>:5044"]
```

![](./img/Screenshot%20(407).png)

Start Filebeat:

```
sudo systemctl start filebeat
```

![](./img/Screenshot%20(409).png)

---

### 4️⃣ Configure Logstash

Create config file:

```
sudo nano /etc/logstash/conf.d/nginx.conf
```
```
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }

  if [response] >= 400 and [response] < 500 {
    mutate { add_tag => ["4xx_error"] }
  }

  if [response] >= 500 {
    mutate { add_tag => ["5xx_error"] }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "nginx-logs-%{+YYYY.MM.dd}"
  }
}
```

Restart Logstash:

```
sudo systemctl restart logstash
```
---

### 5️⃣ Configure Kibana


![](./img/Screenshot%20(403).png)

Access Kibana:

```
http://<ELK-SERVER-IP>:5601
```
![](./img/Screenshot%20(405).png)

Create index pattern:

```
nginx-logs-*
```
![](./img/Screenshot%20(412).png)

---


### 📊 Dashboards

![](./img/Screenshot%20(410).png)

![](./img/Screenshot%20(411).png)

Create visualizations in Kibana:   
📈 Error Rate Graph (4xx & 5xx)  
📊 Requests per Minute   
🌐 Top Client IPs   
🔍 Log Search Dashboard

---

### 📌 Conclusion

This project successfully demonstrates the implementation of a centralized logging and monitoring system using the ELK Stack on AWS. By aggregating logs from multiple application servers into a single platform, it significantly improves visibility, reduces troubleshooting time, and enhances system reliability.

The integration of log parsing, real-time visualization, and proactive alerting enables faster detection of anomalies and efficient incident response. This solution reflects a scalable and production-ready approach to log management, aligning with modern DevOps and Site Reliability Engineering practices.