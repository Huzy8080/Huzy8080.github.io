---
layout: post
title: Django开发Web项目
# subtitle: subtitle
date: 2022-03-20
author: redme
header-img: img/post-bg-cook.jpg
catalog: true
tags:
  - Python
  - Django
---

## 创建应用程序

```shell
# 创建一个app
python manage.py startapp learning_logs

```

新建 model:

```python
from django.db import models

# Create your models here.

class Topic(models.Model):
    """用户学习的主题"""
    text = models.CharField(max_length=200)
    date_added = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        """返回模型的字符串表示"""
        return self.text

```
