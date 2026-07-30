---
title: "Linux davfs2 搭载 Webdav 协议网盘"
description: "Linux课程大作业的一点记录"
publishDate: 2023-12-05
tags: ["linux"]
---

## 服务器配置

| CPU | 规格 | OS镜像 | 系统盘 |
| :---: | :---: | :---: | :---: |
| 鲲鹏计算 | 鲲鹏通用计算增强型：kc1.large.2：2vCPUs：4GB | openEuler 20.03 64bit with ARM(40GB) | 通用型SSD：40GB |

# 基本步骤

:::tip
尝试中发现，后续进行编译安装时，须使用 **root** 用户权限，因此建议一开始就在 **root** 用户的工作目录下进行
:::

:::note
**思路**

- 因为作业要求是封装挂载过程开放给用户，那么用户的挂载点应当在自己的工作目录下，而不需要统一放在 `/mnt` 这样的系统根目录；
- 因此，只需封装出一个具有sudo权限的脚本，使得所有用户都具有不可读写、只可执行且无需密码的权限，
  1. 创建本地挂载点
  2. 更新 secrets 文件
  3. 配置执行挂载命令
- 之后将该脚本通过 `visudo` 命令为所有用户设置无密码执行的权限，之后通过C语言代码执行。
- sh脚本和其调用的C语言程序都含有 `sudo` 命令，在配置 visudo 后依旧存在权限问题。
- 无论是C程序的编译文件还是shell脚本，在执行时都须加上 `sudo`
:::

## 环境安装与配置

> 以下步骤均以root用户进行

### (1) 安装neon

- 安装相关依赖libxml2：

  ```bash
  yum install libxml2 libxml2-devel -y --nogpgcheck
  ```

  :::caution
  报错 `Error: GPG check FAILED`，表示GPG检查不通过，在命令后添加 `--nogpgcheck` 规避检查
  :::

- 安装neon

  ```bash
  yum install neon neon-devel -y --nogpgcheck
  ```

### (2) 安装davfs2

- 下载安装包

  ```bash
  wget http://download.savannah.nongnu.org/releases/davfs2/davfs2-1.7.0.tar.gz
  ```

- 解压缩并安装

  ```bash
  tar -zxvf davfs2-1.7.0.tar.gz

  cd davfs2-1.7.0

  ./configure CFLAGS='-fPIC' --prefix=/root/davfs_install # 指定安装路径

  make
  make install
  ```

  > 这个步骤卡我好久，就是要在 `configure` 的时候指定参数 `-fPIC`

- 测试安装

  ```bash
  export PATH=$PATH:/root/davfs_install/sbin/
  mount.davfs2 --version
  ```

  :::note
  davfs2 的路径为 `/usr/local/etc/davfs2`；（该信息是文档看到的，文档在 `/root/davfs_install/share/doc`）
  :::

- 配置davfs2模块，编辑 `/usr/local/etc/davfs2/davfs2.conf` 文件，添加或更改以下行：

  ```bash
  use_locks   0
  ignore_dav_header 1
  ```

### (3) 测试网盘挂载

- 创建davfs2用户

  ```bash
  useradd davfs2
  # 【作业的低语】以下操作都没必要，只要将sudo操作封装进一个执行脚本，使用户不可读写只可无密码执行即可

  # 添加root入davfs2用户组
  usermod -aG davfs2 root
  # 设置权限，使root可修改、davfs2用户组可读可执行
  chmod 750 /usr/local/sbin/mount.davfs /usr/local/sbin/umount.davfs
  # 设置davfs2为davfs2包的文件所有者
  # chown -R davfs2:davfs2 /usr/local/etc/davfs2 # -R同步处理文件夹下所有文件
  # 配置文件权限，使davfs用户组可读可执行但不可写
  chmod -R 750 /usr/local/etc/davfs2
  # 授权davfs2用户组能够执行挂载、卸载命令
  sudo chown root:davfs2 /usr/local/sbin/mount.davfs /usr/local/sbin/umount.davfs
  sudo chmod 750 /usr/local/sbin/mount.davfs /usr/local/sbin/umount.davfs
  ```

- 编辑 `/usr/local/etc/davfs2/secrets` 文件，添加webdav访问的凭证，格式如下：

  ```bash
  <WebDav_URL> <用户名> <密码>
  ```

  之后更改设置权限，确保只有root权限才可访问：

  ```bash
  chmod 600 /usr/local/etc/davfs2/secrets
  ```

- 网盘挂载和卸载

  ```bash
  # 添加环境变量
  export PATH=$PATH:/usr/local/sbin
  # 挂载
  mount.davfs <WebDav_URL> <mount_point>
  # 卸载
  umount <mount_point>
  ```

## 网盘挂载

- 创建davfs2用户

  ```bash
  useradd davfs2
  ```

