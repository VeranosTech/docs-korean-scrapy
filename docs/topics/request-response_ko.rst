.. _topics-request-response:

============================================================
리퀘스트(Requests)와 리스펀스(Responses)
============================================================

.. module:: scrapy.http
   :synopsis: Request and Response classes

스크래피(Scrapy)는 웹사이트 크롤링에 :class:`Request`\ 와 :class:`Response` 객체를 사용한다.

일반적으로, :class:`Request` 객체는 스파이더 내에서 생성되고 시스템을 지나서
다운로더(Downloader)에 도달한다. 다운로더는 리퀘스트를 실행하고
:class:`Response` 객체를 반환하는데 이 객체는 리퀘스트를 생성했던 스파이더로
돌아간다.

:class:`Request`\ 와 :class:`Response` 클래스는 모두 베이스 클래스에서
필요하지 않은 기능을 추가하는 상속클래스를 가지고 있다. 이 기능은 아래에
:ref:`topics-request-response-ref-request-subclasses`\ 와
:ref:`topics-request-response-ref-response-subclasses`\ 에서 설명하고
있다.


리퀘스트 객체
=========================

.. class:: Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback, flags])

    :class:`Request` 객체는 HTTP 리퀘스트를 나타내며, 스파이더에서 생성되고 다운로드에서 실행된다.
    그 결과로 :class:`Response`\ 를 생성한다.

    :param url: 이 리퀘스트의 URL
    :type url: 문자열

    :param callback: 첫 번째 파라미터로서 이 리퀘스트의 (다운로드된) 리스펀스를 받아서
       호출되는 함수. 상세한 정보는 아래의
       :ref:`topics-request-response-ref-request-callback-arguments`\ 를 참고하라.
       리퀘스트가 콜백(callback)을 지정하지 않으면 스파이더의
       :meth:`~scrapy.spiders.Spider.parse` 메서드가 사용될 것이다.
       처리중에 예외가 발셍하면, 에러백(errback)이 대신 호출된다.

    :type callback: 컬러블(callable)

    :param method: 이 리퀘스트의 HTTP 메서드, 디폴트는 ``'GET'``\ 이다.
    :type method: 문자열

    :param meta: :attr:`Request.meta` 속성의 초기 값은. 주어졌다면
       이 파라미터에 전달된 딕셔너리가 쉘로우(shallow) 복사될 것이다.
    :type meta: 딕셔너리

    :param body: 리퀘스트 바디(body). ``unicode``\ 가 전달되면, 전달된
      (``utf-8``\ 로 기본설정되어 있는) `encoding`\ 을
      사용해 ``str``\ 로 인코딩된다. ``body``\ 가 주어지지 않으면, 빈 문자열이 저장된다.
      이 인자의 타입과 상관없이 저장되는 최종적인 값은, ``str``\ 이 된다 (절대
      ``unicode`` 또는 ``None``\ 이 아니다).
    :type body: 문자열 또는 유니코드

    :param headers: 이 리퀘스트의 헤더(header). (단일 헤더인 경우) 딕셔너리 값은 문자열이나
       (다수의 헤더인 경우) 리스트가 될 수 있다. ``None``\ 이 값으로 주어지면, HTTP 헤더는
       아예 보내지지 않을 것이다.
    :type headers: 딕셔너리

    :param cookies: 리퀘스트 쿠키(cookie), 두 가지 형태로 보내질 수 있다.

        1. 딕셔너리 사용::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies={'currency': 'USD', 'country': 'UY'})

        2. 딕셔너리 리스트 사용::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies=[{'name': 'currency',
                                                    'value': 'USD',
                                                    'domain': 'example.com',
                                                    'path': '/currency'}])

        후자의 형태는 쿠키의 ``domain``\ 과 ``path`` 속성을 커스터마이징할 수 있게
        해준다. 이는 쿠키가 추후의 리퀘스트를 위해 저장된 때만 유용하다.

        어떤 사이트가 (리스펀스에서) 쿠키를 반환할 때, 쿠키는 그 도메인을 위한 쿠키에 저장되고
        미래의 리퀘스트에서 다시 보내진다. 이것이 일반적인 웹 브라우저의 전형적인 동작이다.
        그러나 만약에, 어떤 이유로 인해 기존의 쿠키와 병합하는 것을 피하고 싶다면
        :attr:`Request.meta`\ 의 ``dont_merge_cookies`` 키를 True로 설정하면 된다.

        쿠기 병합을 하지 않는 리퀘스트 예시::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies={'currency': 'USD', 'country': 'UY'},
                                           meta={'dont_merge_cookies': True})

        더 자세한 내용은 :ref:`cookies-mw`\ 를 참고하라.
    :type cookies: 리스트 또는 딕셔너리

    :param encoding: 이 리퀘스트의 인코딩. (디폴트는 ``'utf-8'``\ 이다.)
       이 인코딩은 URL를 퍼센트인코드(percent-encode)하고 바디를 (``unicode``\ 로 주어젔다면) ``str``\ 으로
       변환하기 위해 사용된다.
    :type encoding: 문자열

    :param priority: 이 리퀘스트의 우선순위. (디폴트는 ``0``\ 이다.)
       우선순위는 스케쥴러에서 리퀘스트를 처리할 때 사용하는 순서를 정의하기 위해 사용된다.
       높은 우선순위를 가진 리퀘스트는 먼저 실행된다.
       상대적으로 낮은 우선순위를 나타내기위해 음수 값이 허용된다.
    :type priority: 정수

    :param dont_filter: 이 리퀘스트는 스케쥴러에 의해 필터링되지 않음을 나타낸다.
       이는 동일한 리퀘스트에 대해 여러번 작업을 수행하고 중복 필터를 무시하고 싶을 때 사용한다.
       주의해서 사용하지 않으면 크롤링 루프에 빠질 수 있다. 디폴트는 ``False``\ 다.
    :type dont_filter: 불리언(boolean)

    :param errback: 리퀘스트 처리중에 예외가 발생하면 호출되는 함수. 이것은
       404 HTTP 등의 에러가 발생하는 페이지를 포함한다. 이 함수는 `Twisted Failure`_ 인스턴스를
       첫 번째 파라미터로 받는다.
       더 자세한 내용은 :ref:`topics-request-response-ref-errbacks`\ 를 참고하라.
    :type errback: 컬러블

    :param flags:  리퀘스트에 보내지는 플래그(Flag), 로깅(logging)이나 유사한 목적으로 사용될 수 있다.
    :type flags: 리스트

    .. attribute:: Request.url

        이 리퀘스트의 URL을 포함하고 있는 문자열. 이 속성은 이스케이프(escape)된 URL을 포함하고 있으므로
        컨스트럭터(constructo)에 전달된 URL과 다를 수 있다는 점을 명심하라.

        이 속성은 읽기 전용이다. 리퀘스트의 URL을 변경하려면 :meth:`replace`\ 를 사용하라라.

    .. attribute:: Request.method

        리퀘스트의 HTTP 메서드를 나타내는 문자열. 대문자로만 표현된다. 에: ``"GET"``, ``"POST"``, ``"PUT"`` 등

    .. attribute:: Request.headers

        리퀘스트 헤더를 포함하는 딕셔터리 형태의 객체.

    .. attribute:: Request.body

        리퀘스트 바디를 포함하는 문자열.

        이 속성은 읽기 전용이다. 리퀘스트의 바디를 변경하고 싶으면 :meth:`replace`\ 를 사용하라.

    .. attribute:: Request.meta

        이 리퀘스트의 임의의 메타데이터를 포함하는 딕셔너리. 이 딕셔너리는 새로운 리퀘스트를 위해 비어있고,
        일반적으로 다른 스크래피 구성요소(확장, 미들웨어)에 의해 추가된다. 따라서
        이 사전에 포함된 데이터는 활성화시킨 확장에 의존한다.

        스크래피가 인식하는 특수 메타 키 리스트에 관해서는 :ref:`topics-request-meta`\ 를 참고하라.

        이 딕셔너리는 리퀘스트가 ``copy()`` 또는 ``replace()`` 메소드를 사용해 복제될 때 `쉘로우 복사`_\ 되며
        ``response.meta`` 속성으로 스파이더에서 접근 할 수 있다.

    .. _쉘로우 복사: https://docs.python.org/2/library/copy.html

    .. method:: Request.copy()

       이 리퀘스트의 복사본인 새로운 리퀘스트를 반환한다.
       :ref:`topics-request-response-ref-request-callback-arguments`\ 를 참고하라.

    .. method:: Request.replace([url, method, headers, body, cookies, meta, encoding, dont_filter, callback, errback])

       키워드 인자로 지정해서 새로운 값이 주어진 멤버를 제외하고 같은 멤버를 포함한 리퀘스트 객체를 반환한다.
       (새로운 값이 ``meta`` 인자로 주어지지 않으면) :attr:`Request.meta` 속성은 기본적으로 복사된다.
       :ref:`topics-request-response-ref-request-callback-arguments`\ 를 참고하라.

