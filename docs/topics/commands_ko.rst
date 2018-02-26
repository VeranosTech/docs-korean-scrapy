.. _topics-commands:

=====================
커맨드 라인 툴
=====================

.. versionadded:: 0.10

스크래피는 ``scrapy`` 커맨드라인 툴을 이용해서 제어한다, 이 문서에서는
우리가 단순히 "커맨드" 또는 "스크래피 커맨드"라 부르는 서브 커맨드와 구별하기 위해
"스크래피 툴"이라고 할 것이다.

스크래피 툴은 다양한 목적을 위해 몇 가지 커맨드를 제공하며, 각각은
다른 인자와 옵션 세트를 사용한다.

(``scrapy deploy`` 커맨드는 독립된 ``scrapyd-deploy``\ 을 위해 1.0 버전에서부터 제거되었다.
`Deploying your project`_\ 를 참고하라.)

.. _topics-config-settings:

설정 세팅
======================

스크래피는 기준 위치에 있는 ini 스타일의 ``scrapy.cfg`` 파일에서 설정 파라미터를 찾는다:

1. ``/etc/scrapy.cfg`` 또는 ``c:\scrapy\scrapy.cfg`` (시스템 수준),
2. ``~/.config/scrapy.cfg`` (``$XDG_CONFIG_HOME``) 그리고 ``~/.scrapy.cfg`` (``$HOME``)
   전역 (사용자 수준) 세팅용, 그리고
3. 스크래피 프로젝트 루트 내부의 ``scrapy.cfg`` (다음 섹션 참고).

이 파일들에서의 세팅은 나열된 선호도 순서대로 병합된다:
사용자 지정 값은 시스템 수준 기본값보다 우선순위가 높으며
프로젝트 수준 세팅은 다른 모든 세팅을 오버라이드한다.

또한 스크래피는 많은 환경 변수를 이해할 수 있고 그것들을 통해 설정될 수 있다.
현재는 아래와 같다:

* ``SCRAPY_SETTINGS_MODULE`` (:ref:`topics-settings-module-envvar` 참고)
* ``SCRAPY_PROJECT``
* ``SCRAPY_PYTHON_SHELL`` (:ref:`topics-shell` 참고)

.. _topics-project-structure:

스크래피 프로젝트의 기본 구조
=========================================

커맨드라인 툴과 하위 커맨드를 알아보기 전에, 먼저
스크래피 프로젝트의 디렉토리 구조를 먼저 살펴보자.

수정할 수 있지만, 모든 스크래피 프로젝트는 기본적으로 아래와 유사한
같은 파일을 가지고 있다::

   scrapy.cfg
   myproject/
       __init__.py
       items.py
       middlewares.py
       pipelines.py
       settings.py
       spiders/
           __init__.py
           spider1.py
           spider2.py
           ...

``scrapy.cfg`` 파일은 *프로젝트 루트 디렉토리*\ 라는 디렉토리에 위치하다.
이 파일은 프로젝트 세팅을 정의하는 파이썬 모듈의 이름을 포함하고 있다.
아래는 예시다::

    [settings]
    default = myproject.settings

``scrapy`` 툴 사용하기
=========================

인자 없이 스크래피 툴을 실행해서 시작하면 사용 도움말과 이용 가능한 커맨드를 출력한다::

    Scrapy X.Y - no active project

    Usage:
      scrapy <command> [options] [args]

    Available commands:
      crawl         Run a spider
      fetch         Fetch a URL using the Scrapy downloader
    [...]

만약 스크래피 프로젝트 내에 있으면 첫 번재 줄은 현재 활성화되어있는 프로젝트를 출력한다.
이 예시에서는 프로젝트 바깥에서 실행됐다. 만약 프로젝트 안에서 실행하면
아래와 같이 출력된다::

    Scrapy X.Y - project: myproject

    Usage:
      scrapy <command> [options] [args]

    [...]

프로젝트 생성하기
-----------------------

``scrapy`` 툴로 첫 번째로 하는 일은 스크래피 프로젝트를 만드는 것이다::

    scrapy startproject myproject [project_dir]

위 명령은 스크래피 프로젝트를 ``project_dir`` 디렉토리 안에 생성시킨다.
``proejct_dir``\ 이 지정되어있지 않으면 ``project_dir``\ 은 ``myproject``\ 가 된다.

새 프로젝트 디렉토리로 들어가자::

    cd project_dir

이제 프로젝트를 관리하고 제어하기 위한 ``scrapy`` 커맨드를 사용할 준비가 되었다.

프로젝트 제어하기
---------------------------

사용자는 프로젝트 내부에서 ``scrapy`` 툴을 사용해서 프로젝트를 관리하고 제어한다.

