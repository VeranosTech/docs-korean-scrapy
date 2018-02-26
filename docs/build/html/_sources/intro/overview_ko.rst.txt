.. _intro-overview:

================================
스크래피(Scrapy) 한눈에 보기
================================

스크래피는 웹사이트를 크롤링하고 구조화된 데이터를 추출하는 어플리케이션 프레임워크로
데이터 마이닝, 정보 처리 또는 기록 아카이브 같은 응용 분야에 널리 유용하게 쓰인다.

비록 스크래피가 `웹 스크래핑`_\ 용으로 설계되었지만,
(`Amazon Associates Web Services`_ 같은) API를 사용해서 데이터를 추출하거나
범용 웹크롤러로도 사용할 수 있다.


예제 스파이더 살펴보기
=================================

스크래피가 무엇을 제공하는지 보여주기 위해,
가장 간단한 형태로 스크래피 스파이더 예제를 살펴볼 것이다.

아래는 http://quotes.toscrape.com 웹사이트에서
페이지를 따라가면서 유명한 인용구를 스크랩하는 스파이더 코드다::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/tag/humor/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.xpath('span/small/text()').extract_first(),
                }

            next_page = response.css('li.next a::attr("href")').extract_first()
            if next_page is not None:
                yield response.follow(next_page, self.parse)


위 코드를  ``quotes_spider.py``\ 와 같은 이름으로 텍스트 파일로 저장한 후
:command:`runspider` 커맨드를 사용해서 스파이더를 실행한다.::

    scrapy runspider quotes_spider.py -o quotes.json


작업이 끝나면 ``quotes.json`` 파일에 텍스트와 저자를 포함하는 JSON 포맷 인용구 리스트가
생길 것이다. 파일 내용은 다음처럼 보일 것이다.(여기서는 가독성을 위해서 재포맷팅을 했다)::

    [{
        "author": "Jane Austen",
        "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"
    },
    {
        "author": "Groucho Marx",
        "text": "\u201cOutside of a dog, a book is man's best friend. Inside of a dog it's too dark to read.\u201d"
    },
    {
        "author": "Steve Martin",
        "text": "\u201cA day without sunshine is like, you know, night.\u201d"
    },
    ...]


어떤 일이 일어났나?
-------------------

``scrapy runspider quotes_spider.py`` 커맨드를 실행하면,
스크래피는 그 안에서 스파이더 정의(Spier definition)를 찾고 크롤러 엔진을 통해
스파이더(spider)를 실행한다.

크롤링은 ``start_urls`` 속성에서 정의된 (예제의 경우, *humor* 카테고리에 있는 인용구를 위한
URL) URL에 대한 리퀘스트를 생성한 후 기본 콜백 메서드인 ``parse`` 를 호출하고,
리스펀스 객체를 인자로 전달한다. ``parse`` 콜백에서, CSS 셀렉터를 사용해
인용구 요소에 대해 루프를 돌리고 추출된 인용구와 텍스트, 저자를 포함한 파이썬 dict를
생산한 다음, 다음 페이지로 향하는 링크를 찾고 같은 ``parse`` 메서드를 콜백으로
사용해 다음 리퀘스트를 예약한다.

여기에서 스크래피의 주요한 장점 중 하나를 알 수 있는데
리퀘스트의 :ref:`비동기적 처리 <topics-architecture>`\ 가 가능하다는 점이다.
스크래피는 리퀘스트가 종료되고 처리되는 것을 기다릴 필요가 없이
동시에 다른 리퀘스트를 보내거나 다른 작업을 할 수 있다.
즉, 일부 리퀘스트가 처리될 때 실패하거나 에러가 발생하는 경우에도
다른 리퀘스트는 계속 진행이 된다.

