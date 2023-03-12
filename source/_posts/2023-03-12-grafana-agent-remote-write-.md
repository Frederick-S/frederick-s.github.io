title: 通过 Grafana Agent 上传 Prometheus 指标数据到 Grafana Cloud
tags:
- Grafana
- Prometheus
---

## 介绍
`Grafana Cloud` 为免费账户提供了一万条指标的存储额度，对于业余项目来说可以考虑将指标上传到由 `Grafana Cloud` 托管的 `Prometheus` 中。

## 安装 Grafana Agent
`Prometheus` 指标数据的上传需要通过 `Grafana Agent` 来完成，以下安装步骤以 `Ubuntu` 为例：

```sh
mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana-agent
```

然后通过 `sudo systemctl start grafana-agent` 将其启动，并可通过 `sudo systemctl status grafana-agent` 显示 `grafana-agent` 的当前状态：

```
● grafana-agent.service - Monitoring system and forwarder
     Loaded: loaded (/lib/systemd/system/grafana-agent.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-03-12 05:08:43 UTC; 13s ago
       Docs: https://grafana.com/docs/agent/latest/
   Main PID: 1049084 (grafana-agent)
      Tasks: 7 (limit: 1041)
     Memory: 125.6M
     CGroup: /system.slice/grafana-agent.service
             └─1049084 /usr/bin/grafana-agent --config.file /etc/grafana-agent.yaml -server.http.address=127.0.0.1:9090 -server.grpc.address=127.0.0.1:9091

Mar 12 05:08:43 example-name systemd[1]: Started Monitoring system and forwarder.
```

同时，如果希望系统重启后自动启动 `grafana-agent` 服务，可以执行如下的命令：

```sh
sudo systemctl enable grafana-agent.service
```

## 参考
* [Install Grafana Agent on Linux](https://grafana.com/docs/agent/latest/set-up/install-agent-linux/)