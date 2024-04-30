# Netprobe Lite

Simple and effective tool for measuring ISP performance at home. The tool measures several performance metrics including packet loss, latency, jitter, and DNS performance. It also aggregates these metrics into a common score, which you can use to monitor overall health of your internet connection.

## Support the Project

If you'd like to support the development of this project, feel free to buy me a coffee!

https://buymeacoffee.com/plaintextpm

## Full Tutorial

Visit YouTube for a full tutorial on how to install and use Netprobe:

https://youtu.be/Wn31husi6tc


## Requirements and Setup

To run Netprobe Lite, you'll need a PC running Docker connected directly to your ISP router. Specifically:

1. Netprobe Lite requires the latest version of Docker. For instructions on installing Docker, see YouTube, it's super easy.

2. Netprobe Lite should be installed on a machine (the 'probe') which has a wired Ethernet connection to your primary ISP router. This ensures the tests are accurately measuring your ISP performance and excluding and interference from your home network. An old PC with Linux installed is a great option for this.

## Installation

1. Clone the repo locally to the probe machine:

```
git clone https://github.com/plaintextpackets/netprobe_lite.git
```

2. From the cloned folder, use docker compose to launch the app:

```
docker compose up
```

3. To shut down the app, use docker compose again:

```
docker compose down
```

## How to use

1. Navigate to: http://x.x.x.x:3001/d/app/netprobe where x.x.x.x = IP of the probe machine running Docker.

2. Default user / pass is 'admin/admin'. Login to Grafana and set a custom password.

### Change Netprobe port

To change the port that Netprobe is running on, edit the 'compose.yml' file, under the 'grafana' section:

```    
ports:
    - '3001:3000'
```

Change the port on the left to the port you want to access Netprobe on

### Customize DNS test

If the DNS server your network uses is not already monitored, you can add your DNS server IP for testing.

To do so, modify this line in .env:

```
DNS_NAMESERVER_4_IP="8.8.8.8" # Replace this IP with the DNS server you use at home
```

Change 8.8.8.8 to the IP of the DNS server you use, then restart the application (docker compose down / docker compose up)

### Data storage - default method

By default, Docker will store the data collected in several Docker volumes, which will persist between restarts.

They are:

```
netprobe_lite_grafana_data (used to store Grafana user / pw)
netprobe_lite_prometheus_data (used to store time series data)
```

To clear out old data, you need to stop the app and remove these volumes:

```
docker compose down
docker volume rm netprobe_lite_grafana_data
docker volume rm netprobe_lite_prometheus_data
```

When started again the old data should be wiped out.

### Data storage - bind mount method

Using the default method, the data is stored within Docker volumes which you cannot easily access from the host itself. If you'd prefer storing data in mapped folders from the host, follow these instructions (thank you @Jeppedy):

1. Clone the repo

2. Inside the folder create two directories:

```
mkdir -p data/grafana data/prometheus 
```

3. Modify the compose.yml as follows (volume path as well as adding user ID):

```
  prometheus:
    restart: always
    container_name: netprobe-prometheus
    image: "prom/prometheus"
    volumes:
      - ./config/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data/prometheus:/prometheus # modify this to map to the folder you created

    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - custom_network  # Attach to the custom network
    user: "1000" # set this to the desired user with correct permissions to the bind mount

  grafana:
    restart: always
    image: grafana/grafana-enterprise
    container_name: netprobe-grafana
    volumes:
      - ./config/grafana/datasources/automatic.yml:/etc/grafana/provisioning/datasources/automatic.yml
      - ./config/grafana/dashboards/main.yml:/etc/grafana/provisioning/dashboards/main.yml
      - ./config/grafana/dashboards/netprobe.json:/var/lib/grafana/dashboards/netprobe.json
      - ./data/grafana:/var/lib/grafana  # modify this to map to the folder you created
    ports:
      - '3001:3000'
    networks:
      - custom_network  # Attach to the custom network
    user: "1000" # set this to the desired user with correct permissions to the bind mount
```

4. Remove the volumes section from compose.yml


### Run on startup

To configure the tool to work as a daemon (run on startup, keep running), edit 'compose.yml' and add the following to each service:

```
restart: always
```

More information can be found in the Docker documentation.
