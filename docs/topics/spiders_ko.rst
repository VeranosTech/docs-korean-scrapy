.. _topics-spiders:

==================
스파이더(Spider)
==================

스파이더는 특정 사이트(또는 사이트 그룹)을 어떻게 스크랩할 건지 정의하는 클래스이며,
크롤링을(예, 링크 따라가기 등) 수행하는 방식과, 페이지에서 구조화된 데이터(스크랩되는 아이템)를
추출하는 방식도 포함하고 있다. 즉, 스파이더는 사용자가 특정한 사이트(또는 사이트 그룹)를 파싱하고 크롤링 하기
위한 커스텀 행동을 정의하는 곳이다.

스파이더의 스크랩 사이클은 아래와 같이 진행된다:

1. 첫 번째 URL을 크롤링 하는 최초의 리퀘트르르 생성함으로써 시작하고,
   리퀘스트로부터 다운로드된 리스펀스와 호출되는 콜백 함수를 지정한다.

   수행될 첫 번째 리퀘스트는
   (기본적으로) :attr:`~scrapy.spiders.Spider.start_urls`\ 에서 지정되는 URL을 위해
   :class:`~scrapy.http.Request`\ 를 생성하는 :meth:`~scrapy.spiders.Spider.start_requests` 메서드와
   리퀘스트를 위한 콜백 함수로써 :attr:`~scrapy.spiders.Spider.parse` 메서드를 호출함으로써 얻어진다.

2. 콜백 함수에서, 리스펀스(웹 페이지)를 파싱하고 추출된 데이터가 있는 dict와
   :class:`~scrapy.item.Item` 객체, :class:`~scrapy.http.Request` 객체,
   또는 반복가능한 이러한 객체들을 반환한다.
   반환되는 리리퀘스트들도 콜백 함수를 포함하고 있으며 (일반적으로 자기자신)
   스크래피로 다운로드된 후 리스펀스는 콜백 함수에 의해 처리된다.

3. 콜백 함수는 페이지 컨텐츠를 일반적으로 :ref:`topics-selectors`\ 를 사용해서
   파싱하고(BeautifuulSoup, lxml 등 원하는 어떤 메커니즘을 사용해도 상관없다)
   파싱된 데이터로 이루어진 아이템을 생성한다.

4. 최종적으로 스파이더에서 반환된 아이템을 일반적으로 데이터베이스에서 유지되거나
   (:ref:`Item Pipeline <topics-item-pipeline>`\ 을 통해서) 또는 :ref:`topics-feed-exports`\ 를
   사용해서 파일로 작성된다.

이 사이클은 모든 종류의 스파이더에 적용되지만, 스크래피에 번들로 포함된 다른 목적을 위한
다른 종류의 스파이더도 있다. 앞으로 이 타입에 대해서 다루어볼 것이다.

.. module:: scrapy.spiders
   :synopsis: Spiders base class, spider manager and spider middleware

.. _topics-spiders-ref:

scrapy.Spider
=============