.. _topics-request-response-ref-request-callback-arguments:

추가 데이터를 콜백 함수로 전달
------------------------------------------------------

리퀘스트의 콜백은 리퀘스트의 리스펀스가 다운로드 되었을 때 호출되는 함수다.
콜백 함수는 첫 번째 인자로 다운로드된 :class:`Response` 객체를 받으면서 호출된다.

예::

    def parse_page1(self, response):
        return scrapy.Request("http://www.example.com/some_page.html",
                              callback=self.parse_page2)

    def parse_page2(self, response):
        # this would log http://www.example.com/some_page.html
        self.logger.info("Visited %s", response.url)

종종 콜백 함수에 인자를 전달해서 나중에 두 번째 콜백에서 인자를 받게 하고 싶을 때가 있을 것이다.
이를 위해서는 :attr:`Request.meta` 속성을 사용하면 된다.

아래는 다른 페이지로부터 다른 필드를 추가하기 위해 이 메커니즘을 사용해서 아이템을 전달한
예시이다::

    def parse_page1(self, response):
        item = MyItem()
        item['main_url'] = response.url
        request = scrapy.Request("http://www.example.com/some_page.html",
                                 callback=self.parse_page2)
        request.meta['item'] = item
        yield request

    def parse_page2(self, response):
        item = response.meta['item']
        item['other_url'] = response.url
        yield item


