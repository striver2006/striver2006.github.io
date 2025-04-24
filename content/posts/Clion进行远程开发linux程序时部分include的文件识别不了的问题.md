+++
date = '2025-04-24T12:40:13+08:00'
draft = false
title = 'Clion进行远程开发linux程序时部分include的文件识别不了的问题'
tags = ["CLion","远程开发"]
+++

### Clion进行远程开发linux程序时部分include的文件识别不了的问题

代码文件include远程的头文件时，提示找不到头文件的错误，经过多方查找，如果头文件在linux系统上的路径是：

```shell
/usr/include/xxx/xxx.h
```

如果这个头文件所在的文件夹已经在工程中正确添加了include路径，但还是提示找不到的话，使用以下方法就行纠正：  

如果是在远程主机上进行开发，CLion 可能没有正确同步远程主机上的文件。可以尝试使用 CLion 的“Tools | Resync with Remote Hosts”功能来重新同步远程文件。
