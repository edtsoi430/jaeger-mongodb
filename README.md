# jaeger-mongodb 

## Prerequisites:
1. Download latest version of [Docker]
2. Download latest version of [Go]
3. Download [MongoDB-Compass] to visualize documents in the database with ease (optional)

## Step by step instructions
1. Install (if you haven't already) and run MongoDB locally using Docker:
   - There are multiple ways to install mongoDB. In this guide we will be using a docker image:
        ```bash
        docker run -p 27017:27017 mongo:4.4
        ```
    - You can also run it in detached mode (with the -d option):
        ```bash
        docker run -d -p 27017:27017 mongo:4.4
        ```
2. Download and run Jaeger's sample application [HotROD] (which we will be tracing with Jaeger later on)

    ```bash
    docker run -it \
    -p8080-8083:8080-8083 \
    -e JAEGER_AGENT_HOST=<network_ip_address> \
    jaegertracing/example-hotrod:1.22 \
    all
    ```
    where the current <network_ip_address> can be found with the following command:

    ```bash
    ifconfig -l | xargs -n1 ipconfig getifaddr
    ```
   
3. Cd to the root directory (jaeger-mongodb) and run the following commands:
   - Build main.go with the following command. By default, we will be running and storing traces in MongoDB locally.
        ```bash
        GOOS=linux go build ./cmd/jaeger-mongodb
        ```
   - Start Jager with the following command
        ```bash
        docker run \
        -p 5775:5775/udp \
        -p 6831:6831/udp \
        -p 6832:6832/udp \
        -p 5778:5778 \
        -p 16686:16686 \
        -p 14268:14268 \
        -p 14250:14250 \
        -p 9411:9411 \
        -v $(pwd):/app \
        -e SPAN_STORAGE_TYPE=grpc-plugin \
        -e GRPC_STORAGE_PLUGIN_BINARY=/app/jaeger-mongodb \
        -e GRPC_STORAGE_PLUGIN_CONFIGURATION_FILE=/app/configs/default-config.yaml \
        jaegertracing/all-in-one:1.22
        ```
    - Note: If you have a custom configuration yaml file that you would like to use, change default-config.yaml to the name of your custom config file.

4. In a separate terminal tab/window, start mongodb
    ```bash
    mongodb
    ```

5. Now, you can open Jaeger (port 16682) and HotROD (port 8080) locally:
   - http://localhost:16686 (Jaeger)
   - http://localhost:8080 (HotROD)


6. You can then play around with the HotROD application, and the traces will be stored to the local MongoDB database that we previously set up. The traces will reflect in the Jaeger UI. If you have MongoDB compass install, you can connect to it to visualize the data.


## Archive
- We have attempted to roll out archive storage capability using grpc plugin, but currently Jaeger UI does not have an easy way to tell whether traces have been archived or not. In addition, you can also archive the same trace for an unlimited amount of times, which could result in lots of duplicate data in the archive storage. Therefore we have decided to skip the feature at the moment.
  
## Credit

This project is based on work from [jaeger] and [jaeger-influxdb]. Thank you authors!

[jaeger]: https://github.com/jaegertracing/jaeger/
[jaeger-influxdb]: https://github.com/influxdata/jaeger-influxdbjaeger-mongodb
[Docker]: https://www.docker.com/products/docker-desktop
[Go]: https://golang.org/doc/install
[MongoDB-Compass]: https://www.mongodb.com/products/compass
[HotROD]: https://github.com/jaegertracing/jaeger/tree/master/examples/hotrod