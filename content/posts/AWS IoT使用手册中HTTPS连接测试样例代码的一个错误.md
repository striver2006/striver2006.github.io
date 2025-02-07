+++
date = '2025-01-21T09:35:00+08:00'
draft = false
title = 'AWS IoT教程中的“HTTPS”章节错误'
tags = ["AWS IoT"]
+++

### 1\. 问题现象
在AWS IoT的教程中的“[HTTPS](https://docs.aws.amazon.com/iot/latest/developerguide/http.html#codeexample)”时，
当直行到运行示例代码时，出现以下错误：
```shell
  Traceback (most recent call last):
  File "/home/pi/pi5_0207/aws-iot-device-sdk-python-v2/http_pubsub.py", line 22, in <module>
    publish = requests.request('POST',
              ^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/requests/api.py", line 59, in request
    return session.request(method=method, url=url, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/requests/sessions.py", line 589, in request
    resp = self.send(prep, **send_kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/requests/sessions.py", line 703, in send
    r = adapter.send(request, **kwargs)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/requests/adapters.py", line 633, in send
    conn = self.get_connection_with_tls_context(
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/requests/adapters.py", line 489, in get_connection_with_tls_context
    conn = self.poolmanager.connection_from_host(
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/urllib3/poolmanager.py", line 246, in connection_from_host
    return self.connection_from_context(request_context)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/urllib3/poolmanager.py", line 261, in connection_from_context
    return self.connection_from_pool_key(pool_key, request_context=request_context)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/urllib3/poolmanager.py", line 274, in connection_from_pool_key
    pool = self.pools.get(pool_key)
           ^^^^^^^^^^^^^^^^^^^^^^^^
  File "<frozen _collections_abc>", line 774, in get
  File "/home/pi/aws-iot-python-venv/lib/python3.11/site-packages/urllib3/_collections.py", line 57, in __getitem__
    item = self._container.pop(key)
           ^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: unhashable type: 'list'
```

### 2\. 查看源码
通过查看源码，发现这个错误是在示例中的22行抛出的异常，代码如下：
```python
import requests
import argparse

# define command-line parameters
parser = argparse.ArgumentParser(description="Send messages through an HTTPS connection.")
parser.add_argument('--endpoint', required=True, help="Your AWS IoT data custom endpoint, not including a port. " +
                                                      "Ex: \"abcdEXAMPLExyz-ats.iot.us-east-1.amazonaws.com\"")
parser.add_argument('--cert', required=True, help="File path to your client certificate, in PEM format.")
parser.add_argument('--key', required=True, help="File path to your private key, in PEM format.")
parser.add_argument('--topic', required=True, default="test/topic", help="Topic to publish messages to.")
parser.add_argument('--message', default="Hello World!", help="Message to publish. " +
                                                      "Specify empty string to publish nothing.")

# parse and load command-line parameter values
args = parser.parse_args()

# create and format values for HTTPS request
publish_url = 'https://' + args.endpoint + ':8443/topics/' + args.topic + '?qos=1'
publish_msg = args.message.encode('utf-8')

# make request
publish = requests.request('POST',
            publish_url,
            data=publish_msg,
            cert=[args.cert, args.key])

# print results
print("Response status: ", str(publish.status_code))
if publish.status_code == 200:
    print("Response body:", publish.text)
```
这个错误是因为在第22行中，cert 参数被传递为一个列表，而 requests.request 方法期望的是一个元组。你可以将列表转换为元组来解决这个问题。
```python
publish = requests.request('POST',
            publish_url,
            data=publish_msg,
            cert=(args.cert, args.key))
```
### 3\. 解决方案
修正后的代码如下：
```python
import requests
import argparse

# define command-line parameters
parser = argparse.ArgumentParser(description="Send messages through an HTTPS connection.")
parser.add_argument('--endpoint', required=True, help="Your AWS IoT data custom endpoint, not including a port. " +
                                                      "Ex: \"abcdEXAMPLExyz-ats.iot.us-east-1.amazonaws.com\"")
parser.add_argument('--cert', required=True, help="File path to your client certificate, in PEM format.")
parser.add_argument('--key', required=True, help="File path to your private key, in PEM format.")
parser.add_argument('--topic', required=True, default="test/topic", help="Topic to publish messages to.")
parser.add_argument('--message', default="Hello World!", help="Message to publish. " +
                                                      "Specify empty string to publish nothing.")

# parse and load command-line parameter values
args = parser.parse_args()

# create and format values for HTTPS request
publish_url = 'https://' + args.endpoint + ':8443/topics/' + args.topic + '?qos=1'
publish_msg = args.message.encode('utf-8')

# make request
publish = requests.request('POST',
            publish_url,
            data=publish_msg,
            cert=(args.cert, args.key))

# print results
print("Response status: ", str(publish.status_code))
if publish.status_code == 200:
    print("Response body:", publish.text)
```

