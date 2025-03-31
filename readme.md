# OpenTelemetry Demo using Dynatrace

This repository contains an OpenTelemetry instrumentation guide using the popular OpenTelemetry Astronomy Shop Demo application integrated with Dynatrace.

### Overview

In this guide you'll learn how to:
* Set up the OpenTelemetry demo application
* Instrument your application with OpenTelemetry
* Configure Dynatrace to collect and visualize telemetry data
* Analyze traces, metrics and logs within the Dynatrace platform

Preview:
- Compare Otel instrumentation with Dynatrace OneAgent auto-instrumentation 
- Compare the data we see from traces, metrics, logs using Otel vs OneAgent 

## Pre-requisities:
- Dynatrace Account, get your free 15-day SaaS trial account [here](https://www.dynatrace.com/signup/)
- Docker and Docker Compose, steps [here](https://opentelemetry.io/docs/demo/docker-deployment/)



### 1) Download the Astronomy Shop Demo 

First we will clone the Astronomy Shop from its Github repository:

`git clone https://github.com/open-telemetry/opentelemetry-demo.git cd opentelemetry-demo/`


### 2) Configure the demo app to send telemetry data to Dynatrace

Head to `src/otelcollector` from the cloned repository's root. There will be a `otelcol-config.yml` file present which contains the base Collector configurations for receivers, processors, 
connectors and exporters. We **do not want to edit this file but instead** we will create a new `otelcol-config-extras.yml` file to override and merge the configuration to send data back to Dynatrace. 

Copy this into the newly created 
`otelcol-config-extras.yml` file:

```
exporters: 
  # otlp/http exporter to Dynatrace.  
  otlphttp/dynatrace:  
    endpoint: "${DT_ENDPOINT}"  
    headers:  
      Authorization: "Api-Token ${DT_API_TOKEN}"  
 
processors:  
  cumulativetodelta:  
  
service:  
  pipelines:  
    traces/dynatrace:  
      receivers: [otlp]  
      processors: [batch]  
      exporters: [otlphttp/dynatrace]  
    metrics/dynatrace:  
      receivers: [otlp, spanmetrics]  
      processors: [batch, cumulativetodelta]  
      exporters: [otlphttp/dynatrace]  
    logs/dynatrace:  
      receivers: [otlp]  
      processors: [batch]  
      exporters: [otlphttp/dynatrace]`
```
This configuration will export logs, metrics, and traces to Dynatrace using OTLP

### 3) Configure the environment variables

In the cloned repository's root, create the file `docker-compose.override.yml` and paste this into the file:

```
services: 
  otel-collector: 
    environment: 
      - DT_ENDPOINT 
      - DT_API_TOKEN
```

This ensures the environment variables from Dynatrace will be passed onto all services where:

- `DT_ENDPOINT` is your Dynatrace tenant that will be the OTLP endpoint used by the collector to send data to the backend
- `DT_API_TOKEN` is your Access Token for your Dynatrace Environment

### 4) Create and configure the Access Token from your Dynatrace environment

Create an Access Token with the following scopes to ingest metrics, traces and logs from Otel. For more details on creating tokens, refer to [here](https://docs.dynatrace.com/docs/discover-dynatrace/references/dynatrace-api/basics/dynatrace-api-authentication?_gl=1*et5t8r*_gcl_au*MTQ5NjAzOTE3MC4xNzQzMzg2Mzc3*_ga*MTg0MDU4MDY1LjE3MzE1NDQzNjU.*_ga_1MEMV02JXV*MTc0MzM4NjM2NS44MS4xLjE3NDMzOTAzODYuMC4wLjA.)
* Ingest OpenTelemetry traces (openTelemetryTrace.ingest)
* Ingest metrics (metrics.ingest)
* Ingest logs (logs.ingest)

![alt_text](https://github.com/terrypengdt/OpenTelemetry-Demo/blob/main/images/Lab%202%20Otel%20token%20scope.png)

### 5) Export the environment variables 

Pass the Access Token and Dynatrace Endpoint by replacing the placeholder values with your token and Environment ID. We can pass the values via the terminal in below's example however if you are in a production environment you would use a secret handling mechanism.

```
export DT_ENDPOINT=https://{your-env-id}.live.dynatrace.com/api/v2/otlp 
export DT_API_TOKEN=dt0c01.MY_SECRET_TOKEN 
export OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=delta
```

### 6) Run the demo app

Now that everything is configured, we can deploy the demo app

`docker compose up --no-build`

This may take a couple of minutes the first time you deploy this app to download all of the service images.

When the app is running, you'll start to see this:

![alt_text](https://github.com/terrypengdt/OpenTelemetry-Demo/blob/main/images/Lab%202%20compose.png)
You should now be able to access the Web store: http://localhost:8080/

Other pages you can access:
- Grafana: http://localhost:8080/grafana/
- Load Generator UI: http://localhost:8080/loadgen/
- Jaeger UI: http://localhost:8080/jaeger/ui/
- More pages [here](https://opentelemetry.io/docs/demo/docker-deployment/#verify-the-web-store-and-telemetry)

### 7) Visualize the data in Dynatrace


