.. _intro-tutorial:

===============
스크래피 튜토리얼
===============

이 튜토리얼은 독자의 시스템에 스크래피가 이미 설치된 것으로 간주한다.
스크래피가 없다면 :ref:`intro-install` 를 먼저 진행한다.

`quotes.toscrape.com <http://quotes.toscrape.com/>`_ 웹사이트를 스크랩 할 것이다.
웹사이트는 유명한 작가들의 구절로 구성되어 있다.

이 튜토리얼은 다음 단계를 따라 진행된다.:

1. 새로운 스크래피 프로젝트 생성
2. :ref:`spider <topics-spiders>` 를 작성해 웹사이트를 크롤링하고 데이터를 추출한다.
3. 추출된 스크랩 데이터를 커맨드 라인을 사용해 내보낸다.
4. spider 를 수정해 반복적으로 링크를 따라가게 한다.
5. spider 인수를 사용한다.


스크래피는 `파이썬`_ 으로 제작되었다. 파이썬이 처음이라면 이 언어에 대해 먼저 알고 싶을 것이다.

다른 언어들에 이미 익숙하고 파이썬을 빠르게 배우고 싶다면 `Dive Into Python 3`_ 을 권장한다.
`Python Tutorial`_ 을 따라 연습해도 좋다.

프로그래밍이 처음이고 파이썬으로 시작하고 싶다면 `Learn Python The Hard Way`_ 이 좋은 교재가 될 것이다.
`this list of Python resources for non-programmers`_ 도 참고하라.

.. _파이썬: https://www.python.org/
.. _this list of Python resources for non-programmers: https://wiki.python.org/moin/BeginnersGuide/NonProgrammers
.. _Dive Into Python 3: http://www.diveintopython3.net
.. _Python Tutorial: https://docs.python.org/3/tutorial
.. _Learn Python The Hard Way: http://learnpythonthehardway.org/book/


프로젝트 생성
==================

스크랩을 시작하기 전에 새로운 스크래피 프로젝트를 생성해야 한다.
코드를 저장하고 싶은 경로에 들어가 다음을 실행한다.::

    scrapy startproject tutorial

위 코드는 ``tutorial`` 디렉토리와 다음 컨텐츠를 생성한다.::

    tutorial/
        scrapy.cfg            # deploy configuration file

        tutorial/             # project's Python module, you'll import your code from here
            __init__.py

            items.py          # project items definition file

            pipelines.py      # project pipelines file

            settings.py       # project settings file

            spiders/          # a directory where you'll later put your spiders
                __init__.py


첫번째 스파이더
================

스파이더는 사용자가 정의하는 클래스로 스크래피는 스파이더를 사용해 웹사이트로부터 정보를 얻는다.
스파이더는 반드시 하위 클래스 :class:`scrapy.Spider`가 되어야 하며 처음으로 생성되는 요청을 정의한다.
선택적으로 페이지의 링크를 따라가는 방법과 페이지 내용물을 다운 받아 데이터를 추출할 때 파싱하는 방법을 정의할 수 있다.

다음 코드로 첫번째 스파이더를 정의해보자. 이를 프로젝트 디렉토리의 ``tutorial/spiders`` 에 있는 ``quotes_spider.py`` 파일에 저장한다.::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            urls = [
                'http://quotes.toscrape.com/page/1/',
                'http://quotes.toscrape.com/page/2/',
            ]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = 'quotes-%s.html' % page
            with open(filename, 'wb') as f:
                f.write(response.body)
            self.log('Saved file %s' % filename)


다음에서 알 수 있듯이 스파이더에 하위 클래스 :class:`scrapy.Spider <scrapy.spiders.Spider>` 가 있다.
여기에 몇몇 인수와 매서드를 정의한다.:

* :attr:`~scrapy.spiders.Spider.name`: 인수는 스파이더를 정의한다.
  프로젝트 별로 고유학 이름을 가져야 하며 동일한 명칭의 스파이더를 여러개 설정할 수 없다.

