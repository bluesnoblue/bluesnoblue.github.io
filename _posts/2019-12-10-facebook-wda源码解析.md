---
layout: post
title:  "facebook-wda源码解析"
date:   2019-12-10 00:01:22 +0800
categories: Python, autotest，ATX, facebook-wda
---

# facebook-wda源码解析

facebook-wda
项目地址：https://github.com/openatx/facebook-wda
简介：Facebook WebDriverAgent的Python客户端库（非官方）

主要代码位于/wda/\_\_init\_\_.py
## Error

定义了一些常见的异常
~~~python
class WDAError(Exception):
    """ base wda error """


class WDARequestError(WDAError):
    def __init__(self, status, value):
        self.status = status
        self.value = value

    def __str__(self):
        return 'WDARequestError(status=%d, value=%s)' % (self.status,
                                                         self.value)


class WDAEmptyResponseError(WDAError):
    """ response body is empty """


class WDAElementNotFoundError(WDAError):
    """ element not found """


class WDAElementNotDisappearError(WDAError):
    """ element not disappera """
~~~

## HTTPClient（请求层）

主要对request进行了封装，个人感觉主要新增了线程锁和将响应的json装换成对象的功能。

```python
class HTTPClient(object):
    def __init__(self, address, alert_callback=None):
        """
        Args:
            address (string): url address eg: http://localhost:8100
            alert_callback (func): function to call when alert popup
        """
        self.address = address
        self.alert_callback = alert_callback

    def new_client(self, path):
        return HTTPClient(
            self.address.rstrip('/') + '/' + path.lstrip('/'),
            self.alert_callback)

    def fetch(self, method, url, data=None):
        return self._fetch_no_alert(method, url, data)
        # return httpdo(urljoin(self.address, url), method, data)

    def _fetch_no_alert(self, method, url, data=None, depth=0):
        target_url = urljoin(self.address, url)
        try:
            return httpdo(target_url, method, data)
        except WDARequestError as err:
            if depth >= 10:
                raise
            if err.status != 26:  # alert status code
                raise
            if not callable(self.alert_callback):
                raise
            self.alert_callback()
            return self._fetch_no_alert(method, url, data, depth=depth + 1)

    def __getattr__(self, key):
        """ Handle GET,POST,DELETE, etc ... """
        return functools.partial(self.fetch, key)
```

### http_do() 

1.解析url

2.发送请求 (线程安全锁)

###  _unsafe_httpdo(url,method='GET',data=None)

1.发送请求 

2.响应转化为namedtuple对象

3.返回响应对象（异常处理）

~~~python
def namedlock(name):
    """
    Returns:
        threading.Lock
    """
    if not hasattr(namedlock, 'locks'):
        namedlock.locks = defaultdict(threading.Lock)
    return namedlock.locks[name]

def convert(dictionary):
    """
    Convert dict to namedtuple
    """
    return namedtuple('GenericDict', list(dictionary.keys()))(**dictionary)

def httpdo(url, method="GET", data=None):
    """
    thread safe http request
    """
    p = urlparse(url)
    with namedlock(p.scheme+"://"+p.netloc):
        return _unsafe_httpdo(url, method, data)

def _unsafe_httpdo(url, method='GET', data=None):
    """
    Do HTTP Request
    """
    # ...

    try:
        response = requests.request(method,
                                    url,
                                    json=data,
                                    timeout=HTTP_TIMEOUT)
    except (requests.exceptions.ConnectionError,
            requests.exceptions.ReadTimeout) as e:
        raise

    # ...

    try:
        retjson = response.json()
        retjson['status'] = retjson.get('status', 0)
        r = convert(retjson)
        if r.status != 0:
            raise WDARequestError(r.status, r.value)
        return r
    except JSONDecodeError:
        # ...
~~~

## Client (设备的抽象)

持有一个HTTPClient对象，默认url为http://localhost:8100

~~~python
class Client(object):
    def __init__(self, url=None):
        """
        Args:
            target (string): the device url
        
        If target is empty, device url will set to env-var "DEVICE_URL" if defined else set to "http://localhost:8100"
        """
        if not url:
            url = os.environ.get('DEVICE_URL', 'http://localhost:8100')
        assert re.match(r"^https?://", url), "Invalid URL: %r" % url

        self.http = HTTPClient(url)
        
~~~

另外还封装了一些设备操作的方法，主要都是发送http请求。

例如：

~~~python
    def home(self):
        """Press home button"""
        return self.http.post('/wda/homescreen')

    def healthcheck(self):
        """Hit healthcheck"""
        return self.http.get('/wda/healthcheck')

    def locked(self):
        """ returns locked status, true or false """
        return self.http.get("/wda/locked").value

    def lock(self):
        return self.http.post('/wda/lock')

    def unlock(self):
        """ unlock screen, double press home """
        return self.http.post('/wda/unlock')
~~~

最重要的是提供一个session()方法，通过传入bundle_id（包名）获取一个session对象（APP抽象）

~~~python
    def session(self,
                bundle_id=None,
                arguments=None,
                environment=None,
                alert_action=None):
        # ...
        
        try:
            res = self.http.post('session', data)
        except WDAEmptyResponseError:
            # ...
        httpclient = self.http.new_client('session/' + res.sessionId)
        return Session(httpclient, res.sessionId)