.. _topics-request-response-ref-errbacks:

리퀘스트 처리중 예외를 잡기 위한 에러백 사용
--------------------------------------------------------------------

리퀘스트의 에러백은 처리중에 예외가 발생했을 때 호출되는 함수다.

이 함수는 `Twisted Failure`_ 인스턴스를 첫 번째 파라미터로 받으며
연결 설정 시간 초과, DNS 에러 등을 추적하기 위해 사용된다.

아래는 모든 에러를 로깅하고 필요한 경우 특정한 에러를 잡아내는 스파이더 예시다::

    import scrapy

    from scrapy.spidermiddlewares.httperror import HttpError
    from twisted.internet.error import DNSLookupError
    from twisted.internet.error import TimeoutError, TCPTimedOutError

    class ErrbackSpider(scrapy.Spider):
        name = "errback_example"
        start_urls = [
            "http://www.httpbin.org/",              # HTTP 200 expected
            "http://www.httpbin.org/status/404",    # Not found error
            "http://www.httpbin.org/status/500",    # server issue
            "http://www.httpbin.org:12345/",        # non-responding host, timeout expected
            "http://www.httphttpbinbin.org/",       # DNS error expected
        ]

        def start_requests(self):
            for u in self.start_urls:
                yield scrapy.Request(u, callback=self.parse_httpbin,
                                        errback=self.errback_httpbin,
                                        dont_filter=True)

        def parse_httpbin(self, response):
            self.logger.info('Got successful response from {}'.format(response.url))
            # do something useful here...

        def errback_httpbin(self, failure):
            # log all failures
            self.logger.error(repr(failure))

            # in case you want to do something special for some errors,
            # you may need the failure's type:

            if failure.check(HttpError):
                # these exceptions come from HttpError spider middleware
                # you can get the non-200 response
                response = failure.value.response
                self.logger.error('HttpError on %s', response.url)

            elif failure.check(DNSLookupError):
                # this is the original request
                request = failure.request
                self.logger.error('DNSLookupError on %s', request.url)

            elif failure.check(TimeoutError, TCPTimedOutError):
                request = failure.request
                self.logger.error('TimeoutError on %s', request.url)

.. _topics-request-meta:

Request.meta 특수 키
=================================

:attr:`Request.meta` 속성은 임의의 데이터를 포함할 수 있다, 하지만
스크래피와 빌트인 확장에서 인식되는 몇 가지 특수 키가 존재한다.

