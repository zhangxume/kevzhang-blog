---
title: "Superset 配置环境"
date: 2022-10-16T12:00:59+08:00
description: "出现版本不兼容的情况，参考链接解决问题"
tags: [Superset]
featured_image: "/superset_conf/featured_image.png"
draft: false
---

## Superset 配置环境
### 版本依赖
- 由于 CentOS 7 内置的 Python 版本为 2.7.5，而在[旧版本的文档中提到](https://apache-superset.readthedocs.io/en/latest/installation.html)  
  `Superset supports Python versions >3.7 to take advantage of the newer Python features and reduce the burden of supporting previous versions.`  
在[新版本的文档](https://superset.apache.org/docs/installation/installing-superset-from-scratch)中按照提供的命令即可完成所需依赖的安装。
### Miniconda
- 安装 Miniconda 用于管理 Python 版本及其依赖。
#### 常用命令
- 创建环境  
  ```shell
  [xxx@hostname ~]$ conda create -n env_name
  ```
- 查看所有环境  
  ```shell
  [xxx@hostname ~]$ conda info --envs
  ```
- 删除一个环境  
  ```shell
  [xxx@hostname ~]$ conda remove -n env_name --all
  ```
- 切换环境  
  ```shell
  (base) [xxx@hostname ~]$ conda activate superset
  (superset) [xxx@hostname ~]$ conda deactivate
  ```
#### 创建 ~~Python 3.7 环境~~（出现版本不兼容的问题，改成3.6）
- 创建环境  
  ```shell
  [xxx@hostname ~]$ conda create --name superset python=3.6
  ```
## Superset 安装部署
- 安装依赖  
  ```shell
  (superset) [xxx@hostname ~]$ sudo yum install -y gcc gcc-c++ libffi-devel python-devel python-pip python-wheel python-setuptools openssl-devel cyrus-sasl-devel openldap-devel
  ```
- 安装Superset（指定镜像会快一些，可以更换其他的镜像源试试）  
  ```shell
  (superset) [xxx@hostname ~]$ pip install --upgrade setuptools pip -i https://pypi.douban.com/simple/  
  (superset) [xxx@hostname ~]$ pip install apache-superset -i https://pypi.douban.com/simple/
  ```
- 解决 warning，执行以下两条命令  
  ```shell
  (superset) [xxx@hostname ~]$ pip install sqlalchemy==1.3.24  
  (superset) [xxx@hostname ~]$ pip install dataclasses
  ```
- 初始化数据库  
  ```shell
  (superset) [xxx@hostname ~]$ superset db upgrade
  ```
- 创建管理员用户  
  ```shell
  (superset) [xxx@hostname ~]$ export FLASK_APP=superset  
  (superset) [xxx@hostname ~]$ superset fab create-admin
  ```
- 初始化  
  ```shell
  (superset) [xxx@hostname ~]$ superset init
  ```
### 启动 Superset
- 安装 gunicorn  
  ```shell
  (superset) [xxx@hostname ~]$ pip install gunicorn -i https://pypi.douban.com/simple/
  ```
- 启动  
  ```shell
  (superset) [xxx@hostname ~]$ gunicorn --workers 5 --timeout 120 --bind hostname:8787 "superset.app:create_app()" --daemon
  ```
- 停止  
  ```shell
  (superset) [xxx@hostname ~]$ ps -ef | awk '/superset/ && !/awk/{print $2}' | xargs kill -9
  ```

## 参考链接
[installation.html](https://apache-superset.readthedocs.io/en/latest/installation.html)  
[installing-superset-from-scratch](https://superset.apache.org/docs/installation/installing-superset-from-scratch)  
[article](https://blog.csdn.net/m0_46914845/article/details/125868049?spm=1001.2014.3001.5502)