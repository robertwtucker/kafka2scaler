# Integrating Apache Kafka with Inspire Scaler

This is the companion repository for the article *Integrating Apache Kafka with Inspire Scaler* posted on my [personal website](https://robertwtucker.com). For detailed instructions and additional context, please see [the article](https://robertwtucker.com/posts/kafka2scaler).

## Getting Started

The article assumes that the following software is properly installed and configured on the machine being used. For more information, please see the installation links provided.

* A **Java JDK for version 8** with the latest updates. At the time of this writing, that is 8u222-b10. Given the recent change in [Oracle's licensing terms for Java](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html) releases, there are new restrictions around what is permissible under the license and this has created a lot of uncertainty (i.e. What activities qualify as "development?"). As such, [Azul System's](https://www.azul.com) Zulu Community builds of OpenJDK provide a great cross-platform alternative. So, if you do not already have a JDK installed, see the [Zulu Community download page](https://www.azul.com/downloads/zulu-community/) to find the latest version 8 JDK for your platform.

* **Inspire Scaler** running locally and using default port (`30600`). A remote Scaler server will work as long as the appropriate host name and port are substituted where `localhost:30600` is used in the instructions. This guide was created and tested with Scaler 12.5.0.18-FMAP.

* The **[Docker Engine](https://docs.docker.com/)** and the **[Docker Compose](https://docs.docker.com/compose/) tool** for defining and building multi-container Docker applications. This step is optional if you are not using the [Docker demo environment](#using-the-docker-demo-environment) described later in this document. See the multi-platform installation instructions available for the [Docker Engine](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) tool, respectively. The latest stable version for your platform is recommended but any Docker Engine release after 17.12.0 should work (the compose file uses format version 3.5). This guide was prepared and tested with Docker Engine version 18.09.2 and Docker Compose version 1.23.2.

* The latest binary download of **[Apache Kafka](https://kafka.apache.org/)** (currently 2.3.0). See the [Kafka download page](https://kafka.apache.org/downloads) for more information. If Docker is already installed, there is an option to use Docker images in lieu of installing Kafka. Read the [Create a Topic in Apache Kafka](https://robertwtucker.com/posts/kafka2scaler/#create-a-topic-in-apache-kafka) section of the post for more information.

## Roadmap

See the [open issues](https://github.com/robertwtucker/scaler2kafka-demo/issues) for a list of proposed features (and known issues).

## Contributing

Contributions are what make the open source community such an amazing place to be learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License
Copyright (c) 2019 Quadient Group AG and distributed under the MIT License. See `LICENSE` for more information.

## Contact
Robert Tucker - [@robertwtucker](https://twitter.com/robertwtucker)

Project Link: [https://github.com/robertwtucker/scaler2kafka-demo](https://github.com/robertwtucker/scaler2kafka-demo)

## Acknowledgements

* [Quadient Inspire](https://www.quadient.com/experience/omnichannel-communications-interactions/inspire-platform)
* [Apache Kafka](https://kafka.apache.org)
* [Apache Camel](https://camel.apache.org)
* [Spring Boot](https://spring.io/projects/spring-boot)