특수 키:

* :reqmeta:`dont_redirect`
* :reqmeta:`dont_retry`
* :reqmeta:`handle_httpstatus_list`
* :reqmeta:`handle_httpstatus_all`
* ``dont_merge_cookies`` (:class:`Request` 컨스트럭트의 ``cookies`` 파라미터를 참고하라)
* :reqmeta:`cookiejar`
* :reqmeta:`dont_cache`
* :reqmeta:`redirect_urls`
* :reqmeta:`bindaddress`
* :reqmeta:`dont_obey_robotstxt`
* :reqmeta:`download_timeout`
* :reqmeta:`download_maxsize`
* :reqmeta:`download_latency`
* :reqmeta:`download_fail_on_dataloss`
* :reqmeta:`proxy`
* ``ftp_user`` (더 자세한 정보는 :setting:`FTP_USER`\ 를 참고하라)
* ``ftp_password`` (더 자세한 정보는 :setting:`FTP_PASSWORD`\ 를 참고하라)
* :reqmeta:`referrer_policy`
* :reqmeta:`max_retry_times`

.. reqmeta:: bindaddress

bindaddress
-----------

리퀘스트를 수행할 대 사용되는 발신 IP 주소의 IP.

.. reqmeta:: download_timeout

download_timeout
----------------

타임 아웃하기 전에 다운로더가 대기하는 (초 단위) 시간.
:setting:`DOWNLOAD_TIMEOUT`\ 를 참고하라.

.. reqmeta:: download_latency

download_latency
----------------

요청이 시작된 이후, 리스펀스를 불러오기 위해 소모되는 시간. 예, 네트워크를 통해 전송되는 메시지.
이 메타기는 리스펀스가 다운로드 되었을 때만 사용할 수 있다. 대부분의 다른 메타키는 스크래피의 동작을
제어하기 위해 사용되지만 이 키는 읽기 전용이다.

.. reqmeta:: download_fail_on_dataloss

download_fail_on_dataloss
-------------------------

깨진 응답에 대해 실패할지 여부.
:setting:`DOWNLOAD_FAIL_ON_DATALOSS`\ 를 참고하라.

.. reqmeta:: max_retry_times

max_retry_times
---------------

이 메타 키는 리퀘스트 당 재시도 횟수를 설정한다.
초기화 됐을 때, :reqmeta:`max_retry_times` 메타키는 :setting:`RETRY_TIMES` 설정보다 우선한다.

.. _topics-request-response-ref-request-subclasses:

Request subclasses
==================

이 섹션에는 :class:`Request`\ 의 빌트인 상속 클래스 리스트가 있다.
사용자는 커스텀 기능을 구현하기위해서 아래의 클래스를 상속받을 수도 있다.

FormRequest 객체
-------------------

FormRequest 클래스는 기본 :class:`Request`\ 에 HTML 형식을 처리하는 기능을 추가한다.
이 클래스는 :class:`Response` 객체의 형식 데이터가 있는 형식 필드를 사전에 추가하기 위해
`lxml.html forms`_\ 를 사용한다.

.. _lxml.html forms: http://lxml.de/lxmlhtml.html#forms

