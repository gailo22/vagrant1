## Kibana vs Grafana

#### install ELK stack using docker compose

Create docker-compose.yml file using below content and run `docker-compose up`

```
version: '2.1'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.6.0
    volumes:
      - /home/vagrant/esdata:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - es

  kibana:
    image: docker.elastic.co/kibana/kibana:5.6.0
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: "http://elasticsearch:9200"
    networks:
      - es
    depends_on:
      - elasticsearch

networks:
  es:
    driver: bridge
````

Open browser to http://192.168.33.20:5601

**note** Some issue with elastic search license, you need to run this command:
```
curl -XPUT -u elastic 'http://localhost:9200/_xpack/license' -H "Content-Type: application/json" -d @license.json
```
and key in assword as changeme

#### install Grafana using docker
```
docker run -d --name=grafana -p 3000:3000 grafana/grafana

```

Open browser to http://192.168.33.20:3000/login and login as admin/admin