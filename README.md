# elk
Elasticsearch Logstash and Kibana in Container and sample consumer use cases


sudo dnf install -y podman podman-compose

mkdir elk-podman && cd elk-podman
mkdir logstash
touch docker-compose.yml logstash/logstash.conf

docker-compose.yml:

version: '3.8'

services:
  elasticsearch:
    image: docker.io/library/elasticsearch:8.12.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  logstash:
    image: docker.io/library/logstash:8.12.0
    container_name: logstash
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
      - "5000:5000/udp"

  kibana:
    image: docker.io/library/kibana:8.12.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  esdata:

Bring-up the container Image:

podman-compose down
podman-compose up -d

Firewall settings:

Port	Protocol	Service	Purpose
5601/tcp	TCP	Kibana	Web UI for visualizing logs and Elasticsearch data
9200/tcp	TCP	Elasticsearch	REST API for search, indexing, and queries
5000/tcp	TCP	Logstash	Generic TCP input for receiving logs (e.g., from Filebeat or custom scripts)
5000/udp	UDP	Logstash	Generic UDP input (often used for syslog or network devices)
5044/tcp	TCP	Logstash	Beats input – used by Filebeat, Winlogbeat, etc., to send logs

sudo systemctl status firewalld

sudo firewall-cmd --permanent --add-port=5601/tcp
sudo firewall-cmd --permanent --add-port=9200/tcp
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --permanent --add-port=5000/udp
sudo firewall-cmd --permanent --add-port=5044/tcp

sudo firewall-cmd --reload

sudo firewall-cmd --list-ports

5601/tcp 9200/tcp 5000/tcp 5000/udp 5044/tcp

Access from Remote Machine Now, from another machine, you can access:

http://<your-redhat-ip>:5601 → Kibana
http://<your-redhat-ip>:9200 → Elasticsearch API
