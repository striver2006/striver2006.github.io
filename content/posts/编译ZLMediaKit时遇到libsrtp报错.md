+++
date = '2024-12-22T22:40:13+08:00'
draft = false
title = '编译ZLMediaKit遇到问题：调用VS 2022生成的libsrtp报错："imp_clock未找到"'
tags = ["ZLMediaKit","libsrtp"]
+++

### 编译ZLMediaKit时遇到libsrtp报错

经过多放查找，我工程配置为release版本，在调用的lib里面，编译的时候选择的MD

![错误的配置](../images/编译ZLMediaKit时遇到libsrtp报错/1.png "错误的配置")

在编译主项目时，就会提示LNK2001: 无法解析的外部符号 __imp__clock xxx

需要把lib编译的时候，选择为MT

 ![正确的配置](../images/编译ZLMediaKit时遇到libsrtp报错/2.png "正确的配置")

 在重新编译即可成功

参考：

[visual c++ - Linker can not find __imp_clock and __imp_time64 when building a sample of GTSAM - Stack Overflow](https://stackoverflow.com/questions/68330445/linker-can-not-find-imp-clock-and-imp-time64-when-building-a-sample-of-gtsam)

-------------------------------------
原文链接：[https://blog.csdn.net/a343981218/article/details/129161128](https://blog.csdn.net/a343981218/article/details/129161128)
