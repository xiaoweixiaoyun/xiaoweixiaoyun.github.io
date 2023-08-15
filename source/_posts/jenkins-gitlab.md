---
title: jenkins自动部署（远程编译、部署）
date: 2023/08/15
tags: jenkins,部署
categories: 运维
description:  节约每次上线部署时间，提高作业效率，保证正确性。
keywords: jenkins,部署
cover: /img/md/jenkins.png
---

# 设置参数
## 设置分支参数：branch（develop）
## 设置工程名参数：project（logisticsif-back、logisticsif-front）

## 编写生成镜像脚本
```shell
echo `date +"%Y%m%d%H%M%S%N" | cut -b 1-16` >> $JENKINS_HOME/workspace/$JOB_NAME/$BUILD_DISPLAY_NAME
PJ_LOGISTICSIF_TIMESTAMP=`cat $JENKINS_HOME/workspace/$JOB_NAME/$BUILD_DISPLAY_NAME`
ssh systemuser@172.21.16.85 "cd /tmp && rm -rf $project"
ssh systemuser@172.21.16.85 "cd /tmp && git clone --depth=1 -b $branch git@docker环境服务器地址:project-trial/md/control/$project.git"
ssh systemuser@172.21.16.85 "cd /tmp/$project && docker build -t docker环境服务器地址/project-trial/md/control/$project:$PJ_LOGISTICSIF_TIMESTAMP ."
ssh systemuser@172.21.16.85 "docker logout docker环境服务器地址"
ssh systemuser@172.21.16.85 "echo password | docker login docker环境服务器地址 --username 用户名 --password-stdin"
ssh systemuser@172.21.16.85 "docker push docker环境服务器地址/project-trial/md/control/$project:$PJ_LOGISTICSIF_TIMESTAMP"
ssh systemuser@172.21.16.85 "docker logout docker环境服务器地址"
ssh systemuser@172.21.16.85 "docker rmi docker环境服务器地址/project-trial/md/control/$project:$PJ_LOGISTICSIF_TIMESTAMP"
```

## 编写容器启动脚本（docker-compose.yml）[下载地址](/file/logisticeif.zip)
```shell
version: "3.8"

services:
  logisticsifview:
    image: docker环境服务器地址/project-trial/md/control/logisticsif-front:2023081509515721
    restart: always
    tty: true
    network_mode: host
  logisticsifapi:
    image: docker环境服务器地址/project-trial/md/control/logisticsif-back:2023081509530949
    restart: always
    tty: true
    network_mode: host
    environment:
      - active=dev
    volumes:
      - ./logs/api:/logs
      - ./conf/api:/config
  nginx:
    image: nginx
    container_name: logisticsif-gateway
    restart: always
    volumes:
      - ./conf/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf
      - ./conf/nginx/etc/localtime:/etc/localtime
      - ./conf/nginx/etc/timezone:/etc/timezone
      - ./logs/nginx:/var/log/nginx
    depends_on:
      - logisticsifview
      - logisticsifapi
    network_mode: host
```


## 调用容器启动脚本
```shell
PJ_LOGISTICSIF_TIMESTAMP=`cat $JENKINS_HOME/workspace/$JOB_NAME/$BUILD_DISPLAY_NAME`
ssh manager@10.100.1.125 "docker logout docker环境服务器地址"
ssh manager@10.100.1.125 "echo password | docker login docker环境服务器地址 --username 用户名 --password-stdin"
ssh manager@10.100.1.125 "docker pull docker环境服务器地址/project-trial/md/control/$project:$PJ_LOGISTICSIF_TIMESTAMP"
ssh manager@10.100.1.125 "docker logout docker环境服务器地址"
ssh manager@10.100.1.125 "cd /home/manager/${ENV} && docker-compose down && sed -i "s/${project}:[0-9]*/${project}:${PJ_LOGISTICSIF_TIMESTAMP}/g" docker-compose.yml && docker-compose up -d"
```