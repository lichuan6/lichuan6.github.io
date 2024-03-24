# Install Prometheus and setup grafana and node_exporter

<!-- toc -->

- [Install Prometheus](#install-prometheus)
- [Install Grafana](#install-grafana)
- [Install Node Exporter](#install-node-exporter)
- [Config Prometheus to scape metrics from node_exporter](#config-prometheus-to-scape-metrics-from-node_exporter)
- [Grafana Dashboard](#grafana-dashboard)
  - [Import Dashboard](#import-dashboard)
  - [View the dashboard](#view-the-dashboard)
- [Refs](#refs)

<!-- tocstop -->

# Install Prometheus

First, let's download and install prometheus.

```bash
version=2.51.0
wget -c https://github.com/prometheus/prometheus/releases/download/v${version}/prometheus-${version}.darwin-amd64.tar.gz
tar xf prometheus-*.tar.gz
```

Running prometheus is pretty easy. All you need to do is running `prometheus` binary.

```bash
./prometheus
```

Output:

```bash
./prometheus
ts=2024-03-24T06:52:11.327Z caller=main.go:573 level=info msg="No time or size retention was set so using the default time retention" duration=15d
ts=2024-03-24T06:52:11.327Z caller=main.go:617 level=info msg="Starting Prometheus Server" mode=server version="(version=2.51.0, branch=HEAD, revision=c05c15512acb675e3f6cd662a6727854e93fc024)"
ts=2024-03-24T06:52:11.327Z caller=main.go:622 level=info build_context="(go=go1.22.1, platform=darwin/amd64, user=root@4c99b76a6ccd, date=20240319-10:54:45, tags=netgo,builtinassets,stringlabels)"
ts=2024-03-24T06:52:11.327Z caller=main.go:623 level=info host_details=(darwin)
ts=2024-03-24T06:52:11.327Z caller=main.go:624 level=info fd_limits="(soft=61440, hard=unlimited)"
ts=2024-03-24T06:52:11.327Z caller=main.go:625 level=info vm_limits="(soft=unlimited, hard=unlimited)"
ts=2024-03-24T06:52:11.334Z caller=web.go:568 level=info component=web msg="Start listening for connections" address=0.0.0.0:9090
ts=2024-03-24T06:52:11.336Z caller=main.go:1129 level=info msg="Starting TSDB ..."
ts=2024-03-24T06:52:11.340Z caller=tls_config.go:313 level=info component=web msg="Listening on" address=[::]:9090
ts=2024-03-24T06:52:11.340Z caller=tls_config.go:316 level=info component=web msg="TLS is disabled." http2=false address=[::]:9090
ts=2024-03-24T06:52:11.342Z caller=head.go:616 level=info component=tsdb msg="Replaying on-disk memory mappable chunks if any"
ts=2024-03-24T06:52:11.343Z caller=head.go:698 level=info component=tsdb msg="On-disk memory mappable chunks replay completed" duration=188.04µs
ts=2024-03-24T06:52:11.343Z caller=head.go:706 level=info component=tsdb msg="Replaying WAL, this may take a while"
ts=2024-03-24T06:52:11.345Z caller=head.go:778 level=info component=tsdb msg="WAL segment loaded" segment=0 maxSegment=1
ts=2024-03-24T06:52:11.346Z caller=head.go:778 level=info component=tsdb msg="WAL segment loaded" segment=1 maxSegment=1
ts=2024-03-24T06:52:11.346Z caller=head.go:815 level=info component=tsdb msg="WAL replay completed" checkpoint_replay_duration=89.814µs wal_replay_duration=2.840538ms wbl_replay_duration=89ns total_replay_duration=3.149532ms
ts=2024-03-24T06:52:11.350Z caller=main.go:1150 level=info fs_type=1a
ts=2024-03-24T06:52:11.350Z caller=main.go:1153 level=info msg="TSDB started"
ts=2024-03-24T06:52:11.350Z caller=main.go:1335 level=info msg="Loading configuration file" filename=prometheus.yml
ts=2024-03-24T06:52:11.361Z caller=main.go:1372 level=info msg="Completed loading of configuration file" filename=prometheus.yml totalDuration=11.479233ms db_storage=4.975µs remote_storage=4.641µs web_handler=305ns query_engine=706ns scrape=10.903788ms scrape_sd=42.235µs notify=21.224µs notify_sd=9.849µs rules=7.179µs tracing=211.699µs
ts=2024-03-24T06:52:11.361Z caller=main.go:1114 level=info msg="Server is ready to receive web requests."
ts=2024-03-24T06:52:11.361Z caller=manager.go:163 level=info component="rule manager" msg="Starting rule manager..."
```

# Install Grafana

Next, let's download and install grafana.

```bash
curl -O https://dl.grafana.com/oss/release/grafana-10.4.1.darwin-amd64.tar.gz
tar -zxvf grafana-10.4.1.darwin-amd64.tar.gz
```

You can run grafana using `./bin/grafana server` command.

```bash
./bin/grafana server
INFO [03-24|14:45:48] Starting Grafana                         logger=settings version=10.4.1 commit=d3ce857c0eb86f571ffa993a9cd8493b6f47b630 branch=HEAD compiled=2024-03-24T14:45:48+08:00
INFO [03-24|14:45:48] Config loaded from                       logger=settings file=/Users/hahahahaha/Documents/programs/grafana/grafana-v10.4.1/conf/defaults.ini
INFO [03-24|14:45:48] Target                                   logger=settings target=[all]
INFO [03-24|14:45:48] Path Home                                logger=settings path=/Users/hahahahaha/Documents/programs/grafana/grafana-v10.4.1
INFO [03-24|14:45:48] Path Data                                logger=settings path=/Users/hahahahaha/Documents/programs/grafana/grafana-v10.4.1/data
INFO [03-24|14:45:48] Path Logs                                logger=settings path=/Users/hahahahaha/Documents/programs/grafana/grafana-v10.4.1/data/log
INFO [03-24|14:45:48] Path Plugins                             logger=settings path=/Users/hahahahaha/Documents/programs/grafana/grafana-v10.4.1/data/plugins
INFO [03-24|14:45:48] Path Provisioning                        logger=settings path=/Users/hahahahaha/Documents/programs/grafana/grafana-v10.4.1/conf/provisioning
INFO [03-24|14:45:48] App mode production                      logger=settings
INFO [03-24|14:45:48] Connecting to DB                         logger=sqlstore dbtype=sqlite3
INFO [03-24|14:45:48] Starting DB migrations                   logger=migrator
INFO [03-24|14:45:48] migrations completed                     logger=migrator performed=0 skipped=548 duration=521.065µs
INFO [03-24|14:45:48] Envelope encryption state                logger=secrets enabled=true current provider=secretKey.v1
ERROR[03-24|14:45:48] Failed to get renderer plugin sources    logger=renderer.manager error="failed to open plugins path"
INFO [03-24|14:45:48] Loading plugins...                       logger=plugin.store
ERROR[03-24|14:45:48] Failed to load external plugins          logger=plugin.sources error="failed to open plugins path"
INFO [03-24|14:45:48] Plugin registered                        logger=plugins.registration pluginId=input
INFO [03-24|14:45:48] Plugins loaded                           logger=plugin.store count=56 duration=68.232976ms
INFO [03-24|14:45:48] Query Service initialization             logger=query_data
INFO [03-24|14:45:48] Live Push Gateway initialization         logger=live.push_http
INFO [03-24|14:45:48] Starting                                 logger=ngalert.migration
INFO [03-24|14:45:48] Running in alternative execution of Error/NoData mode logger=ngalert.state.manager
INFO [03-24|14:45:48] registering usage stat providers         logger=infra.usagestats.collector usageStatsProvidersLen=2
INFO [03-24|14:45:48] starting to provision alerting           logger=provisioning.alerting
INFO [03-24|14:45:48] finished to provision alerting           logger=provisioning.alerting
INFO [03-24|14:45:48] Warming state cache for startup          logger=ngalert.state.manager
INFO [03-24|14:45:48] Starting MultiOrg Alertmanager           logger=ngalert.multiorg.alertmanager
INFO [03-24|14:45:48] Storage starting                         logger=grafanaStorageLogger
ERROR[03-24|14:45:48] Failed to get renderer plugin sources    logger=renderer.manager error="failed to open plugins path"
INFO [03-24|14:45:48] State cache has been initialized         logger=ngalert.state.manager states=0 duration=565.65µs
INFO [03-24|14:45:48] Starting scheduler                       logger=ngalert.scheduler tickInterval=10s maxAttempts=1
INFO [03-24|14:45:48] starting                                 logger=ticker first_tick=2024-03-24T14:45:50+08:00
INFO [03-24|14:45:48] HTTP Server Listen                       logger=http.server address=[::]:3000 protocol=http subUrl= socket=
INFO [03-24|14:45:48] starting to provision dashboards         logger=provisioning.dashboard
INFO [03-24|14:45:48] finished to provision dashboards         logger=provisioning.dashboard
INFO [03-24|14:45:48] Adding GroupVersion playlist.grafana.app v0alpha1 to ResourceManager logger=grafana-apiserver
INFO [03-24|14:45:48] Adding GroupVersion featuretoggle.grafana.app v0alpha1 to ResourceManager logger=grafana-apiserver
INFO [03-24|14:45:52] Update check succeeded                   logger=grafana.update.checker duration=3.946779633s
INFO [03-24|14:45:55] Request Completed                        logger=context userId=0 orgId=0 uname= method=GET path=/ status=302 remote_addr=[::1] time_ms=0 duration=138.517µs size=29 referer= handler=/ status_source=server
INFO [03-24|14:45:56] Update check succeeded                   logger=plugins.update.checker duration=8.418350576s
INFO [03-24|14:46:34] Usage stats are ready to report          logger=infra.usagestats
INFO [03-24|14:46:40] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/live/ws status=-1 remote_addr=[::1] time_ms=0 duration=917.361µs size=0 referer= handler=/api/live/ws status_source=server
INFO [03-24|14:48:31] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/live/ws status=-1 remote_addr=[::1] time_ms=4 duration=4.228579ms size=0 referer= handler=/api/live/ws status_source=server
INFO [03-24|14:48:38] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/plugins/grafana-llm-app/settings status=404 remote_addr=[::1] time_ms=0 duration=488.121µs size=64 referer="http://localhost:3000/dashboard/new?editPanel=1&orgId=1" handler=/api/plugins/:pluginId/settings status_source=server
INFO [03-24|14:48:38] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/plugins/grafana-llm-app/settings status=404 remote_addr=[::1] time_ms=0 duration=630.382µs size=64 referer="http://localhost:3000/dashboard/new?editPanel=1&orgId=1" handler=/api/plugins/:pluginId/settings status_source=server
INFO [03-24|14:55:48] Database locked, sleeping then retrying  logger=sqlstore.transactions error="database is locked" retry=0 code="database is locked"
INFO [03-24|14:55:48] Completed cleanup jobs                   logger=cleanup duration=19.821422ms
INFO [03-24|14:56:05] Update check succeeded                   logger=plugins.update.checker duration=8.570043551s
ERROR[03-24|14:59:51] Dashboard not found                      logger=context userId=1 orgId=1 uname=admin error="Dashboard not found" remote_addr=[::1] traceID=
INFO [03-24|14:59:51] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/dashboards/uid/rYdddlPWk status=404 remote_addr=[::1] time_ms=1 duration=1.496887ms size=46 referer=http://localhost:3000/dashboard/import handler=/api/dashboards/uid/:uid status_source=server
INFO [03-24|15:00:47] Initialized channel handler              logger=live channel=grafana/dashboard/uid/rYdddlPWk address=grafana/dashboard/uid/rYdddlPWk
INFO [03-24|15:05:48] Completed cleanup jobs                   logger=cleanup duration=13.724174ms
```

# Install Node Exporter

After Prometheus and Grafana are installed, we can setup node exporter and configure prometheus to scrape metrics from node exporter.

Let's download node exporter binary file and setup node exporter.

```bash
wget -c https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.darwin-amd64.tar.gz
tar xf node_exporter*.gz
```

Start node exporter:

```bash
❯ ./node_exporter
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:192 level=info msg="Starting node_exporter" version="(version=1.7.0, branch=HEAD, revision=7333465abf9efba81876303bb57e6fadb946041b)"
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:193 level=info msg="Build context" build_context="(go=go1.19.12, platform=darwin/amd64, user=root@192f292aac5e, date=20231112-23:56:56, tags=netgo osusergo static_build)"
ts=2024-03-24T06:51:03.871Z caller=filesystem_common.go:111 level=info collector=filesystem msg="Parsed flag --collector.filesystem.mount-points-exclude" flag=^/(dev)($|/)
ts=2024-03-24T06:51:03.871Z caller=filesystem_common.go:113 level=info collector=filesystem msg="Parsed flag --collector.filesystem.fs-types-exclude" flag=^devfs$
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:110 level=info msg="Enabled collectors"
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=boottime
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=cpu
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=diskstats
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=filesystem
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=loadavg
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=meminfo
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=netdev
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=os
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=powersupplyclass
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=textfile
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=thermal
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=time
ts=2024-03-24T06:51:03.871Z caller=node_exporter.go:117 level=info collector=uname
ts=2024-03-24T06:51:03.872Z caller=tls_config.go:274 level=info msg="Listening on" address=[::]:9100
ts=2024-03-24T06:51:03.872Z caller=tls_config.go:277 level=info msg="TLS is disabled." http2=false address=[::]:9100
```

# Config Prometheus to scape metrics from node_exporter

With Prometheus, Grafana and node exporter up and running, we can configuring Prometh eus instance to scrape metrics from node exporter.

Your locally running Prometheus instance needs to be properly configured in order to access Node Exporter metrics. The following prometheus.yml example configuration file will tell the Prometheus instance to scrape, and how frequently, from the Node Exporter via `localhost:9100`:

```yaml
global:
  scrape_interval: 15s

# Add the following to prometheus.yml
scrape_configs:
  - job_name: node
    static_configs:
      - targets: ["localhost:9100"]
```

# Grafana Dashboard

You can create a dashboard or you can import dashboard from https://grafana.com/grafana/dashboards

The node exporter dashboard is located at:
https://grafana.com/grafana/dashboards/1860-node-exporter-full/

## Import Dashboard

Importing a dashboard is easy. All you need to do is to specify the dashboard id.

Don't forget to choose prometheus as data source.

![Import Dashboard](https://github.com/lichuan6/i/blob/main/grafana/Screen%20Shot%202024-03-24%20at%2015.00.45.png?raw=true)

## View the dashboard

After the dashboard is imported, you can view the dashboard.

Go to Home - Dashboard, search `Node Exporter Full` and you will see the information about the local machine.

![Dashboard - Node Exporter Full](https://github.com/lichuan6/i/blob/main/grafana/Screen%20Shot%202024-03-24%20at%2015.36.40.png?raw=true)

# Refs

https://grafana.com/docs/grafana/latest/datasources/prometheus/configure-prometheus-data-source/
