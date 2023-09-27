# graylog-docker-compose
version: '3'
services:
# MongoDB: https://hub.docker.com/_/mongo/
  mongodb:
    image: mongo:5.0.13
    restart: always
    volumes:
      - mongo_data:/data/db
# Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/7.10/docker.html
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:7.10.2
    volumes:
      - es_data:/usr/share/elasticsearch/data
    restart: always
    environment:
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - "ES_JAVA_OPTS=-Dlog4j2.formatMsgNoLookups=true -Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
# Graylog: https://hub.docker.com/r/graylog/graylog/
  graylog:
    image: graylog/graylog:5.1
    volumes:
      - graylog_journal:/usr/share/graylog/data/journal
    restart: always
    environment:
# CHANGE ME (must be at least 16 characters)!
      - GRAYLOG_PASSWORD_SECRET=xxxxxxxxXXXXXXXX
# Password: admin
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      - GRAYLOG_HTTP_EXTERNAL_URI=http://127.0.0.1:9000/
      - GRAYLOG_ROOT_TIMEZONE=Asia/Taipei
      - GRAYLOG_MESSAGE_JOURNAL_ENABLED=true
      - GRAYLOG_MESSAGE_JOURNAL_MAX_AGE=183d
    entrypoint: /usr/bin/tini -- wait-for-it elasticsearch:9200 --  /docker-entrypoint.sh
    links:
      - mongodb:mongo
      - elasticsearch
    depends_on:
      - mongodb
      - elasticsearch
    ports:
# Graylog web interface and REST API
      - 9000:9000
# Syslog TCP
      - 1514:1514
# Syslog UDP
      - 1514:1514/udp
# GELF TCP
      - 12201:12201
# GELF UDP
      - 12201:12201/udp
volumes:
  mongo_data:
    driver: local
  es_data:
    driver: local
  graylog_journal:
    driver: local
