.. _intro-tutorial:

=============================
스크래피(Scrapy) 튜토리얼
=============================

이 튜토리얼에서 독자의 시스템에 스크래피가 이미 설치되었다고 가정한다.
만약 아직 설치하지 않았다면 :ref:`intro-install` 를 참조하여 스크래피를 설치하라.

여기에선 `quotes.toscrape.com <http://quotes.toscrape.com/>`_ 웹사이트를 스크랩 할 것이다.
이 웹사이트는 유명한 작가들의 구절로 구성되어 있다.

본 튜토리얼은 다음 단계를 따라 진행된다:

1. 새로운 스크래피 프로젝트 생성
2. :ref:`스파이더(spider) <topics-spiders>`\ 를 작성해 웹사이트를 크롤링하고 데이터를 추출한다.
3. 추출된 스크랩 데이터를 커맨드 라인을 사용해 내보낸다.
4. 스파이더를 수정해 재귀적으로 링크를 따라가게 한다.
5. 스파이더 인수를 사용한다.


스크래피는 `파이썬`_\ 으로 제작되었다. 파이썬이 처음이라면 이 언어에 대해 먼저 알고 싶을 것이다.

다른 언어들에 이미 익숙하고 파이썬을 빠르게 배우고 싶다면 `Dive Into Python 3`_\ 를 권장한다.
`Python Tutorial`_\ 을 따라 연습해도 좋다.

프로그래밍이 처음이고 파이썬으로 시작하고 싶다면 `Learn Python The Hard Way`_\ 가 좋은 교재가 될 것이다.
`프로그래머가 아닌 사람들을 위한 파이썬 자료 리스트`_\ 도 참고하라.

.. _파이썬: https://www.python.org/
.. _프로그래머가 아닌 사람들을 위한 파이썬 자료 리스트: https://wiki.python.org/moin/BeginnersGuide/NonProgrammers
.. _Dive Into Python 3: http://www.diveintopython3.net
.. _Python Tutorial: https://docs.python.org/3/tutorial
.. _Learn Python The Hard Way: http://learnpythonthehardway.org/book/


프로젝트 생성
==================

스크랩을 시작하기 전에 새로운 스크래피 프로젝트를 생성해야 한다.
코드를 저장하고 싶은 경로에 들어가 다음을 실행한다.::

    scrapy startproject tutorial

위 코드는 ``tutorial`` 디렉토리와 다음 파일들을 생성한다.::

    tutorial/
        scrapy.cfg            # deploy configuration file

        tutorial/             # project's Python module, you'll import your code from here
            __init__.py

            items.py          # project items definition file

            middlewares.py    # project middlewares file

            pipelines.py      # project pipelines file

            settings.py       # project settings file

            spiders/          # a directory where you'll later put your spiders
                __init__.py


첫 번째 스파이더
=======================

스파이더는 사용자가 정의하는 클래스로 스크래피가 웹사이트(또는 웹사이트 그룹)로부터 정보를 스크랩할 때 사용한다.
스파이더는 반드시 :class:`scrapy.Spider`\ 의 상속클래스여야 하며 초기 리퀘스트(request)를 정의해야 한다.
원하는 경우 페이지의 링크를 따라가는 방법과 페이지 내용물을 다운 받아 데이터를 추출할 때 파싱하는 방법도 정의할 수 있다.

다음 코드로 첫 번째 스파이더를 정의해보자.
이 파일을 프로젝트 디렉토리 ``tutorial/spiders``\ 에 ``quotes_spider.py``
라는 이름으로 저장한다.::

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


다음에서 알 수 있듯이 우리의 스파이더는 :class:`scrapy.Spider <scrapy.spiders.Spider>`\ 의
상속클래스이며 다음과 같은 특성과 메서드를 정의했다.

* :attr:`~scrapy.spiders.Spider.name`: 스파이더 식별자.
  프로젝트 내에서 유일해야 한다. 즉, 다른 스파이더에 같은 이름을 설정할 수 없다는
  것을 의미한다.

