NetData for YunoHost
---------------------

[![Integration level](https://dash.yunohost.org/integration/netdata.svg)](https://ci-apps.yunohost.org/jenkins/job/netdata%20%28Community%29/lastBuild/consoleFull) ![](https://ci-apps.yunohost.org/ci/badges/netdata.status.svg) ![](https://ci-apps.yunohost.org/ci/badges/netdata.maintain.svg)  
[![Install NetData with YunoHost](https://install-app.yunohost.org/install-with-yunohost.svg)](https://install-app.yunohost.org/?app=netdata)

[NetData](http://my-netdata.io/) is a system for **distributed real-time performance and health monitoring**.
It provides **unparalleled insights, in real-time**, of everything happening on the
system it runs (including applications such as web and database servers), using
**modern interactive web dashboards**.

_netdata is **fast** and **efficient**, designed to permanently run on all systems
(**physical** & **virtual** servers, **containers**, **IoT** devices), without
disrupting their core function._

**Shipped version:** 1.30.0

**Customization brought by the package:**

* Netdata Cloud functionality disabled
* grant MySQL statistics access via a `netdata` user
* nginx root log statistics via putting `netdata` user in the `adm` group
* Dovecot statistics via giving access to Dovecot stats stocket to `netdata` user (works only with Dovecot 2.2.16+)

**Further recommendations:**
We don't allow YunoHost packages to make changes to sensitive system files. So here are further customizations you can make to allow more monitoring:

* Nginx:
  * requests/connections: follow [these recommandations](https://github.com/firehol/netdata/tree/master/python.d#nginx) to enable `/stab_status`; for example by putting this `location` section in `/etc/nginx/conf.d/yunohost_admin.conf`:
```
location /stub_status {
  stub_status on;
  # Security: Only allow access from the IP below.
  allow 127.0.0.1;
  # Deny anyone else
  deny all;
 }
```
  * weblogs: you can monitor all your nginx weblogs for errors; follow [these recommendations](https://github.com/firehol/netdata/tree/master/python.d#nginx_log)
* phpfpm: follow [these recommandations](https://github.com/firehol/netdata/tree/master/python.d#phpfpm)
* NetData registry: set the instance as its own NetData registry server to avoid leaking any information by default

It has been tested on x86_64 and ARM.

## How it works

Netdata is a highly efficient, highly modular, metrics management engine. Its lockless design makes it ideal for
concurrent operations on the metrics.

![image](https://user-images.githubusercontent.com/2662304/48323827-b4c17580-e636-11e8-842c-0ee72fcb4115.png)

This is how it works:

| Function    | Description                                                                                                                                                                                                                                                    | Documentation                                       |
| :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------- |
| **Collect** | Multiple independent data collection workers are collecting metrics from their sources using the optimal protocol for each application and push the metrics to the database. Each data collection worker has lockless write access to the metrics it collects. | [`collectors`](/collectors/README.md)                |
| **Store**   | Metrics are first stored in RAM in a custom database engine that then "spills" historical metrics to disk for efficient long-term metrics storage.                                                                                                             | [`database`](/database/README.md)                    |
| **Check**   | A lockless independent watchdog is evaluating **health checks** on the collected metrics, triggers alarms, maintains a health transaction log and dispatches alarm notifications.                                                                              | [`health`](/health/README.md)                        |
| **Stream**  | A lockless independent worker is streaming metrics, in full detail and in real-time, to remote Netdata servers, as soon as they are collected.                                                                                                                 | [`streaming`](/streaming/README.md)                  |
| **Archive** | A lockless independent worker is down-sampling the metrics and pushes them to **backend** time-series databases.                                                                                                                                               | [`backends`](/backends/README.md)                    |
| **Query**   | Multiple independent workers are attached to the [internal web server](/web/server/README.md), servicing API requests, including [data queries](/web/api/queries/README.md).                                                                                     | [`web/api`](/web/api/README.md)                      |

The result is a highly efficient, low-latency system, supporting multiple readers and one writer on each metric.

## Infographic

This is a high level overview of Netdata feature set and architecture. Click it to to interact with it (it has direct
links to our documentation).

[![image](https://user-images.githubusercontent.com/43294513/60951037-8ba5d180-a2f8-11e9-906e-e27356f168bc.png)](https://my-netdata.io/infographic.html)

## Features

![finger-video](https://user-images.githubusercontent.com/2662304/48346998-96cf3180-e685-11e8-9f4e-059d23aa3aa5.gif)

This is what you should expect from Netdata:

### General

-   **1s granularity** - The highest possible resolution for all metrics.
-   **Unlimited metrics** - Netdata collects all the available metrics—the more, the better.
-   **1% CPU utilization of a single core** - It's unbelievably optimized.
-   **A few MB of RAM** - The highly-efficient database engine stores per-second metrics in RAM and then "spills"
    historical metrics to disk long-term storage.   
-   **Minimal disk I/O** - While running, Netdata only writes historical metrics and reads `error` and `access` logs.
-   **Zero configuration** - Netdata auto-detects everything, and can collect up to 10,000 metrics per server out of the
    box.
-   **Zero maintenance** - You just run it. Netdata does the rest.
-   **Zero dependencies** - Netdata runs a custom web server for its static web files and its web API (though its
    plugins may require additional libraries, depending on the applications monitored).
-   **Scales to infinity** - You can install it on all your servers, containers, VMs, and IoT devices. Metrics are not
    centralized by default, so there is no limit.
-   **Several operating modes** - Autonomous host monitoring (the default), headless data collector, forwarding proxy,
    store and forward proxy, central multi-host monitoring, in all possible configurations. Each node may have different
    metrics retention policies and run with or without health monitoring.

### Health Monitoring & Alarms

-   **Sophisticated alerting** - Netdata comes with hundreds of alarms **out of the box**! It supports dynamic
    thresholds, hysteresis, alarm templates, multiple role-based notification methods, and more.
-   **Notifications**: [alerta.io](/health/notifications/alerta/), [amazon sns](/health/notifications/awssns/),
    [discordapp.com](/health/notifications/discord/), [email](/health/notifications/email/),
    [flock.com](/health/notifications/flock/), [hangouts](/health/notifications/hangouts/),
    [irc](/health/notifications/irc/), [kavenegar.com](/health/notifications/kavenegar/),
    [messagebird.com](/health/notifications/messagebird/), [pagerduty.com](/health/notifications/pagerduty/),
    [prowl](health/notifications/prowl/), [pushbullet.com](/health/notifications/pushbullet/),
    [pushover.net](health/notifications/pushover/), [rocket.chat](/health/notifications/rocketchat/),
    [slack.com](/health/notifications/slack/), [smstools3](/health/notifications/smstools3/),
    [syslog](/health/notifications/syslog/), [telegram.org](/health/notifications/telegram/),
    [twilio.com](/health/notifications/twilio/), [web](/health/notifications/web/) and [custom
    notifications](/health/notifications/custom/).

### Integrations

-   **Time-series databases** - Netdata can archive its metrics to **Graphite**, **OpenTSDB**, **Prometheus**, **AWS
    Kinesis**, **MongoDB**, **JSON document DBs**, in the same or lower resolution (lower: to prevent it from congesting
    these servers due to the amount of data collected). Netdata also supports **Prometheus remote write API**, which
    allows storing metrics to **Elasticsearch**, **Gnocchi**, **InfluxDB**, **Kafka**, **PostgreSQL/TimescaleDB**,
    **Splunk**, **VictoriaMetrics** and a lot of other [storage
    providers](https://prometheus.io/docs/operating/integrations/#remote-endpoints-and-storage).

## Visualization

-   **Stunning interactive dashboards** - Our dashboard is mouse-, touchpad-, and touch-screen friendly in 2 themes:
    `slate` (dark) and `white`.
-   **Amazingly fast visualization** - Even on low-end hardware, the dashboard responds to all queries in less than 1 ms
    per metric.
-   **Visual anomaly detection** - Our UI/UX emphasizes the relationships between charts so you can better detect
    anomalies visually.
-   **Embeddable** - Charts can be embedded on your web pages, wikis and blogs. You can even use [Atlassian's Confluence
    as a monitoring dashboard](/web/gui/confluence/README.md).
-   **Customizable** - You can build custom dashboards using simple HTML. No JavaScript needed!

### Positive and negative values

To improve clarity on charts, Netdata dashboards present **positive** values for metrics representing `read`, `input`,
`inbound`, `received` and **negative** values for metrics representing `write`, `output`, `outbound`, `sent`.

![positive-and-negative-values](https://user-images.githubusercontent.com/2662304/48309090-7c5c6180-e57a-11e8-8e03-3a7538c14223.gif)

_Netdata charts showing the bandwidth and packets of a network interface. `received` is positive and `sent` is
negative._

### Autoscaled y-axis

Netdata charts automatically zoom vertically, to visualize the variation of each metric within the visible time-frame.

![non-zero-based](https://user-images.githubusercontent.com/2662304/48309139-3d2f1000-e57c-11e8-9a44-b91758134b00.gif)

_A zero-based `stacked` chart, automatically switches to an auto-scaled `area` chart when a single dimension is
selected._

### Charts are synchronized

Charts on Netdata dashboards are synchronized to each other. There is no master chart. Any chart can be panned or zoomed
at any time, and all other charts will follow.

![charts-are-synchronized](https://user-images.githubusercontent.com/2662304/48309003-b4fb3b80-e578-11e8-86f6-f505c7059c15.gif)

_Charts are panned by dragging them with the mouse. Charts can be zoomed in/out with`SHIFT` + `mouse wheel` while the
mouse pointer is over a chart._

> The visible time-frame (pan and zoom) is propagated from Netdata server to Netdata server when navigating via the
> [My nodes menu](/registry/README.md).

### Highlighted time-frame

To improve visual anomaly detection across charts, the user can highlight a time-frame (by pressing `Alt` + `mouse
selection`) on all charts.

![highlighted-timeframe](https://user-images.githubusercontent.com/2662304/48311876-f9093300-e5ae-11e8-9c74-e3e291741990.gif)

_A highlighted time-frame can be given by pressing `Alt` + `mouse selection` on any chart. Netdata will highlight the
same range on all charts._

> Highlighted ranges are propagated from Netdata server to Netdata server, when navigating via the [My nodes
> menu](/registry/README.md).

## What Netdata monitors

Netdata can collect metrics from 200+ popular services and applications, on top of dozens of system-related metrics
jocs, such as CPU, memory, disks, filesystems, networking, and more. We call these **collectors**, and they're managed
by [**plugins**](/collectors/plugins.d/README.md), which support a variety of programming languages, including Go and
Python.

Popular collectors include **Nginx**, **Apache**, **MySQL**, **statsd**, **cgroups** (containers, Docker, Kubernetes,
LXC, and more), **Traefik**, **web server `access.log` files**, and much more. 

See the **full list of [supported collectors](/collectors/COLLECTORS.md)**.

Netdata's data collection is **extensible**, which means you can monitor anything you can get a metric for. You can even
write a collector for your custom application using our [plugin API](/collectors/plugins.d/README.md).

## Links

 * Report a bug: https://github.com/YunoHost-Apps/netdata_ynh/issues
 * NetData website: http://my-netdata.io/
 * YunoHost website: https://yunohost.org/