.. class:: Spider()

   이것은 가장 간단한 스파이더이며 다른 모든 스파이더들이 반드시 상속받아야 하는 것 중에 하나다.
   (직접 제작한 스파이더뿐만 아니라 스크래피와 번들로 제공되는 스파이더 포함)
   이 스파이더는 특별한 기능을 제공하지는 않는다.
   이것은 단지 :attr:`start_urls`\ 스파이더 속성에서 리퀘스트를 보내는 :meth:`start_requests` 구현을
   제공하고 얻어진 각 결과의 리스펀스를 위한 스파이더의 메서드 ``parse``\ 를 호출한다.

   .. attribute:: name

       스파이더의 이름을 정의하는 문자열. 스파이더 이름은 스크래피에 의해
       찾아지고 (그리고 인스턴스화되는) 방식이므로, 따라서 반드시 유일해야 한다.
       그러나 같은 스파이더를 하나 이상 인스턴스화하는 것은 가능하다ㅏ.
       이것은 스파이더 속성중에서 가장 중요한 것이며 꼭 필요하다.

       스파이더가 단일 도메인을 스크랩한다면, 일반적인 관습은 `TLD`_\ 를 포함하거나
       포함하지 않은 도메인을 따서 스파이더 이름을 짓는 것이다. 예를 들면,
       ``mywebsite.com``\ 를 크롤링하는 스파이더는 ``mywebsite``\ 라고 하는
       것이다.

       .. note:: 파이썬 2에서는 ASCII만 가능하다.

   .. attribute:: allowed_domains

       스파이더의 크롤링을 허용하는 도메인을 포함한 선택적인 문자열 리스트다.
       이 리스트에서 지정된 도메인 이름(또는 하위 도메인까지)에 속하지 않은 URL에 대한 리퀘스트는
       :class:`~scrapy.spidermiddlewares.offsite.OffsiteMiddleware`\ 가 활성화되어 있으면
       따라가지지 않는다.

       만약 타겟 url이 ``https://www.example.com/1.html``\ 면,
       ``'example.com'``\ 를 이 리스트에 추가하라.

   .. attribute:: start_urls

       특별히 지정된 URL이 없으면 스파이더가 크롤링을 시작할 URL 리스트다.
       따라서, 처 번째로 다운로드되는 페이지는 여기에 리스팅될 것이다.
       후속 URL은 시작 URL에 포함된 데이터로부터 연속해서 생성될 것이다.

   .. attribute:: custom_settings

      스파이더를 실행할 때 전체 프로젝트 설정에서 덮어쓰여질 세팅의 딕셔너리.
      인스턴스화하기 전에 세팅이 업데이트 되기 때문에 클래스 속성으로 정의되어야 한다.

      이용가능한 빌트인 세팅 리스트는 다음을 참고하라:
      :ref:`topics-settings-ref`.

   .. attribute:: crawler

      이 속성은 클래스를 초기화한 이후의 :meth:`from_crawler` 클래스 메서드에 의해 설정되고
      스파이더의 인스턴스가 묶이는 :class:`~scrapy.crawler.Crawler` 객체로 연결된다.

      Crawler는 단일 엔트리 접근(확장, 미들웨어, 시그널 관리, 등)을 위해 프로젝트 내부의
      많은 구성요소들을 캡슐화하고 있다.
      더 알고 싶은 경우 :ref:`topics-api-crawler`\ 를 참고하라.

   .. attribute:: settings

      스파이더 실행에 관한 설정. 이것은
      :class:`~scrapy.settings.Settings` 인스턴스다,
      이 주제에 대한 자세한 사항은 :ref:`topics-settings`\ 를 참고하라.

   .. attribute:: logger

      스파이더의 :attr:`name`\ 과 함께 생성된 파이썬 logger.
      :ref:`topics-logging-from-spiders`\ 에 나온대로 이것을 사용해서 로그 메세지를
      보낼 수 있다.

   .. method:: from_crawler(crawler, \*args, \**kwargs)

       스크래피가 스파이더를 만들기위해 사용하는 클래스 메서드.

       직접 이것을 오버라이드할 필요는 없을 것이다, 왜냐하면 기본 구현이
       :meth:`__init__` 메서드에 대한 프록시로서 주어진 인자 `args`\ 와 키워드 인자
       `kwargs`\ 를 호출하면서 작동하기 때문이다.

       그럼에도 불구하고, 이 메서드는 새 인스턴스의 :attr:`crawler`\ 와 :attr:`settings` 속성을
       지정해서 나중에 스파이더 코드 내에서 접근 가능하게 만든다.

       :param crawler: 스파이더와 바인드될 크롤러
       :type crawler: :class:`~scrapy.crawler.Crawler` 인스턴스

       :param args: :meth:`__init__` 메서드에 전달될 인자
       :type args: 리스트

       :param kwargs: :meth:`__init__` 메서드에 전달될 키워드 인자
       :type kwargs: 딕셔너리

   .. method:: start_requests()

       이 메서드는 스파이더가 크롤링할 첫 번째 리퀘스트가 있는 이터러블을 반환해야 된다.
       스크랩을 위해서 스파이더가 열렸을 때 스크래피에 의해서 호출된다.
       스크래피는 이것을 한 번만 호출해서 :meth:`start_requests`\ 를
       제네레이터로 구현해도 안전하다.

       기본 구현은 :attr:`start_urls`\ 에 있는 각 url에 대한 ``Request(url, dont_filter=True)``\ 를
       생성한다.

       만약 도메인 스크랩핑을 시작하기위해 사용되는 리퀘스트를 변경하고 싶으면,
       이 메서드를 오버라이드 하면 된다. 예를 들어, POST 리퀘스트를 사용해서 로그인하면서 시작해야
       한다면 아래와 같이 쓸 수 있다::

           class MySpider(scrapy.Spider):
               name = 'myspider'

               def start_requests(self):
                   return [scrapy.FormRequest("http://www.example.com/login",
                                              formdata={'user': 'john', 'pass': 'secret'},
                                              callback=self.logged_in)]

               def logged_in(self, response):
                   # here you would extract links to follow and return Requests for
                   # each of them, with another callback
                   pass

   .. method:: parse(response)

       리퀘스트가 콜백을 지정하지 않았을 때, 스크래피가 다운로드된
       리스펀스를 처리하기 위해 사용하는 기본 콜백이다.

       ``parse`` 메소드는 리스펀스를 처리하고 스크랩된 데이터와 추가적으로 따라갈 URL을
       반환하는 역할을 맡고 있다. 다른 리퀘스트 콜백도 :class:`Spider` 클래스와 동일한
       요구사항을 가지고 있다.

       다른 리퀘스트 콜백뿐만 아니라 이 메서드도 :class:`~scrapy.http.Request` 또는 딕셔너리,
       :class:`~scrapy.item.Item` 객체 이터러블을 반환해야 한다.

       :param response: 파싱할 리스펀스
       :type response: :class:`~scrapy.http.Response`

   .. method:: log(message, [level, component])

       스파이더의 :attr:`logger`\ 를 통해 로그 메세지를 보내는 래퍼(Wrapper),
       백워드 호환성을 위해 유지됨. 더 자세한 정보는
       :ref:`topics-logging-from-spiders` 참고하라.

   .. method:: closed(reason)

       스파이더가 종료될 때 호출된다. 이 메서드는 :signal:`spider_closed` 시그널을 위해
       signals.connect()에 대한 숏컷을 제공한다.

