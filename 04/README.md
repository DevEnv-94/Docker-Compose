```bash
user@server:~$ git clone https://gitlab.rebrainme.com/apps/voting.git
user@server:~$ vim docker-compose.yml
```
```yaml
version: "3.7"

services:
  mysql:
    image: mysql:5.7
    restart: on-failure
    depends_on:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-async: "true"
        fluentd-address: 127.0.0.1:24224
        tag: mysql
    environment:
      MYSQL_DATABASE: voting
      MYSQL_USER: voting
      MYSQL_PASSWORD: voting
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"

  redis:
    image: redis:7.0.2-alpine
    depends_on:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-async: "true"
        fluentd-address: 127.0.0.1:24224
        tag: redis
    restart: on-failure

  voting:
    build: ./voting
    image: voting:local
    logging:
      driver: "fluentd"
      options:
        fluentd-async: "true"
        fluentd-address: 127.0.0.1:24224
        tag: voting_app
    expose:
      - 9000
    restart: on-failure
    depends_on:
      - mysql
      - redis
      - fluentd
    env_file: ./voting/.env.dist

  nginx:
    image: nginx:1.23.0-alpine
    logging:
      driver: "fluentd"
      options:
        fluentd-async: "true"
        fluentd-address: 127.0.0.1:24224
        tag: nginx
    restart: on-failure
    ports:
      - "127.0.0.1:20000:80"
    volumes:
      - ./voting/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - voting
      - fluentd

  fluentd:
    image: govtechsg/fluentd-elasticsearch:1.13.0
    restart: on-failure
    volumes:
      - ./fluentd/conf:/fluentd/etc
    ports:
      - "127.0.0.1:24224:24224"
      - "127.0.0.1:24224:24224/udp"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
    restart: on-failure
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - bootstrap.memory_lock=false
      - cluster.initial_master_nodes=es01
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"

  kibana:
    image: docker.elastic.co/kibana/kibana:7.8.1
    restart: on-failure
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
```
```bash
user@server:~$ sudo docker-compose build
user@server:~$ sudo docker-compose -p rbm31 up -d

user@server:~$ sudo docker exec rbm31_voting_1  sh -c "php artisan migrate --force"
user@server:~$ sudo docker exec rbm31_voting_1  sh -c "php artisan db:seed --force"

user@server:~$ curl 127.0.0.1:20000/polls
{"current_page":1,"data":[{"id":1,"question":"Maxime vel dicta magnam omnis dolorem.","created_at":"2022-07-05 16:21:04","updated_at":"2022-07-05 16:21:04"}
```
```bash
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match **>
  @type copy

    <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix ${tag}
    logstash_dateformat %Y.%m.%d
    </store>

  <store>
    @type stdout
  </store>

</match>
```

![indexes](https://github.com/DevEnv-94/logging_project/blob/master/images/Screenshot%202022-06-15%20at%2023.56.40.png) 