예를 아래 명령으로 새 스파이더를 생성할 수 있다::

    scrapy genspider mydomain mydomain.com

몇몇 스크래피 커맨드(:command:`crawl` 등)는 반드시 스크래피 프로젝트 안에서 실행
해야 한다. 어떤 커맨드를 프로젝트 안에서 써야 하는지에 대한 자세한 정보는
아래의 :ref:`commands reference <topics-commands-ref>`\ 를 참고하라라.

또한 몇몇 커맨드는 프로젝트 안에서 실행 될 때 약간 다른 동작을 한다는 사실을 명심하라
예를 들어, 가져오고 있는 url이 특정한 스파이더와 연결되어 있으면 fetch 커맨드는
오버라이드된 스파이더 동작을 사용한다. (``user_agent`` 속성이
user-agent를 오버라이드 하는 것처럼) 이것은 의도된 것으로, ``fetch`` 커맨드는 스파이더 페이지를 다운로드하고 있는 방식을
확인하기 위해 사용되는 것이기 때문이다

.. _topics-commands-ref:

사용 가능한 툴 커맨드
================================

이 섹션은 사용 가능한 내장 커맨드에 대한 설명과 사용 예시를 포함한
리스트로 구성되어 있다. 기억하라, 언제나 아래와 같이 실행하면 더 많은 정보를
얻을 수 있다::

    scrapy <command> -h

모든 사용가능한 커맨드는 아래와 같이 실행해서 볼 수 있다::

    scrapy -h

커맨드는 스크래피 프로젝트 안에서만 작동하는 것(프로젝트 한정 커맨드)과 프로젝트 안에서
실행할 때 (프로젝트의 오버라이드된 설정을 사용하기 때문에) 약간 다르게 작동하지만
활성화된 스크래피 프로젝트 없이도 작동하는 것(글로벌 커맨드)이 있다.

글로벌 커맨드:

* :command:`startproject`
* :command:`genspider`
* :command:`settings`
* :command:`runspider`
* :command:`shell`
* :command:`fetch`
* :command:`view`
* :command:`version`

프로젝트 한정 커맨드:

* :command:`crawl`
* :command:`check`
* :command:`list`
* :command:`edit`
* :command:`parse`
* :command:`bench`

.. command:: startproject

startproject
------------

* Syntax: ``scrapy startproject <project_name> [project_dir]``
* Requires project: *no*

``project_name``의 새로운 스크래피 프로젝트를 ``project_dir`` 디렉토리 내에 생성한다.
``project_dir``\ 가 지정되지 않으면 ``project_dir``\ 는 ``project_name``\ 가 된다.

사용 예시::

    $ scrapy startproject myproject

.. command:: genspider

genspider
---------

* Syntax: ``scrapy genspider [-t template] <name> <domain>``
* Requires project: *no*

프로젝트 내에서 호출되면 현재 폴더에나 현재 폴더의 ``spiders`` 폴더에 새로운 스파이더를 생성한다.
``<name>`` 파라미더는 스파이더의 ``name``\ 으로 설정되고, ``<domain>``\ 는 ``allowed_domains``\ 과 ``start_urls`` 스파이더 속성을
생성하기 위해 사용된다.

사용 예시::

    $ scrapy genspider -l
    Available templates:
      basic
      crawl
      csvfeed
      xmlfeed

    $ scrapy genspider example example.com
    Created spider 'example' using template 'basic'

    $ scrapy genspider -t crawl scrapyorg scrapy.org
    Created spider 'scrapyorg' using template 'crawl'

이것은 사전에 정의된 템플릿을 기반으로 스파이더를 만드는 편리한 숏컷 커맨드다.
하지만 스파이더를 만드는 방법은 하나가 아니다.
이 커맨드를 사용하는 대신에 직접 스파이더 소스코드 파일을 생성해도 된다.

.. command:: crawl

crawl
-----

* Syntax: ``scrapy crawl <spider>``
* Requires project: *yes*

스파이더를 사용해서 크롤링을 시작한다.

사용 예시::

    $ scrapy crawl myspider
    [ ... myspider starts crawling ... ]


.. command:: check

check
-----

* Syntax: ``scrapy check [-l] <spider>``
* Requires project: *yes*

contract 확인을 실행한다.

사용 예시::

    $ scrapy check -l
    first_spider
      * parse
      * parse_item
    second_spider
      * parse
      * parse_item

    $ scrapy check
    [FAILED] first_spider:parse_item
    >>> 'RetailPricex' field is missing

    [FAILED] first_spider:parse
    >>> Returned 92 requests, expected 0..4

.. command:: list

list
----

* Syntax: ``scrapy list``
* Requires project: *yes*

