.. _topics-logging:

=======
로깅
=======

.. note::
    파이썬 표준 로깅을 사용하면서
    :mod:`scrapy.log` 모듈을 더이상 사용하지 않는다.

스크래피는 이벤트 로깅을 위해 파이썬의 내장 로깅 시스템
<https://veranostech.github.io/docs-korean-cpython/Doc/build/html/library/logging_ko.html>`_\ 을 사용한다.

여기에서도 몇가지 간단한 예를 제공하긴 하지만 더 복잡한 경우는 파이썬 로그 공식 문서를
참조하기 바란다.

로그 기능은 쉽게 사용할 수 있고 :ref:`topics-logging-settings` 스크래피 설정으로
어느 정도는 설정할 수도 있다.

스크래피는 :func:`scrapy.utils.log.configure_logging` 함수를 호출해서
디폴트 설정을 하고 명령을 실행할 때는 :ref:`topics-logging-settings` 설정을 사용한다.
만약 :ref:`run-from-script`\ 에서처럼 스크립트에서 스크래피를 사용할 때는 이 함수를 수동으로
호출해서 사용하기를 권장한다.

.. _topics-logging-levels:

로그 레벨
====================

파이썬의 내장 로그 시스템에서는 로그 메세지의 중요도에 대해 5가지의 다른 레벨을 정의한다.
표준적인 레벨 시스템은 다음과 같다. 가장 중요한 것부터 나열하였다.:

1. ``logging.CRITICAL`` - for 치명적 오류 (가장 높은 중요도)
2. ``logging.ERROR`` - for 일반적 오류
3. ``logging.WARNING`` - for 워닝 메세지
4. ``logging.INFO`` - for 일반적 정보
5. ``logging.DEBUG`` - for 디버깅 메세지 (가장 낮은 중요도)

메세지 로깅하는 법
======================================

다음은 ``logging.WARNING`` 레벨에서 메세지를 로깅하는 간단한 예이다.::

    import logging
    logging.warning("This is a warning")

표준 5 레벨에 대해서는 각 레벨의 로그 메세지를 남기는 단축 명령어가 있다.
일반적인 ``logging.log`` 메서드는 레벨을 인수로 준다.
필요한 경우에는 위 예제를 다음처럼 쓸 수 있다.::

    import logging
    logging.log(logging.WARNING, "This is a warning")

이외에도 여러가지 다른 종류의 로거를 만들 수도 있다.
(모든 모듈에 대해 여러가지 다른 로거를 생성하는 것이 일반적이다.)
이 로거들은 각각 독립적으로 설정할 수 있고 계층구조를 가지게 할 수도 있다.

앞의 예에서는 모든 메세지를 전달하는 가장 상위의 루트 로거를 사용한다.
``logging`` 헬퍼는 루트 로거를 얻기 위한 단축 명령이다.
따라서 위 코드는 다음 코드와 같다::

    import logging
    logger = logging.getLogger()
    logger.warning("This is a warning")

``logging.getLogger`` 함수를 쓰면 이름을 지정해서
여러가지 다른 로거를 사용할 수 있다.::

    import logging
    logger = logging.getLogger('mycustomlogger')
    logger.warning("This is a warning")

마지막으로 현재 모듈의 패스를 지정하는 ``__name__`` 변수를 사용하면
모든 모듈에 대해 서로 다른 모듈을 쓸 수 있다.::

    import logging
    logger = logging.getLogger(__name__)
    logger.warning("This is a warning")

.. seealso::

    Module logging, `HowTo <https://veranostech.github.io/docs-korean-cpython/Doc/build/html/howto/logging_ko.html>`_
        기초적인 로깅 튜토리얼

    Module logging, `Loggers <https://veranostech.github.io/docs-korean-cpython/Doc/build/html/library/logging_ko.html#logger>`_
        로거에 대한 공식 문서

.. _topics-logging-from-spiders:

스파이더에서 로그 남기기
========================================

스크래피는 각 스파이더에 대해 :data:`~scrapy.spiders.Spider.logger` 인스턴스를 제공한다.
다음처럼 사용할 수 있다.::

    import scrapy

    class MySpider(scrapy.Spider):

        name = 'myspider'
        start_urls = ['https://scrapinghub.com']

        def parse(self, response):
            self.logger.info('Parse function called on %s', response.url)

이 로거는 스파이더 이름으로 생성된다. 하지만 원하는 종류의 파이썬 로거를
사용하도록 커스터마이징할 수 있다.::

    import logging
    import scrapy

    logger = logging.getLogger('mycustomlogger')

    class MySpider(scrapy.Spider):

        name = 'myspider'
        start_urls = ['https://scrapinghub.com']

        def parse(self, response):
            logger.info('Parse function called on %s', response.url)

.. _topics-logging-configuration:

로그 설정
=====================

로거 자체는 메세지가 처리되고 표시되는 것을 관리하지 못한다.
이 기능은 여러가지 다른 "핸들러(handlers)"가 처리한다.
핸들러는 로거 인스턴스에 붙어서 메세지를 표준 출력이나 파일, 이메일 등의
특정 목적지로 보낸다.

디폴트로 스크래피는 다음 설정방법을 사용하여 루트 로거 핸들러를 설정할 수 있다.

.. _topics-logging-settings:

로그 세팅
----------------

로그 세팅에는 다음 설정값을 사용한다.:

* :setting:`LOG_FILE`
* :setting:`LOG_ENABLED`
* :setting:`LOG_ENCODING`
* :setting:`LOG_LEVEL`
* :setting:`LOG_FORMAT`
* :setting:`LOG_DATEFORMAT`
* :setting:`LOG_STDOUT`
* :setting:`LOG_SHORT_NAMES`

