# Fritz! exporter for prometheus

This is a prometheus exporter for AVM Fritz! home network devices commonly found in Europe. This exporter uses the devices builtin TR-064 API via the fritzconnection python module.

The exporter still is a bit simplistic in places but it should work with Fritz!Box and Fritz!Repeater Devices (and maybe others). It actively checks for supported metrics and queries the for all devices configured (Yes, it has multi-device support for all you Mesh users out there.)

It has been tested against an AVM Fritz!Box 7590 (DSL), a Fritz!Repeater 2400 and a Fritz!WLAN Repeater 1750E. If you have another box and data is missing, please file an issue or PR on GitHub.

## WARNING

This current version of the exporter has completely reworked how this exports metrics! If you have used this exporter in the past, you will get a completely new set of metrics.

### Changes

* the metrics prefix has changed from `fritzbox_` to `fritz_`
* all labels have been converted to lower-case (e.g. `Serial` -> `serial`) to be in line with the common usage of labels
* some matrics have been renamed to better reflect their content

## Metrics

The following groups of metrics are currently available:

* Base Information (Model, Serial, Software Version, Uptime)
* Software Information (Update available)
* LAN Statistics (Ethernet only)
* WAN Statistics
* DSL Statistics
* PPP Statistics

WiFi statistics are next on the list...

If there are information missing or not displayed on your specific device, please open an issue.

## Building and running

### Requirements

* Python >=3.6
* fritzconnection >= 1.0.0
* prometheus-client >= 0.6.0

A Pipfile for `pipenv` ist included with the source. So after install `pipenv` you can just type `pipenv install` inside the source directory and a virtual environment for running the exporter will be configured for you.

### System Service

This exporter can directly be run from a shell. Set the environment vars as describe in the configuration section of this README and run "python3 -m fritzbox_exporter" from the code directory.
There is a sample systemd unit file included in the docs directory.

### Docker

The recommended way to run this exporter is from a docker container. The included Dockerfile will build the exporter using an python:alpine container using python3.

To build execute

```bash
docker build -t fritz_exporter:latest .
```

from inside the source directory.

To run the resulting image use

```bash
docker run -d --name fritz_exporter -p <PORT>:<FRITZ_EXPORTER_PORT> -e FRITZ_EXPORTER_CONFIG="192.168.178.1,username,password" fritz_exporter:latest
```

Verify correct operation:

```bash
curl localhost:<FRITZ_EXPORTER_PORT>
```

```text
# HELP python_gc_objects_collected_total Objects collected during gc
# TYPE python_gc_objects_collected_total counter
python_gc_objects_collected_total{generation="0"} 481.0
python_gc_objects_collected_total{generation="1"} 112.0
python_gc_objects_collected_total{generation="2"} 0.0
# HELP python_gc_objects_uncollectable_total Uncollectable object found during GC
# TYPE python_gc_objects_uncollectable_total counter
python_gc_objects_uncollectable_total{generation="0"} 0.0
python_gc_objects_uncollectable_total{generation="1"} 0.0
python_gc_objects_uncollectable_total{generation="2"} 0.0
...
```

## Configuration

Configuration is done via environment vars.

The main configuration is done inside the environment variable `FRITZ_EXPORTER_CONFIG`. This variable must contain the comma-separated address, username and password for the device. If you need multiple devices simply repeat the three values for the other devices.

Example for a single device (at 192.168.178.1 username monitoring and the password "mysupersecretpassword"):

```bash
export FRITZ_EXPORTER_CONFIG="192.168.178.1,monitoring,mysupersecretpassword"
```

Example for three devices:

```bash
export FRITZ_EXPORTER_CONFIG="fritz.box,user1,password1,fritz.repeater,user2,password2,192.168.178.123,user3,password3"
```

NOTE: If you are using WiFi Mesh all your devices should have the same username password - you will still have to specify them all.

| Env variable | Description | Default |
|--------------|-------------|---------|
| FRITZ_EXPORTER_CONFIG   | Comma separated "hostname","user","password" triplets | none |
| FRITZ_EXPORTER_PORT | Listening port for the exporter | 9787 |

## Disclaimer

Fritz! and AVM are registered trademarks of AVM GmbH. This project is not associated with AVM or Fritz other than using the devices and their names to refer to them.

## Copyright

Copyright 2019-2021 Patrick Dreker <patrick@dreker.de>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
