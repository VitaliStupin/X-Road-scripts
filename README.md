# X-Road-scripts

Python2 is no longer supported. You can find previous scripts with Python2
support under `py2` branch.

This repository contains helper scripts that can simplify usage and
administration of X-Road.

Provided scripts support TLS authentication with Security Server.
Self-signed key and certificate can be created with openssl:
```
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
```
If Security Server requires TLS authentication then you can add your TLS
certificate in Security Server administration interface:
MEMBER/SUBSYSTEM -> Internal Servers -> INTERNAL TLS CERTIFICATES

Scripts were tested with Python 3.8 (Ubuntu 20.04 LTS) and Python 3.10 (Ubuntu 22.04 LTS)

## Messagelog:
[cat_mlog.sh](messagelog/cat_mlog.sh) - This script prints the contents
of archived X-Road messagelog files to STDOUT. Therefore, providing a way
to "grep" the contents of messagelog files.

[cat_mlog_st.sh](messagelog/cat_mlog_st.sh) - Slower version of
cat_mlog.sh that additionally outputs message SigningTime.

## Global Configuration and Metadata services
[xrdinfo](xrdinfo-src/xrdinfo) - Python module that can be imported and used in any Python3 application.
It implements the following:
* Loading of global configuration from Security Server, Central Server, or Configuration Proxy.
* Parsing of shared_params.xml and returning various elements.
* Handling of listMethods and allowedMethods X-Road requests for SOAP.
* Handling of listMethods and allowedMethods X-Road requests for REST.
* Handling of getWsdl X-Road requests.
* Handling of getOpenApi X-Road requests.
* Listing endpoints from OpenAPI documents.

xrdinfo module can be installed using pip:
```bash
# From checked out repository
pip install xrdinfo-src/
# Or directly from github
pip install 'xrdinfo @ git+https://github.com/ria-ee/X-Road-scripts.git@main#subdirectory=xrdinfo-src'
```

There are also some scripts that use xrdinfo module. Note that output
identifiers consist of slash separated Percent-Encoded parts to be
compatible with X-Road REST identifiers:
* [xrd_all_methods.py](xrdinfo-src/xrd_all_methods.py) - Returns the list of
  all or allowed (services with access rights granted to the issuer of
  the X-Road request) methods (services) in X-Road instance.
* ~~xrd_all_wsdls.py~~ - Downloads all WSDL documents of specified X-Road
  instance and creates an index for these WSDL's. This functionality
  was moved to repositories:
  * https://koodivaramu.eesti.ee/x-tee/x-road-catalogue-collector
  * https://koodivaramu.eesti.ee/x-tee/x-road-catalogue
* [xrd_methods.py](xrdinfo-src/xrd_methods.py) - Returns the list of methods
  provided or allowed by a specified X-Road Member.
* [xrd_registered_subsystems.py](xrdinfo-src/xrd_registered_subsystems.py) -
  Returns the list of all Subsystems in X-Road instance that are
  registered in at least one Security Server.
* [xrd_servers_ips.py](xrdinfo-src/xrd_servers_ips.py) - Returns the list of
  IP addresses of all Security Servers in X-Road instance. Can be used
  to configure Security Servers firewall.
* [xrd_servers.py](xrdinfo-src/xrd_servers.py) - Returns the list of all
  Security Servers in X-Road instance.
* [xrd_subsystems.py](xrdinfo-src/xrd_subsystems.py) - Returns the list of
  all Subsystems in X-Road instance.
* [xrd_members.py](xrdinfo-src/xrd_members.py) - Returns the list of
  all Members in X-Road instance.
* [xrd_subsystems_with_membername.py](xrdinfo-src/xrd_subsystems_with_membername.py) -
  Returns the list of all Subsystems in X-Road instance and additionally
  adding Member names to that list.
* [xrd_subsystems_with_server.py](xrdinfo-src/xrd_subsystems_with_server.py) - 
  Returns the list of all Subsystems in X-Road instance and additionally
  adding Security Servers to that list.
* [xrd_wsdl.py](xrdinfo-src/xrd_wsdl.py) - Returns the service WSDL using
  X-Road request. This script can be useful when
  http://SECURITYSERVER/wsdl service is disabled or TLS authentication
  is required to access Security Server.
* [xrd_openapi.py](xrdinfo-src/xrd_openapi.py) - Returns the service OpenAPI
  description using X-Road request.

## Health and Environment monitoring
[metrics.py](zabbix/metrics.py) - X-Road Health and Environment
monitoring collector for Zabbix. Can be used by:
* Central monitoring to collect Environmental and Health data about all
  Security Servers in X-Road instance.
* Security Server owners to collect Environmental data of their Security
  Server.
* X-Road members to collect other members Health data.

When using MySQL as Zabbix database pay attention to Zabbix
installation guide! You need to use **"character set utf8mb4 collate utf8mb4_bin"** database
creation parameters because service names and versions are case-sensitive
in X-Road and "service.v1" is not equal to "Service.V1".

**NB! Tested with Zabbix 7.0 LTS.**

[zabbix_cron.sh](zabbix/zabbix_cron.sh) - Sample shell script that can
be executed from crontab to periodically collect the data.

Use the provided examples to create your configuration file:
* [metrics.cfg_example](zabbix/metrics.cfg_example) - Example
  configuration file.
* [metrics.cfg_local_example](zabbix/metrics.cfg_local_example) -
  Example configuration file for local Security Server monitoring.

Environment monitoring is using Zabbix template
[envmon_template.xml](zabbix/envmon_template.xml) that should be
imported into Zabbix prior to execution of the collector.

Collector requires [zabbix_utils](https://github.com/zabbix/python-zabbix-utils) package
which can be installed with:
```
sudo pip install zabbix_utils
```

## Miscellaneous
[rights_given.py](misc/rights_given.py) - Can be executed inside
Security Server to display the list of Access Rights granted. Time is
displayed in local timezone. This script is using psycopg2 python module
that can be installed with the following command:
```
sudo apt-get install python3-psycopg2
```

The result is in CSV format and is outputted to STDOUT.

Usage:
```
sudo su xroad -c "python3 rights_given.py" > rights.csv
```

[oldest_log_without_timestamp.py](misc/oldest_log_without_timestamp.py) -
Can be executed inside Security Server to display time of the oldest
message that was not timestamped. Returns nothing if all messages are
timestamped. This script is using psycopg2 python module. Returns time
in UTC.

[last_successful_message.py](misc/last_successful_message.py) - Can
be executed inside Operational Monitoring machine (by default Security
Server machine) to display time of the last successful message. This
script is using psycopg2 python module. Returns time in UTC.

[certs_expiration.py](misc/certs_expiration.py) - Can be executed inside
Security Server to display expiration dates of all active and registered
certificates. Returns time in UTC.

[ocsp_produced.py](misc/ocsp_produced.py) - Can be executed inside
Security Server to display OCSP production time (the time of OCSP
response production) for all active and registered certificates. Returns
"ERROR" if ocsp response is not found. Returns time in UTC.

[globalconf_expiration.py](misc/globalconf_expiration.py) - Can be
executed inside Security Server to display expiration times of global
configuration parts.

[updated_hosts.py](misc/updated_hosts.py) - Can be used to check how
many hosts in Zabbix were updated recently. Zabbix URL and credentials
can be passed as command line arguments or via configuration file.
Example configuration file:
[updated_hosts.cfg_example](misc/updated_hosts.cfg_example).