.. class:: FormRequest(url, [formdata, ...])

    :class:`FormRequest` 클래스는 컨스트럭터에 새로운 인자를 추가한다.
    나머지 인자는 :class:`Request` 클래스와 같으며 이곳에 문서화하지 않았다.

    :param formdata: 파라미터는 url 인코딩된 후 리퀘스트의 바디에 할당되는 HTML 형식 데이터를 포함하는
       딕셔너리(또는 (키, 값) 튜플의 이터러블)다.
    :type formdata: 딕셔너리 또는 튜플의 이터러블(iterable)

    :class:`FormRequest` 객체는 기존 :class:`Request` 메서드를 포함해
    아래의 클래스 메서드를 지원한다:

    .. classmethod:: FormRequest.from_response(response, [formname=None, formid=None, formnumber=0, formdata=None, formxpath=None, formcss=None, clickdata=None, dont_click=False, ...])

       주어진 리스펀스에 포함된 HTML ``<form>`` 요소에서 찾아진 형식 필드 값으로 사전에 채워진
       새로운 :class:`FormRequest` 객체를 반환한다. 예시는
       :ref:`topics-request-response-ref-request-userlogin`\ 를 참고하라.

       기본적으로 정책은 ``<input type="submit">`` 같이 클릭할 수 있는 모든 형식 컨트롤에 대한
       클릭을 자동적으로 시뮬레이션하는 것이다. 이것은 꽤 편리하고 때로는 원하는 동작이기는 하지만
       때로는 디버그를 하기 힘든 문제를 일으킬 수 있다.
       예를 들어, 자바스크립트(javascript)를 사용해서 제출되거나 채워진 형식으로 작업을 할 때
       기본 :meth:`from_response` 동작은 가장 적합한 것이 아닐 수 있다.
       ``dont_click``\ 을 ``True``\ 로 설정해서 이 동작을 비활성화할 수 있다.
       또한, 클릭되는 컨트롤을 (비활성화하는 대신) 변경하고 싶다면 ``clickdata`` 인자를 사용하면 된다.

       .. caution:: 옵션 값에 앞이나 뒤에 공백이 있는 셀렉트 요소에 이 메서드를 사용하면
          `bug in lxml`_ 때문에 작동하지 않는다. 이 버그는 lxml 3.8 이상 버전에서 고쳐져야 한다.

       :param response: 형식 필드를 사전에 채우기위해 사용되는 HTML 형식을 포함하는 리스펀스.
       :type response: :class:`Response` 객체

       :param formname: 주어지는 경우, 이름 속성이 이 값으로 지정된 형식이 사용된다.
       :type formname: 문자열

       :param formid: 주어진 경우, id 속성이 이 값으로 지정된 형식이 사용된다.
       :type formid: 문자열

       :param formxpath: 주어진 경우, xpath에 첫 번쨰로 매치된 형식이 사용된다.
       :type formxpath: 문자열

       :param formcss: 주어진 경우, ccs 셀렉터(selector)에 첫 번째로 매치된 형식이 사용된다.
       :type formcss: 문자열

       :param formnumber: 리스펀스가 다수의 형식을 포함하고 있을 때 사용할 형식의 수.
           첫 번째는 (또한 기본값은) ``0``\ 이다.
       :type formnumber: 정수

       :param formdata: 형식 데이터 내에서 오버라이드(override)할 필드.
          만약 필드가 이미 리스펀스의 ``<form>`` 요소에 존재한다면, 이 파라미터에
          전달된 값으로 오버라이드 된다. 이 파라미터에 전달된 값이 ``None``\ 이면,
          리스펀스의 ``<form>`` 요소에 값이 존재하더라도 필드는 리퀘스트에 포함되지 않을 것이다.
       :type formdata: 딕셔너리

       :param clickdata: 클릭된 컨트롤을 찾는 속성. 주어지지 않은 경우
         형식 데이터는 클릭가능한 첫 번째 요소를 클릭하는 것을 시뮬레이션하면서 제출된다.
         html 속성에 외에도, 컨트롤은 형식 내에 다른 제출가능한 입력에 관련된 제로 베이스(zero-based)
         인덱스에 의해 ``nr`` 속성을 통해서 식별될 수 있다.
       :type clickdata: 딕셔너리

       :param dont_click: 참인 경우, 형식 데이터는 요소 클릭 없이 제출될 것이다.
       :type dont_click: 불리언

       이 클래스 메서드의 다른 파라미터는 :class:`FormRequest` 컨스트럭터로 바로 전달된다.

       .. versionadded:: 0.10.3
          The ``formname`` parameter.

       .. versionadded:: 0.17
          The ``formxpath`` parameter.

       .. versionadded:: 1.1.0
          The ``formcss`` parameter.

       .. versionadded:: 1.1.0
          The ``formid`` parameter.

리퀘스트 사용 예시
---------------------------

HTTP POST를 통한 데이터 전송을 위한 FormRequest 사용
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

HTML 형식 POST를 스파이더 내에서 시뮬레이션하고 여러 키-값 필드를 전송하고 싶으면
(스파이더에서) 아래처럼 :class:`FormRequest` 객체를 반환하면 된다::

   return [FormRequest(url="http://www.example.com/post/action",
                       formdata={'name': 'John Doe', 'age': '27'},
                       callback=self.after_post)]

.. _topics-request-response-ref-request-userlogin:

사용자 로그인을 시뮬레이션하기 위한 FormRequest.from_response() 사용
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

