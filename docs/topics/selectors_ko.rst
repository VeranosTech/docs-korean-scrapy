.. _topics-selectors:

====================
셀렉터(Selector)
====================

웹 페이지를 스크랩할 때, 가장 많이 해야 하는 작업은 HTML 자료로부터 데이터를
추출하는 일이다. 이런 일을 할 수 있는 라이브러리는 몇 가지가 있다.:

 * `BeautifulSoup`_\ 는 파이썬(Python) 프로그래머 사이에서 가장 인기있는 웹 스크랩핑
   라이브러리로 HTML의 구조를 기반으로 파이썬 객체를 생성하고 나쁜 마크업을 꽤 잘
   처리한다. 하지만 한 가지 단점이 있는데 느리다는 것이다.

 * `lxml`_\ 는 XML 파싱 라이브러리로 (HTML도 파싱할 수 있다) `ElementTree`_\ 을 기반으로
   하는 파이썬스러운 API다. (lxml은 파이썬 기본 라이브러리는 아니다.)

스크래피(Scrapy)는 자체적인 추출 매커니즘을 가지고 있다. 이 매커니즘은 `XPath`_\ 나 `CSS`_\ 표현으로
지정된 HTML 문서의 특정 부분을 선택하기 때문에 셀렉터라고 불린다.

`XPath`_\ 는 XML 문서에 있는 노드를 선택하는 언어이며 HTML에서도 사용할 수 있다.
`CSS`_\ 는 HTML 문서에 스타일을 적용시키기 위한 언어로 그 스타일과 특정한 HTML 요소를
연관시키는 셀렉터를 정의한다.

스크래피 셀렉터는 `lxml`_ 라이브러리를 기반으로 만들어져서 속도와 파싱의 정확도 면에서
매우 유사하다.

이 페이지는 셀렉터가 작동하는 방식과 매우 작고 간단한 API에 대해 설명했다.
(`lxml`_ API는 마크업 문서를 선택하는 것 뿐만 아니라 다른 많은 작업에서 사용되는
라이브러리이기 때문에 훨씬 크다.)

셀렉터 API에 대한 완전한 레퍼런스는 다음을 참고하라.
:ref:`Selector reference <topics-selectors-ref>`

.. _BeautifulSoup: https://www.crummy.com/software/BeautifulSoup/
.. _lxml: http://lxml.de/
.. _ElementTree: https://docs.python.org/2/library/xml.etree.elementtree.html
.. _cssselect: https://pypi.python.org/pypi/cssselect/
.. _XPath: https://www.w3.org/TR/xpath
.. _CSS: https://www.w3.org/TR/selectors


셀렉터 사용하기
=========================

셀렉터 생성하기
----------------------

.. highlight:: python

스크래피 셀렉터는 **텍스트** 또는 :class:`~scrapy.http.TextResponse` 객체를
전달해서 생성된 :class:`~scrapy.selector.Selector` 클래스의 인스턴스이다
셀렉터는 입력 타입에 기반해 자동으로 최고의 파싱 규칙 (XML vs HTML)을 선택한다::

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse

텍트스로 생성하기::

    >>> body = '<html><body><span>good</span></body></html>'
    >>> Selector(text=body).xpath('//span/text()').extract()
    [u'good']

리스펀스로 생성하기::

    >>> response = HtmlResponse(url='http://example.com', body=body)
    >>> Selector(response=response).xpath('//span/text()').extract()
    [u'good']

편의를 위해, 리스펀스 객체는 `.selector` 속성으로 셀렉터를 사용할 수 있다,
가능할 때 이 방법을 이용하는 것은 전혀 문제가 없다::

    >>> response.selector.xpath('//span/text()').extract()
    [u'good']


셀렉터 사용하기
-----------------------------

셀렉터를 사요하는 방법을 설명하기 위해서 우리는 (인터렉티브한 테스팅을 제공하는) `Scrapy Shell`\ 과
스크래피 문서 서버에 있는 예제 페이지를 활용할 것이다:

    https://doc.scrapy.org/en/latest/_static/selectors-sample1.html

