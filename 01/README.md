docker-compose-unmounted.yml
```yaml 
version: "3"                
services:                  
  Frontend:                    
    image: nginx:stable
    ports:
      - "127.0.0.1:18888:80"
```

docker-compose-mounted.yml
```yaml
version: "3"                
services:                  
  Frontend:                    
    image: nginx:stable
    ports:
      - "127.0.0.1:18888:80"
    volumes:
      - ./nginx.conf:/etc/nginx.conf
```

```bash
user@server:~/rbm27$ sudo docker-compose -f docker-compose-unmounted.yml -p rbm27 up -d

user@server:~/rbm27$ sudo docker-compose -f docker-compose-mounted.yml -p rbm27  up -d
Recreating rbm27_Frontend_1 ... done

user@server:~/rbm27$ curl 127.0.0.1:18888
Welcome to the training program RebrainMe: Docker!
```