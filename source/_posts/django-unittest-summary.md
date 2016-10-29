title: django 单元测试小结
comments: true
toc: true
date: 2016-10-09 19:23:27
tags: [django,test]
categories: [技术分享]
---

<!-- more -->
> 从前很少写单元测试了，特别是web应用。最近不知不觉喜欢起来这个事情了，发现单元测试对于软件的模块，正交性有很大促进作用，因为函数，模块写的不合理，单元测试写起来就麻烦的多呀。公司的项目一直都是用Django，所以写点django单元测试的小总结，记录为主，备查。

## 测试的场景
框架Django1.8 测试工具 unittest, 要记得给test设置一个独特的settings。

1. 测试请求 也就是测试整个view部分 [官方案例](https://docs.djangoproject.com/en/1.8/topics/testing/advanced/#example) 其中可能会遇到登录，或者时session怎么模拟的问题

2. 测试带有orm的模块
3. 需要mock的测试，比较多的情况是有第三方API调用, 发邮件，发短信这种

unittest提供的断言种类挺多,但是经常用的也就几个 `self.assertContains`, `self.assertEqual`, `self.assertTrue`

顺便提下有用的选项(我这里是单独给测试写了一个settings), 为了提高测试速度，可以把用不到的中间件，installed_apps之类的多余配置给去掉。

```
测试全部用例
python manage.py test  --setting settings_test

测试某个APP
python manage.py test appname --setting settings_test

测试某个app下的TeseCase类
python manage.py test alarm.tests.ModelTestCase --setting settings_test

-v {1,2,3} 数字越大，显示的输出越详细，测试的日志信息
python manage.py test --setting settings_test -v3

其他的选项请查看 --help
python manage.py test --help
```

### 用请求测试 views函数
DJANGO中提供了Client类来模拟http请求，可以模拟不同的method，然后就是请求参数的模拟，用起来很方面。


```
#coding:utf-8
from django.test import TestCase, Client

from sendviews import *
from core.tests import create_user


class SendviewsTestCase(TestCase):

    def setUp(self):
        self.user = create_user()
        self.device = Device(hostname="CN-BJ-0000-00",
                             mac="ff:ff:ff:ff:ff:ff", user=self.user).save()

    def test_creat_sms(self):
        c = Client()
        rep = c.post("/acquireportal/createsms",{"phone": "13988902345",
                                                 "ssid": "erya",
                                                 "dmac": "ff:ff:ff:ff:ff:ff"})
        # 测试http请求的返回码是否正确
        self.assertEqual(rep.status_code, 200)
        # 测试response的内容是否包含字符串
        self.assertContains(rep, "OK")
        # 测试response的内容是否包含字符串 方法二
        self.assertTrue('OK' in rep.content)
```

* 使用 RequestFactory 对象来进行测试，不是从 http client来发起，某些情况会用到

```
from django.test import TestCase, RequestFactory
from django.http import HttpResponse
from util.sign import generate_sign, validate_sign
from util.decorators import apiauth_required, SIGN_KEY

@apiauth_required()
def simpleapi(request):
    return HttpResponse('ok')


class DecoratorsTestCase(TestCase):

    def setUp(self):
        self.factory = RequestFactory()

    def test_apiauth(self):
        # create request object
        key = SIGN_KEY
        query_string = {u"name": u"lzz", u"age": u"20", u"data": u"[python, java, golang, lua]"}
        token = generate_sign(query_string, key)
        query_string.update({u"sign": token})
        req = self.factory.post("/api/test", data=query_string)
        response = simpleapi(req)
        self.assertEqual(response.status_code, 200)

```


* HTML 文本测试，使用 constants 来判断并不是个好的选择，可以用render之后的字符串对比。

对于需要登陆的view，有client也比较容易操作，还有一些特殊的session的检测等, 我这里做了一个简单的封装

```
from django.test import Client

def init_client(user):
    client = Client()
    client.login(username=user.username, password="lzz")
    s = client.session
    s['cur_user_id'] = user.id
    s.save()
    return client
```


### 带有mock的测试
对模块中的方法mock或者是对一个对象中的方法进行mock。真对测试函数中一些无法直接测试的函数设置默认的返回值, py3标准库中已经有了[mock模块](https://docs.python.org/3/library/unittest.mock.html)，py2需要自己安装, 推荐教程 [使用Pyhton Mock进行单元测试1](http://www.oschina.net/translate/unit-testing-with-the-python-mock-class)。 下面是个实际的代码片段。

```
import mock
from django.test import TestCase
from core.models import Tenant
from alarm.models import *
from .controler import TenantAlarm

class ModelTestCase(TestCase):
    def setUp(self):
        self.tenant = Tenant.objects.create(domainname="erya", comname=u"尔雅")

    @mock.patch.object(TenantAlarm, "sendAlarm")
    def test_record_alarm(self, mock_method):
        # record_alarm 这个中会调用sendAlarm方法
        mock_method.return_value = None
        content = "ccccc"
        atype = 0
        rec_uid = 0
        Alarm().record_alarm(content=content, atype=0,
                             rec_tid=self.tenant.id)


class TenantAlarmTestCase(TestCase):
    def setUp(self):
        self.tenant = Tenant.objects.create(domainname="erya", comname=u"尔雅")

    @mock.patch.object(TenantAlarm, "sendSMS", return_value=None)
    @mock.patch.object(TenantAlarm, "sendEmail", return_value=None)
    def test_send_alarm(self, method1, method2):
        content = u"报警了"
        ta = TenantAlarm(self.tenant.id, content, {u'SMS': 0, u'EMAIL': 0})
        ta.sendAlarm()

    @mock.patch('util.sendsms_com.send', return_value=1)
    def test_sendsms(self, send):
        ta = TenantAlarm(self.tenant.id, self.content, {u'SMS': 0, u'EMAIL': 0})
        ta.sendSMS()
        self.assertEqual(0, Account.objects.get(tenant=self.tenant).sms_num)
        self.account.sms_num = 100
        self.account.save()
        ta.sendSMS()
        self.assertEqual(99, Account.objects.get(tenant=self.tenant).sms_num)
```


## coverage
coverage是一个检查单元测试覆盖率的工具，django的文档中也有简要的说明coverage的集成 [文档地址](https://docs.djangoproject.com/en/1.8/topics/testing/advanced/#integration-with-coverage-py)

```bash
#测试并收集测试信息
coverage run --source='.' manage.py test --setting mandela.settings_test
#查看测试结果
coverage report -m
Name                                                       Stmts   Miss  Cover   Missing
----------------------------------------------------------------------------------------
acquireportal/__init__.py                                      0      0   100%
acquireportal/controler.py                                    65     47    28%   22-56, 60-71, 76-79
acquireportal/migrations/0001_initial.py                       6      0   100%
acquireportal/migrations/0002_auto_20160622_1059.py            6      0   100%
acquireportal/migrations/0003_auto_20160622_1100.py            5      0   100%
....

----------------------------------------------------------------------------------------
TOTAL                                                       8013   5858    27%
```

覆盖率挺低的😉