예시를 보자::

    import scrapy


    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            self.logger.info('A response from %s just arrived!', response.url)

하나의 콜백으로부터 여러 리퀘스트와 아이템을 반환한다::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield {"title": h3}

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

:attr:`~.start_urls` 대신 :meth:`~.start_requests`\ 를 직접 사용할 수 있다;
데이터를 더 구조화하기 위해서 :ref:`topics-items`\ 를 사용할 수 있다::

    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

.. _spiderargs:

스파이더 인자
================

스파이더는 스파이더의 작동을 변경시킬 수 있는 인자를 받는다.
일반적으로는 시작 URL을 정의하거나 사이트의 특정 섹션을 크롤링하도록 제한하는 데
사용하지만 스파이더의 기능을 설정하는 데도 사용된다.

스파이더 인자는 :command:`crawl` 커맨드의 ``-a`` 옵션을 사용해서 전달된다. 예를 들면::

    scrapy crawl myspider -a category=electronics

스파이더는 `__init__` 메서드의 인자에 접근할 수 있다::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
            super(MySpider, self).__init__(*args, **kwargs)
            self.start_urls = ['http://www.example.com/categories/%s' % category]
            # ...

기본 `__init__` 메서드는 모든 스파이더 인자를 받아서 스파이더에
속성으로 복사하는 것이다.
위의 예제는 아래와 같이 작성될 수도 있다::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/categories/%s' % self.category)

스파이더 인자는 문자열이라는 사실을 명심하라.
스파이더는 알아서 파싱을 하지 않는다.
만약 `start_urls` 속성을 커맨드라인에서 설정하려면
직접 `ast.literal_eval <https://docs.python.org/library/ast.html#ast.literal_eval>`_ 또는
`json.loads <https://docs.python.org/library/json.html#json.loads>`_ 같은 것을 활용해
리스트로 파싱을 한 다음 속성으로 지정해야 한다.
그렇지 않으면, `start_urls` 문자열에 대해서 문자 각각을 별개의 url로 인식하는 이터레이션 (매우
일반적인 파이썬 함정)을 발생킬 수 있다.