* :meth:`~scrapy.spiders.Spider.start_requests`: 매서드는
  반드시 requests 의 iterable 을 출력해야 한다.
  (requests 의 리스트 또는 제너레이터 함수를 작성할 수 있다.)
  출력된 iterable로 스파이더가 크롤링을 시작한다.
  이후에 오는 requests 는 최초의 requests 로부터 연속적으로 생성된다.

* :meth:`~scrapy.spiders.Spider.parse`: 매서드는 각각의 requests 에서
  다운로드 된 response 를 관리하기 위해 호출된다. response 매개 변수는 :class:`~scrapy.http.TextResponse` 클래스이다.
  이 클래스는 페이지 내용을 포함하며 이를 관리할 때 유용한 매서드를 가지고 있다.

  :meth:`~scrapy.spiders.Spider.parse` 매서드는 보통 response 를 파싱하며
  스크랩 된 데이터를 딕셔너리로 추출하고 새 url 을 찾아낸다. 이 url로부터 새로운 request (:class:`~scrapy.http.Request`) 를 생성한다.

스파이더 실행
---------------------

스파이더를 사용하기 위해 프로젝트의 상위 디렉토리로 이동해 다음을 실행한다.::

   scrapy crawl quotes

이 커맨드는 우리가 앞서 추가한 ``quotes`` 명칭으로 스파이더를 실행해 ``quotes.toscrape.com`` 도메인으로 request 를 보낸다.
다음과 같은 출력을 얻을 수 있다.::

    ... (omitted for brevity)
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
    2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
    ...

이제 현재 디렉토리에서 파일을 확인하자. *quotes-1.html* 와 *quotes-2.html* 두 파일이 생성되어 있어야 한다.
파일에는 각각의 url이 ``parse`` 매서드 명령에 따라 담겨 있다.

.. note:: 이 단계에서 HTML 파싱하지 않는 이유에 대해선 곧 다룰 것이다.


밑단에서 일어나는 일에 대해
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

스크래피는 스파이더의 매서드 ``start_requests`` 에 의해 출력된 객체 :class:`scrapy.Request <scrapy.http.Request>` 를 예약한다.
각각의 request 의 response 를 받으면서 :class:`~scrapy.http.Response` 객체를 인스턴스화 하고
인수로서 response 를 지나는 request 와 관련된 콜백 매서드를 호출한다. 이번 경우엔 콜백 매서드는 ``parse`` 매서드이다.


start_requests 매서드 지름길
---------------------------------------
URL로부터 :class:`scrapy.Request <scrapy.http.Request>` 객체를 생성하는 :meth:`~scrapy.spiders.Spider.start_requests`
매서드를 실행하는 대신 URL 리스트를 포함하는 :attr:`~scrapy.spiders.Spider.start_urls` 클래스 속성을 정의해도 된다.
이 리스트는 :meth:`~scrapy.spiders.Spider.start_requests` 의 디폴트 실행에 의해 스파이더를 위한 첫 request 를 생성하는데 사용된다.::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = 'quotes-%s.html' % page
            with open(filename, 'wb') as f:
                f.write(response.body)

:meth:`~scrapy.spiders.Spider.parse` 매서드는 URL 들을 위한 각각의 request 를 처리하기 위해 호출된다.
매서드는 스크래피에게 명시해주지 않아도 호출되며 이는 :meth:`~scrapy.spiders.Spider.parse` 가 스크래피의 디폴트 콜백 매서드이기 때문이다.
디폴트 콜백 매서드는 명시적으로 지정하지 않아도 request 를 위해 호출된다.

데이터 추출
---------------

스크래피로 데이터를 추출하는 방법을 배우는데는 :ref:`Scrapy shell <topics-shell>` 쉘을 사용한 선택기를 해보는 것이 가장 좋다.
다음을 실행한다.::

    scrapy shell 'http://quotes.toscrape.com/page/1/'

