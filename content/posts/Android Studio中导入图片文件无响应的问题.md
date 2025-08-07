+++
date = '2025-08-07T15:49:26+08:00'
draft = false
title = 'Android Studio中导入图片文件无响应的问题'
tags = ["Android", "Android Studio", "图片资源导入"]
+++

### 1\. 问题现象
在Mac操作系统上使用Android Studio中通过Resource Manager导入图片无响应。具体现象情况如下：<br>
![导入图片](../images/AndroidStudio中导入图片文件无响应的问题/导入操作.png "导入图片")
<p style="text-align: center;">导入图片</p>
<br>
![导入后没有反应](../images/AndroidStudio中导入图片文件无响应的问题/导入没有反应.png "导入后没有反应")
<p style="text-align: center;">导入后没有反应</p>
<br>
操作后未能显示正常的导入界面：
![正常导入的界面](../images/AndroidStudio中导入图片文件无响应的问题/正常导入的界面.png "正常导入的界面")

### 2\. 解决方案
经过多次尝试，发现只要导入的文件路径上存在中文则导入不成功，把图片文件放到一个纯英文路径下面就可以正常使用了。<br>