유요한 사용 케이스는 :class:`~scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware`\ 가 사용하는
사용되는 http 인증 자격증명이나 :class:`~scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware`\ 가
사용하는 사용자 에이전트를 설정하는 경우다.

    scrapy crawl myspider -a http_user=myuser -a http_pass=mypassword -a user_agent=mybot

스파이더 인자는 Scrapyd ``schedule.json`` API를 통해서 전달될 수도 있다.
`Scrapyd documentation`_\ 를 참고하라

.. _builtin-spiders:

범용 스파이더
==================

스크래피는 사용자의 스파이더가 상속하는 데 사용할 수 있는 유용한 범용 스파이더를 제공한다.
그것들의 목표는 몇가지 일반적인 스크랩핑의 경우를 위한 편리한 기능을 제공하는 것이다.
일반적인 경우는 특정한 룰에 따라 사이트에 있는 모든 링크를 따라가거나, `Sitemaps`_\ 에서
크롤링 하거나, 또는 XML/CSV 피드를 파싱하는 경우를 말한다.

아래의 스파이더에서 사용되는 예시의 경우, ``myproject.items`` 모듈에 ``TestItem``\ 이 선언된
프로젝트가 있다고 가정한다::

    import scrapy

    class TestItem(scrapy.Item):
        id = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()


.. currentmodule:: scrapy.spiders

CrawlSpider
-----------

.. class:: CrawlSpider

   일반적인 웹사이트를 크롤링하는데 가장 보편적으로 사용되는 것으로,
   규칙을 정의함으로써 링크를 따라가는 데 편리한 메커니즘을 제공한다.
   특정한 웹사이트나 프로젝트에 가장 적합한 스파이더는 아닐 수 있지만
   여러 경우에 일반적으로 쓰기에는 충분하다. 그러므로 이 스파이더로 시작해서
   커스텀 기능이 필요한 경우는 오버라이드 하거나 직접 구현하면 된다.

   스파이더에서 상속받은 속성 외에도 이 클래스는 새 속성을 지원한다:

   .. attribute:: rules

       하나 이상의 :class:`Rule` 객체 리스트. 각각의 :class:`Rule`\ 은
       사이트 크롤링에 관한 특정한 작동을 정의한다.같은 링크에
       여러 rule이 매칭된다면, 속성에서 정의된 순서에 따라 첫 번째가 사용된다.

   이 스파이더는 오버라이드 가능한 메서드도 노출하고 있다:

   .. method:: parse_start_url(response)

      이 메서드는 start_urls 리스펀스를 위해서 호출된다. 이것은
      최초 리스펀스를 파싱하게 해주고 반드시
      :class:`~scrapy.item.Item` 객체나, :class:`~scrapy.http.Request`
      객체, 또는 둘 중 하나를 포함하는 이터러블을 반환해야 한다.

Crawling rules
~~~~~~~~~~~~~~

