# 🚣 ELK Stack + Frappe Logging Integration (via Docker Compose)

This project demonstrates how to set up an **ELK stack** (Elasticsearch, Logstash, Kibana) using Docker Compose, and **pipe logs** from a separate 3rd-party Docker Compose stack (e.g., Frappe app) into Logstash

2425II_INT3310_1

---

## 📦 Project Structure

```
elk-compose/
│
├── .env
├── docker-compose.yml
├── pipelines.yml
├── extension
│   ├── filebeat (ingest_data, .yml)
│   ├── metricbeat (.yml)
│   └── packetbeat (.yml)
├── logstash_pipelines/
│   ├── delivery.conf
│   └── (any_pipelines).conf
└── README.md
```

---

## ✨ Quick Start

### 1. Clone this repository

```bash
git clone https://github.com/lyhongduc123/docker_elk
cd docker_elk
```

---

### 2. Start the ELK stack

```bash
docker compose up -d
```

---

### 3. Start Frappe

If you’re running a separate **Frappe Docker Compose** (3rd-party), connect it to the ELK network (or Logstash to the 3rd-party network):

1. Clone the Frappe repo

```bash
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

2. Build it

```bash
docker compose -f pwd.yml up -d
```

This launches:

- Elasticsearch on port `9200`
- Logstash on port `5044`,`9600`
- Kibana on `http://localhost:5601`

> ℹ️ Logstash is preconfigured to listen for FileBeat on port **5044**.
---

## 🔍 Verify in Kibana

1. Visit [http://localhost:5601](http://localhost:5601)
2. Go to **"Discover"**
3. Choose or create index pattern: `docker-logs-*`, `erpnext-logs-*`
4. You should see logs from your Frappe services in real-time.

---
## Tips
Change your hosts in your local so that browser don't reset the session
```bash
127.0.0.1 app.local
127.0.0.1 kibana.local
```
After that 
- Kibana is at [http://kibana.local:5601](http://kibana.local:5601)
- Frappe is at [http://app.local:8080](http://app.local:8080)

**Login Kibana with default user:**
- Username: elastic
- Password: (.env password, default is changeme)

**Login ERPNext with default user:**
- Username: Administrator
- Password: admin
---

## 🛠️ Logstash pipelines

### delivery.conf
```conf
input {
  beat {
    port => 5044
  }
}

# Modify tags to send to different pipelines
# Currently, tags are set in the filebeat.yml configuration file
# Tags: container-{id}, docker-container, ingest-data, docker-{container-name}


output {
  if "docker-container" in [tags] {
    pipeline {
      send_to => "docker_logs"
    }
  } else if "ingest-data" in [tags] {
    pipeline {
      send_to => "file_logs"
    }
  } else {
    stdout { codec => rubydebug }
  }
}

```
### docker_logs.conf
```conf
input {
  pipeline {
    address => "docker_logs"
  }
}

filter {

}

output {
  elasticsearch {
    hosts => ["https://es01:9200"]
    user => "elastic"
    password => "${ELASTIC_PASSWORD}"
    cacert => ["certificates/ca/ca.crt"]
    index => "filebeat-docker-logs-%{+YYYY.MM.dd}"
  }
}

```


## Metricbeat
Use Metricbeat at Stack Monitoring


## 💜 Cleanup

```bash
docker compose down -v
docker compose -f pwd.yml down -v
```