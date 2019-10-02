# CANedge over-the-air manager (BETA)

Manage multiple CANedge devices connected to S3. Supports firmware and configuration updates of multiple devices. Works on a single firmware version at a time.

Features:
- Migrate devices to new Firmwares
- Migrate Configuration Files for MINOR/MAJOR updates
- Update Configuration Files (without updating the Firmware)
- Validates updated configuration files using the device Rule Schema
- Uses a user provided configuration migration function to update/modify configuration files

## Project structure
The project structure is:
```
.gitignore
LICENSE
README.md (this readme!)
canedge.py (main canedge manager module)
canedge_cli.py (canedge manager command line interface)
servers.json (example of S3 server details file)
config (configuration function examples)
│   compatible configuration function examples)
│   ...
|
└───obsolete
    │   (compatible configuration function examples)
    |   ...

```

## Installation

Requires:
- [Python 3.7.X](https://www.python.org/downloads/).
- Python modules (`pip install minio jsonschema pycryptodome`):
 - minio
 - jsonschema
 - pycryptodome

Tested with the following installed modules:
```
$ pip freeze
attrs==19.2.0
certifi==2019.9.11
future==0.17.1
jsonschema==3.0.2
minio==5.0.1
pycryptodome==3.9.0
pyrsistent==0.15.4
python-dateutil==2.8.0
pytz==2019.2
six==1.12.0
urllib3==1.25.6
```

## Module usage
The CANedge module can be imported and used directly to update device firmware and configuration.

```python
from canedge import CANedge, CANedgeReturnCodes

# Create client connection to S3 server
mc = Minio(endpoint="S3_SERVER_ENDPOINT",
           access_key="ACCESS_KEY",
           secret_key="SECRET_KEY",
           secure=True)

# Init a CANedge object
ce = CANedge(mc=mc,
             bucket="BUCKETNAME",
             fw_old_path="OLD_FIRMWARE_PATH.bin",
             fw_new_path="NEW_FIRMWARE_PATH.bin")

# Update configuration for a list of devices
ce.cfg_update(device_ids_to_update=[LIST OF DEVICE IDs],
              cfg_cb=CONFIGURATION_FUNCTION,
              config_name=config_name)

# Update firmware for a list of devices
ce.fw_update(device_ids_to_update=[LIST OF DEVICE IDs])

# Cleans unused Configuration and Rule Schema files
ce.cfg_clean()

```

## Command Line Interface (CLI) usage

The CLI is a simple front-end to the CANedge manager module. The CLI supports batch firmware, configuration and clean-up operations.

Things to note:
- The list of devices is created when the tool is started. To refresh the list, re-start the tool
- Even when using the CLI, the configuration function needs to be written in Python. The `config/` directory provides examples to be used as a starting point
- It is recommended to initially test with one or a few devices before updates are applied to all devices

Typing `python canedge_cli.py --help` provides the following help:
```
usage: canedge_cli.py [-h] [-s SERVER] [-a ALIAS] -b BUCKET -f CFGFUNC -c
                      FWCUR [-n FWNEW]

CANedge Command Line Interface

optional arguments:
  -h, --help            show this help message and exit
  -s SERVER, --server SERVER
                        A json file containing a list S3 server details (if
                        omitted, servers.json is used) (default: servers.json)
  -a ALIAS, --alias ALIAS
                        The alias of the relevant server from server to use (if
                        omitted, the 1st entry is used) (default: None)
  -b BUCKET, --bucket BUCKET
                        S3 bucket name (default: None)
  -f CFGFUNC, --cfgfunc CFGFUNC
                        A user-provided configuration function applied during
                        the configuration update (default: None)
  -c FWCUR, --fwcur FWCUR
                        The Firmware file currently used on the devices to be
                        updates (*.bin) (default: None)
  -n FWNEW, --fwnew FWNEW
                        The Firmware file to be migrated to (*.bin) (default:
                        None)
```

Tips:
- If the "-s" argument is omitted, the tool searches for a file named "servers.json"
- If the "-a" argument is omitted, the tool will select the first server alias in server file
- If used for firmware update to a new major / minor version, first run `config` to create a compatible configuration file
- MinIO server configuration files can be used directly (-s argument)

The format of the S3 server file is:
```
{
  "ALIAS": {
    "url": "hostname:port",
    "accessKey": "ACCESS_KEY",
    "secretKey": "SECRET_KEY"
  }
}
```

Running the above will connect you to the server and fetch a list of devices with specified firmware.

Once started, type ´help´ for more information on the available commands:
```
(CANedge)help

Documented commands (type help <topic>):
========================================
clean  config  exit  help  ls  update
```

### Example on starting the CLI tool:
`python canedge_cli.py -s servers.json -b office -f config_func_00_07_XX-00_07_XX.py -c firmware_07_01.bin`

### Example on using the CLI tool:
List devices:
```
(CANedge)ls
   0, ID: 071E61AD, FW: 00.07.01, HW: 00.00, CFG_CRC: 2218F2BC
   1, ID: 0DB63A93, FW: 00.07.01, HW: 00.00, CFG_CRC: D8CA618F
   2, ID: 0EEBBC43, FW: 00.07.01, HW: 00.00, CFG_CRC: 3D550818
   3, ID: 12C1A48D, FW: 00.07.01, HW: 00.00, CFG_CRC: 8C4E8370
   4, ID: 26AEB06C, FW: 00.07.01, HW: 00.00, CFG_CRC: F3CC41A8
   5, ID: 2FB30D1C, FW: 00.07.01, HW: 00.00, CFG_CRC: C5FCB62F
   6, ID: 3B912722, FW: 00.07.01, HW: 00.00, CFG_CRC: 4C14CA13
   7, ID: 4B23D6A0, FW: 00.07.01, HW: 00.00, CFG_CRC: 443BECCD
   8, ID: 51697F4D, FW: 00.07.01, HW: 00.00, CFG_CRC: 3E7D99C3
   9, ID: A9110452, FW: 00.07.01, HW: 00.00, CFG_CRC: 3BB5B899
  10, ID: ADC02130, FW: 00.07.01, HW: 00.00, CFG_CRC: 6127A604
  11, ID: C8B4DC35, FW: 00.07.01, HW: 00.00, CFG_CRC: ABD94077
  12, ID: D2920F21, FW: 00.07.01, HW: 00.00, CFG_CRC: E8F96938
  13, ID: E04A72BB, FW: 00.07.01, HW: 00.00, CFG_CRC: DED0F604
  14, ID: F7261DAD, FW: 00.07.01, HW: 00.00, CFG_CRC: 8C4C5238
```

Configuration update devices 0-5:
```
(CANedge)config -r 0-5
- 071E61AD | Config updated
- 0DB63A93 | Config updated
- 0EEBBC43 | Config updated
- 12C1A48D | Config updated
- 26AEB06C | Config updated
- 2FB30D1C | Config updated
```

Configuration update device 6 with --dry argument (a unused configuration file is created):
```
(CANedge)config -r 6-6 --dry
- 3B912722 | Config updated
```

# Changelog CANedge module
All notable changes to this project will be documented in this file.

## [00.00.02] - 2019-10-02
### Added

### Changed
- Configuration function prototype changed. Now takes additional arguments "tools", "index" and "device_id"
- Configuration tools object now includes helper functions for credential encryption

# Changelog CANedge CLI
All notable changes to this project will be documented in this file.

## [00.00.02] - 2019-10-02
### Added
- Configuration change `config` command now takes `--dry` argument to create test configuration file `config-XX.XX.json`. Eg. `config -r 0-0 --dry`
