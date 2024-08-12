---
title: 在 Linux 上给用户赋予指定目录的读写权限
urlname: 2019-09-29-linux-acl-permission
author: 章鱼猫先生
date: 2019-09-29
updated: "2021-06-25 11:02:02"
---

在 Linux 上指定目录的读写权限赋予用户，有两种方法可以实现这个目标：第一种是使用 ACL (访问控制列表)，第二种是创建用户组来管理文件权限，下面会一一介绍。为了完成这个教程，我们将使用以下设置：

- 操作系统：CentOS 7
- 测试目录：/data/share
- 测试用户：shenweiyan
- 文件系统类型：ext4

请确认所有的命令都是使用 root 用户执行的，或者使用 sudo 命令来享受与之同样的权限。让我们开始吧！下面，先使用 mkdir 命令来创建一个名为 share 的目录。

```bash
$ mkdir -p /data/share
```

## 1. 使用 ACL 来为用户赋予目录的读写权限

**重要提示：** 打算使用此方法的话，您需要确认您的 Linux 文件系统类型（如 ext3 和 ext4, NTFS, BTRFS）支持 ACL。

**1.1. 首先， 依照以下命令在您的系统中检查当前文件系统类型，并且查看内核是否支持 ACL：**

```bash
$ df -T | awk '{print $1,$2,$NF}' | grep "^/dev"
$ grep -i acl /boot/config*
```

从下方的截屏可以看到，文件系统类型是 ext4，并且从 CONFIG_EXT4_FS_POSIX_ACL=y 选项可以发现内核是支持 POSIX ACL 的。
![](https://shub.weiyan.tech/yuque/elog-cookbook-img/FmYmZeCp5xG3nJs8Z7LUaA-QzME7.png)

**1.2. 查看文件系统（分区）挂载时是否使用了 ACL 选项。**

```bash
$ tune2fs -l /dev/vda1 | grep acl
```

![](https://shub.weiyan.tech/yuque/elog-cookbook-img/FjbOrbZJgdXuVum5OsrQITWOjnn1.png)
通过上边的输出可以发现，默认的挂载项目中已经对 ACL 进行了支持。如果发现结果不如所愿，你可以通过以下命令对指定分区（此例中使用 /dev/sda3）开启 ACL 的支持）。

```bash
$ mount -o remount,acl /
$ tune2fs -o acl /dev/sda3
```

**1.3. 指定目录 share 的读写权限分配给名为 shenweiyan 的用户了，依照以下命令执行即可。**

```bash
# 检查目录默认的 ACL 设置（Check the default ACL settings for the directory）
$ getfacl /data/share

# 指定用户读写权限（Give rw access to user shenweiyan）
# 对于目录必须增加 x (执行)权限, 否则无法进入目录
$ setfacl -m user:shenweiyan:rwx /data/share

# 再次检查目录 ACL 设置（Check new ACL settings for the directory）
$ getfacl /data/share
```

![](https://shub.weiyan.tech/yuque/elog-cookbook-img/FkspVGfvw4EAuZvACndk2GaDBnA6.png)
在上方的截屏中，通过输出结果的第二行 getfacl 命令可以发现，用户 shenweiyan 已经成功的被赋予了 /data/share 目录的读写权限。

如果想要获取 ACL 列表的更多信息。请参考：

- [如何使用访问控制列表（ACL）为用户/组设置磁盘配额](http://www.tecmint.com/set-access-control-lists-acls-and-disk-quotas-for-users-groups/)
- [如何使用访问控制列表（ACL）挂载网络共享](http://www.tecmint.com/rhcsa-exam-configure-acls-and-mount-nfs-samba-shares/)

## 2. 使用用户组来为用户赋予指定目录的读写权限

**2.1. 如果用户已经拥有了默认的用户组（通常组名与用户名相同），就可以简单的通过变更文件夹的所属用户组来完成。**

```python
$ chgrp shenweiyan /data/share
```

另外，我们也可以通过以下方法为多个用户（需要赋予指定目录读写权限的）新建一个用户组。如此一来，也就创建了一个共享目录。

```python
$ groupadd dbshare
```

**2.2. 接下来将用户 shenweiyan 添加到 dbshare 组中：**

```python
# add user to projects
$ usermod -aG dbshare shenweiyan

# check users groups
$ groups tecmint
```

**2.3. 将目录的所属用户组变更为 dbshare：**

```python
$ chgrp projects /data/share
```

**2.4. 现在，给组成员设置读写权限。**

```bash
$ chmod -R 0760 /data/share

# check new permissions
$ ls -l /data/share
```

ok，在 Linux 上给用户赋予指定目录的读写权限就介绍到这里 ！

## 参考资料：

- 高延斌，《[Linux ACL 体验](https://www.ibm.com/developerworks/cn/linux/l-acl/index.html)》，IBM Developer
- Mr-Ping，《在 Linux 上给用户赋予指定目录的读写权限》，Linux 中国

<script src="https://giscus.app/client.js"
        data-repo="shenweiyan/Knowledge-Garden"
        data-repo-id="R_kgDOKgxWlg"
        data-mapping="number"
        data-term="9"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="light"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
