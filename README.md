# Simulation-Docker

Instructions on how to deploy and use the NLAE simulation.

## Prerequisites
- Install docker engine : https://docs.docker.com/engine/install/
- Install docker-compose: https://docs.docker.com/compose/install/

## Containers
The docker compose will deploy two containers `elasticsearch` and `simulation`.
- `elasticsearch` : this container contains elasticsearch:6.8.1 deployed using default configuration
- `simulation`    : contains the REST API.

## Deployment

Below is an example docker-compose used to deploy the simulation:

**Docker-compose.yml example**

```yml
version: "2.2"

services:

  elasticsearch:
    container_name: elasticsearch
    restart: always
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    image: "docker.elastic.co/elasticsearch/elasticsearch:6.8.1"
    networks:
      nlae:
        aliases:
          - elasticsearch
    ports:
      - "9200:9200"
    ulimits:
      memlock:
        hard: -1
        soft: -1
    volumes:
      - "esdata1:/usr/share/elasticsearch/data"

  simulation:
    image: "danny2097/nlae-simulation:latest"
    container_name: simulation
    depends_on:
      - elasticsearch
    networks:
      nlae:
        aliases:
          - nlp
    ports:
      - "8080:8080"
      
volumes:
  esdata1:
    driver: local

networks:
  nlae:
    driver: bridge
```



Simply run the one of the following commands:

For immediate console output from the containers

```
docker-compose up --build --force-recreate
```

For detached mode (with no immediate console output)

```
docker-compose up --build --force-recreate -d
```


The product of this will result in the creation of two containers with the names `simulation` and `elasticsearch`


**NOTE :** Please allow for upto **40 - 60 seconds** for the Elasticsearch cluster and Simulation Api to load and configure themselves before using. This is a result of a delay in the exposure of the host and port that Elasticsearch is bound to when cluster is starting. This then requires the API to essentially wait so it can create a client.   


## API Features

There are 3 Endpoints:
- **/process** : send an object to be processed by the Text Analytics Engine 
- **/query** : request entities that have been processed by the Text Analytics Engine 
- **/delete** : deletes object (document) from the Text Analytics Engine store.

## Models

The models associated with the **Endpoints** are presented below.

### Process

```json
{
  "entityType": "type : String",
  "fieldName": "type : String",
  "id": "type : String",
  "nlpFeatures": "type : [String]",
  "text": "type : String"
}
```

### Query

```json
{
  "entityType": "type : String",
  "fieldName": "type : String",
  "nlpExpression": "type : NlpExpression"
}
```

### Delete

```json
{
  "id": "type : String"
}
```

## API Documentation

- __**UI Endpoint Documentation**__ :
  More information see `<hostname>:port/swagger-ui.html` when deployed.

- __**Open API Specification**__ : More information see `<hostname>:port/v2/api-docs` when deployed.
