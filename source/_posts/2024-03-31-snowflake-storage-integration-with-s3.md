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

使用 `ACCOUNTADMIN` 角色在 `Snowflake` 中执行：
```sql
USE ROLE ACCOUNTADMIN;
SELECT SYSTEM$GET_SNOWFLAKE_PLATFORM_INFO();
```

记录下返回的 `VPC ID`：
```sql
{"snowflake-vpc-id":["vpc-abc"]}
```

## 参考
* [Allowing the Virtual Private Cloud IDs](https://docs.snowflake.com/en/user-guide/data-load-s3-allow)