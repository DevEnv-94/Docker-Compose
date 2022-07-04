```bash
user@server:~$ git clone https://gitlab.rebrainme.com/apps/voting.git
user@server:~$ vim docker-compose.yml
```
```yaml
version: "3.7"

networks:
  database_network:
  web_network:

services:
  mysql:
    image: mysql:5.7
    restart: on-failure
    networks:
      - database_network
    environment:
      MYSQL_DATABASE: voting
      MYSQL_USER: voting
      MYSQL_PASSWORD: voting
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"

  redis:
    image: redis:7.0.2-alpine
    networks:
      - database_network
    restart: on-failure

  voting:
    build: ./voting
    image: voting:local
    expose:
      - 9000
    networks:
      - database_network
      - web_network
    restart: on-failure
    depends_on:
      - mysql
      - redis
    env_file: ./voting/.env.dist

  nginx:
    image: nginx:1.23.0-alpine
    networks:
      - web_network
    restart: on-failure
    ports:
      - "127.0.0.1:20000:80"
    volumes:
      - ./voting/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - mysql
      - redis
      - voting
```
```bash
user@server:~$ sudo docker-compose build
user@server:~$ sudo docker-compose -p rbm30 up -d

user@server:~$ sudo docker exec rbm30_voting_1  sh -c "php artisan migrate --force"
user@server:~$ sudo docker exec rbm30_voting_1  sh -c "php artisan db:seed --force"

user@server:~$ curl 127.0.0.1:20000/polls
{"current_page":1,"data":[{"id":1,"question":"Odit magnam laudantium aut animi omnis.","created_at":"2022-07-04 17:48:13","updated_at":"2022-07-04 17:48:13"}
```