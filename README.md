# MUICR-HERI-SURE
This repository contains the project deployment for the final project of the RIS subject for the Master's in Computer Engineering and Networks (UPV, Valencia). And puts in practice some other work performed previously ( jdomlac/anymqtt2hamqtt )

## Usage

Sadly, using ```docker compose up -d``` won't work right away, additional steps must be performed manually.

### Docker volumes
The first consideration is that we have choosen to declare all the docker volumes as external, meaning that they have to be manually created through:

```
docker volume create <name>
```

We did so because when performing ```docker compose down``` on a deployment also erases the docker volumes if not declared externally. You can choose, but remember to create or specify them accordingly.

### Environment file
An enviroment file declaring the following variables must be created during deployment:

```
DOCKER_INFLUXDB_INIT_MODE=setup
DOCKER_INFLUXDB_INIT_USERNAME=<username>
DOCKER_INFLUXDB_INIT_PASSWORD=<somepass>
DOCKER_INFLUXDB_INIT_ORG=<yourorganization>
DOCKER_INFLUXDB_INIT_BUCKET=<bucketname>

HOSTNAME=telegraf

GF_SECURITY_ADMIN_USER=<grafanauser>
GF_SECURITY_ADMIN_PASSWORD=<grafanapass>
```

### Telegraf config
It is mandatory to configure the telegraf data streams and the InfluxDB endpoint, it is necessary to generate an InfluxDB access token to perform this action.

```
# Name this telegraf.conf and place it into the configuration folder.
[global_tags]
  # Global labels

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  debug = false
  quiet = false
  logfile = ""

# Input Plugin para MQTT
[[inputs.mqtt_consumer]]
  servers = ["tcp://mqtt:1883"] # Replace if necessary
  topics = ["anymqtt2hamqtt/#"] # Replace if necessary
  qos = 0
  connection_timeout = "30s"
  client_id = "telegraf"
  username = ""
  password = ""
  data_format = "json"  # Cambia a "value" o "influx" si no usas JSON

# Output Plugin para InfluxDB
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"] # Replace if necessary
  token = "<your_token_here>"
  organization = "<yourorganization>"
  bucket = "<bucketname>"
```

### Grafana setup
During grafana setup you have to take into consideration that:

* The influx host name is the container name in the docker network
* The influx password is not the one set up on the .env file, you have to get an access token for that purpose (Can be the same used for telegraf)

### Setting up homepage (The initial homescreen)
There are a lot of public homepage setups you can get inspired by, we recommend starting at the homepage documentation page: https://gethomepage.dev/configs/


### Setting up the reverse proxy
You will need to setup the NGINX reverse proxy by accesing thoughout the 81 port on a web browser.

In some cases like HomeAssistant, there is a need to allow explicitly the use of proxies.

````
#homeassistant configuration.yaml

http:                        
  use_x_forwarded_for: true                       
  trusted_proxies:
    - <dockernetwork/cidr>
````

### Setting up the MQTT Broker
There is also a need to setup a basic ```mosquitto.conf``` file on the mosquitto's config directory, the minimal contents should like this:

```
listener 1883 0.0.0.0
allow_anonymous true
```

Further authentication or security is up to the implementer.

### Setting up anymqtt2hamqtt
Device configuration, including broker's authentication, and the private broker's endpoint and credentials must be performed, to files config files must be created at the application's config directory:

```
# Name this file to config.yaml and fill in the required fields
# The homeassistant broker configuration
broker_url: mqtt
broker_port: 1883
broker_username:
broker_password:
```

```
# Example of devices configuration file
# Name this file to devices.yaml and fill in the required fields
broker1:
  devices:
    em320:
      topic: ttn/em320
      type: em320_ttn
  username: exampleuser
  password: examplepassword
  port: 1883
  url: eu1.cloud.thethings.network
```