현재 프로젝트 내의 사용가능한 모든 스파이더를 나열한다. 한 줄에 한 스파이더씩 출력된다.

사용 예시::

    $ scrapy list
    spider1
    spider2

.. command:: edit

edit
----

* Syntax: ``scrapy edit <spider>``
* Requires project: *yes*

``EDITOR`` 환경 변수나 (설정되어 있지 않으면) :setting:`EDITOR`\ 에서
정의된 에디터를 사용해 주어진 스파이더를 편집한다.

이 커맨드는 가장 일반적인 경우를 위한 편리한 숏컷으로 제공되는 것이다.
개발자는 당연히 스파이더를 작성하고 디버그하는 데 사용할 IDE나 툴을 마음대로 고를 수 있다.

사용 예시::

    $ scrapy edit spider1

.. command:: fetch

fetch
-----

* Syntax: ``scrapy fetch <url>``
* Requires project: *no*

스크래피 다운로더를 사용해 주어진 URL을 다운로드하고 표준 출력에 컨텐츠를 작성한다.

이 커맨드의 흥미로운 점은 스파이더가 다운로드하는 방식을 페이지에 불러온다는 점이다.
예를 들어, 만약 스파이더가 User Agent를 오버라이드 하는 ``USER_AGENT`` 속성을
가지고 있으면 이 커맨드는 그것을 사용한다.

따라서 이 커맨드는 스파이더가 특정 페이지를 불러오는 방식을 보기위해 사용된다.

프로젝트 밖에서 사용되면, 스파이더에 따른 특별핸 작동은 일어나지 않으며
기본 스크래피 다운로더 세팅을 사용할 것이다.

지원되는 옵션:

* ``--spider=SPIDER``: 우회 스파이더 자동감지 그리고 특정 스파이더 사용 강제

* ``--headers``: 리스펀스의 바디 대신 HTTP 헤더 출력

* ``--no-redirect``: HTTP의 3xx 리다이렉션 따라가지 않기 (기본은 따라가는 것을 설정)

사용 예시::

    $ scrapy fetch --nolog http://www.example.com/some/page.html
    [ ... html content here ... ]

    $ scrapy fetch --nolog --headers http://www.example.com/
    {'Accept-Ranges': ['bytes'],
     'Age': ['1263   '],
     'Connection': ['close     '],
     'Content-Length': ['596'],
     'Content-Type': ['text/html; charset=UTF-8'],
     'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
     'Etag': ['"573c1-254-48c9c87349680"'],
     'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
     'Server': ['Apache/2.2.3 (CentOS)']}

.. command:: view

view
----

* Syntax: ``scrapy view <url>``
* Requires project: *no*

스크래피 스파이더가 보는 것처럼, 주어진 URL을 브라우저에서 연다.
종종 스파이더는 페이지를 일반적인 사용자와 다르게 본다, 따라서
이 스파이더가 무엇을 보는지 확인하고 기대했던 것인지 확인한다

지원 옵션:

* ``--spider=SPIDER``: 우회 스파이더 자동 탐지 및 특정 스파이더 사용 강제

* ``--no-redirect``: HTTP 3xx 리다이렉트 따라가지 않기 (기본은 따라가는 것으로 설정)

사용 예시::

    $ scrapy view http://www.example.com/some/page.html
    [ ... browser starts ... ]

.. command:: shell

shell
-----

* Syntax: ``scrapy shell [url]``
* Requires project: *no*

주어진 URL에 대해 shell을 시작하거나 URL이 주어지지 않았으면 빈 상태로 시작한다.
또한 UNIX 스타일의 로컬 파일경로 (``./``, ``../`` 또는 절대경로)를 지원한다.
상세한 정보는 :ref:`topics-shell`\ 를 참고하라.

지원 옵션:

* ``--spider=SPIDER``: 우회 스파이더 자동 탐색 및 특정 스파이더 사용 강제

* ``-c code``: shell에 있는 코드를 평가하고 결고를 출력한 뒤 종료

* ``--no-redirect``: HTTP 3xx 리다이렉트 따라가지 않기 (기본은 따라가는 것으로 설정);
  커맨드라인에서 인자로 전달된 URL에 대해서만 영향을 미친다;
  일단 shell 안에 있으면 ``fetch(url)``\ 가 기본적으로 계속해서 HTTP 리다이렉트를 따라갈 것이다.

사용 예시::

    $ scrapy shell http://www.example.com/some/page.html
    [ ... scrapy shell starts ... ]

    $ scrapy shell --nolog http://www.example.com/ -c '(response.status, response.url)'
    (200, 'http://www.example.com/')

    # shell follows HTTP redirects by default
    $ scrapy shell --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
    (200, 'http://example.com/')

    # you can disable this with --no-redirect
    # (only for the URL passed as command line argument)
    $ scrapy shell --no-redirect --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
    (302, 'http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F')


