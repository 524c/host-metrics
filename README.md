# host-metrics
Dashboard + métricas de servidores, usando Grafana, collectd e influxdb

Setup on ubuntu/debian server (grafana and influxdb running on the same server)

# SERVER SIDE
Install Grafana
Ref.: https://grafana.com/docs/grafana/latest/installation/debian/

```
# To install the latest Enterprise edition:

sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Add this repository for stable releases:
echo "deb https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# After you add the repository:
sudo apt-get update
sudo apt-get -y install grafana-enterprise
```

### influxdb v1 only
```
sudo apt -y install influxdb-client influxdb
```

```
# /etc/influxdb/influxdb.conf

# INFLUXDB: add collectd block config
[[collectd]]
  enabled = true
  bind-address = "0.0.0.0:25827" # the bind address
  database = "collectd" # Name of the database that will be written to
  retention-policy = ""
  batch-size = 5000 # will flush if this many points get buffered
  batch-pending = 10 # number of batches that may be pending in memory
  batch-timeout = "10s"
  read-buffer = 0 # UDP read buffer size, 0 means to use OS default
  typesdb = "/usr/local/share/collectd/types.db"
  security-level = "encrypt" # "none", "sign", or "encrypt"
  auth-file = "/etc/collectd/auth_file"
  parse-multivalue-plugin = "split"  # "split" or "join"
```

### /etc/collectd/auth_file
```
collectd: passwd-xpto
```

# CLIENT SIDE
```
sudo apt-get install --no-install-recommends collectd collectd-core
```

### /etc/collectd/collectd.conf
```
LoadPlugin cpu
LoadPlugin disk
LoadPlugin load
LoadPlugin memory
LoadPlugin processes
LoadPlugin swap
LoadPlugin users
LoadPlugin interface
LoadPlugin df
LoadPlugin uptime
LoadPlugin logfile
LoadPlugin network

<Plugin "logfile">
  LogLevel "info"
  File "/var/log/collectd.log"
  Timestamp true
</Plugin>

<Plugin "network">
  <Server "REMOTE IP" "25827">
    SecurityLevel "none"
    Username "collectd"
    Password "passwd-xpto"
  </Server>
</Plugin>
```


### On grafana interface, after add a admin user, import the dashboard file "Host Overview-1640027639335.json"

