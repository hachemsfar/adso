# Quickstart Guide
This section contains two types of guides: one for local standalone mode using Apache Kafka and the other for operating a confluent platform inside docker.

@@@ note
For using all the functionalities of the connector we recommend going for the Confluent platform.
@@@

## Kafka connect standalone

### Synopsis
This quickstart will show how to set up the aDSO Sink Connector against an existing SAP NetWeaver&reg;
system and run it locally in Kafka Connect standalone mode using Apache Kafka.
We will use a non encrypted communication channel with basic SAP&reg; username/password authentication.
In productive environments it is recommended to use a [SAP&reg Secure Network Communication (SNC)](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/e6/56f466e99a11d1a5b00000e835363f/content.htm?no_cache=true) setup
with optional Single Sign On (SSO).

### Preliminary setup
1. Download and extract [Apache Kafka](https://kafka.apache.org/downloads)
1. Copy the aDSO sink connector jar into the Kafka Connect plugins directory
1. Get a copy of SAP&reg; JCo v@JCO_VERSION@ (sapjco3.jar and native lib sapjco3.dll or sapjco3.so) and copy it to
   the plugins directory

### Configuration
1. Edit the contents of file __<kafka_root>/config/connect-standalone.properties__ like this:
    ```properties
    bootstrap.servers=localhost:9092
    key.converter=org.apache.kafka.connect.json.JsonConverter
    value.converter=org.apache.kafka.connect.json.JsonConverter
    key.converter.schemas.enable=true
    value.converter.schemas.enable=true
    offset.storage.file.filename=/tmp/connect.offsets
    offset.flush.interval.ms=10000
    plugin.path=/kafka_2.12-2.0.0/plugins
    ```
    @@@ note
    Make sure that the plugin path exists 
    @@@
1. Extract the properties template for the aDSO sink connector, which is located at __etc/ohja-adso-sink-connector.properties__ within the connector package. Then, copy this template to __<kafka_root>/config/ohja-adso-sink-connector.properties__.
1. Get in contact with your administration team for the connection properties of your SAP NetWeaver&reg; installation and maintain the following minimum connector properties:  
    ```properties
    # SAP Netweaver application host DNS or IP
    jco.client.ashost = 127.0.0.1
    # SAP system number
    jco.client.sysnr = 20
    # SAP client number
    jco.client.client = 100
    # SAP RFC user
    jco.client.user = user
    # SAP user password
    jco.client.passwd = password
    ```  
    @@@ note
    Make sure the SAP&reg; user has enough privileges to access RCF-enabled function modules and the SAP NetWeaver&reg; Gateway is configured to accept RFC connections from your host.
    @@@
1. Maintain the input Kafka topic name, and the technical name for the aDSO of your choice in SAP&reg;:  
    ```properties
    # technical name of the aDSO
    sap.adso#00.name=ZADSOTEST
    # name of the topic where data will be fetched from
    sap.adso#00.topic=ZADSOTEST
    ```

### Execution
1. Start a local Zookeeper instance from the shell e.g., for Windows OS type:  
    ```bash
    cd <KAFKA_ROOT>
    set KAFKA_LOG4J_OPTS=-Dlog4j.configuration=file:<KAFKA_ROOT>/config/tools-log4j.properties
    bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
    ```   
1. Start a local Kafka server instance from separate shell e.g., for Windows OS type:  
    ```bash
    bin\windows\kafka-server-start.bat .\config\server.properties
    ```
1. Start a simple kafka consumer from separate shell e.g., for Windows OS type:  
    ```bash
    bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic ODPSAPITEST --from-beginning
    ```
1. Start a local standalone Kafka Connect instance and execute the ODP source connector from separate shell e.g., for Windows OS type:  
    ```bash
    bin\windows\connect-standalone.bat .\config\connect-standalone.properties .\config\ohja-odp-source-connector.properties > log.txt 2>&1
    ```  
    The logging outputs will be written to the file __log.txt__

To view the JSON representations of the ODP data messages along with their schema, switch to the Kafka consumer shell. If everything is set up correctly, you should see the messages being printed out to the console output in JSON format.

### Logging
Check the log outputs by opening file __log.txt__ in an editor of your choice or for Windows OS just type:
```
type log.txt
```

## Confluent Platform

### Synopsis
This section shows how to launch the ODP source connector on a Confluent Platform running locally
within a docker environment.

### Preliminary setup
1. Install [Docker Engine](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) on the machine where you plan to run the Confluent Platform.
    1.  If you're using a Mac or Windows, [Docker Desktop](https://docs.docker.com/desktop/) includes both.
1. Download and launch a ready-to-go Confluent Platform Docker image as described in [Confluent Platform Quickstart Guide](<https://docs.confluent.io/platform/current/platform-quickstart.html#ce-docker-quickstart>)
1. Make sure that the machine running the Confluent Platform is connected to the SAP® source.
1. Make sure you have a licensed version of the  [SAP&reg; Java Connector 3.1 SDK](https://support.sap.com/en/product/connectors/jco.html) before proceeding.

### Connector Installation
The connector can either be installed manually or through the Confluent Hub Client.

@@@ note
In both scenarios it is beneficial to use a [volume](<https://docs.docker.com/compose/compose-file/#volumes>) to easily transfer the connector file into the Kafka Connect service container.
If running Docker on a Windows machine make sure to add a new system variable COMPOSE_CONVERT_WINDOWS_PATHS and set it to 1.
@@@

#### Manual Installation
To install the aDSO Sink Connector, follow these steps:

1. Unzip the downloaded file, *init-kafka-connect-adso-x.x.x.zip*.
1. Get a copy of SAP&reg; Java Connector v@JCO_VERSION@ SDK and move it to the *lib* folder inside the unzipped connector folder.
   SAP&reg; JCo consists of *sapjco3.jar*, and the native libraries like sapjco3.dll for Windows OS or sapjco3.so for Unix.
   Include e.g. the native lib *sapjco3.so* next to the *sapjco3.jar*
1. Move the unzipped connector folder into the configured CONNECT_PLUGIN_PATH of the Kafka Connect service
1. Within the directory where the *docker-compose.yml* of the Confluent Platform is located you can start the Confluent Platform using Docker Compose
    ```
    docker-compose up -d
    ```

#### Confluent Hub Client
Install the zipped connector *init-kafka-connect-adso-x.x.x.zip*using the Confluent Hub Client from outside the Kafka Connect docker container

```bash
docker-compose exec connect confluent-hub install {PATH_TO_ZIPPED_CONNECTOR}/init-kafka-connect-adso-x.x.x.zip
```

### Configuration
The connector can be configured and launched using the control-center service of the Confluent Platform.

1. In the control-center (default: [localhost:9091](localhost:9091)) select a Connect Cluster in the Connect tab.
1. Click the "Add connector" button and select the aDSOSinkConnector.
1. Enter a unique name for the connector instance and provide any other required configuration.

## Hints
- To prevent from SAP&reg; user lockout during the connector configuration process it is recommended to enter the *jco.client.user*
  and *jco.client.passwd* properties in the *JCo Destination* configuration block with caution since the control-center
  validates the configuration with every user input which includes trying to establish a connection to the SAP&reg; system
- If you enter the properties in the *JCo Destination* configuration block first, value recommendations for some other
  properties will be loaded directly from the SAP&reg; source system
- You can display the advanced JCo configuration properties in the Confluent Control Center UI by setting the configuration
  property *Display advanced properties* to 1
- Since you can push data to multiple *aDSO sinks* in one connector instance, an additional
  *aDSO sinks* configuration block will appear once you provided the required information for the first
  *aDSO sinks* configuration block
- There's a limit to the number of *aDSO sinks* that can be configured in the Confluent Control Center UI for one connector instance. If you need to configure additional sources beyond this limit, you can do so in the UI without any recommendations by using the Additional Properties section. The same applies to the number of selection conditions in full initialization mode.
