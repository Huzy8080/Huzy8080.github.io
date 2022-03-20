---
layout: post
title: Pycharm社区版创建Django项目
# subtitle: subtitle
date: 2022-03-20
author: redme
header-img: img/post-bg-cook.jpg
catalog: true
tags:
  - Python
  - Django
---

## 一、背景

- Windows环境
- Pycharm社区版
- 使用Django创建项目

## 二、问题

Pycharm社区版的`File -> New Project...`没有创建Django的快捷方式，只能手动创建项目

## 三、解决方法

1. 创建项目文件夹**django_learning**
2. 在项目文件夹下进入命令行, 后续代码如下：

```Bash
# 建立虚拟环境
django_learning> python -m venv dl_env
# 激活虚拟环境
django_learning> dl_env\Scrippt\activate
# 激活后，目录前面有(dl_env)
# 安装Django
(dl_env) django_learning> pip install Django
#创建Django项目
(dl_env) django_learning> django-admin startproject django_learning .
#创建数据库
(dl_env) django_learning> python manage.py migrate
#在9000端口启动项目(默认8000，经常会被占用而启动报错)
(dl_env) django_learning> python manage.py runserver 9000
```

## 总结
1. 如果系统安装了多个版本，创建虚拟环境时要指定Python版本
2. 必须要激活虚拟环境，才能使用django-admin命令
3. runserver 命令可指定IP和port