웹사이트는 데이터와 연관된 세션이나 (로그인 페이지를 위한) 토큰 인증 같은 ``<input type="hidden">`` 요소를 통해서
사전에 채워진 형식 필드를 제공하는 것이 일반적이다. 스크랩을 할 때, 이 필드들이 자동적으로 채워지고
사용자 이름이나 패스와드 같은, 일부 필드만 오버라이드하기를 원할 수 있다.
이런 작업을 위해서는 :meth:`FormRequest.from_response`
method for this job. Here's an example spider which uses it::


    import scrapy

    class LoginSpider(scrapy.Spider):
        name = 'example.com'
        start_urls = ['http://www.example.com/users/login.php']

        def parse(self, response):
            return scrapy.FormRequest.from_response(
                response,
                formdata={'username': 'john', 'password': 'secret'},
                callback=self.after_login
            )

        def after_login(self, response):
            # check login succeed before going on
            if "authentication failed" in response.body:
                self.logger.error("Login failed")
                return

            # continue scraping with authenticated session...


Response objects
================

.. class:: Response(url, [status=200, headers=None, body=b'', flags=None, request=None])

    A :class:`Response` object represents an HTTP response, which is usually
    downloaded (by the Downloader) and fed to the Spiders for processing.

    :param url: the URL of this response
    :type url: string

    :param status: the HTTP status of the response. Defaults to ``200``.
    :type status: integer

    :param headers: the headers of this response. The dict values can be strings
       (for single valued headers) or lists (for multi-valued headers).
    :type headers: dict

    :param body: the response body. To access the decoded text as str (unicode
       in Python 2) you can use ``response.text`` from an encoding-aware
       :ref:`Response subclass <topics-request-response-ref-response-subclasses>`,
       such as :class:`TextResponse`.
    :type body: bytes

    :param flags: is a list containing the initial values for the
       :attr:`Response.flags` attribute. If given, the list will be shallow
       copied.
    :type flags: list

    :param request: the initial value of the :attr:`Response.request` attribute.
        This represents the :class:`Request` that generated this response.
    :type request: :class:`Request` object

    .. attribute:: Response.url

        A string containing the URL of the response.

        This attribute is read-only. To change the URL of a Response use
        :meth:`replace`.

    .. attribute:: Response.status

        An integer representing the HTTP status of the response. Example: ``200``,
        ``404``.

    .. attribute:: Response.headers

        A dictionary-like object which contains the response headers. Values can
        be accessed using :meth:`get` to return the first header value with the
        specified name or :meth:`getlist` to return all header values with the
        specified name. For example, this call will give you all cookies in the
        headers::

            response.headers.getlist('Set-Cookie')

    .. attribute:: Response.body

        The body of this Response. Keep in mind that Response.body
        is always a bytes object. If you want the unicode version use
        :attr:`TextResponse.text` (only available in :class:`TextResponse`
        and subclasses).

        This attribute is read-only. To change the body of a Response use
        :meth:`replace`.

    .. attribute:: Response.request

        The :class:`Request` object that generated this response. This attribute is
        assigned in the Scrapy engine, after the response and the request have passed
        through all :ref:`Downloader Middlewares <topics-downloader-middleware>`.
        In particular, this means that:

        - HTTP redirections will cause the original request (to the URL before
          redirection) to be assigned to the redirected response (with the final
          URL after redirection).

        - Response.request.url doesn't always equal Response.url

        - This attribute is only available in the spider code, and in the
          :ref:`Spider Middlewares <topics-spider-middleware>`, but not in
          Downloader Middlewares (although you have the Request available there by
          other means) and handlers of the :signal:`response_downloaded` signal.

    .. attribute:: Response.meta

        A shortcut to the :attr:`Request.meta` attribute of the
        :attr:`Response.request` object (ie. ``self.request.meta``).

        Unlike the :attr:`Response.request` attribute, the :attr:`Response.meta`
        attribute is propagated along redirects and retries, so you will get
        the original :attr:`Request.meta` sent from your spider.

        .. seealso:: :attr:`Request.meta` attribute

    .. attribute:: Response.flags

        A list that contains flags for this response. Flags are labels used for
        tagging Responses. For example: `'cached'`, `'redirected`', etc. And
        they're shown on the string representation of the Response (`__str__`
        method) which is used by the engine for logging.

    .. method:: Response.copy()

       Returns a new Response which is a copy of this Response.

    .. method:: Response.replace([url, status, headers, body, request, flags, cls])

       Returns a Response object with the same members, except for those members
       given new values by whichever keyword arguments are specified. The
       attribute :attr:`Response.meta` is copied by default.

    .. method:: Response.urljoin(url)

        Constructs an absolute url by combining the Response's :attr:`url` with
        a possible relative url.

        This is a wrapper over `urlparse.urljoin`_, it's merely an alias for
        making this call::

            urlparse.urljoin(response.url, url)

    .. automethod:: Response.follow


