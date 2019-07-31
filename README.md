# Integrating Apache Kafka with Inspire Scaler

## Introduction

Explain the purpose, provide an overview

## Solution Overview

High-level description

### Architecture

Picture

### Solution Components

#### Apache Kafka

Blurb on Kafka

#### Apache Camel

Blurb on Camel

#### Spring Boot

Blurb on Spring


## How-To Guide

The instructions in this section provide a step by step walk-through of creating and configuring a Spring Boot application using Apache Camel to provide a gateway to Inspire Scaler. A full copy of the completed project and its assets can be found LINK_HERE.


### Pre-requisites

This guide assumes that the following software is properly installed and configured on the machine being used. For more information, please see the installation links provided.

* A **Java JDK for version 8** with the latest updates. At the time of this writing, that is 8u222-b10. Given the recent change in [Oracle's licensing terms for Java](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html) releases, there are new restrictions around what is permissable under the license and this has created a lot of uncertainty (i.e. What activities qualify as "development?"). As such, [Azul System's](https://www.azul.com) Zulu Community builds of OpenJDK provide a great cross-platform alternative. So, if you do not already have a JDK installed, see the [Zulu Community download page](https://www.azul.com/downloads/zulu-community/) to find the latest version 8 JDK for your platform.

* **Inspire Scaler** running locally and using default port (30600). A remote Scaler server will work as long as the appropriate host name and port are substituted where *localhost:30600* is used in the instructions. This guide was created and tested with Scaler v12.5.0.18-FMAP.

* The **[Docker Engine](https://docs.docker.com/)** and the **[Docker Compose](https://docs.docker.com/compose/) tool** for defining and building multi-container Docker applications. This step is optional if you are not using the [Docker demo environment](#using-the-docker-demo-environment) described later in this document. See the multi-platform installation instructions available for the [Docker Engine](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) tool, respectively. The latest stable version for your platform is recommended but any Docker Engine release after 17.12.0 should work (the compose file uses format version 3.5). This guide was prepared and tested with Docker Engine version 18.09.2 and Docker Compose version 1.23.2.

* The latest binary download of **[Apache Kafka](https://kafka.apache.org/)** (currently 2.3.0). See the [Kafka download page](https://kafka.apache.org/downloads) for more information. If Docker is already installed, there is an option to use Docker images in lieu of installing Kafka. This is not recommended unless you are familiar with Docker and how networking and volume creation work.


### Create an Inspire Scaler Workflow

In many cases, the Scaler workflow(s) we will need to integrate with (invoke) will already exist. For this guide, we will create a simple workflow with only those items needed to prove the efficacy of the process.

1. To begin, access the user interface for the local instance of Scaler at [http://localhost:30600](http://localhost:30600).

1. Log in as an administrator and create a new *On Demand* workflow. Our example will be named *Simple Scaler Endpoint*.

1. Using the workflow editor, add three Scaler workflow components in this order:

   * HTTP Input
   * Script
   * HTTP Output

   You can either drag and drop the components from the pallette to the canvas or just double-click them in the pallette to move them over. Once finished, make sure that the components are all linked together in the aforementioned order.

   ![Scaler workflow components linked](docs/images/scaler-workflow.png)

1. Edit the **HTTP Input** component. Enter *simplews* for the *URL Endpoint* and set the *API authentication group* to *no authentication*. Take note of the *Request URL*. We will need this information later in order to contact this workflow via HTTP. Be sure to click *OK* to close the component detail popup and not the *X* so that the changes made will be persisted in memory.

   ![HTTP Input component](docs/images/http-input.png)

1. Open the **Script** component. In the *Variables* tab, click the *Required* checkbox for the *body* variable.

1. Add a simple line of script, `console.info(getvar("body"))`, in the code area. This will allow us to see exactly what was passed to Scaler in the body of the HTTP request from the *Job* details once the request is completed. Click *OK* to close the window.

   ![Script component](docs/images/script-component.png)

1. Edit the **HTTP Output** component, setting the *Response Type* to *Confirmation (HTTP 204)* before clicking *OK*.

1. At this point, our workflow is complete. Click the *disk* icon in the toolbar to save all of the in-memory changes to the database and the *up and right arrow* nearby to publish the initial draft of the workflow as version 1. Click *OK* to confirm.

1.  With the *Sample Scaler Endpoint* workflow selected in the *On Demand* workflows screen, click the *play* icon in the toolbar to deploy the workflow.


### Create a Topic in Apache Kafka

For simplicity's sake, this section is written as though the [Docker demo environment](#using-the-docker-demo-environment) is being used. If this is not the case and you have your own Kafka/Zookeeper environment available, please adjust the command parameters below and throughout the rest of this guide as appropriate.

Before proceeding further, read the [Using the Docker Demo Environment](#using-the-docker-demo-environment) section and follow the instructions to start it up. Once the Kafka server is running, open a console window or command prompt and switch to the directory you installed Kafka to. To create a topic to use for passing messages to Scaler, run one of the following script with the supplied parameters based on your operating system:

*MacOS & Linux*
```console
> bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic inspire --replication-factor 1 --partitions 1
```

*Windows*
```console
> bin\windows\kafka-topics.bat --bootstrap-server localhost:9092 --create --topic inspire --replication-factor 1 --partitions 1
```

If you would like to verify that the topic was properly created, you can use the command with the `--list` parameter and you should see **inspire** listed among the output.

*MacOS & Linux*
```console
> bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

*Windows*
```console
> bin\windows\kafka-topics.bat --bootstrap-server localhost:9092 --list
```

As mentioned previously, there is an option to use a Docker container based on the same Kafka image used for the demo environment in lieu of installing Kafka's binary distribution. For example, to create the topic in Kafka, you could execute the following:

```console
> docker run --rm --network kafka-demo confluentinc/cp-kafka \
>   kafka-topics --zookeeper zookeeper:2181 --create \
>   --topic inspire --replication-factor 1 --partitions 1
```

**Important**: Because the [Confluent Platform](https://www.confluent.io/product/confluent-platform/) Docker image for Kafka defines three volume mount-points, each time a `docker run` command using the image is executed, three new Docker volumes are created. Even though these volumes are very small in size, this can lead to the creation of plethora of Docker volumes without an associated container. In order to make sure that these are cleaned up, use a Docker command similar to the following (adjust as appropriate for your operating system):

```console
> docker volume rm $(docker ls -qf dangling=true)
```


### Create a Spring Boot project with Spring Initializr

1. In the browser, navigate to the Spring Initializr at [https://start.spring.io](https://start.spring.io).

   ![Spring Initializr](docs/images/spring-initializr.png)

1. Choose *Gradle Project* as the type of project and leave the *Language* and *Spring Boot* settings at their default.

1. Feel free to customize the name and description values in the *Project Metadata* section as appropriate. For the purposes of this walk-through, please do not change the *Packaging* or *Java* selections from *Jar* and *8*, respectively.

1. In the *Dependencies* section, type in '`camel`' and click the plus sign to add *Apache Camel*.

1. Click *Generate the project* to download a ZIP file containing the Spring Boot project structure and files that have been created according to the previous choices. Later on, we'll customize the project's configuration to build and run a Java application.

1. Expand (unzip) the project to a directory on disk. This will now be referred to as the `${project-root-dir}` or project root directory.


### Configure Apache Camel

[Apache Camel's](https://camel.apache.org) integration framework provides the mechanism we will use to connect Kafka and Scaler. Camel's routing and mediation engine is central to its power and flexibility. Using Camel's *route* idiom, we will define the system endpoints (*connector* configuration), how information (in the form of a *message*) will travel between them and what, if any, transformation and/or processing of the *message* might be required along the way.

Before we can define the route, we need to add the Camel connectors we will be using with our system endpoints. Camel already has a connector build specifically for Kafka. For Scaler, we need to use a connector for one of the supported input methods and Camel has existing connectors for HTTP, JMS and RabbitMQ. We will use the HTTP connector based on version 4 of Apache's HTTP client.

1. Using a text editor or Java development IDE, if you have one, edit the `build.gradle` file in the project's root directory. In the *dependency* section,  duplicate the line that starts with *implementation* twice and modify the copied lines so that the section looks like the following:

   ```
   dependencies {
	   implementation 'org.apache.camel:camel-spring-boot-starter:2.24.0'
	   implementation 'org.apache.camel:camel-kafka:2.24.0'
	   implementation 'org.apache.camel:camel-http4:2.24.0'
	   testImplementation 'org.springframework.boot:spring-boot-starter-test'
   }
   ```

   The version numbers might have changed by the time you are reading this, so just use whatever is current. Save and close the file.

1. Create a new sub-directory under the `${project-root-dir}/src/main/resources` directory named `camel`.

1. Create a file named `routes.xml` in the `${project-root-dir}/src/main/resources/camel` directory and open it for editing.

1. Copy and paste the route definition that follows into the editor:

   ```
   <routes xmlns="http://camel.apache.org/schema/spring">
      <route id="kafka2scaler">
         <from uri="kafka:{{kafka.consumer.topic}}?brokers={{kafka.brokers}}&amp;groupId={{kafka.consumer.groupId}}&amp;seekTo={{kafka.consumer.seekTo}}"/>
         <log message="Kafka sent: [${body}]"/>
         <to uri="http4:{{http.host}}:{{http.port}}{{http.resourceUrl}}?httpMethod={{http.method}}"/>
         <log message="Scaler returned:[HTTP ${header.CamelHttpResponseCode}], Body:[${body}]"/>
      </route>
   </routes>
   ```

   Save and close the file.

1. Rename the `application.properties` file under `${project-root-dir}/src/main/resources` to `application.yaml` and open it for editing.

1. Copy and paste the application configuration information below into the editor:

   ```
   camel:
     springboot:
       main-run-controller: true

   kafka:
     brokers: localhost:9092
     consumer:
       topic: inspire
       groupId: demo
       seekTo: end

   http:
     host: localhost
     port: 30600
     resourceUrl: /rest/api/submit-job/simplews
     method: POST
   ```

   Save and close the file.


### Build and Run the Application

Using a console windows or command prompt, switch to the project's root directory. If all of the steps in the [previous section](#configure-apache-camel) were completed correctly, we should be able to build and run our application. Use the [Gradle](https://gradle.org) build tool provided with the project to compile the source files.

```
> ./gradlew build
```

Windows users can omit the `./` prefix on the command above. This is only required for MacOS and *nix platforms.

If everything is correct, you will see a `BUILD SUCCESSFUL` message once all of the dependencies are downloaded, the files are compiled and tests are run. If not and errors are produced, recheck the changes made to the source files in the [previous section](#configure-apache-camel). Make sure the files have been saved before trying again.

When the application is compiled, we can now use the Spring Boot plugin to run the application directly.

```
> ./gradlew bootRun
```

As the application starts, several informational messages will be logged to the console. Once connected to Kafka, the application will wait for a message to be published in the **inspire** topic.

### Send a Message to Kafka


### Verify Scaler Received the Message


## Using the Docker Demo Environment

In order to provide ready access to a Kafka environment for testing this integration, a Docker Compose file that creates a single-node instance of Apache Kafka along with an Apache Zookeeper server is included with the project assets. The Zookeeper server is required by Kafka to support many of the distributed features (leader elections, partitions, etc.).

To start the demo environment, from a console window or command prompt in the project root directory, execute:
```console
> docker-compose up -d
```

This will create a network and start both server instances in *daemon* or background mode. By default, Kafka's primary listener will be bound to port `9092` of the host machine (`localhost:9092`). Because of the peculiarities of networking with Docker, a second listener has been configured to run on port `29092`. If you plan to connect to the demo Kafka server from another container (internal to Docker, on the `kafka-demo` network), you will need to use `kafka:29092` for the commands or parameters that require supplying the address of the Kafka broker.


## Areas for Further Study and/or Investigation

### Calling Scaler with Authentication Enabled

### Create a Response Object in Scaler and Publish it to Kafka
