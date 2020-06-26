title: 如何使用 rsync 同步数据到其他服务器
tags:
- rsync
---

假设我们希望将 `server1` 下的 `/data1` 目录中的数据同步到 `server2` 下的 `/data2` 目录，首先需要建立 `server1` 和 `server2` 的免密登陆，在 `server1` 上执行 `ssh-keygen`，默认情况会在 `~/.ssh` 目录下生成 `id_rsa` 和 `id_rsa.pub` 两个文件，然后将 `~/.ssh/id_rsa.pub` 文件的内容复制到 `server2` 的 `~/.ssh/authorized_keys` 文件中即可。

接着，就可以使用 `rsync` 进行数据同步，具体命令为 `rsync -az --delete /data1/ server2-user@server2-ip:/data2`，其中 `-a` 表示递归同步 `/data1` 下的子文件夹及保留文件的权限、组、软连接等信息，如果不需要这些额外的文件信息而只想要递归同步可以使用 `-r` 来代替 `-a`；`-z` 表示开启文件压缩来减少网络传输；`--delete` 表示在 `/data1` 中删除的文件在 `/data2` 中也会同步删除。最后需要注意命令中 `/data1/` 末尾的 `/`，加了 `/` 表示将 `/data1` 下的所有文件同步到 `/data2`，没有 `/` 则表示将 `/data1` 这个文件夹同步到 `/data2` 下，假设 `/data1` 下有 `a`、`b`、`c` 三个文件，两种写法最后的同步区别为：

* `/data1/`：`/data2/a,b,c`
* `/data1`：`/data2/data1/a,b,c`

最后，我们需要将 `rsync` 加入到定时任务中进行自动备份。执行 `crontab -e`，将定时任务添加到文件中，如每小时执行一次：`0 * * * * rsync -az --delete /data1/ server2-user@server2-ip:/data2`。

参考：

- [How To Use Rsync to Sync Local and Remote Directories on a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps)
- [How to Use rsync to Backup Your Data on Linux](https://www.howtogeek.com/135533/how-to-use-rsync-to-backup-your-data-on-linux/)