.. command:: parse

parse
-----

* Syntax: ``scrapy parse <url> [options]``
* Requires project: *yes*

주어진 URL을 불러오고 ``--callback`` 옵션으로 전달된 메서드나 옵션이 없으면 ``parse``\ 를 사용해 스파이더로
파싱한다.

지원 옵션:

* ``--spider=SPIDER``: 우회 스파이더 자동 탐색 및 특정 스파이더 사용 강제

* ``--a NAME=VALUE``: 스파이더 인자 설정 (반복해서 쓸 수 있음)

* ``--callback`` 또는 ``-c``: 리스펀스를 파싱하기 위해 콜백으로 사용되는 메서드

* ``--pipelines``: 파이프라인을 통한 아이템 처리

* ``--rules`` 또는 ``-r``: 리스펀스를 파싱하기 위해 사용할 콜백을 탐색하기 위해
  :class:`~scrapy.spiders.CrawlSpider` rule 사용

* ``--noitems``: 스크랩된 아이템 보이지 않기

* ``--nolinks``: 추출된 링크 보이지 않기

* ``--nocolour``: 출력에 색을 입히지 않기

* ``--depth`` 또는 ``-d``: 재귀적으로 리퀘스트를 따라가야 하는 깊이 수준(기본: 1)

* ``--verbose`` 또는 ``-v``: 깊이 수준에 대한 정보 표시

사용 예시::

    $ scrapy parse http://www.example.com/ -c parse_item
    [ ... scrapy log lines crawling example.com spider ... ]

    >>> STATUS DEPTH LEVEL 1 <<<
    # Scraped Items  ------------------------------------------------------------
    [{'name': u'Example item',
     'category': u'Furniture',
     'length': u'12 cm'}]

    # Requests  -----------------------------------------------------------------
    []


.. command:: settings

settings
--------

* Syntax: ``scrapy settings [options]``
* Requires project: *no*

스크래피 세팅 값을 얻는다.

프로젝트 안에서 사용되면 프로젝트의 세팅 값을 보여준다, 그렇지 않은 경우
세팅에 대한 기본 스크래피 값을 보여준다.

예시::

    $ scrapy settings --get BOT_NAME
    scrapybot
    $ scrapy settings --get DOWNLOAD_DELAY
    0

.. command:: runspider

runspider
---------

* Syntax: ``scrapy runspider <spider_file.py>``
* Requires project: *no*

프로젝트 생성할 필요 없이 파이썬 파일에 포함되어있는 스파이더를 실행한다.

사용 예시::

    $ scrapy runspider myspider.py
    [ ... spider starts crawling ... ]

.. command:: version

version
-------

* Syntax: ``scrapy version [-v]``
* Requires project: *no*

스크래피 버전을 출력한다.
``-v``\ 와 같이 쓰면 버그 리포트에 유용한 파이썬, Twisted, Platform 정보도 출력한다.

.. command:: bench

bench
-----

.. versionadded:: 0.17

* Syntax: ``scrapy bench``
* Requires project: *no*

간단한 벤치마크 테스트를 실행한다. :ref:`benchmarking`.

Custom project commands
=======================

:setting:`COMMANDS_MODULE` 설정을 사용해서 사용자의 커스텀 프로젝트 커맨드를 추가할 수 있다.
커맨드를 구현 방식에 대한 예시는 `scrapy/commands`_\ 에 있는 예제를 참고하라.

.. _scrapy/commands: https://github.com/scrapy/scrapy/tree/master/scrapy/commands
.. setting:: COMMANDS_MODULE

COMMANDS_MODULE
---------------

Default: ``''`` (empty string)

커스텀 스크래피 커맨드를 찾기위해 사용되는 모듈.
사용자의 스크래피 프로젝트에 커스텀 커맨드를 추가하기 위해 사용된다.

예시::

    COMMANDS_MODULE = 'mybot.commands'

.. _Deploying your project: https://scrapyd.readthedocs.io/en/latest/deploy.html

setup.py 엔트리 포인트를 통한 커맨드 등록
--------------------------------------------------------

.. note:: 이것은 실험적인 기능이므로 사용에 주의를 요한다.

라이브러리 ``setup.py`` 파일의 엔트리 포인트에 ``scrapy.commands`` 섹션을
추가해서 외부 라이브러리로부터 스크래피 커맨드를 추가할 수도 있다.

다음의 예시는 ``my_command`` 커맨드를 추가했다::

  from setuptools import setup, find_packages

  setup(name='scrapy-mymodule',
    entry_points={
      'scrapy.commands': [
        'my_command=my_scrapy_module.commands:MyCommand',
      ],
    },
   )
