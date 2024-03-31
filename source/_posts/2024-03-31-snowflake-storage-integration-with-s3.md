title: 'Snowflake 配置 S3 Storage Integration'
tags:
- Snowflake
---

## 介绍
`Snowflake` 的 `Data Loading` 和 `Data Unloading` 可以通过 `S3` 导入和导出数据。用户可以使用 `AWS_KEY_ID` 和 `AWS_SECRET_KEY` 来授权 `Snowflake` 访问 `S3`，不过出于安全和权限控制的考虑，一般不会这么做。

`Snowflake` 建议通过 `Storage Integration` 来管理权限。

## 获取 VPC ID
在配置 `Storage Integration` 前，需要设置 `S3` 策略。首先获取 `Snowflake` 的 `VPC ID`，后续的 `S3` 策略配置中将只允许该 `VPC` 访问。

> 允许特定 VPC 访问的功能要求 Snowflake 实例和对应的 S3 Bucket 运行在相同的 AWS 区域内。

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

下面的策略中 `vpc-abc` 是 `Snowflake` 实例的 `VPC`，`snowflake-storage-integration-example` 是示例 `Bucket` 的名字，`unloading` 和 `loading` 是该 `Bucket` 下的两个文件夹，分别用于 `Data Unloading` 和 `Data Loading` 使用：

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

## 创建 IAM 角色
接着，创建一个 `IAM` 角色并绑定前一步创建的 `S3` 策略。在 `IAM` 左侧导航栏 `Access management` 下选择 `Roles`，之后点击 `Create role`：

![alt](/images/snowflake-5.png)

`Trusted entity type` 选择 `AWS account`，然后在 `An AWS account` 下选择 `Another AWS account`，`Account ID` 暂时先填当前账号的 `ID`，之后会修改：

![alt](/images/snowflake-6.png)

同时，选择 `Require external ID (Best practice when a third party will assume this role)`，`External ID` 暂时用一个假的例如 `0000` 替代，之后同样会修改：

![alt](/images/snowflake-7.png)

最后绑定先前创建的 `S3` 策略：

![alt](/images/snowflake-8.png)

创建角色之后，记录下角色的 `ARN`，接下来会用到：

![alt](/images/snowflake-9.png)

## 创建 Storage Integration
这时就可以在 `Snowflake` 中创建 `Storage Integration` 了：

```sql
CREATE STORAGE INTEGRATION snowflake_storage_integration_example
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123:role/snowflake-integration-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://snowflake-storage-integration-example/loading/', 's3://snowflake-storage-integration-example/unloading/')
```

其中 `STORAGE_AWS_ROLE_ARN` 是之前所创建的 `IAM` 角色的 `ARN`，`STORAGE_ALLOWED_LOCATIONS` 是示例 `Bucket` 下的两个文件夹的地址。

> 只有授权了 `CREATE INTEGRATION` 权限的角色才能创建 `STORAGE INTEGRATION`，默认只有 `ACCOUNTADMIN` 才有这个权限。

## 获取 Snowflake 的用户 ARN 和 External ID
接着需要获取所创建的 `Storage Integration` 对应的 `Snowflake` `IAM` 用户的 `ARN` 和 `External ID`：

```sql
desc integration snowflake_storage_integration_example;
```

![alt](/images/snowflake-10.png)

记录下 `STORAGE_AWS_IAM_USER_ARN` 和 `STORAGE_AWS_EXTERNAL_ID`。

## 授权 Snowflake 用户
回到之前创建的 `IAM` 角色，在 `Trust relationships` 下替换掉之前填写的临时 `Account ID` 和 `External ID`：

![alt](/images/snowflake-11.png)

![alt](/images/snowflake-12.png)

完成后，我们就可以执行一条 `Data Unloading` 命令来验证配置是否成功：

```sql
copy into 's3://snowflake-storage-integration-example/unloading/'
from (select OBJECT_CONSTRUCT_KEEP_NULL(*) from (select * from MY_DATABASE.MY_SCHEMA.MY_TABLE limit 10))
FILE_FORMAT = (type = json, COMPRESSION = NONE)
STORAGE_INTEGRATION = snowflake_storage_integration_example
```

如果配置成功，那么 `Snowflake` 会将表 `MY_DATABASE.MY_SCHEMA.MY_TABLE` 的数据导出到 `s3://snowflake-storage-integration-example/unloading/` 文件夹下。

## 参考
* [Allowing the Virtual Private Cloud IDs](https://docs.snowflake.com/en/user-guide/data-load-s3-allow)
* [Option 1: Configuring a Snowflake storage integration to access Amazon S3](https://docs.snowflake.com/en/user-guide/data-load-s3-config-storage-integration)
* [CREATE STORAGE INTEGRATION](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)