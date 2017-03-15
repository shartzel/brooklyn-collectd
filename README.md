# Collectd

[Collectd](https://collectd.org/) is a deamon that collects, transmits and stores
peformance metrics.

## Basic Configuration


| Config Key                                | Description                                                         | Default                      |
|-------------------------------------------|---------------------------------------------------------------------|------------------------------|
| collectd.configfile                       | Path to the configuration file                                      | /etc/collectd.conf           |
| collectd.fqdnlookup                       | Look up FQDN                                                        | true                         |
| collectd.basedir                          | Collectd Root directory                                             | /var/lib/collectd            |
| collectd.pidfile                          | Path to pidfile                                                     | /var/run/collectd.pid        |
| collectd.plugindir                        | Directory where plugins are stored                                  | /usr/lib64/collectd          |
| collectd.typesdb                          | Path to types.db                                                    | /usr/share/collectd/types.db |
| collectd.autoloadplugins                  | Automatically load plugins for which a plugin stanza is found       | false                        |
| collectd.collectinternalstats             | Collect internal stats                                              | false                        |
| collectd.interval                         | Default interval to collect stats in seconds                        | 10                           |
| collectd.maxreadinterval                  | Max interval after backoff to attempt to read data                  | 86400                        |
| collectd.timeout                          | Number of iterations before data is considered missing              | 2                            |
| collectd.readthreads                      | Number of threads to start for reading plugins                      | 5                            |
| collectd.writethreads                     | Number of threads to start                                          | 5                            |
| collectd.writequeuelimithigh              | Max size of write queue                                             | 0 (unlimited)                |
| collectd.writequeuelimitlow               | Min size of write queue                                             | 0 (unlimited)                |
| collectd.configurl                        | URL to a file which will be appended to the collectd configuration. | ""                           |
| collectd.plugins.syslog.enable            | Enable the syslog plugin                                            | true                         |
| collectd.plugins.rrdtool.enable           | Enable the RRDTool plugin                                           | false                        |
| collectd.plugins.rrdtool.datadir          | Path to store RRD data                                              | /var/lib/collectd/rrd        |
| collectd.plugins.rrdtool.createfilesasync | Use background thread to write RRD files                            | false                        |
| collectd.plugins.rrdtool.cachetimeout     | If greater than zero, write values to cache                         | 120                          |
| collectd.plugins.rrdtool.cacheflush       | Write cached values to disk this often (seconds)                    | 900                          |
| collectd.plugins.rrdtool.writespersecond  | How many updates to write per second                                | 50                           |
| collectd.plugins.cpu.enable               | Enable the CPU plugin                                               | true                         |
| collectd.plugins.interface.enable         | Enable the Interface plugin                                         | true                         |
| collectd.plugins.load.enable              | Enable the Load plugin                                              | true                         |
| collectd.plugins.memory.enable            | Enable the Memory plugin                                            | true                         |
| collectd.plugins.network.enable           | Enable the Network plugin                                           | false                        |
| collectd.plugins.network.listen.enable    | Set collectd as listener (server)                                   | false                        |
| collectd.plugins.network.listen.address   | Address on which to listen                                          | *                            |
| collectd.plugins.network.listen.port      | Port on which to listen                                             | "25826"                      |
| collectd.plugins.network.server.enable    | Enable sending data to a server                                     | false                        |
| collectd.plugins.network.server.address   | Address of the collectd server                                      | ""                           |
| collectd.plugins.network.server.port      | Port on which the collectd server is listening                      | "25826"                      |
| collectd.plugins.disk.enable              | Enable the Disk plugin                                              | false                        |
| collectd.plugins.processes.enable         | Enable the Processes plugin                                         | false                        |
| collectd.plugins.protocols.enable         | Enable the Protocols plugin                                         | false                        |

This blueprint only exposes collectd and plugin configuration that is most commonly
used. More plugins and configuration may be added over time.
See [Collectd documentation](https://collectd.org/documentation/manpages/collectd.conf.5.shtml)
For more information on configuration.


## Example Use


In order to  use collectd to capture meaninful metrics on a system
you deploy, "attach" it to an entity using the SameServerEntity.

```
brooklyn.catalog:
  version: "0.1"
  item:
    type: org.apache.brooklyn.entity.software.base.SameServerEntity
    id: Tomcat8-with-collectd
    name: Tomcat8 With Collectd
    brooklyn.children:
      - type: org.apache.brooklyn.entity.webapp.tomcat.Tomcat8Server
        id: Tomcat8
      - type: 'collectd:0.1'

```

Collectd will then be deployed alongside your entity, on the same server, and
will capture the data from the system on which it is running, sending it 
to a collectd server for example:


```

name: my-app
location: my-location
services:
  - type: 'collectd:0.1'
    id: collectd-server
    name: CollectdServer
    brooklyn.config:
      collectd.plugins.rrdtool.enable: true
      collectd.plugins.disk.enable: true
      collectd.plugins.network.enable: true
      collectd.plugins.network.listen.enable: true
      collectd.plugins.network.listen.address: $brooklyn:attributeWhenReady("host.subnet.address")
  - type: 'Tomcat8-with-collectd:0.1'
    id: my-tomcat8
    name: My Tomcat8
    brooklyn.config:
      collectd.plugins.network.enable: true
      collectd.plugins.disk.enable: true
      collectd.plugins.protocols.enable: true
      collectd.plugins.network.server.enable: true
      collectd.plugins.network.server.address: $brooklyn:component("collectd-server").attributeWhenReady("host.subnet.address")

```
