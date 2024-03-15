# Redis Clustering Demo

This repository contains a demo application showcasing how to use Redis clustering with a Spring Boot application. The application manages user data using Redis as a cache. It provides CRUD operations for managing user entities and leverages Redis clustering for caching.

## Prerequisites

Before running the application, ensure you have the following installed:

- Java Development Kit (JDK) 8 or higher
- MySQL database server
- Redis server

Install Redis Using below commands : 
```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis-stack-server
```

## Create Redis Cluster 

Go to redis installed location and create new folder (using root user if required)
```bash
cd /etc/redis
mkdir cluster-test
cd cluster-test
```

Create few nodes for cluster and copy conf file of other server 
```bash
mkdir 7000
cp ../redis.conf /7000
```

Replace the port and node details in that conf file 
```bash
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes
```

Do the same for rest of the nodes (7001, 7002, 7003, 7004, 7005). 
Note : We need atleast 3 master nodes (3 master + 3 slave nodes) to create Redis cluster.

After this start the server
```bash
redis-server /7000/redis.conf
redis-server /7001/redis.conf
...
```

Check the server startup in the logs
```bash
tail -f /var/log/redis/redis-server.log
```

Now all nodes should be up and running. Now we need to create a cluster of this 6 nodes we just started.
```bash
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```
It will show you the configuration, give yes. Now Redis Cluster is created and You can use this cluster in the SpringBoot project.


## Installation and Setup

### 1. Clone the Repository

```bash
git clone https://github.com/example/redis-cluster-demo.git
```

### 2. Database Configuration

Update the `application.properties` file located in `src/main/resources` with your MySQL database configuration:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/redisclusterdemo?createDatabaseIfNotExist=true
spring.datasource.username=<your_mysql_username>
spring.datasource.password=<your_mysql_password>
```

### 3. Redis Configuration

Update the Redis configuration in the `application.properties` file to point to your Redis cluster:

```properties
spring.cache.type=redis
spring.data.redis.cluster.nodes=localhost:7000,localhost:7001,localhost:7002,localhost:7003,localhost:7004,localhost:7005
spring.data.redis.cluster.max-redirects=1
spring.cache.redis.time-to-live=60m
```

Note : We can also give only 1 node url in "spring.data.redis.cluster.nodes=localhost:7000" and it will work. 
When starting the Spring Boot application, using the above given url it fetches all the nodes present in cluster so if 7000 node goes down spring boot will manage caching using other nodes available.
But if 7000 node is down and then we need to restart the Spring Boot application, then it will fail to fetch cluster details as Spring Boot doesn't know all the nodes url.
So as a fail safe, best practice is to give all nodes url, just in case some nodes are down, so Application's caching doesn't fail.

### 4. Build and Run

Build the application using Maven:

```bash
mvn clean package
```

Run the application:

```bash
java -jar target/redis-cluster-demo.jar
```

## Usage

The application exposes RESTful endpoints for managing users:

- `GET /api/users`: Get all users
- `GET /api/users/{id}`: Get user by ID
- `POST /api/users`: Create a new user
- `PUT /api/users/{id}`: Update an existing user
- `DELETE /api/users/{id}`: Delete a user by ID

## Technologies Used

- Spring Boot
- Spring Data JPA
- Redis
- MySQL
- Maven

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This project is licensed under the Yash Kakrecha License.
