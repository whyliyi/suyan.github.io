---
layout: post
title: python中tenacity的用法浅析
category: python
keywords: python, tenacity 
---

# 背景

最近研读OpenStack相关代码，发现了这个tenacity这个好玩的包，能够在异常或者不满足条件的情况下自动重试函数，只需要一个*装饰器*即可搞定，很任性。

Google了一下相关资料，写个博客以做记录。

# 场景
拿Heat中一段代码做解释：
代码如下：
[示例代码](https://github.com/openstack/heat/blob/master/heat/engine/clients/os/nova.py#L120-L135)


```python
    @tenacity.retry(
        stop=tenacity.stop_after_attempt(
            max(cfg.CONF.client_retry_limit + 1, 0)),
        retry=tenacity.retry_if_exception(
            client_plugin.retry_if_connection_err),
        reraise=True)
    def get_server(self, server):
        """Return fresh server object.
        Substitutes Nova's NotFound for Heat's EntityNotFound,
        to be returned to user as HTTP error.
        """
        try:
            return self.client().servers.get(server)
        except exceptions.NotFound:
            raise exception.EntityNotFound(entity='Server', name=server)
```
`stop=`是retry退出的条件

`retry=`是retry执行的条件

`reraise=`是是否在retry退出时跑出异常。

*在本段代码中，tenacity.retry的作用是：当client_plugin.retry_if_connection_err执行为true时，则重试调用get_server函数，重试次数由max(cfg.CONF.client_retry_limit + 1, 0)指定，如果达到重试次数后仍然不成功，则抛出异常。*

# 解析tenacity.retry

# 参考
[Package Index > tenacity > 4.8.0](https://pypi.python.org/pypi/tenacity)

[Python Never Gives Up: The tenacity Library](https://dzone.com/articles/python-never-gives-up-the-tenacity-library)

 

