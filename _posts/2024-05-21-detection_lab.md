---
title: Building a Cybersecurity Detection Lab - Unity
date: 2024-05-21 06:00:00 -0600
categories: [blueteam, tools]
tags: [monitoring, low, dev]
image:
  path: https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/7ca00a6e-63c6-4726-97b4-8ef5ec3e7b68
---

In cybersecurity, having an efficient detection platform is critical to identifying and responding to threats. This blog will guide you through three essential steps to build your own detection lab: installation, integration and testing.

## Requirements
**Hardware/Hyper-V**:
 - 2 cores of CPU,
 - 4 GB of RAM, and 
 - 256 GB of disk space.

**Base Operating System**: [Ubuntu Server](https://ubuntu.com/download/server)

## Installation
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
- Test > put `http://<HOST-IP>:5601/` on your favorite browser, change `<HOST_IP>` with your host-ip
- Fill your credentials and enter
- And click on "Explore on my own"

### Fleet Server as Agent Management**

## Integration

## Testing