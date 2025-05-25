# Patient Management Microservices System

## Overview:
A comprehensive microservices-based Patient Management System utilizing Java and Spring Boot. The system is architected to handle patient data, billing processes, authentication, and analytics, ensuring scalability and maintainability.

## Key Features:

- **Microservices Architecture**: Implemented distinct services for patients, billing, authentication, analytics, and an API gateway to manage inter-service communication.

- **Spring Boot & Spring Cloud**: Leveraged Spring Boot for rapid application development and Spring Cloud for service discovery and configuration management.

- **gRPC Integration**: Utilized gRPC for efficient, low-latency communication between services, particularly between billing and analytics services.

- **Kafka Messaging**: Integrated Apache Kafka to handle asynchronous messaging and ensure reliable data exchange between services.

- **Docker**: Containerized all services and orchestrated them using Docker for streamlined development and deployment.

- **PostgreSQL Database**: Used PostgreSQL for persistent data storage for Auth and Patient services.

- **Security**: Implemented authentication and authorization mechanisms to secure service endpoints.

- **Testing**: Conducted integration testing to ensure seamless interaction between microservices.

## Docker Setup

kafka container
```
docker run -p 9092:9092 -p 9094:9094 --env KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094 --env KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER --env KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093 --env KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT --env KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094 --env KAFKA_CFG_NODE_ID=0 --env KAFKA_CFG_PROCESS_ROLES=controller,broker --name kafka --pull missing --network internal bitnami/kafka:latest 
```

auth-service-db
```
docker run -p 5001:5432 -v C:\Users\bhara\OneDrive\Desktop\Trying\docker\db_volumes\auth-service-db:/var/lib/postgresql/data --env POSTGRES_DB=db --env POSTGRES_PASSWORD=password --env POSTGRES_USER=admin_user --name auth-service-db --pull missing --network internal postgres:latest 
```

auth-service
```
docker build -f auth-service\Dockerfile -t auth-service:latest . && docker run --env JWT_SECRET=f710fcf314e2874e5912e441c007c81fe6519ddff44ad079a3a5e59bf8e0c44276b7057cf36a21f7c346d52b85b786ac9ef153522b03ede426efb1ea48ccb5d570daa0881fe2108e68ee51613991b19f03c58c78d40563802d8a5b32c2a8205cbcff3e4d97a5e411e675573446669e34a59df020e805289367c651b0d44bada00afa1cbd8e413df40b9b70079541f7040a27d8f0b80935f417596aec5c569420bb279ed5e7541055c042a2937f75304cd8813c9f0a7e730e427d105773d9bc79403611e5c23dc7b7416c695b779fcfdabe1c119fbcce7ebdd601860df014f3066ce3aa97fe11512377085b1f142af9f041f3a165693d9bd143e53132d5b420c5 --env SPRING_DATASOURCE_PASSWORD=password --env SPRING_DATASOURCE_URL=jdbc:postgresql://auth-service-db:5432/db --env SPRING_DATASOURCE_USERNAME=admin_user --env SPRING_JPA_HIBERNATE_DDL_AUTO=update --env SPRING_SQL_INIT_MODE=always --name auth-service --network internal auth-service:latest 
```

billing-service
```
docker build -f billing-service\Dockerfile -t billing-service:latest . && docker run -p 4001:4001 -p 9001:9001 --name billing-service --network internal billing-service:latest 
```

patient-service-db
```
docker run -p 5000:5432 -v C:\Users\bhara\OneDrive\Desktop\Trying\docker\db_volumes\patient-service-db:/var/lib/postgresql/data --env POSTGRES_USER=admin_user --env POSTGRES_PASSWORD=password --env POSTGRES_DB=db --name patient-service-db --pull missing --network internal postgres:latest 
```

patient-service
```
docker build -f patient-service\Dockerfile -t patient-service:latest . && docker run --env BILLING_SERVICE_ADDRESS=billing-service --env BILLING_SERVICE_GRPC_PORT=9001 --env SPRING_DATASOURCE_PASSWORD=password --env SPRING_DATASOURCE_URL=jdbc:postgresql://patient-service-db:5432/db --env SPRING_DATASOURCE_USERNAME=admin_user --env SPRING_JPA_HIBERNATE_DDL_AUTO=update --env SPRING_SQL_INIT_MODE=always --env SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092 --name patient-service --network internal patient-service:latest 
```

analytic-service
```
docker build -f analytics-service\Dockerfile -t analytics-service:latest . && docker run -p 4002:4002 --env SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092 --name analytics-service --network internal analytics-service:latest 
```

api-gateway
```
docker build -f api-gateway\Dockerfile -t api-gateway:latest . && docker run -p 4004:4004 --env AUTH_SERVICE_URL=http://auth-service:4005 --name api-gateway --network internal api-gateway:latest 
```