* :meth:`~scrapy.spiders.Spider.start_requests`: 반드시 스파이더가 크롤링을 시작할 수 있는
  이터러블한 리퀘스트를 리턴해야 한다(리퀘스트 리스트를 리턴하거나 제네레이터 함수를 만들면 된다).
  후속 리퀘스트는 이 최초의 리퀘스트로부터 연속적으로 생성될 것이다.

* :meth:`~scrapy.spiders.Spider.parse`: 생성된 각 리퀘스트로부터 다운로드된 리스판스(response)를 처리하기 위해
  호출될 메서드. 리스판스 파라미터는 페이지 내용을 포함하고 있는 :class:`~scrapy.http.TextResponse` 인스턴스이며
  이 인스턴스는 내용을 처리할 수 있는 유용한 메서드를 가지고 있다.

  :meth:`~scrapy.spiders.Spider.parse` 메서드는 보통 리스판스를 파싱하며
  스크랩된 데이터를 딕셔너리로 추출하고 새 url을 찾아낸다.
  이 url로부터 새로운 리퀘스트(:class:`~scrapy.http.Request`)를 생성한다.

스파이더 실행
-----------------------

스파이더를 사용하기 위해 프로젝트의 최상위 디렉토리로 이동해 다음을 실행한다.::

   scrapy crawl quotes

이 커맨드는 우리가 앞서 추가한 ``quotes`` 명칭으로 스파이더를 실행해 ``quotes.toscrape.com`` 도메인으로 리퀘스트를 보낸다.
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
파일에는 각각의 url이 ``parse`` 메서드 명령에 따라 담겨 있다.

.. note:: 이 단계에서 HTML 파싱하지 않는 이유에 대해선 곧 다룰 것이다.


밑단에서 일어나는 일에 대해
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

스크래피는 스파이더의 ``start_requests`` 메서드에 의해 반환된 객체 :class:`scrapy.Request <scrapy.http.Request>` 를 예약한다.
각각에 대한 리스판스를 받으면 스파이더는 :class:`~scrapy.http.Response` 객체를 인스턴스화하고
리퀘스트와 연결된 콜백 메서드를 호출하는데 리스판스를 인자로서 전달한다(이 경우에는 ``parse`` 메서드다).


start_requests 메서드 대체
---------------------------------------
URL로부터 :class:`scrapy.Request <scrapy.http.Request>` 객체를 생성하는 :meth:`~scrapy.spiders.Spider.start_requests`
메서드를 구현하는 대신 URL 리스트를 포함하는 :attr:`~scrapy.spiders.Spider.start_urls` 클래스 속성을 정의해도 된다.
이 리스트는 :meth:`~scrapy.spiders.Spider.start_requests`\ 의 디폴트 구현에서 사용되며 스파이더를 위한 첫 리퀘스트를 생성한다::

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

:meth:`~scrapy.spiders.Spider.parse` 메서드는 우리가 명시적으로 스크래피에 명령하지 않아도
각 URL의 리퀘스트를 처리하기 위해 호출된다.
왜냐하면 :meth:`~scrapy.spiders.Spider.parse`\ 는 스크래피의 디폴트 콜백 메서드이기 때문이며
명시적인 콜백 할당 없이 리퀘스트를 위해 호출된다.


데이터 추출
---------------

스크래피로 데이터를 추출하는 방법을 배우는 데는 :ref:`스크래피 셸 <topics-shell>`\ 을 사용한 셀렉터(Selector)를
사용 해보는 것이 가장 좋다.
다음을 실행한다.::

    scrapy shell 'http://quotes.toscrape.com/page/1/'

.. note::

   커맨드 라인에서 스크래피 셸을 실행할 때는 url에 항상 따옴표를 둘러야 한다.
   그렇지 않으면 url은 인자를 포함한 (예시. ``&`` 문자) url은 작동하지 않을 것이다.

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

