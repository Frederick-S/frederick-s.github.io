title: AWS EC2 监控内存使用
tags:
- AWS
---

`AWS` `EC2` 的监控页面默认没有显示内存使用率，需要搭配 `CloudWatch` 配置使用。

由于需要在 `EC2` 上安装 `CloudWatch agent` 来上报监控数据到 `CloudWatch`，所以需要先为 `EC2` 配置 `IAM` 角色来授予需要的权限。创建 `IAM` 角色时，在第一步的 `Trusted entity type` 选择 `AWS service`，`Use case` 选择 `EC2`；在第二步的 `Permissions policies` 添加 `CloudWatchAgentServerPolicy` 即可。更多细节可参考 [Create IAM roles and users for use with CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent-commandline.html)。

接着，在 [Download and configure the CloudWatch agent using the command line](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html) 中根据实际 `EC2` 的操作系统下载和安装 `CloudWatch agent`，这里以 `ARM64` 的 `Ubuntu` 系统为例：

```sh
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/arm64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

然后，为 `CloudWatch agent` 创建一个配置文件，例如 `cloudwatch.json`，写入如下内容：

```
{
   "metrics":{
      "metrics_collected":{
         "mem":{
            "measurement":[
               "mem_used_percent"
            ],
            "metrics_collection_interval":60
         }
      },
      "append_dimensions": {
        "InstanceId": "${aws:InstanceId}"
      }
   }
}
```

这表示每隔60秒收集一次内存使用率，接着启动 `CloudWatch agent`：

```sh
sudo amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:$HOME/cloudwatch.json -s
```

可以通过 `amazon-cloudwatch-agent-ctl -a status` 来查看 `CloudWatch agent` 的状态：

```
{
  "status": "running",
  "starttime": "2022-10-09T13:23:11+00:00",
  "configstatus": "configured",
  "cwoc_status": "stopped",
  "cwoc_starttime": "",
  "cwoc_configstatus": "not configured",
  "version": "1.247355.0b252062"
}
```

此时 `CloudWatch agent` 的状态为运行中。

如果一切正常，那么在 `AWS` 控制台中 `CloudWatch` 的 `All metrics` 下会多出一项 `CWAgent`（如果原来没有添加过的话）：

![alt](/images/cloudwatch-1.png)

点击进入后选择相应的 `EC2`，点击 `Add to graph`：

![alt](/images/cloudwatch-2.png)

在当前页面上方就会显示对应的内存使用率的监控：

![alt](/images/cloudwatch-3.png)

之后也可以创建一个 `Dashboard`，将这个监控加入到自定义的 `Dashboard` 中。

如果在 `AWS` 控制台没有看到 `CWAgent` 项目，那么可以查看 `EC2` 上 `CloudWatch agent` 的日志是否有异常，日志保存在 `/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log`。例如，如果忘记为 `EC2` 配置 `IAM` 角色，同时 `EC2` 上又没有其他的权限访问信息，`CloudWatch agent` 就无法上报监控数据，会提示如下类似的异常：

```
2022-10-09T13:27:36Z E! WriteToCloudWatch failure, err:  NoCredentialProviders: no valid providers in chain
caused by: EnvAccessKeyNotFound: failed to find credentials in the environment.
SharedCredsLoad: failed to load profile, .
EC2RoleRequestError: no EC2 instance role found
caused by: EC2MetadataError: failed to make EC2Metadata request
```

最后，本文只监控了内存使用率，如果想要添加更多的监控，可以参考 [Metrics collected by the CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html) 添加相应的指标。

## 参考
* [How to monitor memory usage on AWS EC2 ??](https://lepczynski.it/en/aws_en/how-to-monitor-memory-usage-on-aws-ec2/)
* [Download and configure the CloudWatch agent using the command line](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html)
* [Metrics collected by the CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html)
* [Create IAM roles and users for use with CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent-commandline.html)