.. _topics-selectors-htmlcode:

아래는 페이지의 HTML 코드다:

.. literalinclude:: ../_static/selectors-sample1.html
   :language: html

.. highlight:: sh

우선 shell을 열자::

    scrapy shell https://doc.scrapy.org/en/latest/_static/selectors-sample1.html

shell이 로드되면 사용자는 ``response`` shell 변수를 써서 사용 가능한 리스펀스와
``response.selector` 속성으로 붙은 셀렉터를 갖게 된다.

HTML을 처리하고 있기 대문에, 셀렉터는 자동적으로 HTML 파서를 사용한다.

.. highlight:: python

페이지의 :ref:`HTML 코드 <topics-selectors-htmlcode>`\ 를 보고 타이틀 태그에 있는 텍스트를
선택하기 위한 XPath 만들어 보자::

    >>> response.selector.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]

XPath나 CSS를 사용해 리스펀스를 쿼리하는 것은 매우 일반적인 일이라서 리스펀스는
두 숏컷을 포함하고 있다. ``response.xpath()``, ``response.css()``::

    >>> response.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    >>> response.css('title::text')
    [<Selector (text) xpath=//title/text()>]

보다시피, ``.xpath()``\ 와 ``.css()`` 메서드는
:class:`~scrapy.selector.SelectorList` 인스턴스를 반환하며, 이는 새로운
셀렉터 리스트다. 이 API는 중첩된 데이터를 빠르게 선택하는 데 사용할 수 있다::

    >>> response.css('img').xpath('@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

텍스트 데이터를 실제로 추출하려면 반드시 셀렉터의 ``.extract()`` 메서드를 아래처럼 호출해야 한다::

    >>> response.xpath('//title/text()').extract()
    [u'Example website']

가장 첫 번째에 메치된 요소를 추출하려면 셀렉터의 ``.extract_first()`` 메서드를 호출해야 한다.

    >>> response.xpath('//div[@id="images"]/a/text()').extract_first()
    u'Name: My image 1 '

찾아진 요소가 없으면 ``None``\ 을 반환한다:

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first() is None
    True

기본 리턴 값을 ``None`` 대신에 주어진 인자로 반환시킬 수 있다.:

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first(default='not-found')
    'not-found'

CSS 셀렉터는 CSS3 수에도-요소(pseudo-element)를 사용해서 텍스트와 속성 노드를
선택할 수 있다::

    >>> response.css('title::text').extract()
    [u'Example website']

이제 베이스 URL과 몇 가지 이미지 링크를 얻어볼 것이다::

    >>> response.xpath('//base/@href').extract()
    [u'http://example.com/']

    >>> response.css('base::attr(href)').extract()
    [u'http://example.com/']

    >>> response.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

    >>> response.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

.. _topics-selectors-nesting-selectors:

셀렉터 중첩하기
---------------------------------

선택 메서드(``.xpath()`` 또는 ``.css()``)는 같은 타입의 셀렉터 리스트를 반환한다,
따라서 그 셀렉터에도 선택 메서드를 호출할 수 있다. 아래는 그 예시다::

    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
     u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
     u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
     u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
     u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']

    >>> for index, link in enumerate(links):
    ...     args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
    ...     print 'Link number %d points to url %s and image %s' % args

    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']

정규 표현식으로 셀렉터 사용하기
----------------------------------------

:class:`~scrapy.selector.Selector`\ 는 정규식을 사용한 데이터 추출을 위해 ``.re()`` 메서드를
제공한다. 그러나 ``.xpath()``\ 나 ``.css()`` 메서드와는 다르게, ``.re()``\ 는 유니코드 문자열 리스트를
반환한다. 따라서 중첩된 ``.re()`` 호출을 생성할 수 없다.

아래는 위의 :ref:`HTML code <topics-selectors-htmlcode>`\ 에서 이미지 이름을 추출하는 예제다::

    >>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']

``.re()``\ 를 위해 ``.extract_first()``\ 를 반복하는 추가적인 헬퍼가 있으며,
이름은 ``.re_first()``\ 다. 첫 번째로 매치되는 문자열을 추출하기 위해 사용할 수 있다::

    >>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r'Name:\s*(.*)')
    u'My image 1'

.. _topics-selectors-relative-xpaths:

상대 XPath로 작업하기
--------------------------------

만약 셀렉터를 중첩하면서 ``/``\ 로 시작하는 XPath를 사용하는 경우, 그 XPath는 문서에 대해
절대적이며 호출되는 ``Selector``\ 에 대해 상대적이지 않다.

예를 들어 ``<div>`` 요소에 있는 모든 ``<p>`` 요소를 추출하고 싶다고 하자.
우선, 모든 ``<div>`` 요소를 얻는다::

    >>> divs = response.xpath('//div')

처음에, 아래와 같은 접근방식을 사용하려고 시도할 수도 있는데, 이는 잘못됐다,
왜냐하면 ``<div>`` 요소에 있는 것 뿐만 아니라 문서에 있는 모든 ``<p>`` 요소를 다 추출하기
때문이다::

    >>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
    ...     print p.extract()

아래가 적절한 방식이다 (``.//p`` XPath 가장 앞에 있는 점을 명심하기 바란다)::

    >>> for p in divs.xpath('.//p'):  # extracts all <p> inside
    ...     print p.extract()

다른 일반적인 경우는 모든 ``<p>`` 자식을 추출하는 것이다::

    >>> for p in divs.xpath('p'):
    ...     print p.extract()

상대적 XPath에 관한 자세한 정보는 XPath 설명서에 있는 `Location Paths`_\ 를
참고하라.

.. _Location Paths: https://www.w3.org/TR/xpath#location-paths

.. _topics-selectors-xpath-variables:

XPath 표현식의 변수
---------------------------------

XPath는 XPath 표현식에서 ``$somevariable`` 신택스를 사용해
변수를 참조하는 것을 허용한다. 이는 쿼리 내부의 인자를 ``?`` 같은 플레이스홀더로
대체해놓고 나중에 쿼리에 값을 전달하는 SQL 세계의 파라미터로 나타내진 쿼리 또는 준비된 명령문과 꽤 흡사하다.

아래는 (이전에 보여줬었던) 하드 코딩 없이 "id" 속성 값에 기반한 요소와 일치시키는 예제다::

    >>> # `$val` used in the expression, a `val` argument needs to be passed
    >>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
    u'Name: My image 1 '

아래는 다섯 개의 ``<a>`` 자식을 포함하는 ``<div>``  태그의 "id" 속성을 찾는
또 다른 예제다 (여기서는 integer인 ``5``\ 를 전달했다)::

    >>> response.xpath('//div[count(a)=$cnt]/@id', cnt=5).extract_first()
    u'images'

모든 변수 참조는 ``.xpath()``\ 를 호출할 때 바인드하는 값이 있어야 한다.
(그렇지 않으면 ``ValueError: XPath error:`` 예외가 발생한다).
이는 네임드 인자를 필요한 만큼 전달해서 실행한다.

스크래피 셀렉터를 강력하게 해주는 `parsel`_ 라이브러리에 관한 자세한 내용과 예제는 `XPath variables`_\ 에 나와있다.

.. _parsel: https://parsel.readthedocs.io/
.. _XPath variables: https://parsel.readthedocs.io/en/latest/usage.html#variables-in-xpath-expressions

EXSLT 확장 사용하기
--------------------------------

`lxml`_ 위에 빌드되어서, 스크래피 셀렉터는 `EXSLT`_ 확장도 지원하며
XPath 표현식에서 사용할 수 있는 사전 등록된 네임스페이스가 존재한다:


======  =====================================    =======================
prefix  namespace                                usage
======  =====================================    =======================
re      \http://exslt.org/regular-expressions    `regular expressions`_
set     \http://exslt.org/sets                   `set manipulation`_
======  =====================================    =======================

정규 표현식
~~~~~~~~~~~~~~~~~~~~~~

예를 들어 ``test()`` 함수는, XPath의 ``starts-with()`` 또는 ``contains()``\ 가 충분하지 않을 때
꽤 유용하다.

숫자로 끝나는 "class" 속성을 가진 리스트 아이템에서 링크를 선택하는 예제::

    >>> from scrapy import Selector
    >>> doc = """
    ... <div>
    ...     <ul>
    ...         <li class="item-0"><a href="link1.html">first item</a></li>
    ...         <li class="item-1"><a href="link2.html">second item</a></li>
    ...         <li class="item-inactive"><a href="link3.html">third item</a></li>
    ...         <li class="item-1"><a href="link4.html">fourth item</a></li>
    ...         <li class="item-0"><a href="link5.html">fifth item</a></li>
    ...     </ul>
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> sel.xpath('//li//@href').extract()
    [u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
    >>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()
    [u'link1.html', u'link2.html', u'link4.html', u'link5.html']
    >>>

.. warning:: C 라이브러리 ``libxslt``\ 는 기본적으로 EXSLT 정규 표현식을 지원하지 않는다.
    따라서 `lxml`_\ 의 구현은 파이썬의 ``re`` 모듈을 사용한다.
    그러므로 정규식 함수를 XPath 표현식에서 사용하는 것은 성능 측면에서 작은 패널티를
    주게 된다.

세트 작업
~~~~~~~~~~~~~~

텍스트 요소를 추출하기 전에 문서트리의 일부를 제외시키는 것이 편리할 수 있다.

아이템스코프(itemscope)와 대응하는 아이템프롭(itemprop)이 그룹이 있는 마이크로데이터를 추출하는 예제
(샘플 컨텐츠는 http://schema.org/Product\ 에서 가져왔다::

    >>> doc = """
    ... <div itemscope itemtype="http://schema.org/Product">
    ...   <span itemprop="name">Kenmore White 17" Microwave</span>
    ...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
    ...   <div itemprop="aggregateRating"
    ...     itemscope itemtype="http://schema.org/AggregateRating">
    ...    Rated <span itemprop="ratingValue">3.5</span>/5
    ...    based on <span itemprop="reviewCount">11</span> customer reviews
    ...   </div>
    ...
    ...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    ...     <span itemprop="price">$55.00</span>
    ...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
    ...   </div>
    ...
    ...   Product description:
    ...   <span itemprop="description">0.7 cubic feet countertop microwave.
    ...   Has six preset cooking categories and convenience features like
    ...   Add-A-Minute and Child Lock.</span>
    ...
    ...   Customer reviews:
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Not a happy camper</span> -
    ...     by <span itemprop="author">Ellie</span>,
    ...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1">
    ...       <span itemprop="ratingValue">1</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">The lamp burned out and now I have to replace
    ...     it. </span>
    ...   </div>
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Value purchase</span> -
    ...     by <span itemprop="author">Lucas</span>,
    ...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1"/>
    ...       <span itemprop="ratingValue">4</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">Great microwave for the price. It is small and
    ...     fits in my apartment.</span>
    ...   </div>
    ...   ...
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> for scope in sel.xpath('//div[@itemscope]'):
    ...     print "current scope:", scope.xpath('@itemtype').extract()
    ...     props = scope.xpath('''
    ...                 set:difference(./descendant::*/@itemprop,
    ...                                .//*[@itemscope]/*/@itemprop)''')
    ...     print "    properties:", props.extract()
    ...     print

    current scope: [u'http://schema.org/Product']
        properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']

    current scope: [u'http://schema.org/AggregateRating']
        properties: [u'ratingValue', u'reviewCount']

    current scope: [u'http://schema.org/Offer']
        properties: [u'price', u'availability']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    >>>

위에서 우리는 일단 ``itemscope`` 요소에 대해 반복을 했고, 각각에 대해
모든 ``itemprops`` 요소를 찾았다. 그 다음 또다른 ``itemscope`` 안에 있는 것들을
제외시켰다.

.. _EXSLT: http://exslt.org/
.. _regular expressions: http://exslt.org/regexp/index.html
.. _set manipulation: http://exslt.org/set/index.html


XPath 팁
---------------

스크래피 셀렉터로 XPath를 사용할 때 유용한 팁들이
`this post from ScrapingHub's blog`_\ 에 있다.
XPath에 아직 익숙하지 않다면 먼저 `XPath tutorial`_\ 을
보는 것도 좋다.


