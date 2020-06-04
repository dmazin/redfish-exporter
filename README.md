# redfish-exporter

This is a Prometheus Exporter for extracting metrics from a server using the Redfish API.
The hostname of the server has to be passed as **target parameter** in the http call.

It has been tested with the following server models:

Cisco UCS C480M5, working properly since BMC FW 4.1(1d)  
Cisco UCS C240M4  
Cisco UCS C240M5  
Cisco UCS C220M4
Cisco UCS C220M5

Cisco BMC FW below 4.x has its flaws with regards to redfish API. Hence I recommend to update at least to 4.0(1c).

Dell PowerEdge R640  
Dell PowerEdge R730  
Dell PowerEdge R740  
Dell PowerEdge R640  
Dell PowerEdge R840  

## Example Call

if you are logged in to the POD running the exporter you can call

```bash
curl http://localhost:9200/redfish?target=node001r-bm001.cc.eu-de-1.cloud.sap&job=redfish/myjob
```

## Prerequisites and Installation

The exporter was written for Python 3.6 or newer. To install all modules needed you have to run the following command:

```bash
pip3 install --no-cache-dir -r requirements.txt
```

There is also a docker file available to create a docker container to run the exporter.

## The config.yml file

* The **listen_port** is providing the port on which the exporter is waiting to receive calls. it is overwritten by the environment variable **LISTEN_PORT**.

* The credentials for login to the switches can either be added to the config.yaml file or passed via environment variables. The environment variables are taking precedence over the entries in config.yaml file.

    If you have several different jobs with different credentials you can do a mapping of job names to environment variables using the **job2user_mapping** entries. `REDFISH_JOB1_USERNAME` and `REDFISH_JOB1_PASSWORD` would be the variables from the example of the first job.

    The environment overwrites the settings in the config file for username and password.

* The **app_env** can be specified in the config file or also via environment variable APP_ENV. If set to `production` it will only log info messages. For all other values the level will be set to debug.

* The **timeout** parameter specifies the amount of time to wait for an answer from the server. Again this can alos be provided via TIMEOUT environment variable.

* The **job** parameter specifies the Prometheus job that will be passed as label if no job was handed over during the API call.

### Example of a config file

```text
listen_port: 9200
username: <your username>
password: <your password>
timeout: 40
job: 'redfish/myjob'
job2user_mapping: {
  redfish/myjob: REDFISH_JOB1,
  redfish/otherjob: REDFISH_JOB2,
  redfish/thirdjob: REDFISH_JOB3
}
app_env: dev
max_retries: 3
```

## Exported Metrics

All metrics returned by the redfish exporter are gauge metrics.

### redfish_up

Indicating if the redfish API was giving useful data back (== 1) or not (== 0).

### redfish_health

Show the health information of the hardware parts like processor, memory, storage controllers, disks, fans, power and chassis if available.

### redfish_memory_correctable

### redfish_memory_uncorrectable

Showing the count of errors per dimm.

Cisco servers do not seem to provide this kind of information via redfish. Dell PowerEdge servers only with certain DIMM manufacturers (Samsung not, Micron Technology and Hynix Semiconductor do).

### redfish_powerstate

Showing the powerstate of the server

### redfish_response_duration_seconds

The duration of the first response of the server to a call to /redfish/v1

### redfish_scrape_duration_seconds

Total duration of scarping all data from the server

### redfish_version

A collection of firmware version data stored in the labels. The value is always zero.
