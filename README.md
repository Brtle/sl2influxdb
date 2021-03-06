# Seedlink to InfluxDB

Dump seedlink (seismological) time series into [InfluxDB](https://influxdata.com). Use
[Grafana](http://grafana.org) to plot waveforms, real time latency delay, etc. Maps uses
the grafana [worldmap-panel plugin](https://github.com/grafana/worldmap-panel).

Dockerfile, docker-compse.yml are available [here](https://github.com/marcopovitch/sl2influxdb/blob/master/docker/README.md).

## Install

```bash
pip install .
```

## Usage

```bash
~$ seedlink2influxdb -help
Usage: seedlink2influxdb [options]

Options:
  -h, --help            show this help message and exit
  --dbserver=DBSERVER   InfluxDB server name
  --dbport=DBPORT       InfluxDB server port
  --slserver=SLSERVER   seedlink server name
  --slport=SLPORT       seedlink server port
  --fdsnserver=FDSN_SERVER[:PORT]
                        fdsn station server name
  --streams=STREAMS     streams to fetch (regexp): [('.*','.*','.*Z','.*')]
  --flushtime=NUMBER    when to force the data flush to influxdb
  --db=DBNAME           InfluxDB name to use
  --dropdb              [WARNING] drop previous database !
  --keep=NUMBER         how many days to keep data
  --recover             use seedlink statefile to save/get streams from last
```

Example :

```bash
seedlink2influxdb
    --dbserver localhost \
    --dbport 8086 \
    --slserver rtserve.resif.fr \
    --fdsnserver http://ws.resif.fr \
    --db eost2 \
    --keep 1
```

> **Note**
>
> Fdsnserver request (`--fdsnserver` option) is optional. It is only used and performed
> at the begining and *could be slow* (if too much stations info are requested) ! Data
> collected are only used to get station coordinates and are converted to geohash,
> needed to plot measurements on a map.

## Screenshot

### Delay/Latency Map

<img src="https://cloud.githubusercontent.com/assets/4367036/22286118/6a4fa65e-e2ee-11e6-93ae-ae1b4f68a7a2.png" width="350">

### Latency & traces for multiple stations

<img src="https://cloud.githubusercontent.com/assets/4367036/12712706/95c4a38c-c8ca-11e5-8fa7-9c40bbdb8d24.png" width="350">

### Waveform, RMS, latency plots for a given station

<img src="https://cloud.githubusercontent.com/assets/4367036/12712707/95e9f498-c8ca-11e5-8115-cabb66dbf692.png" width="350">

## InfluxDB

InluxDB data representation (measurements, tags, fields, timestamps).

Measurements:

* **queue**: internal messages producer queue (seedlink thread) and consumer queue (influxdb exporter thread)
  * tags
    * **type**=(consumer|producer)
  * field
    * **size**=queue size
  * timestamp
* **count** : amplitude in count (waveforms)
  * tags
    * **channel** = channel name (eg. FR.WLS.00.HHZ)
  * field
    * **value** = amplitude
  * timestamp
* **latency** : seedlink packet propagation time from station to datacenter
  * tags
    * **channel** = channel name
  * field
    * **value** = latency value
  * timestamp
* **delay** : time since last seedlink packet was received
  * tags
    * **channel** = channel name (eg. FR.WLS.00.HHZ)
    * **geohash** = station coordinates geohash formated
  * field
    * **value** = latency value
  * timestamp

## Dependencies

* [obspy](https://github.com/obspy/obspy/wiki)
* [python InfluxDB](https://github.com/influxdata/influxdb-python)
* [geohash](https://pypi.org/project/python-geohash/)
* [grafana](http://grafana.org) (with [worldmap-panel plugin](https://github.com/grafana/worldmap-panel))

## Using docker-compose

A `docker-compose` is available to quickly setup influxdb and grafana.

### Data storage

If you are running this project for the first time, you have to create a
*influxdb data docker volume* in order to keep your measurements between restarts:

```bash
docker volume create --name=sl2influxdb_influxdb_data
```

### Start services

#### For RaspberryShake

This configuration is ready to be run, assuming your raspeberryshake is in you local
network and reachable using *raspberryshake.local* address.

To start all the containers (influxdb, seedlink fetcher and grafana):

```bash
docker-compose up -d rshakegrafana
```

Check the logs to see if seedlink data is fetched without problem:

```bash
docker-compose logs -f sl2raspberryshake
```

#### For Generic Seedlink Server

You need to customize the docker-compose.yml file to set properly this environement
variables:

* SEEDLINK_SERVER
* FDSN_WS_STATION_SERVER
* SEEDLINK_STREAMS (without space !!)

Then, starts the container:

```bash
docker-compose up -d grafana
```

To check the logs if seedlink data is fetched well:

```bash
docker-compose logs -f sl2generic
```

### Acces to grafana interface

Then launch you preferred browser and go to
[http://localhost:3000](http://localhost:3000), with:

* user: admin
* passwd : admin

### Stop services

```bash
docker-compose down -v
```