.. _`XPath tutorial`: http://www.zvon.org/comp/r/tut-XPath_1.html
.. _`this post from ScrapingHub's blog`: https://blog.scrapinghub.com/2014/07/17/xpath-tips-from-the-web-scraping-trenches/


조건이 있는 텍스트 노드 사용하기
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`XPath string function`_\ 에 인자로 텍스트 건텐츠를 사용할 필요가 있을 때,
``.//text()``\ 사용을 피하고 ``.``\ 만 사용하라.

왜냐하면 ``.//text()`` 표현은 텍스트 요소의 집합을 생산하기 때문이다 -- *노드-세트* .
그리고 ``contains()`` 또는 ``starts-with()`` 같은 문자열 함수에 인자로 전달돼서 노드-세트가 문자열로 변환될 때,
첫 번째 요소에 대한 텍스트만 불러온다.

예::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')

*노드-세트*\ 를 문자열로 변환::

    >>> sel.xpath('//a//text()').extract() # 노드-셋을 본다
    [u'Click here to go to the ', u'Next Page']
    >>> sel.xpath("string(//a[1]//text())").extract() # 문자열로 변환한다
    [u'Click here to go to the ']

그러나 문자열로 변환된 *노드*\ 는 텍스트와 모든 디센던트(descendant)를 합쳐버린다::

    >>> sel.xpath("//a[1]").extract() # 첫 번째 노드를 선택한다
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
    >>> sel.xpath("string(//a[1])").extract() # 문자열로 변환한다
    [u'Click here to go to the Next Page']