.. _urlparse.urljoin: https://docs.python.org/2/library/urlparse.html#urlparse.urljoin

.. _topics-request-response-ref-response-subclasses:

Response subclasses
===================

Here is the list of available built-in Response subclasses. You can also
subclass the Response class to implement your own functionality.

TextResponse objects
--------------------

.. class:: TextResponse(url, [encoding[, ...]])

    :class:`TextResponse` objects adds encoding capabilities to the base
    :class:`Response` class, which is meant to be used only for binary data,
    such as images, sounds or any media file.

    :class:`TextResponse` objects support a new constructor argument, in
    addition to the base :class:`Response` objects. The remaining functionality
    is the same as for the :class:`Response` class and is not documented here.

    :param encoding: is a string which contains the encoding to use for this
       response. If you create a :class:`TextResponse` object with a unicode
       body, it will be encoded using this encoding (remember the body attribute
       is always a string). If ``encoding`` is ``None`` (default value), the
       encoding will be looked up in the response headers and body instead.
    :type encoding: string

    :class:`TextResponse` objects support the following attributes in addition
    to the standard :class:`Response` ones:

    .. attribute:: TextResponse.text

       Response body, as unicode.

       The same as ``response.body.decode(response.encoding)``, but the
       result is cached after the first call, so you can access
       ``response.text`` multiple times without extra overhead.

       .. note::

            ``unicode(response.body)`` is not a correct way to convert response
            body to unicode: you would be using the system default encoding
            (typically `ascii`) instead of the response encoding.


    .. attribute:: TextResponse.encoding

       A string with the encoding of this response. The encoding is resolved by
       trying the following mechanisms, in order:

       1. the encoding passed in the constructor `encoding` argument

       2. the encoding declared in the Content-Type HTTP header. If this
          encoding is not valid (ie. unknown), it is ignored and the next
          resolution mechanism is tried.

       3. the encoding declared in the response body. The TextResponse class
          doesn't provide any special functionality for this. However, the
          :class:`HtmlResponse` and :class:`XmlResponse` classes do.

       4. the encoding inferred by looking at the response body. This is the more
          fragile method but also the last one tried.

    .. attribute:: TextResponse.selector

        A :class:`~scrapy.selector.Selector` instance using the response as
        target. The selector is lazily instantiated on first access.

    :class:`TextResponse` objects support the following methods in addition to
    the standard :class:`Response` ones:

    .. method:: TextResponse.xpath(query)

        A shortcut to ``TextResponse.selector.xpath(query)``::

            response.xpath('//p')

    .. method:: TextResponse.css(query)

        A shortcut to ``TextResponse.selector.css(query)``::

            response.css('p')

    .. automethod:: TextResponse.follow

    .. method:: TextResponse.body_as_unicode()

        The same as :attr:`text`, but available as a method. This method is
        kept for backwards compatibility; please prefer ``response.text``.


HtmlResponse objects
--------------------

.. class:: HtmlResponse(url[, ...])

    The :class:`HtmlResponse` class is a subclass of :class:`TextResponse`
    which adds encoding auto-discovering support by looking into the HTML `meta
    http-equiv`_ attribute.  See :attr:`TextResponse.encoding`.

.. _meta http-equiv: https://www.w3schools.com/TAGS/att_meta_http_equiv.asp

XmlResponse objects
-------------------

.. class:: XmlResponse(url[, ...])

    The :class:`XmlResponse` class is a subclass of :class:`TextResponse` which
    adds encoding auto-discovering support by looking into the XML declaration
    line.  See :attr:`TextResponse.encoding`.

.. _Twisted Failure: https://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html
.. _bug in lxml: https://bugs.launchpad.net/lxml/+bug/1665241