.. class:: Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

   ``link_extractor``\ 는 크롤링된 페이지로부터 링크를 어떻게 추출할 것인지
   정의하는 :ref:`Link Extractor <topics-link-extractors>` 객체다.

   ``callback``\ 는 지정된 link_extractor로 추출된 각 링크를 위해 호출되는 callable 또는 문자열(이 경우
   그 이름을 가진 스파이더 객체의 메서드가 사용된다)이다. 이 콜백은
   첫 인자로 리스펀스를 받으며 반드시 :class:`~scrapy.item.Item` 또는
   :class:`~scrapy.http.Request` 객체를 포함하는 리스트를 리턴해야 한다(또는


   .. warning:: crawl 스파이더의 rule을 작성할 때는, 콜백으로 ``parse``\ 를 사용하는 것을
       피하라, 왜냐하면 :class:`CrawlSpider`\ 가 자신의 로직을 구현하기 위해
       ``parse`` 메서드를 사용하기 때문이다.
       따라서 ``parse`` 메서드를 오버라이드한다면, crawl 스파이더는 더이상 작동하지 않을
       것이다.

   ``cb_kwargs`` \ 는 콜백 함수에 전달될 키워드 인자를 포함하고 있는 딕셔너리다.

   ``follow``\ 는 불리언으로 rule로 추출된 리스펀스로부터 링크를 따라가야 하는지를
   지정한다. 만약 ``Callback``\ 이 None이면 ``follow``\ 는 ``True``\ 가 기본이고
   그렇지 않은 경우 기본은 ``False``\다.

   ``process_links`` \ 는 callable이나 문자열(이 경우 해당 이름을 가진 스파이더 객체의
   메서드가 사용된다)로 지정된 ``link_extractor``\ 을 사용해 리스펀스로부터 추출된 링크
   리스트에 대해 호출된다. 주로 필터링을 목적으로 사용된다.

   ``process_request``\ 는 callable이나 문자열(이 경우 해당 이름을 가진 스파이더 객체의
   메서드가 사용된다)로 rule에 따라 추출된 모든 리퀘스트에 대해 호출되며, 반드시
   리퀘스트나 (요청을 필터링하기 위해서) None을 반환해야 한다.

CrawlSpider example
~~~~~~~~~~~~~~~~~~~

이제 rule이 있는 CrawlSpider를 보자::

    import scrapy
    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor

    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']

        rules = (
            # Extract links matching 'category.php' (but not matching 'subsection.php')
            # and follow links from them (since no callback means follow=True by default).
            Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

            # Extract links matching 'item.php' and parse them with the spider's method parse_item
            Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )

        def parse_item(self, response):
            self.logger.info('Hi, this is an item page! %s', response.url)
            item = scrapy.Item()
            item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
            return item


이 스파이더는 example.com의 홈페이지를 크롤링하기 시작해서, 카테고리 링크와 아이템 링크를
수집하고 ``parse_item`` 메서드로 아이템링크를 파싱한다. 모든 아이템
리스펀스에서는 XPath를 사용해 HTML로부터 데이터를 추출하고, :class:`~scrapy.item.Item`\ 을
데이터로 채운다.

XMLFeedSpider
-------------

.. class:: XMLFeedSpider

    XMLFeedSpider는 특정 노드 이름에 따라 XML 피드를 반복해서 파싱하도록 설계 되었다.
    이터레이터는 다음 중에서 선택될 수 있다: ``iternodes``, ``xml``,
    ``html``. 성능을 위해서 ``iternodes`` 이터레이터를 사용하는 것을 추천한다.
    ``xml``\ 과 ``html`` 이터레이터는 파싱하기 위해서 전체 DOM을 한꺼번에 생성한다.
    그러나, ``html`` 이터레이터는 마크업이 좋지 못한 상태인 XML을 파싱할 때는 유용하다.

    이터레이터와 태그명을 설텅하기 위해서 반드시 아래의 클래스 속성을
    정의해야 한다:

    .. attribute:: iterator

        사용할 이터레이터를 정의하는 문자열. 다음 중 하나일 수 있다:

           - ``'iternodes'`` - 정규식에 기반한 빠른 이터레이터

           - ``'html'`` - :class:`~scrapy.selector.Selector`\ 를 사용하는 이터레이터.
             DOM 파싱을 사용하며 무조건 모든 DOM을 메모리에 로드한다는 사실을 명심하라.
             큰 피드의 경우 문제가 될 수 있다.

           - ``'xml'`` - :class:`~scrapy.selector.Selector`\ 를 사용하는 이터레이터.
             DOM 파싱을 사용하며 무조건 모든 DOM을 메모리에 로드한다는 사실을 명심하라.
             큰 피드의 경우 문제가 될 수 있다.

        기본으로 ``'iternodes'``\ 가 설정되어 있다.

    .. attribute:: itertag

        반복할 노드(또는 요소)의 이름 문자열. 예시::

            itertag = 'product'

    .. attribute:: namespaces

        스파이더가 처리할 문서에서 이용가능한 네임스페이스를 정의한 ``(prefix, uri)`` 튜플 리스트.
        ``prefix``\ 와 ``uri``\ 는 :meth:`~scrapy.selector.Selector.register_namespace` 메서드를
        이용해서 네임스페이스로 등록될 것이다.

        그 다음 :attr:`itertag` 속성에 네임스페이스가 있는 노드를 지정할 수 있다.

        예시::

            class YourSpider(XMLFeedSpider):

                namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
                itertag = 'n:url'
                # ...

    이러한 새로운 속성 이외에도, 이 스파이더는 아래의 오버라이드가능한 메서드들을
    가지고 있다:

    .. method:: adapt_response(response)

        스파이더가 파싱을 시작하기 전에, 스파이더 미들웨어에서 도착하자마자 리스펀스를 받는 메서드.
        파싱하기 전에 리스펀스 본문을 수정하기 위해 사용될 수 있다.
        이 메서드는 리스펀스르 받고 리스펀스를 반환한다(똑같은 것일 수도 다른 것일 수도 있다).

    .. method:: parse_node(response, selector)

        제공된 태그명(``itertage``)과 매칭되는 노드에 대해 호출된다.
        리스펀스와 각 노드에 대한 :class:`~scrapy.selector.Selector`\ 를 받는다
        이 메서드는 반드시 오버라이딩 해야 한다.
        그렇지 않으면 스파이더가 작동을 하지 않는다.
        이 메서드는 :class:`~scrapy.item.Item` 객체, :class:`~scrapy.http.Request` 객체,
        또는 이런 것들을 포함하고 있는 이터러블을 반환해야 한다.

    .. method:: process_results(response, results)

        이 메서드는 스파이더를 통해 반환된 각 결과(아이템 또는 리퀘스트)에 대해
        호출되며, 프레임워크 코어에 결과를 반환하기 전에 필요한 마지막 프로세싱을 수행하기 위한 것이다.
        예를 들면 아이템 ID를 세팅 하는 것이 있다. 결과 리스트와 그 결과를 생성한
        응답을 받는다. 반드시 결과 리스트를 반환해야 한다(아이템 또는 리퀘스트).


XMLFeedSpider 예제
~~~~~~~~~~~~~~~~~~~~~

이 스파이더들은 사용하기 매우 쉽다, 예제를 보도록 하자::

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem

    class MySpider(XMLFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.xml']
        iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
        itertag = 'item'

        def parse_node(self, response, node):
            self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

            item = TestItem()
            item['id'] = node.xpath('@id').extract()
            item['name'] = node.xpath('name').extract()
            item['description'] = node.xpath('description').extract()
            return item

기본적으로 위에서 한 일은 주어진 ``start_urls``\ 에서 피드를 다운로드하고 각각의
``item`` 태그를 반복시켜 출력하고, 임의의 데이터를 :class:`~scrapy.item.Item`\ 에
저장하는 스파이를 생성한 것이다.

CSVFeedSpider
-------------

.. class:: CSVFeedSpider

   이 스파이더는 노드 대신 행에 대해 반복된다는 점만 제외하면 XMLFeedSpider과 매우 유사하다.
   반복할 때 호출되는 메서드는 :meth:`parse_row`\ 다.

   .. attribute:: delimiter

       CSV 파일의 각 필드의 구분 문자인 문자열.
       기본은 ```','`` (콤마)로 설정되어 있다.

   .. attribute:: quotechar

       CSV 파일의 각 필드의 둘러싸는 문자인 문자열.
       기본은 ``'"'`` (쌍따옴표)로 설정되어 있다.

   .. attribute:: headers

       CSV 파일의 컬럼명 리스트.

   .. method:: parse_row(response, row)

       리스펀스와 CSV 파일의 제공된 헤더에 대한 키를 포함하는 딕셔너리(각 행을 나타낸다)를 받는다.
       이 스파이더 또한 전처리나 후처리를 위해 ``adapt_response``, ``process_results``
       메서드를 오버라이드를 할 수 있는 기회를 제공한다.

CSVFeedSpider 예제
~~~~~~~~~~~~~~~~~~~~~

이전 것과 비슷하지만 :class:`CSVFeedSpider`\ 를 사용한 예제를
보도록 하자::

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem

    class MySpider(CSVFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.csv']
        delimiter = ';'
        quotechar = "'"
        headers = ['id', 'name', 'description']

        def parse_row(self, response, row):
            self.logger.info('Hi, this is a row!: %r', row)

            item = TestItem()
            item['id'] = row['id']
            item['name'] = row['name']
            item['description'] = row['description']
            return item


SitemapSpider
-------------

.. class:: SitemapSpider

    SitemapSpider는 `Sitemaps`_\ 을 사용해서 URL을 찾아 사이트를 크롤링 해준다.

    내포된 사이트맵을 지원하고 `robots.txt`_\ 에서 사이트맵 url 검색을 지원한다.

    .. attribute:: sitemap_urls

        크롤링하고 싶은 url이 있는 사이트맵을 가리키는 url 리스트.

        `robots.txt`_\ 를 가리킬 수도 있으며 사이트맵 url을 추출하기 위해 파싱될 것이다.

    .. attribute:: sitemap_rules

        튜플 ``(regex, callback)``\ 의 리스트:

        * ``regex``\ 는 정규 표현식으로 사이트맵에서 추출된 url과 매치된다.
          ``regex``\ 는 문자열이나 컴파일된 정규식 객체를 쓸 수 있다.

        * callback은 정규표현식에 일치하는 url 처리를 위해 사용하는 콜백이다.
          ``callback``\ 은 (스파이더 메서드의 이름을 가리키는) 문자열이나 callable이
          될 수 있다.

        예시::

            sitemap_rules = [('/product/', 'parse_product')]

        Rule은 순서에 따라 적용되며, 매치되는 가장 첫 번째 것만 사용된다.

        만약 이 속성을 생략하면, 사이트맵에서 찾아진 모든 url이 ``parse`` 콜백으로 처리될 것이다.

    .. attribute:: sitemap_follow

        따라가야 될 사이트맵의 정규식 리스트. 다른 사이트맵을 가리키는 `Sitemap index files`_\ 를 사용하는
        사이트에만 해당한다.

        기본으로, 모든 사이트맵을 따라가도록 되어있다.

    .. attribute:: sitemap_alternate_links

        한 ``url``\ 에 대한 대체 링크를 따라가야하는지를 지정한다.
        대체 링크는 동일한 ``url`` 블럭에 전달된 다른 언어로된 웹사이트를 위한 링크다.

        예시::

            <url>
                <loc>http://example.com/</loc>
                <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
            </url>

        ``sitemap_alternate_links``\ 이 설정되어 있으면, 두 URL은 모두 획득된다. With
        ``sitemap_alternate_links``\ 이 작동하지 않도록 설덩되어 있으면, ``http://example.com/``\ 만이
        획득된다.

        기본은 ``sitemap_alternate_links``\ 가 작동하지 않도록 되어 있다.


SitemapSpider 예제
~~~~~~~~~~~~~~~~~~~~~~

가장 간단한 예제: ``parse`` 콜백을 사용해 사이트맵에서 탐색된 모든 url을 처리한다::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...

몇몇 url은 특정콜백으로, 다른 url은 다른 콜백으로 처리한다::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']
        sitemap_rules = [
            ('/product/', 'parse_product'),
            ('/category/', 'parse_category'),
        ]

        def parse_product(self, response):
            pass # ... scrape product ...

        def parse_category(self, response):
            pass # ... scrape category ...

`robots.txt`_ 파일에 정의된 사이트맵을 따라가고 ``/sitemap_shop``\ 를 포함한 사이트맵만
따라간다::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...

SitemapSpider를 다른 url 소스와 결합한다::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]

        other_urls = ['http://www.example.com/about']

        def start_requests(self):
            requests = list(super(MySpider, self).start_requests())
            requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
            return requests

        def parse_shop(self, response):
            pass # ... scrape shop here ...

        def parse_other(self, response):
            pass # ... scrape other here ...

.. _Sitemaps: https://www.sitemaps.org/index.html
.. _Sitemap index files: https://www.sitemaps.org/protocol.html#index
.. _robots.txt: http://www.robotstxt.org/
.. _TLD: https://en.wikipedia.org/wiki/Top-level_domain
.. _Scrapyd documentation: https://scrapyd.readthedocs.io/en/latest/
