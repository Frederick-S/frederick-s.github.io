title: 'Snowflake 配置 S3 Storage Integration'
tags:
- Snowflake
---

## 介绍
`Snowflake` 的 `Data Loading` 和 `Data Unloading` 可以通过 `S3` 导入和导出数据。用户可以使用 `AWS_KEY_ID` 和 `AWS_SECRET_KEY` 来授权 `Snowflake` 访问 `S3`，不过出于安全和权限控制的考虑，一般不会这么做。

`Snowflake` 建议通过 `Storage Integration` 来管理授权。

## 获取 VPC ID
在配置 `Storage Integration` 前，需要设置 `S3` 策略，首先获取 `Snowflake` 的 `VPC ID`，后续的 `S3` 策略配置中将只允许该 `VPC` 访问。

> 注意，允许特定 VPC 访问的功能要求 Snowflake 实例和对应的 S3 Bucket 运行在相同的 AWS 区域内。

切换到 `ACCOUNTADMIN` 角色在 `Snowflake` 中执行：
```sql
USE ROLE ACCOUNTADMIN;
SELECT SYSTEM$GET_SNOWFLAKE_PLATFORM_INFO();
```

记录下返回的 `VPC ID`：
```sql
{"snowflake-vpc-id":["vpc-abc"]}
```

## 创建 IAM 策略
然后，需要创建一个 `S3` 策略来定义 `Snowflake` 访问 `S3 Bucket` 的权限。

从 `AWS` 控制台进入 `IAM`，在左侧导航栏 `Access management` 下选择 `Account settings`：

![alt](/images/snowflake-1.png)

在 `Security Token Service (STS)` 下查看所在区域的 `STS` 状态是否是 `Active`：

![alt](/images/snowflake-2.png)

接着，在左侧导航栏 `Access management` 下选择 `Policies`，之后点击 `Create policy`：

![alt](/images/snowflake-3.png)

切换到 `JSON` 后输入 `S3` 策略：

![alt](/images/snowflake-4.png)

下面的策略中 `vpc-abc` 是 `Snowflake` 实例的 `VPC`，`snowflake-storage-integration-example` 是示例 `Bucket` 的名字，`unloading` 和 `loading` 是该 `Bucket` 下的两个文件夹：

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
				"s3:PutObject",
				"s3:GetObject",
				"s3:GetObjectVersion",
				"s3:DeleteObject",
				"s3:DeleteObjectVersion"
			],
			"Resource": [
				"arn:aws:s3:::snowflake-storage-integration-example/unloading/*",
				"arn:aws:s3:::snowflake-storage-integration-example/loading/*"
			],
			"Condition": {
				"StringEquals": {
					"aws:SourceVpc": "vpc-abc"
				}
			}
		},
		{
			"Sid": "Statement2",
			"Effect": "Allow",
			"Action": [
				"s3:ListBucket",
				"s3:GetBucketLocation"
			],
			"Resource": [
				"arn:aws:s3:::snowflake-storage-integration-example"
			],
			"Condition": {
				"StringLike": {
					"s3:prefix": [
						"unloading/*",
						"loading/*"
					]
				},
				"StringEquals": {
					"aws:SourceVpc": "vpc-abc"
				}
			}
		}
	]
}
```

## 参考
* [Allowing the Virtual Private Cloud IDs](https://docs.snowflake.com/en/user-guide/data-load-s3-allow)
* [Option 1: Configuring a Snowflake storage integration to access Amazon S3](https://docs.snowflake.com/en/user-guide/data-load-s3-config-storage-integration)