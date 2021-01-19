# ICSNPP-BSAP-Serial

Industrial Control Systems Network Protocol Parsers (ICSNPP) - BSAP over Serial.

## Overview

ICSNPP-BSAP-Serial is a Zeek plugin for parsing and logging fields within the BSAP (Bristol Standard Asynchronous Protocol) protocol for serial->Ethernet traffic.

This parser requires a serial to Ethernet converter converting serial BSAP to Ethernet using either port 1234 or 1235 over UDP.

This plugin was developed to be fully customizable, so if you would like to drill down into specific BSAP packets and log certain variables, add the logging functionality to [scripts/main.zeek](scripts/main.zeek). The functions within [scripts/main.zeek](scripts/main.zeek) and [src/events.bif](src/events.bif) should prove to be a good guide on how to add new logging functionality.

This parser produces four log files. These log files are defined in [scripts/main.zeek](scripts/main.zeek).
* bsap_serial_header.log
* bsap_serial_rdb.log
* bsap_serial_rdb_ext.log
* bsap_serial_unknown.log

For additional information on these log files, see the *Logging Capabilities* section below.

## Installation

### Package Manager

This script is available as a package for [Zeek Package Manger](https://docs.zeek.org/projects/package-manager/en/stable/index.html)

```bash
zkg refresh
zkg install icsnpp-bsap-serial
```

If this package is installed from ZKG it will be added to the available plugins. This can be tested by running `zeek -N`. If installed correctly you will see `ICSNPP::BSAP_Serial`.

If you have ZKG configured to load packages (see @load packages in quickstart guide), this plugin and scripts will automatically be loaded and ready to go.
[ZKG Quickstart Guide](https://docs.zeek.org/projects/package-manager/en/stable/quickstart.html)

If you are not using site/local.zeek or another site installation of Zeek and just want to run this package on a packet capture you can add `icsnpp/bsap-serial` to your command to run this plugin's scripts on the packet capture:

```bash
git clone https://github.com/cisagov/icsnpp-bsap-serial.git
zeek -Cr icsnpp-bsap-serial/examples/bsap_serial_example.pcapng icsnpp/bsap-serial
```

### Manual Install

To install this package manually, clone this repository and run the configure and make commands as shown below.

```bash
git clone https://github.com/cisagov/icsnpp-bsap-serial.git
cd icsnpp-bsap-serial/
./configure
make
```

If these commands succeed, you will end up with a newly created build directory which contains all the files needed to run/test this plugin. The easiest way to test the parser is to point the ZEEK_PLUGIN_PATH environment variable to this build directory.

```bash
export ZEEK_PLUGIN_PATH=$PWD/build/
zeek -N # Ensure everything compiled correctly and you are able to see ICSNPP::BSAP_SERIAL
```

Once you have tested the functionality locally and it appears to have compiled correctly, you can install it system-wide:
```bash
sudo make install
unset ZEEK_PLUGIN_PATH
zeek -N # Ensure everything installed correctly and you are able to see ICSNPP::BSAP_SERIAL
```

To run this plugin in a site deployment you will need to add the line `@load icsnpp/bsap-serial` to your `site/local.zeek` file in order to load this plugin's scripts.

If you are not using site/local.zeek or another site installation of Zeek and just want to run this package on a packet capture you can add `icsnpp/bsap-serial` to your command to run this plugin's scripts on the packet capture:

```bash
zeek -Cr icsnpp-bsap-serial/examples/bsap-serial_example.pcapng icsnpp/bsap-serial
```

If you want to deploy this plugin on an already existing Zeek implementation and you don't want to build the plugin on the machine, you can extract the ICSNPP_Bsap_serial.tgz file to the directory of the established ZEEK_PLUGIN_PATH (default is `${ZEEK_INSTALLATION_DIR}/lib/zeek/plugins/`).

```bash
tar xvzf build/ICSNPP_Bsap_serial.tgz -C $ZEEK_PLUGIN_PATH 
```

## Logging Capabilities

### BSAP Header Log (bsap_serial_header.log)

#### Overview

This log captures BSAP header information for every BSAP packet converted to Ethernet and logs it to **bsap_serial_header.log**.

#### Fields Captured

| Field             | Type      | Description                                               |
| ----------------- |-----------|-----------------------------------------------------------| 
| ts                | time      | Timestamp                                                 |
| uid               | string    | Unique ID for this connection                             |
| id                | conn_id   | Default Zeek connection info (IP addresses, ports)        |
| ser               | string    | Message Serial Number                                     |
| dadd              | count     | Destination Address                                       |
| sadd              | count     | Source Address                                            |
| ctl               | count     | Control Byte                                              |
| dfun              | string    | Destination Function                                      |
| seq               | count     | Message Sequence                                          |
| sfun              | string    | Source Function                                           |
| nsb               | count     | Node Status Byte                                          |
| type_name         | string    | Local or Global header                                    |

### BSAP RDB (Remote Database Access) Log (bsap_serial_rdb.log)

#### Overview

This log captures BSAP RDB function information and logs it to **bsap_serial_rdb.log**.

The vast majority of BSAP traffic is RDB function traffic. The RDB access is used to read and write variables between master and slave RTU's.

#### Fields Captured

| Field                 | Type      | Description                                               |
| --------------------- |-----------|-----------------------------------------------------------|
| ts                    | time      | Timestamp                                                 |
| uid                   | string    | Unique ID for this connection                             |
| func_code             | string    | RDB function being initiated                              |
| data                  | string    | RDB function specific data                                |


### BSAP BSAP_RDB_EXT (Remote Database Access Extended) Log (bsap_serial_rdb_ext.log)

#### Overview

This log captures BSAP RDB Extension function information and logs it to **bsap_serial_rdb_ext.log**.

These Extension functions of RDB contain information from controllers loading date and time, setting clearing diagnostics, and calling system resets. These only pertain to the GFC 3308 controllers.

#### Fields Captured

| Field                 | Type      | Description                                               |
| --------------------- |-----------|-----------------------------------------------------------|
| ts                    | time      | Timestamp                                                 |
| uid                   | string    | Unique ID for this connection                             |
| dfun                  | string    | Destination Function                                      |
| seq                   | count     | Message Sequence                                          |
| sfun                  | string    | Source Function                                           |
| nsb                   | count     | Node Status Byte                                          |
| extfun                | string    | RDB extension function                                    |
| data                  | string    | RDB Ext function specific data                            |


### BSAP Unknown (bsap_serial_unknown.log)

#### Overview

This log captures all other BSAP traffic that hasn't been defined and logs it to **bsap_serial_unknown.log**.

#### Fields Captured

| Field                 | Type      | Description                                               |
| --------------------- |-----------|-----------------------------------------------------------|
| ts                    | time      | Timestamp                                                 |
| uid                   | string    | Unique ID for this connection                             |
| data                  | string    | BSAP unknown data                                         |

## ICSNPP Packages

All ICSNPP Packages:
* [ICSNPP](https://github.com/cisagov/icsnpp)

Full ICS Protocol Parsers:
* [BACnet](https://github.com/cisagov/icsnpp-bacnet)
    * Full Zeek protocol parser for BACnet (Building Control and Automation)
* [BSAP over IP](https://github.com/cisagov/icsnpp-bsap-ip)
    * Full Zeek protocol parser for BSAP (Bristol Standard Asynchronous Protocol) over IP
* [BSAP Serial->Ethernet](https://github.com/cisagov/icsnpp-bsap-serial)
    * Full Zeek protocol parser for BSAP (Bristol Standard Asynchronous Protocol) over Serial->Ethernet
* [Ethernet/IP and CIP](https://github.com/cisagov/icsnpp-enip)
    * Full Zeek protocol parser for Ethernet/IP and CIP

Updates to Zeek ICS Protocol Parsers:
* [DNP3](https://github.com/cisagov/icsnpp-dnp3)
    * DNP3 Zeek script extending logging capabilites of Zeek's default DNP3 protocol parser
* [Modbus](https://github.com/cisagov/icsnpp-modbus)
    * Modbus Zeek script extending logging capabilites of Zeek's default Modbus protocol parser

### Other Software
Idaho National Laboratory is a cutting edge research facility which is a constantly producing high quality research and software. Feel free to take a look at our other software and scientific offerings at:

[Primary Technology Offerings Page](https://www.inl.gov/inl-initiatives/technology-deployment)

[Supported Open Source Software](https://github.com/idaholab)

[Raw Experiment Open Source Software](https://github.com/IdahoLabResearch)

[Unsupported Open Source Software](https://github.com/IdahoLabCuttingBoard)

### License

Copyright 2020 Battelle Energy Alliance, LLC

Licensed under the 3-Part BSD (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  https://opensource.org/licenses/BSD-3-Clause

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.




Licensing
-----
This software is licensed under the terms you may find in the file named "LICENSE" in this directory.