~~~



## Session(App的抽象)

继续传递封装httpclient，其实这里没有看很懂...session_id传进来有什么用
~~~python
class Session(object):
    def __init__(self, httpclient, session_id):
        """
        Args:
            - target(string): for example, http://127.0.0.1:8100
            - session_id(string): wda session id
            - timeout (seconds): for element get timeout
        """
        self.http = httpclient
        self._target = None
        self._timeout = 30.0
        v = self.http.get('/').value
        self.capabilities = v['capabilities']
        self._sid = v['sessionId']
        self.__scale = None
~~~

封装了一些对app的操作

~~~python
    def tap(self, x, y):
        return self.http.post('/wda/tap/0', dict(x=x, y=y))
    def click(self, x, y):
        """
        x, y can be float(percent) or int
        """
        x, y = self._percent2pos(x, y)
        return self.tap(x, y)
    def double_tap(self, x, y):
        x, y = self._percent2pos(x, y)
        return self.http.post('/wda/doubleTap', dict(x=x, y=y))
~~~
最重要的地方是实现了\_\_call\_\_方法，根据传入的可选参数，返回selector对象（定位器）

~~~python
    def __call__(self, *args, **kwargs):
        if 'timeout' not in kwargs:
            kwargs['timeout'] = self._timeout
        httpclient = self.http.new_client('')
        return Selector(httpclient, self, *args, **kwargs)
~~~

## Selector（元素层/定位器）

实例化的过程，只是把用户传入的参数可用部分进行储存
~~~python
class Selector(object):
    def __init__(self,
                 httpclient,
                 session,
                 predicate=None,
                 id=None,        
                 #  ...
                 ):
        self.http = httpclient
        self.session = session

        self.predicate = predicate
        self.id = id
        #  ...
~~~

示例查到是通过\_wdasearch方法，根据using，value发送http请求，获得一个元素id的数组。

~~~python
    def _wdasearch(self, using, value):
        """
        Returns:
            element_ids (list(string)): example ['id1', 'id2']
        
        HTTP example response:
        [
            {"ELEMENT": "E2FF5B2A-DBDF-4E67-9179-91609480D80A"},
            {"ELEMENT": "597B1A1E-70B9-4CBE-ACAD-40943B0A6034"}
        ]
        """
        element_ids = []
        for v in self.http.post('/elements', {
                'using': using,
                'value': value
        }).value:
            element_ids.append(v['ELEMENT'])
        return element_ids
~~~

在find_element_ids方法和_\_\init\_\_方法可以看到Selector是怎么根据用户入参获取元素id的。

~~~python
    def find_element_ids(self):
        elems = []
        if self.id:
            return self._wdasearch('id', self.id)
        if self.predicate:
            return self._wdasearch('predicate string', self.predicate)
        if self.xpath:
            return self._wdasearch('xpath', self.xpath)
        if self.class_chain:
            return self._wdasearch('class chain', self.class_chain)

        chain = '**' + ''.join(
            self.parent_class_chains) + self._gen_class_chain()
        if DEBUG:
            print('CHAIN:', chain)
        return self._wdasearch('class chain', chain)
~~~

实际使用的时候是通过get方法和find_elements方法获取一个或多个元素。元素的实例化过程是在find_elements方法中，而get方法只是获取数组第一个元素。

~~~python
    def get(self, timeout=None, raise_error=True):
        # ...
        while True:
            elems = self.find_elements()
            if len(elems) > 0:
                return elems[0]
            if start_time + timeout < time.time():
                break
            time.sleep(0.01)
        # ...
    def find_elements(self):
        """
        Returns:
            Element (list): all the elements
        """
        es = []
        for element_id in self.find_element_ids():
            e = Element(self.http.new_client(''), element_id)
            es.append(e)
        return es
~~~

Selector还通过实现魔术方法\_\_getattr\_\_ 将自身和Elment对象的边界模糊掉。

~~~python
    def __getattr__(self, oper):
        return getattr(self.get(), oper)
~~~

## Element(元素层/元素)

继续传递httpclient，然后持有一个element_id, 将id和httpclient又做一次封装提供\_req和\_wda\_req方法发送请求
~~~python
class Element(object):
    def __init__(self, httpclient, id):
        """
        base_url eg: http://localhost:8100/session/$SESSION_ID
        """
        self.http = httpclient
        self._id = id

    def __repr__(self):
        return '<wda.Element(id="{}")>'.format(self.id)

    def _req(self, method, url, data=None):
        return self.http.fetch(method, '/element/' + self.id + url, data)

    def _wda_req(self, method, url, data=None):
        return self.http.fetch(method, '/wda/element/' + self.id + url, data)
~~~

另外就是一些对元素的操作，和获取元素的属性的方法。

~~~python
    # operations
    def tap(self):
        return self._req('post', '/click')

    def click(self):
        """ Alias of tap """
        return self.tap()
    
    def set_text(self, value):
        return self._req('post', '/value', {'value': value})

    def clear_text(self):
        return self._req('post', '/clear')
    
    @property
    def enabled(self):
        return self._prop('enabled')

    @property
    def visible(self):
        return self._prop('attribute/visible')
    
    def _prop(self, key):
        return self._req('get', '/' + key.lstrip('/')).value
    
~~~