.. note::

   커맨드 라인에서 스크래피 쉘을 실행할 때는 url에 항상 따옴표를 둘러야 한다.
   따옴표가 없으면 url 은 인수를 포함하게 되고 작동하지 않는다. (예시. ``&`` 문자)

   윈도우에서는 쌍따옴표를 사용한다.::

       scrapy shell "http://quotes.toscrape.com/page/1/"

다음과 같이 나타날 것이다.::

    [ ... Scrapy log here ... ]
    2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
    [s]   item       {}
    [s]   request    <GET http://quotes.toscrape.com/page/1/>
    [s]   response   <200 http://quotes.toscrape.com/page/1/>
    [s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
    [s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
    [s] Useful shortcuts:
    [s]   shelp()           Shell help (print this help)
    [s]   fetch(req_or_url) Fetch request (or URL) and update local objects
    [s]   view(response)    View response in a browser
    >>>

셀을 사용하면 response 객체와 함께 `CSS`_ 를 사용해 요소를 선택할 수 있다.::

    >>> response.css('title')
    [<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

``response.css('title')`` 실행의 결과물은 :class:`~scrapy.selector.SelectorList` 로 호출된 객체로 리스트와 유사하다.
XML/HTML 요소를 둘러싸는 :class:`~scrapy.selector.Selector` 객체의 리스트를 나타내며
이 객체로 보다 정밀한 선택이나 데이터 추출을 가능하게 하는 쿼리를 실행할 수 있다.

위의 title 로부터 텍스트를 추출하기 위해 다음을 실행한다.::

    >>> response.css('title::text').extract()
    ['Quotes to Scrape']

여기서 알아야 할 것은 두가지이다. 먼저 CSS 쿼리에 ``::text`` 를 추가했으며
이는 ``<title>`` 요소로부터 텍스트 요소만 선택함을 의미한다.
``::text`` 를 명시하지 않으면 title 요소 전체를 가져와 태그까지 포함하게 된다.::

    >>> response.css('title').extract()
    ['<title>Quotes to Scrape</title>']

다음은 ``.extract()`` 를 호출한 결과물이 리스트라는 것이다.
이는 우리가 :class:`~scrapy.selector.SelectorList` 의 인스턴스를 사용하기 때문이다.
이번 예시에서처럼 첫번째 결과물 만을 원한다면 다음을 실행한다.::

    >>> response.css('title::text').extract_first()
    'Quotes to Scrape'

다음 코드로 대체할 수 있다.::

    >>> response.css('title::text')[0].extract()
    'Quotes to Scrape'

그러나 ``.extract_first()`` 의 사용은 ``IndexError`` 를 피할 수 있다.
selection 에 맞는 요소를 찾지 못하면 ``None`` 을 출력하게 된다.

여기서 알아야 할 것이 있다. 대부분의 스크래피 코드에서 페이지에서 찾을 수 없는 것들로 인한 오류를 피하고
몇몇 요소를 페이지에서 찾지 못하더라도 최소한의 데이터는 얻고 싶을 것이다.

:meth:`~scrapy.selector.Selector.extract`, :meth:`~scrapy.selector.SelectorList.extract_first`
매서드에 더해 :meth:`~scrapy.selector.Selector.re` 매서드로 정규 표현식을 사용한 추출을 할 수 있다.::

    >>> response.css('title::text').re(r'Quotes.*')
    ['Quotes to Scrape']
    >>> response.css('title::text').re(r'Q\w+')
    ['Quotes']
    >>> response.css('title::text').re(r'(\w+) to (\w+)')
    ['Quotes', 'Scrape']

``view(response)`` 를 사용해 웹 브라우저의 쉘로부터 response 페이지를 열면
적절한 CSS selector 를 찾는데 도움이 될 것이다. 브라우저 개발자 툴이나 Firebug 와 같은 확장을 사용해도 된다.
(:ref:`topics-firebug` 와 :ref:`topics-firefox` 를 보자.)

`Selector Gadget`_ 은 시각적으로 선택된 요소들의 CSS selector 를 빠르게 찾기 위한 좋은 도구가 될 것이다.
많은 브라우저에서 작동한다.

.. _regular expressions: https://docs.python.org/3/library/re.html
.. _Selector Gadget: http://selectorgadget.com/


XPath: 간략한 소개
^^^^^^^^^^^^^^^^^^^^

`CSS`_ 에 추가로 스크래피 selector 는 `XPath`_ 표현식을 지원한다.::

    >>> response.xpath('//title')
    [<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
    >>> response.xpath('//title/text()').extract_first()
    'Quotes to Scrape'

XPath 표현식은 아주 강력하고 스크래피 selector 의 기초가 된다.
사실 CSS selector 는 밑단에서 XPath 로 변환된다.
쉘 내부의 selector 객체의 텍스트 표현을 자세히 보면 이를 알 수 있다.

CSS selector 만큼 많이 쓰이진 않지만 XPath 표현식은 구조를 탐색할 뿐 아니라
내용까지 보기 때문에 보다 강력한 성능을 제공한다.
XPath 를 사용하면 *"Next Page" 를 포함하는 링크* 와 같은 것들을 선택할 수 있다.
이러한 기능들로 XPath 가 스크래피 작업에 적합해 진다.
이미 CSS selector 에 대해 알고 있더라도 XPath 를 공부하는 것을 권장한다.
스크래피가 더 간편해질 것이다.

이 문서에서 XPath 에 대해 자세히 다루진 않지만 :ref:`using XPath with Scrapy Selectors here <topics-selectors>`
에서 더 많은 정보를 얻을 수 있다. XPath 에 대해 배우고 싶다면 `this tutorial to learn XPath through examples <http://zvon.org/comp/r/tut-XPath_1.html>`_  과
`this tutorial to learn "how to think in XPath" <http://plasmasturm.org/log/xpath101/>`_ 를 보자.

.. _XPath: https://www.w3.org/TR/xpath
.. _CSS: https://www.w3.org/TR/selectors

인용구와 작가 추출
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이제 selection 과 추출에 대해 조금 알게 되었다. 웹 페이지로부터 인용구를 추출하는 코드를 작성해 스파이더를 완성해 보자.

http://quotes.toscrape.com 의 인용구는 각각 다음과 같은 HTML 요소로 나타난다.:

.. code-block:: html

    <div class="quote">
        <span class="text">“The world as we have created it is a process of our
        thinking. It cannot be changed without changing our thinking.”</span>
        <span>
            by <small class="author">Albert Einstein</small>
            <a href="/author/Albert-Einstein">(about)</a>
        </span>
        <div class="tags">
            Tags:
            <a class="tag" href="/tag/change/page/1/">change</a>
            <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
            <a class="tag" href="/tag/thinking/page/1/">thinking</a>
            <a class="tag" href="/tag/world/page/1/">world</a>
        </div>
    </div>

스크래피 쉘을 열고 원하는 데이터를 추출하는 방법을 알아보자.::

    $ scrapy shell 'http://quotes.toscrape.com'

인용구 HTML 요소의 selector 리스트를 다음과 같이 얻는다.::

    >>> response.css("div.quote")

위의 쿼리로부터 출력된 각각의 selector 의 하위 요소에서 다음 쿼리를 실행할 수 있다.
첫번째 selector 를 변수에 할당해 특정 인용구에 CSS selector 를 바로 실행할 수 있게 하자.::

    >>> quote = response.css("div.quote")[0]

방금 생성된 ``quote`` 객체를 사용해 인용구로부터 ``title``, ``author``, ``tags`` 를 추출해 보자.::

    >>> title = quote.css("span.text::text").extract_first()
    >>> title
    '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
    >>> author = quote.css("small.author::text").extract_first()
    >>> author
    'Albert Einstein'

tags 가 문자열 리스트이기 때문에 ``.extract()`` 매서드를 사용해 모두 얻을 수 있다.::

    >>> tags = quote.css("div.tags a.tag::text").extract()
    >>> tags
    ['change', 'deep-thoughts', 'thinking', 'world']

각각의 인용구를 추출하는 법을 알았으면 이제 모든 인용구에서 이를 반복하고 파이썬 딕셔너리로 넣어보자.::

    >>> for quote in response.css("div.quote"):
    ...     text = quote.css("span.text::text").extract_first()
    ...     author = quote.css("small.author::text").extract_first()
    ...     tags = quote.css("div.tags a.tag::text").extract()
    ...     print(dict(text=text, author=author, tags=tags))
    {'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
    {'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
        ... a few more of these, omitted for brevity
    >>>

스파이더에서 데이터 추출
-----------------------------

다시 스파이더로 돌아가 보자. 지금까지는 특정 데이터를 추출하진 않고 전체 HTML 페이지를 로컬 파일에 저장했다.
위의 추출 로직을 통합해 스파이더에 넣어보자.

스크래피 스파이더는 보통 페이지로부터 추출된 데이터를 담고 있는 다수의 딕셔너리를 생성한다.
이를 위해 콜백에서 ``yield`` 파이썬 키워드를 사용한다. 다음과 코드와 같다.::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }

이 스파이더를 실행하면 추출된 데이터와 로그를 출력한다.::

    2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
    {'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
    2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
    {'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}


.. _storing-data:

스크랩 된 데이터 저장
========================

스크랩 된 데이터를 저장하는 가장 간단한 방법은 :ref:`Feed exports <topics-feed-exports>` 를 사용하는 것이다.
다음 커맨드를 실행한다.::

    scrapy crawl quotes -o quotes.json

위 커맨드는 스크랩된 항목을 모두 `JSON`_ 에 넣은 ``quotes.json`` 파일을 생성한다.

역사적인 이유에서 스크래피는 내용을 덮어쓰지 않고 주어진 파일에 내용을 추가한다.
이로 인해 파일을 제거하지 않고 위 커맨드를 두번 실행하면 손상된 JSON 파일이 된다.

`JSON Lines`_ 과 같은 다른 형식을 사용할 수도 있다.::

    scrapy crawl quotes -o quotes.jl

`JSON Lines`_ 형식은 새로운 기록을 추가할 수 있다는 점에서 유용하다.
앞서 다룬 JSON 과 같은 문제를 겪지 않아도 된다.
각각의 기록은 다른 라인에 기록되기 때문에 대용량 파일을 모두 메모리에 넣지 않아도 작업할 수 있다.
이를 위한 툴 `JQ`_ 를 사용하면 된다.

이번 튜토리얼과 같이 작은 프로젝트에서는 필요하지 않지만
보다 복잡한 작업에서 스크랩한 항목을 사용하고 싶다면 :ref:`Item Pipeline <topics-item-pipeline>` 를 작성하면 된다.
Item Pipeline 을 위한 자리표시자 파일은 프로젝트를 생성할 때 ``tutorial/pipelines.py`` 에 설정되어 있다.
스크랩된 항목들을 저장만 하고 싶다면 item pipeline 을 사용하지 않아도 된다.

.. _JSON Lines: http://jsonlines.org
.. _JQ: https://stedolan.github.io/jq


링크 따라가기
===============

http://quotes.toscrape.com 의 두 페이지로부터 스크랩하는 대신 모든 페이지로부터 인용구를 얻고 싶다고 헤보자.

페이지로부터 데이터를 추출하는 방법은 알고 있다. 이제 페이지들로부터 링크를 따라가는 방법을 알아보자.

먼저 따라가고자 하는 페이지 링크를 추출해 보자. 앞서 살펴본 페이지에서 다음 페이지로 가는 링크를 볼 수 있다.
다음 마크업을 보자.:

.. code-block:: html

    <ul class="pager">
        <li class="next">
            <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
        </li>
    </ul>

쉘에서 링크를 추출해보자.::

    >>> response.css('li.next a').extract_first()
    '<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

앵커 요소를 얻었지만 ``href`` 인수가 필요하다. 이를 위해 스크래피는 CSS 확장을 지원한다.
다음과 같이 인수 컨텐츠를 선택할 수 있다.::

    >>> response.css('li.next a::attr(href)').extract_first()
    '/page/2/'

아래 코드는 다음 페이지를 반복적으로 따라가도록 수정된 스파이더이다. 스파이더를 사용해 데이터를 추출한다.::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                next_page = response.urljoin(next_page)
                yield scrapy.Request(next_page, callback=self.parse)


데이터를 추출하면 ``parse()`` 매서드는 다음 페이지로의 링크를 보고 :meth:`~scrapy.http.Response.urljoin`
매서드를 사용해 완전한 절대 URL 을 빌드한다. (추출된 데이터에서는 상대 링크이다.)
절대 URL 로 다음 페이지로의 request 를 생성하고 이를 콜백으로 등록해 다음 페이지에서의 데이터 추출을 관리한다.
이 과정을 반복해 모든 페이지의 크롤링이 진행된다.

여기서 알아야 할 것은 스크래피가 링크를 따라가는 메카니즘이다.
콜백 매서드에서 request 를 생성하면 스크래피는 이를 보내도록 예약하고
request 가 끝났을 때 실행될 콜백 매서드를 등록한다.

이 방법으로 지정한 규칙대로 링크를 따라가는 복잡한 크롤러를 만들어
방문한 페이지에 따라 다양한 종류의 데이터를 추출할 수 있다.

이번 예시에서 다음 페이지를 찾을 수 없을 때까지 다음 페이지를 따라가는 루프를 만들었다.
페이지 번호가 있는 블로그, 포럼 등의 사이트를 크롤링하는데 유용하다.


.. _response-follow-example:

request 생성 지름길
--------------------------------

request 객체를 생성하는 쉬운 방법으로 :meth:`response.follow <scrapy.http.TextResponse.follow>` 를 사용한다.::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('span small::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                yield response.follow(next_page, callback=self.parse)

scrapy.Request 와 달리 ``response.follow`` 는 상대 URL 을 바로 지원한다. (urljoin 을 호출하지 않아도 된다.)
``response.follow`` 는 request 인스턴스만 출력함을 알아두자. 이 request 를 생성해 주어야 한다.

또한 문자열 대신 ``response.follow`` 로 selector 를 보낼 수 있다. 이 selector 는 중요한 인수를 추출해야 한다.::

    for href in response.css('li.next a::attr(href)'):
        yield response.follow(href, callback=self.parse)

``<a>`` 요소로의 지름길이 있다. ``response.follow`` 는 href 의 인수를 자동으로 사용한다.
코드는 다음과 같이 간결해진다.::

    for a in response.css('li.next a'):
        yield response.follow(a, callback=self.parse)

.. note::

    ``response.follow(response.css('li.next a'))`` is not valid because
    ``response.css`` returns a list-like object with selectors for all results,
    not a single selector. A ``for`` loop like in the example above, or
    ``response.follow(response.css('li.next a')[0])`` is fine.

추가 예시와 패턴
--------------------------

다음 스파이더는 콜백을 작성하고 링크를 따라간다. 이번엔 작가 정보를 스크랩 해보자.::

    import scrapy


    class AuthorSpider(scrapy.Spider):
        name = 'author'

        start_urls = ['http://quotes.toscrape.com/']

        def parse(self, response):
            # follow links to author pages
            for href in response.css('.author + a::attr(href)'):
                yield response.follow(href, self.parse_author)

            # follow pagination links
            for href in response.css('li.next a::attr(href)'):
                yield response.follow(href, self.parse)

        def parse_author(self, response):
            def extract_with_css(query):
                return response.css(query).extract_first().strip()

            yield {
                'name': extract_with_css('h3.author-title::text'),
                'birthdate': extract_with_css('.author-born-date::text'),
                'bio': extract_with_css('.author-description::text'),
            }

이 스파이더는 메인 페이지에서 시작해 작가 페이지로의 모든 링크를 따라간다.
이 과정에서 매번 ``parse_author`` 콜백을 호출하고 앞서 본 것과 같이 페이지 번호 링크와 ``parse`` 콜백도 호출한다.

여기서 우리는 콜백을 위치 인수로서 ``response.follow`` 로 보내 코드를 간결하게 했다.
이 방법은 ``scrapy.Request`` 에서도 사용할 수 있다.

``parse_author`` 콜백은 도우미 기능을 정의해 CSS 쿼리로부터 데이터를 추출하고 정리한다.
추출된 작가 데이터로 파이썬 딕셔너리를 작성한다.

이 스파이더의 흥미로운 점은 동일한 작가의 인용구가 여러개 있다고 해도 작가 페이지를 여러번 방문하지 않는다는 것이다.
디폴트에 의해 스크래피는 이미 방문했던 url 로의 request 를 걸러낸다. 이는 프로그램 실수로 인한 서버 과부하를 막기 위함이다.
이 기능은 :setting:`DUPEFILTER_CLASS` 에서 설정할 수 있다..

이제 스크래피로 링크를 따라가고 콜백을 사용하는 매카니즘을 이해했을 것이다.

아직 링크를 따라가는 메카니즘에 도움이 되는 스파이더 예시가 남아 있다.
:class:`~scrapy.spiders.CrawlSpider` 클래스에서 소형 규칙 엔진을 실행하는 포괄 스파이더를 알아 보자.
이 소형 규칙 엔진을 사용하면 스파이더의 최상위에서 크롤러를 작성할 수 있다.

또한 일반적인 패턴은 :ref:`trick to pass additional data to the callbacks <topics-request-response-ref-request-callback-arguments>`
를 사용해 하나 이상의 페이지로부터 얻은 데이터를 하나의 항목으로 생성한다.


스파이더 인수 사용
======================

항목을 실행할 때 ``-a`` 옵션을 사용해 스파이더에 커맨드 라인 인수를 줄 수 있다.::

    scrapy crawl quotes -o quotes-humor.json -a tag=humor

이 인수들은 스파이더의 ``__init__`` 매서드로 보내져 디폴트에 의해 스파이더 인수가 된다.

이번 예시에서 ``tag`` 인수에 부여된 값들은 ``self.tag`` 를 통해 사용 가능해 진다.
이 기능을 사용해 스파이더가 인수에 기반해 URL 을 빌드하고 특정 태그를 포함하는 인용구만 가져오게 할 수 있다.::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            url = 'http://quotes.toscrape.com/'
            tag = getattr(self, 'tag', None)
            if tag is not None:
                url = url + 'tag/' + tag
            yield scrapy.Request(url, self.parse)

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                yield response.follow(next_page, self.parse)


스파이더에 ``tag=humor`` 인수를 보내면 ``humor`` 태그의 URL 만 방문함을 알 수 있다.
``http://quotes.toscrape.com/tag/humor`` 와 같은 URL 만 따르게 된다..

:ref:`learn more about handling spider arguments here <spiderargs>`.

다음 단계
==========

이 튜토리얼은 스크래피의 기초만 다루었고 이 외에도 많은 기능들이 있다.
:ref:`intro-overview` 챕터의 :ref:`topics-whatelse` 섹션에서 중요한 기능들에 대한 간략한 개요를 볼 수 있다.

커맨드 라인 툴, 스파이더, selector나 스크랩 데이터 모델링과 같이 튜토리얼에서 다루지 않은 것들에 대해 알고 싶다면 :ref:`section-basics` 를 보자.
예시 프로젝트로 배우는 것을 선호한다면 ref:`intro-examples` 섹션을 보자.

.. _JSON: https://en.wikipedia.org/wiki/JSON
.. _dirbot: https://github.com/scrapy/dirbot