따라서, ``.//text()`` 노드-세트를 사용하는 것은 이 경우에 아무것도 선택하지 않는다::

    >>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
    []

하지만 노드를 의미하는 ``.``\ 를 사용하면 작동한다::

    >>> sel.xpath("//a[contains(., 'Next Page')]").extract()
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']

.. _`XPath string function`: https://www.w3.org/TR/xpath/#section-String-Functions

//node[1]와 (//node)[1]의 차이를 주의하라
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``//node[1]``\ 는 각각의 부모 아래서 처음으로 발생하는 모든 노드를 선택한다.

``(//node)[1]``\ 는 문서 내에서 모든 노드를 선택하고, 그중 첫 번째만 가져온다.

예::

    >>> from scrapy import Selector
    >>> sel = Selector(text="""
    ....:     <ul class="list">
    ....:         <li>1</li>
    ....:         <li>2</li>
    ....:         <li>3</li>
    ....:     </ul>
    ....:     <ul class="list">
    ....:         <li>4</li>
    ....:         <li>5</li>
    ....:         <li>6</li>
    ....:     </ul>""")
    >>> xp = lambda x: sel.xpath(x).extract()

아래는 부모에 상관없이 모든 첫 번째 ``<li>`` 요소를 가져온다::

    >>> xp("//li[1]")
    [u'<li>1</li>', u'<li>4</li>']

그리고 아래는 전체 문서의 첫 번째 ``<li>`` 요소만 가져온다::

    >>> xp("(//li)[1]")
    [u'<li>1</li>']

아래는 ``<ul>`` 부모 아래 있는 모든 첫 번째 ``<li>`` 요소만 가지고 온다::

    >>> xp("//ul/li[1]")
    [u'<li>1</li>', u'<li>4</li>']

그리고 아래는 전체 문서의 ``<ul>`` 부모 아래 있는 첫 번째 ``<li>`` 요소만 가지고 온다::

    >>> xp("(//ul/li)[1]")
    [u'<li>1</li>']

클래스로 쿼리 할 때, CSS 사용을 고려하라
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

요소가 여러 CSS 클래스를 포함할 수 있기 때문에, 클래스로 요소를 선택하는 XPath 방식은
다소 장황하다::

    *[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

만약 ``@class='someclass'``\ 로 쓴다면 다른 클래스를 가진 요소를 놓치게 될 것이다.
그리고 ``contains(@class, 'someclass')``\ 로 쓰면 ``someclass`` 문자열을 공유하는 다른
클래스 이름이 있으면 원하는 것 보다 많은 요소를 얻게 된다.

스크래피 셀렉터는 셀렉터를 연결시켜서 사용할 수 있기 때문에, 대부분의 경우 CSS를 사용해
클래스로 선택을 한 다음 필요할 때 XPath로 전환하면 된다::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
    >>> sel.css('.shout').xpath('./time/@datetime').extract()
    [u'2014-07-23 19:00']

이것은 위의 장황한 XPath 트릭을 사용하는 것 보다 훨씬 깔끔하다.
단지 뒤따르는 XPath 표현식에서 ``.``\ 를 사용하는 것을 기억하라.


.. _topics-selectors-ref:

빌트인 셀렉터 레퍼런스
====================================

.. module:: scrapy.selector
   :synopsis: Selector class

Selector 객체
----------------

.. class:: Selector(response=None, text=None, type=None)

  :class:`Selector` 인스턴스는 리스펀스의 래퍼(wrapper)로 컨텐츠의 특정 부분을 선택하게 해준다.

  ``response``\ 는:class:`~scrapy.http.HtmlResponse` or an
  :class:`~scrapy.http.XmlResponse` object that will be used for selecting and
  extracting data.

  ``text`` is a unicode string or utf-8 encoded text for cases when a
  ``response`` isn't available. Using ``text`` and ``response`` together is
  undefined behavior.

  ``type`` defines the selector type, it can be ``"html"``, ``"xml"`` or ``None`` (default).

    If ``type`` is ``None``, the selector automatically chooses the best type
    based on ``response`` type (see below), or defaults to ``"html"`` in case it
    is used together with ``text``.

    If ``type`` is ``None`` and a ``response`` is passed, the selector type is
    inferred from the response type as follows:

        * ``"html"`` for :class:`~scrapy.http.HtmlResponse` type
        * ``"xml"`` for :class:`~scrapy.http.XmlResponse` type
        * ``"html"`` for anything else

   Otherwise, if ``type`` is set, the selector type will be forced and no
   detection will occur.

  .. method:: xpath(query)

      Find nodes matching the xpath ``query`` and return the result as a
      :class:`SelectorList` instance with all elements flattened. List
      elements implement :class:`Selector` interface too.

      ``query`` is a string containing the XPATH query to apply.

      .. note::

          For convenience, this method can be called as ``response.xpath()``

  .. method:: css(query)

      Apply the given CSS selector and return a :class:`SelectorList` instance.

      ``query`` is a string containing the CSS selector to apply.

      In the background, CSS queries are translated into XPath queries using
      `cssselect`_ library and run ``.xpath()`` method.

      .. note::

          For convenience this method can be called as ``response.css()``

  .. method:: extract()

     Serialize and return the matched nodes as a list of unicode strings.
     Percent encoded content is unquoted.

  .. method:: re(regex)

     Apply the given regex and return a list of unicode strings with the
     matches.

     ``regex`` can be either a compiled regular expression or a string which
     will be compiled to a regular expression using ``re.compile(regex)``

    .. note::

        Note that ``re()`` and ``re_first()`` both decode HTML entities (except ``&lt;`` and ``&amp;``).

  .. method:: register_namespace(prefix, uri)

     Register the given namespace to be used in this :class:`Selector`.
     Without registering namespaces you can't select or extract data from
     non-standard namespaces. See examples below.

  .. method:: remove_namespaces()

     Remove all namespaces, allowing to traverse the document using
     namespace-less xpaths. See example below.

  .. method:: __nonzero__()

     Returns ``True`` if there is any real content selected or ``False``
     otherwise.  In other words, the boolean value of a :class:`Selector` is
     given by the contents it selects.


SelectorList objects
--------------------

.. class:: SelectorList

   The :class:`SelectorList` class is a subclass of the builtin ``list``
   class, which provides a few additional methods.

   .. method:: xpath(query)

       Call the ``.xpath()`` method for each element in this list and return
       their results flattened as another :class:`SelectorList`.

       ``query`` is the same argument as the one in :meth:`Selector.xpath`

   .. method:: css(query)

       Call the ``.css()`` method for each element in this list and return
       their results flattened as another :class:`SelectorList`.

       ``query`` is the same argument as the one in :meth:`Selector.css`

   .. method:: extract()

       Call the ``.extract()`` method for each element in this list and return
       their results flattened, as a list of unicode strings.

   .. method:: re()

       Call the ``.re()`` method for each element in this list and return
       their results flattened, as a list of unicode strings.


Selector examples on HTML response
----------------------------------

Here's a couple of :class:`Selector` examples to illustrate several concepts.
In all cases, we assume there is already a :class:`Selector` instantiated with
a :class:`~scrapy.http.HtmlResponse` object like this::

      sel = Selector(html_response)

1. Select all ``<h1>`` elements from an HTML response body, returning a list of
   :class:`Selector` objects (ie. a :class:`SelectorList` object)::

      sel.xpath("//h1")

2. Extract the text of all ``<h1>`` elements from an HTML response body,
   returning a list of unicode strings::

      sel.xpath("//h1").extract()         # this includes the h1 tag
      sel.xpath("//h1/text()").extract()  # this excludes the h1 tag

3. Iterate over all ``<p>`` tags and print their class attribute::

      for node in sel.xpath("//p"):
          print node.xpath("@class").extract()

Selector examples on XML response
---------------------------------

Here's a couple of examples to illustrate several concepts. In both cases we
assume there is already a :class:`Selector` instantiated with an
:class:`~scrapy.http.XmlResponse` object like this::

      sel = Selector(xml_response)

1. Select all ``<product>`` elements from an XML response body, returning a list
   of :class:`Selector` objects (ie. a :class:`SelectorList` object)::

      sel.xpath("//product")

2. Extract all prices from a `Google Base XML feed`_ which requires registering
   a namespace::

      sel.register_namespace("g", "http://base.google.com/ns/1.0")
      sel.xpath("//g:price").extract()

.. _removing-namespaces:

Removing namespaces
-------------------

When dealing with scraping projects, it is often quite convenient to get rid of
namespaces altogether and just work with element names, to write more
simple/convenient XPaths. You can use the
:meth:`Selector.remove_namespaces` method for that.

Let's show an example that illustrates this with GitHub blog atom feed.

.. highlight:: sh

First, we open the shell with the url we want to scrape::

    $ scrapy shell https://github.com/blog.atom

.. highlight:: python

Once in the shell we can try selecting all ``<link>`` objects and see that it
doesn't work (because the Atom XML namespace is obfuscating those nodes)::

    >>> response.xpath("//link")
    []

But once we call the :meth:`Selector.remove_namespaces` method, all
nodes can be accessed directly by their names::

    >>> response.selector.remove_namespaces()
    >>> response.xpath("//link")
    [<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     ...

If you wonder why the namespace removal procedure isn't always called by default
instead of having to call it manually, this is because of two reasons, which, in order
of relevance, are:

1. Removing namespaces requires to iterate and modify all nodes in the
   document, which is a reasonably expensive operation to perform for all
   documents crawled by Scrapy

2. There could be some cases where using namespaces is actually required, in
   case some element names clash between namespaces. These cases are very rare
   though.

.. _Google Base XML feed: https://support.google.com/merchants/answer/160589?hl=en&ref_topic=2473799