첫 두 설정값은 로그 메세지의 목적지를 정의한다.
:setting:`LOG_FILE` 값이 주어지면 메세지는 루트 로거를 통해
:setting:`LOG_ENCODING` 인코딩되는  :setting:`LOG_FILE` 파일로 저장된다.
만약 이 값이 주어지지 않고 :setting:`LOG_ENABLED` 값이 ``True``\ 이면
로그 메세지는 표준 출력으로만 주어진다.
마지막으로 만약 :setting:`LOG_ENABLED` 값이 ``False``\ 이면
로그 출력이 보이지 않는다.

:setting:`LOG_LEVEL`\ 은 표시할 최소 중요도의 로그 레벨을 설정한다.
이보다 중요하지 않는 로그는 필터링되어 없어진다. 가능한 로그 레벨은
:ref:`topics-logging-levels`\ 에 있다.

:setting:`LOG_FORMAT`\ 과 :setting:`LOG_DATEFORMAT`\ 는
메세지 포멧에 필요한 형식 문자열을 정의한다.
이 문자열에 대해서는 `로그 레코드 속성
<https://veranostech.github.io/docs-korean-cpython/Doc/build/html/library/logging_ko.html#logrecord-attributes>`_
문서와
`datetime의 strftime and strptime 지시자
<https://veranostech.github.io/docs-korean-cpython/Doc/build/html/library/datetime.html#strftime-and-strptime-behavior>`_
문서를 참조한다.

만약 :setting:`LOG_SHORT_NAMES` 설정이 주어지면 로그에서 스크래피 구성요소가 출력되지 않는다.
보통은 이 값이 설정되지 않고 로그를 출력하는 스크래피 구성요소가 출력에 같이 표시된다.

명령줄 옵션
--------------------

로깅에 대한 스크래피 설정을 오버라이드 할 수 있는 다음 옵션을
모든 명령에서 사용할 수 있다.

* ``--logfile FILE``
    :setting:`LOG_FILE` 값을 오버라이드
* ``--loglevel/-L LEVEL``
    :setting:`LOG_LEVEL`  값을 오버라이드
* ``--nolog``
    :setting:`LOG_ENABLED` 값을 ``False`` 설정

.. seealso::

    로그 핸들러에 대해서는 `logging.handlers <https://veranostech.github.io/docs-korean-cpython/Doc/build/html/library/logging.handlers_ko.html>`_
    참조

고급 커스터마이징
--------------------------------------------

스크래피는 표준 라이브러리의 로깅 모듈을 사용하므로
표준 로깅의 모든 기능을 사용하여 로거를 커스터마이징할 수 있다.

예를 들어 HTTP 404 또는 500를 반환하는 웹사이를 스크래핑하면
다음과 같은 모든 메세지를 감추고  싶을 것이다.::

    2016-12-16 22:00:06 [scrapy.spidermiddlewares.httperror] INFO: Ignoring
    response <500 http://quotes.toscrape.com/page/1-34/>: HTTP status code
    is not handled or not allowed

우선 ``[scrapy.spidermiddlewares.httperror]``\ 라는 로거 이름에 주의하자.
만약 :setting:`LOG_SHORT_NAMES`\ 가 ``True``\ 면 ``[scrapy]``\ 라고만 출력된다.
이 설정을 False로 하고 다시 크롤링을 하라.

다음으로는 메세지가 INFO 레벨이라는 점에 유의하여
``scrapy.spidermiddlewares.httperror`` 로그 레벨을
INFO보다 높게 설정한다. INFO보다 높은 로그 레벨은 WARNING이다.
이 설정은 스파이더의 ``__init__`` 메서드에서 할 수 있다.::

    import logging
    import scrapy


    class MySpider(scrapy.Spider):
        # ...
        def __init__(self, *args, **kwargs):
            logger = logging.getLogger('scrapy.spidermiddlewares.httperror')
            logger.setLevel(logging.WARNING)
            super().__init__(*args, **kwargs)

다시 스파이더를 돌리면
``scrapy.spidermiddlewares.httperror`` 로거는 보이지 않을 것이다.

scrapy.utils.log 모듈
==============================================

.. module:: scrapy.utils.log
   :synopsis: Logging utils

.. autofunction:: configure_logging

    스크래피 명령을 사용하면 ``configure_logging`` 함수가 자동을 실행된다.
    만약 사용자 스크립트를 사용하면 명시적으로 이 함수를 호출한다.
    이 경우에도 반드시 호출할 필요는 없지만 권장한다.

    핸들러를 스스로 설정하고 싶을 때도 `install_root_handler=False` 인수를 넣고
    이 함수를 호출한다.
    이 경우에는 디폴트로 로그 출력이 나오지 않는다.

    만약 로그 출력을 수동으로 설정하고 싶으면
    `logging.basicConfig()`_\ 를 사용하여 루트 핸들러를 설정한다.
    다음은 ``INFO`` 또는 그 이상의 로그 메세지를 파일로 출력하는 예이다.::

        import logging
        from scrapy.utils.log import configure_logging

        configure_logging(install_root_handler=False)
        logging.basicConfig(
            filename='log.txt',
            format='%(levelname)s: %(message)s',
            level=logging.INFO
        )

    이런 식으로 스크래피를 사용하는 것에 대해서는 :ref:`run-from-script`\ 를 참조한다.

.. _logging.basicConfig(): https://veranostech.github.io/docs-korean-cpython/Doc/build/html/library/logging_ko.html#logging.basicConfig


