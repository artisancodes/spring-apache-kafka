# Spring for Apache Kafka Demo
Apache Kafka is a distributed and fault-tolerant stream processing system.

## Setup Kafka using Docker Compose
Docker Compose tool is handy for managing service setups. The Kafka cluster requires Zookeeper and Kafka Brokers.

```
version: '3'
services:
  zookeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports: 
      - "2181:2181"
    networks:
      - kafka-net

  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9092,OUTSIDE://localhost:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9092,OUTSIDE://0.0.0.0:9093
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CREATE_TOPICS: "artisancodes:1:1"
    networks:
      - kafka-net
networks:
  kafka-net:
    driver: bridge
```

## Start the Kafka Cluster
```
docker-compose up -d
```
The above command will simply start the Zookeeper and Kafka containers in detached mode. The -d mode ensures that the
container runs in the background.

## Verify the Kafka Cluster
In order to verify that the Kafka Cluster is running successfully, run the following command to see the running
containers:
```
docker-compose ps
```

# Quick Tour
To use the Apache Kafka with Spring, first, you must install and run Apache Kafka. Then you must declare a dependency
in your build tool:
```
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
</dependency>
```

## Spring Boot Consumer App
Here is a minimal consumer application:
```
@SpringBootApplication
public class KafkaDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(KafkaDemoApplication.class, args);
	}

	@Bean
	public NewTopic topic() {
		return TopicBuilder
				.name("topic1")
				.partitions(10)
				.replicas(1)
				.build();
	}

	@KafkaListener(id = "my-group", topics = "topic1")
	public void consumer(String message) {
		System.out.println("Consumed message: " + message);
	}
}
```

## Spring Boot Producer App
Here is a minimal producer application:
```
@SpringBootApplication
public class KafkaDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(KafkaDemoApplication.class, args);
	}

	@Bean
	public NewTopic topic() {
		return TopicBuilder
				.name("topic1")
				.partitions(10)
				.replicas(1)
				.build();
	}

	@Bean
	public ApplicationRunner runner(KafkaTemplate<String, String> template) {
		return args -> template.send("topic1", "Hello World!");
	}
}
```

# Configuring Topics
To add topics automatically to the broker, define a ```KafkaAdmin``` bean in your application context. To do so, you can
add a ```NewTopic``` ```@Bean``` for each topic to the application context.

```
@Bean
public KafkaAdmin admin() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    return new KafkaAdmin(configs);
}

@Bean
public NewTopic topic1() {
    return TopicBuilder
            .name("artisancodes-1")
            .partitions(10)
            .replicas(3)
            .compact()
            .build();
}
```

> **NOTE:**  When using Spring Boot, a ```KafkaAdmin``` bean is automatically registed so you only need the 
```NewTopic``` ```@Bean```s.

Starting with version 2.7, you can declare multiple ```NewTopic```s in a single ```KafkaAdmin.NewTopics```bean definition:
```
@Bean
public KafkaAdmin.NewTopics createTopics() {
    return new KafkaAdmin.NewTopics(
            TopicBuilder.name("topic-1")
                    .build(),
            TopicBuilder.name("topic-2")
                    .replicas(1)
                    .build(),
            TopicBuilder.name("topic-3")
                    .partitions(3)
                    .build()
    );
}
```

