---
layout: post
title:  "InterfaceTest自己动手实现自动化接口测试框架"
date:   2019-10-09 00:01:22 +0800
categories: Python, request, unittest, autotest
---

# InterfaceTest

灵感来源于写WebUI自动化的PageObject的设计模式；根据被测项目的实际情况，将接口抽离出来封装成接口层。辅以单元测试框架unittest、数据驱动ddt的用例层和数据层，提高测试代码的维护性和可阅读性 ~~（尤其是RESTful风格的api ）~~。

# 开始

**依赖三方库 **

request http请求

ddt 数据驱动

**安装**

克隆或下载项目，安装依赖。

```
pip install -r requirement.txt
```

~~不知道requirement.txt里面有没有漏掉什么依赖包，因为作者没有使用虚拟环境的习惯所以是没有直接用pip生成requirement.txt，如果运行测试代码的时候报错就用pip install直接安装相应的包吧。~~

启动测试代码

```
python run.py
```

# 项目目录

```
InterfaceTest
├── run.py										# 启动测试
├── CONFIG.py									# 配置文件
├── CONFIG_RAW.txt								# 配置源文件
├── case										# 测试用例
│   ├── test_{modulename}.py             
│   ├── ...
├── data										# 测试数据集
│   ├── test_{modulename}_{interfacename}.yaml	# 测试数据
│   ├── ...            
└── interface									# 接口封装
    └── __init__.py
    └── {modelname}.py
    └── BASE.py
```

# 配置文件
对于账号密码，base_url等，这种管全局的参数一般都放到CONFIG.py里维护，这里只是一种方法。当然也可以维护一份配置文件（如ini），

例如有时需要根据不通的环境提供不同的base_url和测试账号

~~~python
ENV = 'prd'

if ENV == 'prd':
    BASE_URL = 'http://api.xxx.con'
    TEST_USERNAME = 'account'
    TEST_PASSWORD = '123456'

elif ENV == 'dev':
    BASE_URL = 'http://api-dev.xxx.con'
    TEST_USERNAME = 'dev'
    TEST_PASSWORD = '123456'

else:
    BASE_URL = 'http://api-sit.xxx.con'
    TEST_USERNAME = 'sit'
    TEST_PASSWORD = '123456'
~~~




# 接口层

接口层是对后台所有接口的进行的抽象，类中的方法一般为对某一个接口的封装。

## 基类

BASE.py

根据实际需求对所有接口定义一些公关方法（例如某些接口需要jwt鉴权、需要带时间戳或request_id等。）

以下例子为获取header的方法，以及定义一个token变量存放登陆后的token

~~~python
from time import time


class BaseInterface:

    def __init__(self):
        self.token = ''

    def _get_headers(self, includes=''):
        t = str(int(time()))

        headers = {
            'timestamp': t,
        }

        if 'token' in includes:
            headers['token'] = self.token

        return headers
~~~

## 各模块

**{modelname}.py**

按后台的模块分割，分别对每个模块以一个py文件一个minxin类进行维护。

假设被测系统有鉴权、用户两个模块。

鉴权有注册登录登出接口，用户有获取用户信息接口，那么可以根据文档封装出以下mixin类。

根据以下的类可以很容易反向看出接口的url，method以及参数等信息。

auth.py

~~~PYTHON
from .Base import BaseInterface
from CONFIG import BASE_URL
import requests

class Auth(BaseInterface):

    def login(self, username, password):
        headers = self._get_headers()
        data = {
            'username': username,
            'password': password
        }
        response = requests.post(BASE_URL+'/auth/login', headers=headers, json=data)
        if response.status_code == 200:
            self.token = response.json()['data']   # 登录成功后自动保存token
        return response

    def register(self, username, password):
        headers = self._get_headers()
        data = {
            'username': username,
            'password': password
        }
        response = requests.post(BASE_URL + '/auth/register', headers=headers, json=data)
        return response

    def logout(self):
        headers = self._get_headers(includes='token')
        response = requests.delete(BASE_URL + '/auth/logout', headers=headers)
        if response.status_code == 200:
            self.token = ''   # 登出后自动保存token
        return response
~~~

user.py

~~~python
from .Base import BaseInterface
from CONFIG import BASE_URL
import requests

class User(BaseInterface):

    def get_profile(self):
        headers = self._get_headers(includes='token')  #headers附带token鉴权
        response = requests.get(BASE_URL + '/user/profile', headers=headers)
        return response
~~~

## 混入

\_\_init\_\_.py

~~~python
from .auth import Auth
from .user import User


class Interface(Auth, User):

    def __init__(self, username=None, password=None):
        super().__init__()
        if username and password:
            self.login(username, password)