셀을 사용하면 리스판스 객체와 함께 `CSS`_ 를 사용해 요소를 선택할 수 있다.::

    >>> response.css('title')
    [<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

``response.css('title')`` 실행의 결과물은 :class:`~scrapy.selector.SelectorList` 로 불리는 객체로 리스트 같은
형태이다. 이 객체는 :class:`~scrapy.selector.Selector` 객체의 리스트를 나타내며
XML/HTML 요소를 감싸서 정밀한 선택이나 데이터를 추출하는 추가적인 쿼리를 사용할 수 있도록 해준다.

위의 ``title``\ 로부터 텍스트를 추출하기 위해 다음을 실행한다.::

    >>> response.css('title::text').extract()
    ['Quotes to Scrape']

여기서 알아야 할 것은 두가지이다. 먼저 CSS 쿼리에 ``::text``\ 를 추가했으며
이는 ``<title>`` 요소로부터 텍스트 요소만 선택함을 의미한다.
``::text`` 를 명시하지 않으면 title 요소 전체를 가져와 태그까지 포함하게 된다.::

    >>> response.css('title').extract()
    ['<title>Quotes to Scrape</title>']

다음은 ``.extract()`` 를 호출한 결과물이 리스트라는 것이다.
이는 우리가 :class:`~scrapy.selector.SelectorList` 의 인스턴스를 처리하고 있기 때문이다.
이번 예시처럼 첫번째 결과만을 원하면 다음을 실행한다.::

    >>> response.css('title::text').extract_first()
    'Quotes to Scrape'

다음 코드로 대체할 수 있다.::

    >>> response.css('title::text')[0].extract()
    'Quotes to Scrape'

``.extract_first()``\ 를 사용하면 ``IndexError``\ 를 피할 수 있다.
선택에 대응하는 요소를 찾지 못하면 ``None``\ 을 출력한다.

여기서 알아야 할 것이 있다. 대부분의 스크래핑 코드의 경우,
사람들은 일부분이 스크랩에 실패하더라도 최소한 **일정** 데이터를 얻을 수 있도록
페이지에서 찾을 수 없는 것들로 인해 발생하는 에러에 대해 코드가 탄력적으로 대응하기를 바랄 것이다.

:meth:`~scrapy.selector.Selector.extract`, :meth:`~scrapy.selector.SelectorList.extract_first`
메서드에 더해 :meth:`~scrapy.selector.Selector.re` 메서드로 정규 표현식을 사용한 추출을 할 수 있다.::

    >>> response.css('title::text').re(r'Quotes.*')
    ['Quotes to Scrape']
    >>> response.css('title::text').re(r'Q\w+')
    ['Quotes']
    >>> response.css('title::text').re(r'(\w+) to (\w+)')
    ['Quotes', 'Scrape']

적절한 CSS 셀렉터를 찾기 위해서 ``view(response)``\ 를 사용해 웹 브라우저의 셸에서 리스판스 페이지를 여는
것도 유용하다. 브라우저의 개발자 도구나 Firebug와 같은 확장을 사용해도 된다.
(:ref:`topics-firebug`\ 와 :ref:`topics-firefox`\ 를 참고하라.)

`Selector Gadget`_ 은 시각적으로 요소를 선택하고 이에 해당하는 CSS 셀렉터를 찾는 도구로
대부분의 브라우저에서 작동하는 훌륭한 도구이다.

.. _regular expressions: https://docs.python.org/3/library/re.html
.. _Selector Gadget: http://selectorgadget.com/


XPath: 간략한 소개
^^^^^^^^^^^^^^^^^^^^^^^^^^

스크래피 셀렉터는 `CSS`_\ 뿐 아니라 `XPath`_\ 표현식도 지원한다.::

    >>> response.xpath('//title')
    [<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
    >>> response.xpath('//title/text()').extract_first()
    'Quotes to Scrape'

XPath 표현식은 아주 강력하며 스크래피 셀렉터의 기본이다.
사실 CSS 셀렉터는 밑단에서 XPath로 변환된다.
셸 내부 셀렉터 객체의 텍스트 표현을 자세히 보면 이를 알 수 있다.

CSS 셀렉터만큼 인기가 있지는 않지만 XPath 표현식은 구조를 탐색할 뿐 아니라
내용까지 보기 때문에 더 강력한 성능을 가지고 있다.
XPath 를 사용하면 *"Next Page" 를 포함하는 링크* 같은 것들을 선택할 수 있다.
이러한 기능들로 인해서 XPath는 스크랩 작업에 적합하며, 그래서
이미 CSS 셀렉터에 대해 알고 있더라도 XPath에 대해 공부하는 것을 권장한다.
스크랩을 훨씬 쉽게 할 수 있을 것이다.

이 문서에서 XPath에 대해 자세히 다루진 않지만 :ref:`스크래피 셀렉터로 XPath 사용하기 <topics-selectors>`\
에서 더 많은 정보를 얻을 수 있다.
XPath 에 더 대해 배우고 싶다면 `예시를 통해 배우는 XPath 튜토리얼 <http://zvon.org/comp/r/tut-XPath_1.html>`_\ 과
`"how to think in XPath" <http://plasmasturm.org/log/xpath101/>`_ 를 추천한다.

.. _XPath: https://www.w3.org/TR/xpath
.. _CSS: https://www.w3.org/TR/selectors

인용구와 작가 추출
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이제 선택과 추출에 대해 조금 알게 되었으므로, 웹페이지에서 인용구를 추출하는 코드를 작성해서
스파이더를 완성시키자.

http://quotes.toscrape.com\ 의 인용구는 각각 다음과 같은 HTML 요소로 나타난다:

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

스크래피 셸을 열고 원하는 데이터를 추출하는 방법을 알아보자::

    $ scrapy shell 'http://quotes.toscrape.com'

인용구 HTML 요소의 셀렉터 리스트를 다음과 같이 얻는다::

    >>> response.css("div.quote")

위의 쿼리로부터 반환된 각각의 셀렉터에서 하위 요소에 대한 쿼리를 더 실행할 수 있다.
첫번째 셀렉터를 변수에 할당해 특정 인용구에 CSS 셀렉터를 바로 실행할 수 있게 하자::

    >>> quote = response.css("div.quote")[0]

방금 생성된 ``quote``\ 객체를 사용해 인용구로부터 ``title``, ``author``, ``tags``\ 를 추출해 보자::

    >>> title = quote.css("span.text::text").extract_first()
    >>> title
    '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
    >>> author = quote.css("small.author::text").extract_first()
    >>> author
    'Albert Einstein'

태그가 문자열 리스트이기 때문에 ``.extract()`` 메서드를 사용해 모두 얻을 수 있다::

    >>> tags = quote.css("div.tags a.tag::text").extract()
    >>> tags
    ['change', 'deep-thoughts', 'thinking', 'world']

각각의 인용구를 추출하는 법을 알았으므로 이제 모든 인용구 요소에 대해 반복해서 파이썬 딕셔너리로 넣을 수 있다::

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

다시 스파이더로 돌아가 보자. 지금까지는 특정 데이터를 추출하진 않고 전체 HTML 페이지를 로컬 파일로 저장했다.
위의 추출 로직을 통합해 스파이더에 통합해 보자.

스크래피 스파이더는 보통 페이지로부터 추출된 데이터를 담고 있는 다수의 딕셔너리를 생성한다.
이를 위해 콜백에서 ``yield`` 파이썬 키워드를 사용한다. 다음과 코드와 같다::

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

스크랩 된 데이터를 저장하는 가장 간단한 방법은 아래의 커맨드를 사용하여
:ref:`Feed exports <topics-feed-exports>`\ 를 이용하는 것이다::

    scrapy crawl quotes -o quotes.json

위 커맨드는 모든 스크랩된 항목을 `JSON`_\ 형식으로 나열한 ``quotes.json`` 파일을 생성한다.

역사적인 이유로 스크래피는 내용을 덮어쓰지 않고 주어진 파일에 내용을 추가한다.
이로 인해 파일을 제거하지 않고 위 커맨드를 두 번 실행하면 손상된 JSON 파일이 된다.

`JSON Lines`_\ 과 같은 다른 형식을 사용할 수도 있다::

    scrapy crawl quotes -o quotes.jl

`JSON Lines`_ 형식은 stream_like하기 때문에 새로운 기록을 쉽게 추가할 수 있어서 유용하다.
두 번 실행했을 때 JSON과 같은 문제가 발생하지 않는다.
각각의 기록은 다른 라인에 기록되기 때문에 모든 것을 메모리에 맞추지 않아도 큰 파일을 처리할 수 있고,
커맨드라인에서 그런 작업을 할수 있게 돕는 `JQ`_\ 라는 툴도 있다.

이번 튜토리얼과 같이 작은 프로젝트에서는 필요하지 않지만
스크랩된 항목으로 보다 복잡한 일을 수행하고 싶으면 :ref:`Item Pipeline <topics-item-pipeline>`\ 를
작성하라. 아이템 파이프라인을 위한 placeholder 파일은 프로젝트가 생성될 때
``tutorial/pipelines.py``\ 에 세팅되어 있다.
스크랩된 항목들을 저장만 하고 싶다면 아이템 파이프라인을 사용하지 않아도 된다.

.. _JSON Lines: http://jsonlines.org
.. _JQ: https://stedolan.github.io/jq


링크 따라가기
====================

http://quotes.toscrape.com\ 의 처음 두 페이지로부터 스크랩하는 대신 모든 페이지로부터 인용구를 얻고 싶다고 헤보자.

페이지로부터 데이터를 추출하는 방법은 알고 있으므로 페이지에서 링크를 따라가는 방법을 알아보자.

첫 번째는 따라가려고 하는 페이지로 향하는 링크를 추출하는 것이다.
페이지를 조사해보면 아래의 마크업으로 표시된 다음 페이지로 가는 링크가 있는 것을
볼 수 있다:

.. code-block:: html

    <ul class="pager">
        <li class="next">
            <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
        </li>
    </ul>

셸에서 링크를 추출해보자.::

    >>> response.css('li.next a').extract_first()
    '<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

앵커 요소를 얻었지만 ``href`` 속성이 필요하다. 이를 위해 스크래피는 다음과 같이 속성 컨텐츠를 선택할 수 있는
CSS 확장을 지원한다::

    >>> response.css('li.next a::attr(href)').extract_first()
    '/page/2/'

재귀적으로 다음페이지 링크를 따라가고 데이터를 추출하는 수정된 스파이더를 보자::

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


데이터를 추출한 후에 ``parse()`` 메서드는 다음 페이지 링크를 찾고
:meth:`~scrapy.http.Response.urljoin` 메서드를 사용해 (링크가 상대적일 수 있기 때문에) 절대 URL을 생성하고
다음 페이지를 위한 새로운 리퀘스트를 생산하고, 자기 자신을 콜백으로 등록해 다음 페이지의 데이터를 추출하고
그런 식으로 모든 페이지를 크롤링한다.

여기서 본 것이 스크래피가 링크를 따라가는 메카니즘이다:
콜백 메서드에서 리퀘스트를 생성할 때 스크래피는 리퀘스트가 보내지도록 예약하고
리퀘스트가 끝났을 때 실행되도록 콜백 메서드를 등록한다.

이 방법을 사용하면 지정한 규칙대로 링크를 따라가는 복잡한 크롤러를 만들고
방문한 페이지에 따라 다양한 종류의 데이터를 추출할 수 있다.

이번 예에서는 다음 페이지를 찾을 수 없을 때까지 다음 페이지를 따라가는 루프를 만들었다 --
이는 번호 표시줄이 있는 블로그, 포럼 등의 사이트를 크롤링하는데 유용하다.


.. _response-follow-example:

리퀘스트 생성 지름길
--------------------------------

리퀘스트 객체를 생성하는 쉬운 방법으로 :meth:`response.follow <scrapy.http.TextResponse.follow>`\ 를 사용할
수 있다::

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

scrapy.Request\ 와 달리 ``response.follow``\ 는 상대 URL을 지원하므로
urljoin을 호출하지 않아도 된다.
하지만 여전히 Request 인스턴스를 반환해야 한다.
``response.follow``\ 가 Request 인스턴스를 반환하도록 되어 있다는 점에 주의한다.


``response.follow``\ 에는 문자열 대신 셀렉터를 넣을수도 있다.
이 셀렉터는 필요한 속성을 추출해야 한다::

    for href in response.css('li.next a::attr(href)'):
        yield response.follow(href, callback=self.parse)

``<a>`` 요소의 경우에는 ``response.follow``\ 가 요소의 href 인자를 자동으로 사용하므로
코드가 다음처럼 간결해진다::

    for a in response.css('li.next a'):
        yield response.follow(a, callback=self.parse)

.. note::

    ``response.follow(response.css('li.next a'))``\ 는 유효하지 않다.
    ``response.css``\ 는 단일 셀렉터가 아니라 모든 결과에 대한 셀렉터를 포함하는 리스트 형태의
    객체를 반환하기 때문이다. 위 예시에 있는 ``for`` 루프나
    ``response.follow(response.css('li.next a')[0])``\ 는 문제가 없다.

추가 예제와 패턴
--------------------------

다음 스파이더는 콜백과 링크 따라가기를 보여주는 또 다른 스파이더다.
이번에는 저자 정보를 스크랩한다::

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
이 과정에서 매번 ``parse_author`` 콜백을 호출하고 앞서 본 것과 같이 ``parse`` 콜백으로
번호줄 링크까지 따라간다.

여기서 우리는 콜백을 위치 인자로서 ``response.follow``\ 로 보내 코드를 간결하게 했다.
이 방법은 ``scrapy.Request``\ 에서도 사용할 수 있다.

``parse_author`` 콜백은 CSS 쿼리로부터의 데이터를 정리하고 추출하는 헬퍼 함수를 정의하며
저자 정보가 담긴 파이썬 딕셔너리를 생산한다.

이 스파이더의 흥미로운 점은 동일한 작가의 인용구가 여러개 있다고 해도 작가 페이지를 여러번 방문하지 않는다는 것이다.
스크래피는 디폴트로 이미 방문했던 url로의 리퀘스트를 걸러낸다. 이는 프로그램 실수로 인한 서버 과부하를 막기 위함이다.
이 기능은 :setting:`DUPEFILTER_CLASS`\ 를 세팅해서 설정을 바꿀 수 있다..

이제 스크래피로 링크를 따라가고 콜백을 사용하는 매카니즘을 이해했을 것이다.

링크 따라가기 메카니즘을 활용하는 예시 스파이더로
그것을 바탕으로 당신의 크롤러를 작성하는데 사용할 수 있는 소형 규칙 엔진을 구현한 일반 스파이더인
:class:`~scrapy.spiders.CrawlSpider` 클래스를 확인해 보아라.

또한 공통 패턴은 :ref:`콜백에 추가적인 데이터를 전달하는 트릭
<topics-request-response-ref-request-callback-arguments>`\ 을 사용해서
한 페이지 이상으로부터 데이터가 있는 아이템을 생성할 수 있다.


스파이더 인수 사용
======================

스파이더를 실행할 때 ``-a`` 옵션을 사용해 커맨드 라인 인자를 제공할 수 있다::

    scrapy crawl quotes -o quotes-humor.json -a tag=humor

이 인자들은 스파이더의 ``__init__`` 메서드로 보내져 기본적으로 스파이더 속성이 된다.

이번 예시에서 ``tag`` 인자로 제공된 값들은 ``self.tag``\ 를 통해 사용 가능해 진다.
이 기능을 사용해 인자에 기반해 URL을 생성하고 스파이더가 특정 태그를 가진 인용구만 가져오도록
만들 수 있다::

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


스파이더에 ``tag=humor`` 인자를 보내면 ``http://quotes.toscrape.com/tag/humor`` 같은
``humor`` 태그의 URL만 방문함을 알 수 있다.

:ref:`스파이더 인자를 다루는 법에 대해 더 배우기 <spiderargs>`.

다음 단계
==========

이 튜토리얼은 스크래피의 기초만 다루었고 이 외에도 많은 기능들이 있다.
:ref:`intro-overview` 챕터의 :ref:`topics-whatelse` 섹션에서 중요한 기능들에 대한 간략한 개요를 볼 수 있다.

커맨드 라인 툴, 스파이더, 셀렉터나 스크랩 데이터 모델링과 같이 튜토리얼에서 다루지 않은 것들에 대해 알고 싶다면
:ref:`section-basics` 를 확인하라.
예시 프로젝트로 배우는 것을 선호한다면 :ref:`intro-examples` 섹션을 보자.

.. _JSON: https://en.wikipedia.org/wiki/JSON
.. _dirbot: https://github.com/scrapy/dirbot
