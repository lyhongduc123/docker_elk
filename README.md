# 🚣 ELK Stack + Frappe Logging Integration (via Docker Compose)

This project demonstrates how to set up an **ELK stack** (Elasticsearch, Logstash, Kibana) using Docker Compose, and **pipe logs** from a separate 3rd-party Docker Compose stack (e.g., Frappe app) into Logstash via the **GELF logging driver**.

2425II_INT3310_1

---

## 📦 Project Structure

```
elk-compose/
│
├── .env
├── docker-compose.yml
├── elk-pwd.yml
├── pipelines.ymp
├── logstash_pipelines/
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

### 2. Start Frappe

If you’re running a separate **Frappe Docker Compose** (3rd-party), connect it to the ELK network (or Logstash to the 3rd-party network):

1. Clone the Frappe repo

```bash
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```
2. Replace **pwd.yml** with **elk-pwd.yml** and rename it back

3. Build it

```bash
docker compose -f pwd.yml up -d
```

---

### 3. Start the ELK stack

```bash
docker compose up -d
```

This launches:

- Elasticsearch on port `9200`
- Logstash on port `5000`,`12201/udp`
- Kibana on `http://localhost:5601`

> ℹ️ Logstash is preconfigured to listen for GELF logs on port **12201/udp**.
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

---

## 🛠️ Logstash Config (logstash_pipelines/your_pipelines.conf)

```conf
input {
  gelf {
    port => 12201
    type => docker
  }
}

filter {
    # Your filter
}

output {
  stdout { }
  elasticsearch {
    hosts => ["https://es01:9200"]
    user => "elastic"
    password => "${ELASTIC_PASSWORD}"
    cacert => ["certificates/ca/ca.crt"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

---

## 🥪 Manual Test

You can test sending a log using a temporary container

Then check:

```bash
docker logs docker_elk-logstash01-1
```

---

## 💜 Cleanup

```bash
docker compose down -v
docker compose -f pwd.yml down -v
```

---

## ⚠️ Attention

- Make sure both Logstash and Frappe services are in the **same Docker network**.
- The gelf-address should be send to **127.0.0.1** not the container (localhost won't work)
- Logstash in this project mount Frappe's logs volume, that is why you need to run Frappe first
- You don't need to directly modify the 3rd compose (pwd.yml), you can create a override yml file and specify it when build

Example:
```yaml
services:
  frontend:
    logging:
      driver: gelf
      options:
        gelf-address: "udp://logstash01:12201"
    networks:
      - elknet
  ... More services

networks:
  docker_elk_frappe_network:
    driver: bridge
```

Consider: 
- Creating your own share network between Logstash and 3rd party before compose. 
- Or add Logstash to the 3rd party's network after running ELK