~~~

## 调用

在其他地方只需要导入包里的Interface类，实例化后即可调接口

~~~python
from interface import Interface

con = Intreface('test_account','123456')
r = con.get_profile()
print(r.status_code)  # 响应码
pirnt(r.json())  #响应的json
~~~

# 用例层

test_{modulename}.py     

为了方便Testloader寻找测试用例以及维护方便，同样按照每个模块以一个py文件一个minxin类进行维护。为了后续可以分端进行测试也可对对模块进行多级区分（某些项目后台管理台有独立的用户体系会有多个用例层），例如：test_app_auth.py,test_admin_auth.py 分别对应app端的鉴权模块和管理台的鉴权模块。或者把UI自动化的用例也统一管理~~，后续可能会写一篇UI自动化的文章。~~

为了简单一下只按一个端的接口进行举例。

test_auth.py

~~~python
import unittest
from interface import Interface
from CONFIG import TEST_USERNAME, TEST_PASSWORD


class TestUser(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        global con
        con = Interface(username=TEST_USERNAME, password=TEST_PASSWORD)

    def test_get_profile(self):
        r = con.get_profile()
        self.assertEqual(r.status_code, 200)
~~~

#　数据层

某些接口有多组测试参数，可以用将测试参数或预期结果进行参数化，结合ddt模块进行维护。

例如登录接口需要有登录成功和登录失败的用例。

当然如果只是少量的、固定不变的数据也可以直接放在用例里，这里的粒度可以根据实际情况来把握。

而一切一次性消耗的数据，可以用随机函数生成

/data/test_auth_login.yaml

~~~yaml
case1:
  username : username1
  password : '123456'
  state_code : 200

case2:
  username: username1
  password: '12345'
  state_code : 499

~~~

~~~python
import unittest
from interface import Interface
from ddt import ddt, file_data
from CONFIG import TEST_USERNAME, TEST_PASSWORD


@ddt
class TestAuth(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        global con
        con = Interface()

    @file_data('../data/test_auth_login.yaml')
    def test_login(self, username, password, state_code):
        r = con.login(username, password)
        self.assertEqual(r.status_code, state_code)

    @file_data('../data/test_auth_register.yaml')
    def test_register(self, username, password):
        r = con.register(username, password)
        self.assertEqual(r.status_code, 200)

    def test_logout(self):
        con2 = Interface(username=TEST_USERNAME, password=TEST_PASSWORD)
        r = con2.logout()
        self.assertEqual(r.status_code, 200)
~~~

# 主入口

run.py

如果只需要简单的实现运行时执行所有测试用例那么run.py脚本很简单。

只需三步走：

1.找到所有用例

2.实例化TestRuner

3.执行用例

~~~python
from unittest import defaultTestLoader, TextTestRunner

cases = defaultTestLoader.discover('./case/', pattern=pattern)
runner = TextTestRunner()
runner.run(cases)
~~~

为了未来可以灵活运行脚本，例如实现在确定在哪个环境运行、运行哪些用例等需求，

这里则引入了命令行选项解析的库getopt。而run.py脚本很简单。则会相对复杂一些。

~~~python
import sys
import getopt
from unittest import defaultTestLoader, TextTestRunner


def build_config(env):  # 构建CONFIG文件的方法
    with open('CONFIG_raw.txt', 'r', encoding='utf-8') as f:
        content = f.read()
    with open('CONFIG.py', 'w', encoding='utf-8') as f:
        f.write(f"ENV = '{env}'\n")
        f.write('\n')
        f.write(content)


def main(argv):  # 主方法
    pattern = 'test_*.py'   # pattern默认搜集全部测试用例
    env = 'test'  # 默认在测试环境运行测试用例
    try:
        opts, args = getopt.getopt(argv, 'hp:e:')
        for opt, arg in opts:
            if opt == '-h':
                print('run.py -p <pattern> -e <environment>')
                sys.exit()
            if opt == '-e':
                env = arg.lower()
            if opt == '-p':
                pattern = f'{arg}*.py'

        build_config(env)  # 构建CONFIG文件
    except getopt.GetoptError as e:
        print(e)
        sys.exit()

    # 根据入参pattern，在case文件夹中收集测试用例，pattern默认搜集全部测试用例
    cases = defaultTestLoader.discover('./case/', pattern=pattern)

    # 实例化一个测试执行器并执行所有测试用例
    runner = TextTestRunner()
    runner.run(cases)


if __name__ == '__main__':
    main(sys.argv[1:])
~~~

添加了需求后则可以添加pattern以及environment参数进行灵活的测试。



































