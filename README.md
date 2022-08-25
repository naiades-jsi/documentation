# JSI NAIADES Documentation

## CAROUGE Deployment

Carouge is deployed on IRCAI machine. All components are dockerized.

### CAROUGE Entities for Input data

Entrypoint for data retrieval (at UDGA):
* `http://5.53.108.182:8668/v2/entities/{ENTITY_NAME}?attrs=deviceState,owner,value&lastN=1`

Note: Alternatively, `http://5.53.108.182:8668/` can be replaced with `http://test.naiades-project.eu:8668`.

Entity names:

* FlowerBed 1 &rarr; `urn:ngsi-ld:Device:Device-1f0d`
* FlowerBed 2 &rarr; `urn:ngsi-ld:Device:Device-1f08`
* FlowerBed 3 &rarr; `urn:ngsi-ld:Device:Device-1f10`
* FlowerBed 4 &rarr; `urn:ngsi-ld:Device:Device-1f06`
* FlowerBed 5 &rarr; `urn:ngsi-ld:Device:Device-1efd`
* FlowerBed 6 &rarr; `urn:ngsi-ld:Device:Device-1eff`
* FlowerBed 7 &rarr; `urn:ngsi-ld:Device:Device-1f02`
* FlowerBed 8 &rarr; `urn:ngsi-ld:Device:Device-1efe`
* Weather Station &rarr; `urn:ngsi-ld:WeatherObserved:EnvironmentalStation`

### CAROUGE Influx Schema

Access to Influx:

* Docker network: `host`
* Influx URL: `localhost:8086`
* Organisation: `naiades`
* Bucket: `carouge`
* Influx token (reference!)

Measurements:

* FlowerBed 1 &rarr; `device_1f0d`
* FlowerBed 2 &rarr; `device_1f08`
* FlowerBed 3 &rarr; `device_1f10`
* FlowerBed 4 &rarr; `device_1f06`
* FlowerBed 5 &rarr; `device_1efd`
* FlowerBed 6 &rarr; `device_1eff`
* FlowerBed 7 &rarr; `device_1f02`
* FlowerBed 8 &rarr; `device_1efe`

### CAROUGE Kafka Topics

* Docker network: `host`
* Kafka URL: `localhost:9092`

Fusion topics:

* FlowerBed 1 &rarr; `features_carouge_flowerbed1`
* FlowerBed 2 &rarr; `features_carouge_flowerbed2`
* FlowerBed 3 &rarr; `features_carouge_flowerbed3`
* FlowerBed 4 &rarr; `features_carouge_flowerbed4`
* FlowerBed 5 &rarr; `features_carouge_flowerbed5`
* FlowerBed 6 &rarr; `features_carouge_flowerbed6`
* FlowerBed 7 &rarr; `features_carouge_flowerbed7`
* FlowerBed 8 &rarr; `features_carouge_flowerbed8`

Prediction topics:

* FlowerBed 1 &rarr; `device_1f0d_pred_output`
* FlowerBed 2 &rarr; `device_1f08_pred_output`
* FlowerBed 3 &rarr; `device_1f10_pred_output`
* FlowerBed 4 &rarr; `device_1f06_pred_output`
* FlowerBed 5 &rarr; `device_1efd_pred_output`
* FlowerBed 6 &rarr; `device_1eff_pred_output`
* FlowerBed 7 &rarr; `device_1f02_pred_output`
* FlowerBed 8 &rarr; `device_1efe_pred_output`

### CAROUGE Entities for Output Data

TODO

### CAROUGE Component Schema
```mermaid
graph LR
    A[UDGA Historic API] --> |FIWARE-adapter| B[(InfluxDB)]
    B --> |Data Fusion| C(Carouge Watering)
    C --> |Uploder| D(UDG DMV)
```

* __FIWARE-adapter__
    * GitHub: `https://github.com/naiades-jsi/naiades-toolkit/tree/master/FIWARE-adapter`
    * Config file: `productionInflux/downloadScheduler.json`
    * DockerHub: `e3ailab/fiware_adapter_ircai`
    * Starting Docker: `docker run -d --network=host e3ailab/fiware_adapter_ircai`
* __Data Fusion__
    * GitHub: `https://github.com/naiades-jsi/data-fusion`
    * Use-case Config file: `config_data/config_carouge_*.json` _(Configs are generated on the fly, config file is not needed.)_
    * General secrets file: `secrets.json` &larr; copy `secrets.example.json` (and update secrets)
    * Starting: `python3 index.NAIDES.carouge_w.py`
    * DockerHub: `e3ailab/df_carouge_w_ircai`
    * Starting Docker: `docker run -d --network=host e3ailab/df_carouge_w_ircai`
* __Carouge Watering__
    * GitHub: `https://github.com/naiades-jsi/Carouge`
    * Config files:
        * Modeling (consumer): `main/config_deployment.json`
        * Scheduling: `schedule/schedule_real_data.json`
    * Starting: `python3 main.py -cc config_real_data.json -cs schedule_real_data.json`
    * DockerHub: `e3ailab/carouge_ircai`
    * Starting docker: `docker run -d --network=host e3ailab/carouge_ircai`
