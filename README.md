# GerminalCodebaseFramework Deploy

### Compile to war file

Eclipse -> GWT -> Compile

The war file is in `target` folder

### Deploy directory 

Create a deploy directory (like this one).

copy the war file to `webapps`

create a `docker-compose.yml`

```yaml
version: '2'
services:
  nginx:
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.0
    environment:
      - "cluster.name=elasticsearch"
      - "bootstrap.memory_lock=true"
      - "xpack.security.enabled=false"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
  app:
    image: jetty
    links:
      - elasticsearch
    depends_on:
      - "elasticsearch"
      - "nginx"
    volumes:
      - ./webapps:/var/lib/jetty/webapps
    ports:
      - "8080"
    environment:
      - VIRTUAL_HOST=${GERMINAL_HOSTNAME}
      - GERMINAL_ELASTIC_ADDR=elasticsearch
volumes:
  esdata1:
    driver: local

networks:
  esnet:
```

This docker compose include three containers: app (which is our project), nginx, and elasticsearch server

### Run on local

```shell
GERMINAL_HOSTNAME=localhost docker-compose up --scale app=3 -d
```

GERMINAL_HOSTNAME specifies the address whci the app is running on. On local, just set GERMINAL_HOSTNAME as localhost

`—scale app=3` specifies there are three app instances running

To shut it down

```shell
docker-compose down
```

### Run on remote server

Suppose remote server address is `www.germinal.com`

```shell
GERMINAL_HOSTNAME=www.germinal.com docker-compose up --scale app=3 -d
```

### Without Elasticsearch server inside

Suppoer elasticsearch server is running on `192.168.0.1`

```yaml
version: '2'
services:
  nginx:
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
  app:
    image: jetty
    depends_on:
      - "nginx"
    volumes:
      - ./webapps:/var/lib/jetty/webapps
    ports:
      - "8080"
    environment:
      - VIRTUAL_HOST=${GERMINAL_HOSTNAME}
      - GERMINAL_ELASTIC_ADDR=${GERMINAL_ELASTIC_ADDR}
volumes:
  esdata1:
    driver: local

networks:
  esnet:
```

Command to run

```shell
GERMINAL_HOSTNAME=www.germinal.com GERMINAL_ELASTIC_ADDR=192.168.0.1 docker-compose up --scale app=3 -d
```

Command to shut it down

```
docker-compose down
```