스크래피는 장애 허용(fault-tolerant) 방식으로 한 번에 다수의 동시적인 리퀘스트를 보내서
빠르게 크롤링을 한다.
동시에 스크래피는 :ref:`몇 가지 세팅 <topics-settings-ref>`\ 을 통해
크롤링의 politeness에 대한 통제권을 제공한다.
각 리퀘스트 사이의 다운로드 딜레이를 조정하고,
각 도메인이나 IP에 대한 동시적인 리퀘스트의 수를 제한하고,
심지어 자동적으로 이런 것들을 산출하는 :ref:`자동 쓰로틀링(auto-throttling) 확장 <topics-autothrottle>`\ 을
사용할 수도 있다.

.. note::

    이 예제는 JSON 파일을 생성하기 위해 :ref:`피드 익스포트(feed exports) <topics-feed-exports>`\ 를
    사용했다. 포맷(XML, CSV 등)이나 스토리지 백엔드(FTP, `Amazon S3`_ 등)는 쉽게 바꿀 수 있다.
    또한 :ref:`아이템 파이프라인 <topics-item-pipeline>`\ 을 만들어서
    아이템을 데이터베이스에 저장할 수도 있다.


.. _topics-whatelse:

그 밖의 사항
==================

스크래피를 사용해서 웹사이트로부터 아이템을 추출하고 저장하는 법을 배웠지만,
이것은 시작에 불과하다. 스크래피는 스크랩핑을 쉽고 효율적으로 만드는
강력한 많은 기능들을 제공한다:

* 정규식을 사용한 추출을 가능하게 하는 헬퍼 메서드를 포함해,
  확장 CSS Selector와 XPath 표현식을 사용해서 HTML/XML 자료로부터
  데이터를 :ref:`선택하고 추출하는 <topics-selectors>` 작업 기능을 내장.

* CSS와 XPath 표현식으로 데이터를 스크랩하는 것을 시험해볼 수 있고, 스파이더를
  디버깅할 때 매우 유용한 :ref:`대화형 쉘 콘솔 <topics-shell>` (IPython 인식).

* 다양한 포맷(JSON, CSV, XML)의 :ref:`피드 익스포트 생성 <topics-feed-exports>`\ 과
  다양한 백엔드로의 저장(FTP, S3, 로컬 파일 시스템) 기능을 내장.

* 외국어, 비표준, 망가진 인코딩 선언 처리를 위한 강력한 인코딩 지원과 자동 감지.

* :ref:`시그널 <topics-signals>`\ 을 사용한 사용자 지정 플러그인을 허용하는
  :ref:`강력한 확장 지원 <extending-scrapy>`, 명확히 정의된 API
  (미들웨어, :ref:`확장 <topics-extensions>`,
  :ref:`파이프라인 <topics-item-pipeline>`).

* 광범위한 미들웨어 및 내장 기능 확장:

  - 쿠키 및 세선 조작
  - 압축, 인증, 캐싱 등의 HTTP 기능
  - 사용자-에이전트 스푸핑
  - robots.txt
  - 크롤링 깊이 제한
  - 기타

* 크롤러를 검사하고 디버깅하기 위해, 스크래피 프로세스 내에서 실행되고 있는 파이썬 콘솔에 연결하는
  :ref:`텔넷 콘솔 <topics-telnetconsole>`

* `Sitemaps`_\ 과 XML/CSV 피드가 있는 사이트를 크롤링하는 재사용 가능한 스파이더와,
  스크랩된 아이템과 연결된 :ref:`자동 이미지(또는 다른 미디어) 다운로드 <topics-media-pipeline>`\ 용
  미디어 파이프라인, 캐싱 DNS resolver, 등

다음 단계
====================

다음 단계로는 :ref:`스크래피를 설치하는 <intro-install>` 것을 설명한다.
이어서 :ref:`튜토리얼 <intro-tutorial>`\ 을 따라하며 완전한 스크래피 프로젝트를
만드는 방법을 배우고 `커뮤니티에 참여하기`_ 바란다.

.. _커뮤니티에 참여하기: https://scrapy.org/community/
.. _웹 스크래핑: https://en.wikipedia.org/wiki/Web_scraping
.. _Amazon Associates Web Services: https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html
.. _Amazon S3: https://aws.amazon.com/s3/
.. _Sitemaps: https://www.sitemaps.org/index.html
