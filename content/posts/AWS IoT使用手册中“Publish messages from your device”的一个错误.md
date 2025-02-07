+++
date = '2025-01-21T09:35:00+08:00'
draft = false
title = 'AWS IoT连接测试程序运行出错'
tags = ["AWS IoT"]
+++

### 1\. 问题位置
在AWS IoT的教程中的“[Publish messages from your device](https://docs.aws.amazon.com/iot/latest/developerguide/sdk-tutorials.html#sdk-tutorials-experiment)”时，
以下说明有误：
```
d.Locate this line of code:
  message_json = json.dumps(message)
e.Change it to:
  message = "{}".json.dumps(json.loads(message))
```

### 2\. 正确做法
经过实践，正确的说明是：
```
d.Locate this line of code:
  message_json = json.dumps(message)
e.Change it to:
  message_json = json.dumps(json.loads(message))
```

