---
title: ClamAV部署及使用
urlname: gew51xlnpitb9lhc
date: '2023-02-06 18:23:45 +0800'
tags: []
categories: []
---

# 安装

RPM 包下载地址
[https://www.clamav.net/downloads](https://www.clamav.net/downloads)

## yum 安装

```bash
sudo yum update -y
sudo yum install repl-release
sudo yum install -y openssl openssl-devel libcurl-devel zlib-devel libpng-devel libxml2-devel json-c-devel bzip2-devel pcre2-devel ncurses-devel
sudo yum install clamav clamd clamav-update
```

# 卸载

```python
sudo yum remove clamav clamd clamav-update
```

# 使用

## 官方文档

[https://docs.clamav.net/](https://docs.clamav.net/)

## ClamAV 主要组成部分

- clamd - 后台进程
- clamav - 用户端扫描程序
- clamav-data - 扫描器威胁特征库
- clamav-devel - 扫描器程序依赖
- clamav-lib - 扫描器动态调用库
- clamav-milter - Milter module for the Clam Antivirus scanner
- clamav-update - 更新扫描器威胁特征库

## 特征库更新

配置文件位于/etc/freshclam.conf，可修改更新频率，默认是两小时检查一次更新，找到其中的如下一段，把数值改为 1：

```bash
# Number of database checks per day.
# Default: 12 (every two hours)
Checks 1
```

去掉配置文件中 UpdateLogFile 选项的注释，并设置日志保存的路径
执行`freshclam`命令更新威胁特征库
启动后台更新服务

```bash
systemctl start clamav-freshclam
systemctl enable clamav-freshclam
```

## 修改配置文件

配置文件路劲：/etc/clamd.d/scan.conf

- 去掉 Logfile 选项的注释，日志文件保存在/var/log/clamd.scan
- LogFileMaxSize 选项建议设置为 10M
- 修改 LogSyslog 选项为 no
- OnAccessIncludePath 参数设置需要实时监测的目录
- LocalSocket 去掉注释，并修改为/tmp/clamd.socket
- LocalSocketGroup 去掉注释并修改为 clamscan
- LocalSocketMode 去掉注释并修改为 660
- 如果需要阻断恶意文件的读写，需要去掉 OnAccessPrevention 选项的注释，并修改为 yes；如果只是进行检测不阻断，只记录日志，则无需修改。

都修改完毕后，保存退出。
执行如下命令创建日志文件：

```
touch /var/log/clamd.scan
chown clamscan:clamscan /var/log/clamd.scan
```

## 定时扫描和实时监控

### 应用目录定时扫描

每天凌晨 2 点进行 sftp 目录扫描

```bash
0 2 * * * /usr/bin/clamscan -r -i --quiet -o -l /var/log/clamscan.log --move=/tmp/infected /sftp
```

### 应用目录实时监控

暂时禁用 会造成被监听目录的 IO 性能受损、缓存性能受损 IO 读写速度差距约 1 倍 缓存性能的影响约 30 倍)

## 上报扫描结果

### 配置 Logstash

简介
工具发展里程
主要用途

安装
几种安装方式
源码安装注意内核以及 curl 版本，还需要安装 Python3 模块 cmake
各模块说明

配置
配置文件位置
主要使用配置
更新特征库
启动后台服务

使用
定期目录扫描
实时文件监控扫描
rest-api 调用

注意事项

谷歌实际案例
