title: 迁移业余项目数据到 Azure MySQL
tags:
- Azure
- MySQL
---

创建了 `Azure MySQL` 实例后（这里使用的是 `Flexible Server`），首先导出原始数据库的数据，因为用的是 `Docker` 所以通过以下方式导出：

```
docker exec ${container_id} /usr/bin/mysqldump -u ${user_name} --password=${password} ${database_name} > backup.sql
```

然后通过 `Azure CLI` 创建一个新的数据库：

```
az mysql flexible-server db create --resource-group ${resource_group} --server-name ${server_name} --database-name ${database_name}
```

最后通过 `Azure CLI` 导入数据：

```
az mysql flexible-server execute -n ${server_name} -u ${user_name} -p ${password} -d ${database_name} -f ${path_to_backup_sql_file}
```

参考：

- [Sample Databases for Azure Database for MySQL flexible server](https://github.com/Azure-samples/mysql-database-samples)
- [az mysql flexible-server](https://docs.microsoft.com/en-us/cli/azure/mysql/flexible-server?view=azure-cli-latest)
