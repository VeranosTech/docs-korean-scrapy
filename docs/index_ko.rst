.. _topics-index_ko:

==============================
Scrapy |version| 문서
==============================

이 문서에는 Scrapy에 대해 알아야 할 모든 것이 있다.

도움 받기
=================

문제가 있는가? 우리가 도움을 줄 수 있다!

* :doc:`FAQ <faq>`\ 를 확인하라 -- 이곳에는 공통적인 질문에 대한 답변이 있다.
* 특정한 내용을 찾고 있다면? :ref:`genindex` 또는 :ref:`modindex`\ 를 참고하라.
* 질문을 하거나 질문을 찾으려면 `StackOverflow using the scrapy tag`_\ 를 이용하라.
* 질문을 하거나 질문을 찾으려면 `Scrapy subreddit`_\ 를 이용하라.
* `scrapy-users mailing list`_ 아카이브에서 질문을 검색하라.
* `#scrapy IRC channel`_\ 에서 질문하라,
* Scrapy 버그는 `issue tracker`_\ 에 리포트하라.

.. _scrapy-users mailing list: https://groups.google.com/forum/#!forum/scrapy-users
.. _Scrapy subreddit: https://www.reddit.com/r/scrapy/
.. _StackOverflow using the scrapy tag: https://stackoverflow.com/tags/scrapy
.. _#scrapy IRC channel: irc://irc.freenode.net/scrapy
.. _issue tracker: https://github.com/scrapy/scrapy/issues


첫 번째 단계
======================

.. toctree::
   :caption: 첫 번째 단계
   :hidden:

   intro/overview
   intro/install
   intro/tutorial
   intro/examples

:doc:`intro/overview`
    Scrapy가 무엇이고 어떻게 도와주는지 이해하기.

:doc:`intro/install`
    컴퓨터에 Scrapy 설치하기.

:doc:`intro/tutorial`
    첫 번째 Scrapy 프로젝트 만들기.

:doc:`intro/examples`
    사전 제작된 Scrapy 프로젝트로 놀면서 더 배워보기.

.. _section-basics:

기본 개념
================

.. toctree::
   :caption: 기본 개념
   :hidden:

   topics/commands
   topics/spiders
   topics/selectors
   topics/items
   topics/loaders
   topics/shell
   topics/item-pipeline
   topics/feed-exports
   topics/request-response
   topics/link-extractors
   topics/settings
   topics/exceptions


:doc:`topics/commands`
    Scrapy 프로젝트를 관리하기 위해 사용되는 커맨드라인 툴에 대배 배우기.

:doc:`topics/spiders`
    당신의 웹사이트를 크롤링하기 위한 룰 만들기.

:doc:`topics/selectors`
    XPath를 사용해서 웹 페이지로부터 데이터 추출하기.

:doc:`topics/shell`
    인터렉티브한 환경에서 추출 코드를 테스트해보기.

:doc:`topics/items`
    스크랩 하고 싶은 데이터 정의하기.

:doc:`topics/loaders`
    아이템을 추출된 데이터로 채우기.

:doc:`topics/item-pipeline`
    스크랩된 데이터를 후처리하고 저장하기.

:doc:`topics/feed-exports`
    다른 포맷과 스토리지를 사용해 스크랩된 데이터를 출력하기.

:doc:`topics/request-response`
    HTTP 리퀘스트와 리스폰스를 나타내기 위해 사용되는 클래스 이해하기.

:doc:`topics/link-extractors`
    페이지로부터 팔로우할 링크를 추출하는 편리한 클래스.

:doc:`topics/settings`
    Scrapy 설정하는 법 배우고 모든 :ref:`이용 가능하나 세팅 <topics-settings-ref>` 확인하기.

:doc:`topics/exceptions`
    사용 가능한 모든 예외와 예외의 이미 확인하기.


빌트인 서비스
======================

.. toctree::
   :caption: Built-in services
   :hidden:

   topics/logging
   topics/stats
   topics/email
   topics/telnetconsole
   topics/webservice

