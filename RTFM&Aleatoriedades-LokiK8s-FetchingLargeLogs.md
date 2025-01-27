---

title: "RTFM & Aleatoriedades - Loki Fetching Large Logs"
date: 2024-11-24T20:40:44-03:00
tags: ["Linux", "Loki", "SRE", "DevOps", "Observability", "Grafana", "K8s"]
author: "Faioli"
toc: true
hideSummary: false
comments: true
showComments: true
showDate: true
showTitle: true
showShare: true
disableShare: false
norss: false
nosearch: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

There are several reasons why Grafana may not be configured to break the 5000 line limit when exploring logs... It is also known that we can export in several other ways. But here I will focus on solving the need to download an unexpectedly large amount of logs...

And for that we will make use of LogCLI: the command line interface for Grafana Loki that makes it easy to run [LogQL](https://grafana.com/docs/loki/latest/query/) queries on a Loki instance.

### Download binary

- Download the `logcli` binary from the [Loki releases page](https://github.com/grafana/loki/releases).
  - [loki-3.3.0.x86_64.rpm](https://github.com/grafana/loki/releases/download/v3.3.0/loki-3.3.0.x86_64.rpm)
  - [logcli_3.3.0_arm64.deb](https://github.com/grafana/loki/releases/download/v3.3.0/logcli_3.3.0_arm64.deb)

### Install

``` bash
sudo dnf isntall https://github.com/grafana/loki/releases/download/v3.3.0/logcli-3.3.0.x86_64.rpm
```

### Bash completion

add this to your `~/.bashrc.d` file

``` bash
cat << EoF >> ~/.bashrc.d/var_loki
# Set up command completion
eval "$(logcli --completion-script-bash)"
EoF
```

If you have questions about the [query](https://grafana.com/docs/loki/latest/query/) that needs to be made, access the Explorer in Grafana

1.  Go to `Grafana` \> `Explore`
    1.  Apply the necessary `label` to filter logs ...

![Loki](img/Loki/lokifetching.png)

In this example, I'm looking for the last 7 days of logs from the NGINX Ingress which is processing approximately 8.8 GiB.

``` bash
{app="ingress-nginx"} |= ``
```

Then, to allow local access, forward a local port to the Loki pod

``` bash
kubectl --namespace loki port-forward svc/loki-gateway 8080:80
```

Configure the LogCli environment variable

``` bash
export LOKI_ADDR=http://localhost:8080
```

And using the query we checked, we can finally extract the log to a local file

``` bahs
$ logcli query '{app="ingress-nginx"} |= ``' --limit=9000000 --since=168h  -o default,raw ou jsonl  > /tmp/log_ingres-nginx.log
```

- `--limit` here I try to set a very high value to ensure that all logs from the 7 days are captured.
- `--since` I set 168h time range.