- 编辑 `/usr/local/etc/davfs2/secrets` 文件，添加webdav访问的凭证，格式如下：

  ```
  <WebDav_URL> <用户名> <密码>
  ```

  之后更改设置权限，确保只有root权限才可访问：

  ```bash
  chmod 600 /usr/local/etc/davfs2/secrets
  ```

- 网盘挂载和卸载

  ```bash
  # 添加环境变量
  export PATH=$PATH:/usr/local/sbin
  # 挂载
  mount.davfs <WebDav_URL> <mount_point>
  # 卸载
  umount <mount_point>
  ```

## 为了写作业搓的程序

```c
// mountWebdav.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <sys/stat.h>

// 检查本地是否存在指定的文件夹
int checkLocalFolder(const char *folderPath) {
    struct stat st;
    if (stat(folderPath, &st) == 0) {
        if (S_ISDIR(st.st_mode)) {
            return 1; // 文件夹存在
        }
    }
    return 0; // 文件夹不存在
}

int main(int argc, char *argv[]) {
    // 若参数不足，则打印使用方法并退出
    if (argc < 2) {
        fprintf(stderr, "Usage: sudo %s [mount|unmount] <local_mount_point> [username:password@webdav_address:port]\n", argv[0]);
        exit(EXIT_FAILURE);
    }
    // 检查本地文件夹是否存在
    if (!checkLocalFolder(argv[2])) {
        fprintf(stderr, "本地挂载点不存在！请确认挂载点存在或使用绝对路径\n");
        exit(EXIT_FAILURE);
    }
    // 添加环境变量
    system("export PATH=$PATH:/usr/local/sbin");

    char command[256];  // 命令缓冲区

    if (strcmp(argv[1], "umount") == 0) {      // 【卸载】
        snprintf(command, sizeof(command), "sudo umount.davfs %s", argv[2]);
        if(system(command) == -1){
            perror("执行失败，请检查是否已经挂载");
            exit(EXIT_FAILURE);
        }
        memset(command, 0, sizeof(command));  // 清空command缓冲区
    } else if (strcmp(argv[1], "mount") == 0) {    // 【挂载】
        // 检查参数数目
        if(argc != 4){
            fprintf(stderr, "Usage: %s [mount|unmount] <local_mount_point> [username:password@webdav_address:port]\n", argv[0]);
            exit(EXIT_FAILURE);
        }
        // 解析参数
        char info[256];
        strcpy(info, argv[3]);  // strtok函数需要修改原字符串，所以需要复制一份到char[]中
        char *username = strtok(info, ":");  // 获取用户名
        char *password = strtok(NULL, "@");  // 获取密码
        // 获取webdav地址
        char *webdav_address = strtok(NULL, ":");
        char *token = strtok(NULL, ":");
        asprintf(&webdav_address, "%s%s%s", webdav_address, ":", token);
        char *port = strtok(NULL, "");       // 获取端口号

        // 更新secrets配置
        if(port == NULL){
            snprintf(command, sizeof(command), "echo \"%s %s %s\" >> /usr/local/etc/davfs2/secrets", webdav_address, username, password);
        } else {
            snprintf(command, sizeof(command), "echo \"%s:%s %s %s\" >> /usr/local/etc/davfs2/secrets", webdav_address, port, username, password);
        }
        if(system(command) == -1){
            perror("更新secrets配置失败");
            exit(EXIT_FAILURE);
        }
        memset(command, 0, sizeof(command));  // 清空command缓冲区

        // 执行挂载命令
        snprintf(command, sizeof(command), "sudo mount.davfs %s %s", webdav_address, argv[2]);
        if(system(command) == -1){
            perror("执行挂载命令失败");
            exit(EXIT_FAILURE);
        } else {
            printf("挂载成功！\n");
        }
        memset(command, 0, sizeof(command));  // 清空command缓冲区
    } else {
        fprintf(stderr, "非法命令，请使用 'mount' 或 'unmount'.\n");
        exit(EXIT_FAILURE);
    }

    return 0;
}
```

## 测试程序

在测试时，为模拟用户使用的真实场景，我们将编译后的可执行文件 `mountWebdav` 移动到了 `/usr/local/sbin/` 文件夹下。

- 执行 `visudo` 命令，修改 `sudoers` 文件，添加下行：使得 davfs2 用户能够无密码执行该文件

  ```
  davfs2 ALL=(ALL) NOPASSWD: /usr/local/sbin/mountWebdav
  ```

- 切换到 davfs2 用户，尝试网盘挂载和卸载：

  > 注意，运行时开头须加上 `sudo`

  ```bash
  # 切换davfs2用户
  su davfs2
  # 创建本地挂载点
  mkdir davfstest
  # 网盘挂载
  sudo mountWebdav mount /home/davfs2/davfstest <username>:<passwd>@<webdav_address>:<port>
  # 网盘卸载
  sudo mountWebdav umount /home/davfs2/davfstest
  ```

  运行成功的输出大概如下：

  ```
  umount.davfs: waiting for mount.davfs (pid xxxxxx) to terminate gracefully .. OK
  ```
