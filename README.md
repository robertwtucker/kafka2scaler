# Integrating Apache Kafka with Inspire Scaler

## Introduction
Explain the purpose, provide an overview

## Solution Overview

### Architecture

### Solution Components

#### Apache Camel
#### Spring Boot

### How-To Guide
The following instructions will guide you step by step through the process of creating and configuring a Spring Boot application using Apache Camel to provide a gateway to Inspire Scaler. A full copy of the completed project and its assets can be found LINK_HERE.

1. In the browser, navigate to the Spring Initializr at [https://start.spring.io](https://start.spring.io).

![Spring Initializr](/docs/images/spring-initializr.png)

2. Choose **Gradle Project** as the type of project and leave the **Language** and **Spring Boot** settings at their default.
3. Feel free to customize the name and description values in the **Project Metadata** section as appropriate. For the purposes of this walkthrough, please do not change the Packaging or Java selections from Jar and 8, respectively.
4. For dependencies, search for `camel` and add **Apache Camel**.
5. Click **Generate the project** to download a ZIP file containing the Spring Boot project (Java application) that has been generated according to the previous choices.
6. Expand (unzip) the project to a directory on disk. This will now be referred to as the `${project_root}` directory.

### Using the Docker Demo Environment
In order to provide ready access to a Kafka environment, a Docker Compose file is included. This provides a single-node instance of Apache Kafka along with an Apache Zookeeper server. The Zookeeper server is required by Kafka to support many of the distributed features (leader elections, partitions, etc.).

From the project root, execute:
```
> docker-compose up -d
```

By default, Kafka's primary listener will be bound to port `9092` the host machine. If you plan to connect to the demo Kafka server from another container (internal to Docker, on the `kafka-demo` network), you will need to use `kafka:29092` when the commands or parameters request a broker.