* __FIWARE-uploader__
    * GitHub: `https://github.com/gal9/FIWARE-uploader`
    * Config file: `config/deployment/carouge_watering.json`
    * Starting: `python3 main.py -c deployment/carouge_watering.json`
    * DockerHub: `e3ailab/uploader_carouge_watering_ircai`
    * Starting docker: `docker run -d --network=host e3ailab/uploader_carouge_watering_ircai`

### Monitoring Carouge Dataflow

Attach to `/bin/bash` on `ubuntu:20.04` container on IRCAI machine (find container id with `docker ps | grep ubuntu`) and attacht to it with `docker exec -it CONTAINER_ID /bin/bash`. If container is running you can skip to the section starting with "Finally, ...".

If it is not running, start it with: `docker run -it --network=host --entrypoint /bin/bash ubuntu:20.04` (TODO: check if this is true).

If you started the new Ubuntu container, you would have to update it by running:
```bash
apt-get update -y
apt-get install -y python3-pip python3-dev
apt-get install git
```

Next, you would have to clone the `NAIADES/wf-monitor` directory into `/home/` by running:
```bash
cd /home
git clone https://github.com/naiades-jsi/wf-monitor
cd wf-monitor
pip install -r requirements.txt
```

Finally, you run the workflow diagnostics with:
```bash
python3 main.py -w carouge
```

Optionally, you can pipe the output into a log file with a pip `python3 main.py -w carouge &> logs/carouge.log` and view the log with some other tool.

## ALICANTE Deployment

TODO

### ALICANTE Component Schema
```mermaid
graph LR
    A[UDGA Historic API] --> |FIWARE-adapter| B[(InfluxDB)]
    B --> |Data Fusion| C(LSTM Alicante Univariate)
    B --> |Data Fusion with Weather| D(LSTM Alicante Multivariate)
    B --> |Data Fusion Raw Cap.| E(Alicante Salinity)
    B --> |Data Fusion Raw Level| E
    C --> |Uploder| X[Simavi DMV]
    D --> |Uploader| X
    E --> |Uploader| X
```



## BRAILA Deployment

TODO

```mermaid
graph LR
    A[SIMAVI Historic API] --> |FIWARE-adapter| B[(InfluxDB)]
    G[EPANET Simulations] --> |File system| E
    B --> |Data Fusion| C(LSTM Braila Univariate)
    B --> |Data Fusion| D(Anomaly Detection Flow)
    B --> |Data Fusion| E(Leakage Detection)
    A --> |?| F(Precise Leakage Detection)
    C --> |Uploader| X[Simavi DMV]
    D --> |Uploader| X
    E --> |Uploader| X
    F --> |?| X
```


# Additional Infrastructure

## Docker

### Building an Image

Deployment is achieved via dockerized components. Docker should prepared on a local machine! Each component has a `Dockerfile` in the root directory. As many repositories are used to create multiple component, `Dockerfile` should be edited before building an image. Usually, the user has to editi the `[CMD]` line in the `Dockerfile` in order to reflect the component being built. Mostly, various options are commented in the `Dockerfile`.

To build the image the command should be run: `docker build -t DOCKERHUB_NAME .` (note the trailing dot in the command!). For Carouge those commands are:

* `docker build -t e3ailab/fiware_adapter_ircai .`
* `docker build -t e3ailab/df_carouge_w_ircai .`
* `docker build -t e3ailab/carouge_ircai .`
* `docker build -t e3ailab/uploader_carouge_watering_ircai .`

### Deploying an Image

After the image is being built (usually on a local machine) we have to push it to the DockerHub with: `docker push DOCKERHUB_NAME`. In order for this to work, you would have to log in into DockerHub with `docker login`.

After this, we move to the IRCAI machine. As we usually need to update the deployment, we should first run:

* `docker pull DOCKERHUB_NAME`

and finally:

* `docker run -d --network="host" DOCKERHUB_NAME`

## DockerHub `e3ailab`

DockerHub password is available at Aleš or Mark.

## InfluxDB

### Starting InfluxDB on IRCAI machine

### Obtaining InfluxDB Token

Token can be obtained on IRCAI machine in the InfluxDB container. User has to identify InfluxDB docekr file by runing `docker ps`. The result would usually be something like this:

```
CONTAINER ID   IMAGE                                               COMMAND                  CREATED        STATUS        PORTS                                                 NAMES
ba4e70a80576   e3ailab/carouge_ircai                               "python3 main.py -cc…"   2 days ago     Up 2 days                                                           suspicious_ellis
9d52325f0039   e3ailab/df_carouge_w_ircai                          "python3 index.NAIAD…"   2 days ago     Up 2 days                                                           upbeat_wing
75d42f41218f   e3ailab/uploader_carouge_watering_ircai             "python3 main.py -c …"   7 days ago     Up 7 days                                                           suspicious_feistel
281b7ca5e2b9   influxdb:latest                                     "/entrypoint.sh infl…"   5 months ago   Up 4 months   0.0.0.0:8086->8086/tcp                                influxdb
```

ContainerID for InfluxDB is `281b7ca5e2b9` in this case. To enter the container you have to run: `docker exec -it 281b7ca5e2b9 /bin/bash`.

Once in the container, run `influx auth list`; you have to choose the token for the `naiades` user.