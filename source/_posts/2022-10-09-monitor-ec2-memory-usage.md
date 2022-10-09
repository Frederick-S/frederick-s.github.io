title: AWS EC2 监控内存使用
tags:
- AWS
---

`AWS` `EC2` 的监控页面默认没有显示内存使用率，需要搭配 `CloudWatch` 配置使用。

由于需要在 `EC2` 上安装 `CloudWatch agent` 来上报监控数据到 `CloudWatch`，所以需要先为 `EC2` 配置 `IAM` 角色来授予需要的权限。创建 `IAM` 角色时，在第一步的 `Trusted entity type` 选择 `AWS service`，`Use case` 选择 `EC2`；在第二步的 `Permissions policies` 添加 `CloudWatchFullAccess`。

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

## 参考
* [How to monitor memory usage on AWS EC2 ??](https://lepczynski.it/en/aws_en/how-to-monitor-memory-usage-on-aws-ec2/)