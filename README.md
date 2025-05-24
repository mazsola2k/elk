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