:doc:`topics/logging`
    파이썬 내장 logging 패키지를 Scrapy에서 사용하는 법 배우기.

:doc:`topics/stats`
    스크랩하고 있는 크롤러에 대한 통계 수집하기.

:doc:`topics/email`
    특정 이벤트가 발생했을 때 이메일 알림 보내기.

:doc:`topics/telnetconsole`
    내장 파이썬 콘솔을 이용해서 작동중인 크롤러 조사하기.

:doc:`topics/webservice`
    웹 서비스를 사용하는 크롤러 모니터링하고 제어하기.


특정 문제 해결하기
==============================

.. toctree::
   :caption: 특정 문제 해결하기
   :hidden:

   faq
   topics/debug
   topics/contracts
   topics/practices
   topics/broad-crawls
   topics/firefox
   topics/firebug
   topics/leaks
   topics/media-pipeline
   topics/deploy
   topics/autothrottle
   topics/benchmarking
   topics/jobs

:doc:`faq`
    자주 묻는 질문에 대한 답변.

:doc:`topics/debug`
    Scrapy 스파이더의 흔한 문제 디버깅하는 법 배우기.

:doc:`topics/contracts`
    스파이더 테스팅을 위해 컨트랙트 사용하는 법 배우기.

:doc:`topics/practices`
    Scrapy 관례에 대해 익숙해지기.

:doc:`topics/broad-crawls`
    병력적으로 많은 도메인을 크롤링하게 Scrapy 튜닝하기.

:doc:`topics/firefox`
    Firefox와 유용한 애드온을 사용해서 스크랩하는 법 배우기.

:doc:`topics/firebug`
    Firebug를 사용해서 효율적으로 스크랩하는 법 배우기.

:doc:`topics/leaks`
    크롤러의 메모리 누수를 찾고 제거하는 법 배우기.

:doc:`topics/media-pipeline`
    스크랩된 아이템과 관련된 이미지나 파일 다운로드 하기.

:doc:`topics/deploy`
    Scrapy 스파이더 배포하고 리모트 서버에서 구동하기.

:doc:`topics/autothrottle`
    로드에 따라서 다이나믹하게 크롤링 레이트 조정하기.

:doc:`topics/benchmarking`
    당신의 하드웨어 위에서 Scrapy가 어떻게 작동하는지 확인하기.

:doc:`topics/jobs`
    거대한 스파이더의 크롤링을 정지하고 재개하는 법 배우기.

.. _extending-scrapy_ko:

Scrapy 확장하기
==============================

.. toctree::
   :caption: Scrapy 확장하기
   :hidden:

   topics/architecture
   topics/downloader-middleware
   topics/spider-middleware
   topics/extensions
   topics/api
   topics/signals
   topics/exporters


:doc:`topics/architecture`
    Scrapy 아키텍쳐 이해하기.

:doc:`topics/downloader-middleware`
    요청되고 다운로드 되는 페이지 커스터마이즈 하기.

:doc:`topics/spider-middleware`
    스파이더의 인풋과 아웃풋 커스터마이징 하기.

:doc:`topics/extensions`
    커스텀 기능으로 Scrapy 확장하기.

:doc:`topics/api`
    Scrapy 기능 확장을 위한 확장과 미들웨어에서 api 사용하기.

:doc:`topics/signals`
    사용 가능한 시그널과 어떻게 사용하는지 확인하기.

:doc:`topics/exporters`
    빠르게 스크랩된 아이템을 파일로 내보내기(XML, CSV, etc).


기타
============

.. toctree::
   :caption: 기타
   :hidden:

   news
   contributing
   versioning

:doc:`news`
    최신 Scrapy 버전에서 변경된 점 확인하기.

:doc:`contributing`
    Scrapy 프로젝트에 기여하는 법 배우기.

:doc:`versioning`
    Scrapy 버전관리와 API 안정성 이해하기.
