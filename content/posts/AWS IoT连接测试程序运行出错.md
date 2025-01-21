+++
date = '2025-01-21T09:35:00+08:00'
draft = false
title = 'AWS IoT连接测试程序运行出错'
tags = ["AWS IoT"]
+++

### 1\. 问题现象
在AWS IoT的教程中的“[试试 AWS IoT Core 快速连接教程](https://docs.aws.amazon.com/zh_cn/iot/latest/developerguide/iot-quick-start.html)”时，
当直行到运行./start.sh这一步时，出现以下错误：
```shell
  Traceback (most recent call last):
  File "/home/czb/AWSIoT/aws-iot-device-sdk-python-v2/samples/pubsub.py", line 84, in <module>
    mqtt_connection = mqtt_connection_builder.mtls_from_path(
                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/czb/AWSIoT/python_aws_venv/lib/python3.12/site-packages/awsiot/mqtt_connection_builder.py", line 276, in mtls_from_path
    return _builder(tls_ctx_options, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/czb/AWSIoT/python_aws_venv/lib/python3.12/site-packages/awsiot/mqtt_connection_builder.py", line 231, in _builder
    tls_ctx = awscrt.io.ClientTlsContext(tls_ctx_options)
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/czb/AWSIoT/python_aws_venv/lib/python3.12/site-packages/awscrt/io.py", line 596, in __init__
    self._binding = _awscrt.client_tls_ctx_new(
                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  RuntimeError: 34 (AWS_ERROR_INVALID_ARGUMENT): An invalid argument was passed to a function.
```

### 2\. 查看源码
通过查看源码，发现这个错误是在pubsub.py文件中的84行抛出的异常，代码如下：
```python
    # Create a MQTT connection from the command line data
    mqtt_connection = mqtt_connection_builder.mtls_from_path(
        endpoint=cmdData.input_endpoint,
        port=cmdData.input_port,
        cert_filepath=cmdData.input_cert,
        pri_key_filepath=cmdData.input_key,
        ca_filepath=cmdData.input_ca,
        on_connection_interrupted=on_connection_interrupted,
        on_connection_resumed=on_connection_resumed,
        client_id=cmdData.input_clientId,
        clean_session=False,
        keep_alive_secs=30,
        http_proxy_options=proxy_options,
        on_connection_success=on_connection_success,
        on_connection_failure=on_connection_failure,
        on_connection_closed=on_connection_closed)
```
这个错误是在调用mqtt_connection_builder.mtls_from_path函数时抛出的异常，经过调试基本确定cmdData传递的参数存在问题，经过查找，问题出在cmdData.input_ca
这个参数，这个参数是一个文件路径，但是由于start.sh下载这个文件时出现错误，该文件是长度为0的空文件，因此出现了这个错误。
### 3\. 解决方案
有两个办法解决这个问题：
1. 重新下载证书文件，确保证书文件的内容不为空。
2. 修改start.sh脚本，把---ca_file root-CA.crt这个参数去掉，也是可以运行成功的。
```shell
printf "\nRunning pub/sub sample application...\n"
python3 aws-iot-device-sdk-python-v2/samples/pubsub.py --endpoint aeq2cc8y2b5ha-ats.iot.eu-north-1.amazonaws.com --cert Door.cert.pem --key Door.private.key --client_id basicPubSub --topic sdk/test/python --count 0
```

