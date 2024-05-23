---
title: Building UN1TY, a Cybersecurity Detection Lab
date: 2024-05-23 06:00:00 -0600
categories: [blueteam, tools]
tags: [monitoring, low, dev]
image:
  path: https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/18d749a0-d814-48cc-82db-bee422e501a0
---

In cybersecurity, having an efficient detection platform is critical to identifying and responding to threats. This blog will guide you through two essential steps to build your own detection lab (Client-Server Model): server installation, collector integration.

![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/01fe11b4-84c5-4d0e-abb1-5938977ac6bc)

## UN1TY Server Requirements
**Hardware**:
 - 2 cores of CPU
 - 8 GB of RAM 
 - 256 GB of Disk space

**Base Operating System**: [Ubuntu Server](https://ubuntu.com/download/server)

## UN1TY Installation
**Detection Platform**: We will use Elastic Security for our lab.
### **Elasticsearch as Database**
- [Installing elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#install-deb)
```sh
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.13.4-amd64.deb
sudo dpkg -i elasticsearch-8.13.4-amd64.deb
# > The generated password for the elastic built-in superuser is : *****
```
- Save or reset and save the password for elastic superuser
- `/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic` for reset pass
- Start Service
```sh
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```
- Test
```sh
curl -XGET -k -uelastic 'https://localhost:9200/_cluster/health'
# > {"cluster_name":"elasticsearch","status":"green",.....
```

### Kibana as UI Console
- [Installing kibana](https://www.elastic.co/guide/en/kibana/current/deb.html#install-deb)
```sh
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.13.4-amd64.deb
sudo dpkg -i kibana-8.13.4-amd64.deb
```
- Configure
```sh
# set server.host value to "0.0.0.0" in /etc/kibana/kibana.yml
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana 
/usr/share/kibana/bin/kibana-setup --enrollment-token <TOKEN>
# > Kibana configured successfully.
```
- Start Service
```sh
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
```
- Test > put `http://<HOST-IP>:5601` on your favorite browser, change `<HOST_IP>` with your host-ip
- Fill your credentials and enter
- And click on "Explore on my own"

### Fleet Server as Collectors Management
[Installing Fleet Server](https://www.elastic.co/guide/en/fleet/current/install-fleet-managed-elastic-agent.html#elastic-agent-installation-steps): After logging into kibana, follow these steps to install fleet server.
- ☰ > Management > Fleet > Agents > Add a Fleet Server
- Fill Name and URL
  - Name: `main`
  - URL: `https://<HOST-IP>:8220`
- Copy Linux Tar Command and add `--fleet-server-es-insecure` to the end of the command
```sh
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.4-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.13.4-linux-x86_64.tar.gz
cd elastic-agent-8.13.4-linux-x86_64
sudo ./elastic-agent install \
  --fleet-server-es=https://<HOST-IP>:9200 \
  --fleet-server-service-token=***** \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-ca-trusted-fingerprint=***** \
  --fleet-server-port=8220 --fleet-server-es-insecure
```
- Test
```sh
curl -XGET -k 'https://<HOST-IP>:8220/api/status'
# > {"name":"fleet-server","status":"HEALTHY"}
```

## Collector Integration
- [Installing Elastic Agent](https://www.elastic.co/guide/en/fleet/current/install-fleet-managed-elastic-agent.html#elastic-agent-installation-steps)
### For Windows
- ☰ > Management > Fleet > Agents > Add agent
- `Agent policy - WIN` for Windows and click on ***Create policy***
- Copy windows Command and add `--insecure` to the end of the command
```powershell
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.4-windows-x86_64.zip -OutFile elastic-agent-8.13.4-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.13.4-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.13.4-windows-x86_64
.\elastic-agent.exe install --url=https://<IP-HOS>:8220 --enrollment-token=***** --insecure
# > Elastic Agent has been successfully installed.
```
### For Linux
- ☰ > Management > Fleet > Agents > Add agent
- Click on ***Create new agent policy***, `Agent policy - LIN` for Linux and click on ***Create policy***
- Copy Linux Tar Command and add `--insecure` to the end of the command
```sh
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.4-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.13.4-linux-x86_64.tar.gz
cd elastic-agent-8.13.4-linux-x86_64
sudo ./elastic-agent install --url=https://<HOST-IP>:8220 --enrollment-token=***** --insecure
# > Elastic Agent has been successfully installed.
```

## UN1TY Results
- Finally, we will have the following result
- ☰ > Management > Fleet > Agents
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/acf29dbf-b0ea-4116-a90d-1a98e64a7f94)
- ☰ > Security > Explore > Hosts > All Hosts
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/affc0cbf-2671-4eb2-8031-fc9c7feb23c5)