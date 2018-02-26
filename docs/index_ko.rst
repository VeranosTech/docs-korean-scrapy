.. _topics-index:

========================================
스크래피 |version| 문서
========================================

이 문서에는 스크래피에 대해 알아야 할 모든 것을 포함한다.

도움 받기
=================

문제가 있으면 다음처럼 도움을 얻을 수 있는 방법들이 있다.

* :doc:`FAQ <faq>`\ 를 확인한다. 이곳에는 공통적인 질문에 대한 답변이 있다.
* 특정한 내용을 찾고 있다면 :ref:`genindex` 또는 :ref:`modindex`\ 를 참고할 수 있다..
* `스택오버플로우(Stack Overflow)에서 스크래피 태그 검색`_\ 을 이용하여 질문을 하거나 질문을 검색할 수 있다.
* `스크래피 서브래딧(sub-reddit)`_\ 을 이용하여 질문을 하거나 질문을 검색할 수 있다.
* `스크래피 사용자 메일링 리스트`_ 아카이브에서 질문을 검색할 수 있다.
* `#scrapy IRC 채널`_\ 에서 질문할 수 있다.
* 스크래피 버그는 `이슈 트래커(issue tracker)`_\ 에 이슈를 작성할 수 있다.

.. _스크래피 사용자 메일링 리스트: https://groups.google.com/forum/#!forum/scrapy-users
.. _스크래피 서브래딧(sub-reddit): https://www.reddit.com/r/scrapy/
.. _스택오버플로우(Stack Overflow)에서 스크래피 태그 검색: https://stackoverflow.com/tags/scrapy
.. _#scrapy IRC 채널: irc://irc.freenode.net/scrapy
.. _이슈 트래커(issue tracker): https://github.com/scrapy/scrapy/issues


처음 시작하기
======================

.. toctree::
   :caption: 처음 시작하기
   :hidden:

   intro/overview_ko
   intro/install_ko
   intro/tutorial_ko
   intro/examples_ko

:doc:`intro/overview_ko`
    스크래피가 무엇이고 어떤 도움을 주는지 이해하기.

:doc:`intro/install_ko`
    컴퓨터에 스크래피 설치하기.

:doc:`intro/tutorial_ko`
    첫 번째 스크래피 프로젝트 제작.

:doc:`intro/examples_ko`
    사전 제작된 스크래피 프로젝트를 이용하여 배우기.

.. _section-basics:

기본 개념
================

.. toctree::
   :caption: 기본 개념
   :hidden:

   topics/commands_ko
   topics/spiders_ko
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


:doc:`topics/commands_ko`
    스크래피 프로젝트를 관리하는 커맨드라인 도구.

:doc:`topics/spiders_ko`
    웹사이트를 크롤링(crawling)하기 위한 규칙 만들기.

:doc:`topics/selectors`
    XPath를 사용해서 웹 페이지로부터 데이터 추출.

:doc:`topics/shell`
    인터렉티브한 환경에서 추출 코드를 테스트.

:doc:`topics/items`
    스크랩하고 싶은 데이터 정의.

:doc:`topics/loaders`
    아이템을 추출된 데이터로 채우기.

:doc:`topics/item-pipeline`
    스크랩된 데이터를 후처리(post-process)하고 저장.

:doc:`topics/feed-exports`
    다른 포맷과 스토리지를 사용해 스크랩된 데이터를 출력.

:doc:`topics/request-response`
    HTTP 리퀘스트(Request)와 리스판스(Response)를 나타내기 위해 사용되는 클래스.

:doc:`topics/link-extractors`
    페이지로부터 팔로우할 링크를 추출하는 편리한 클래스.

:doc:`topics/settings`
    스크래피를 설정하는 법 배우고 모든 :ref:`이용 가능한 세팅 <topics-settings-ref>`.

:doc:`topics/exceptions`
    사용 가능한 모든 예외(exception)와 그 의미.


내장 서비스
======================

.. toctree::
   :caption: Built-in services
   :hidden:

   topics/logging_ko
   topics/stats_ko
   topics/email
   topics/telnetconsole
   topics/webservice

:doc:`topics/logging_ko`
    파이썬에 내장된 logging 패키지를 스크래피에서 사용하는 법.

:doc:`topics/stats_ko`
    스크랩하고 있는 크롤러에 대한 통계 수집.

:doc:`topics/email`
    특정 이벤트가 발생했을 때 이메일 알림 송신.

:doc:`topics/telnetconsole`
    내장 파이썬 콘솔을 이용하여 작동중인 크롤러 조사.

:doc:`topics/webservice`
    웹 서비스를 사용하여 크롤러를 감시 및 제어.


문제 해결
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
    스크래피 스파이더에서 자주 발생하는 문제 디버깅하는 법.

:doc:`topics/contracts`
    스파이더 테스팅을 위해 컨트랙트(contracts)를 사용하는 법.

:doc:`topics/practices`
    일반적인 스크래피 사용 관례.

:doc:`topics/broad-crawls`
    복수 도메인을 병렬로 크롤링할 수 있도록 스크래피 튜닝.

:doc:`topics/firefox`
    Firefox와 각종 유용한 애드온을 사용하여 스크랩하는 법.

:doc:`topics/firebug`
    Firebug를 사용해서 효율적으로 스크랩하는 법.

:doc:`topics/leaks`
    크롤러의 메모리 누수를 찾고 제거하는 법.

:doc:`topics/media-pipeline`
    스크랩된 아이템과 관련된 이미지나 파일 다운로드.

:doc:`topics/deploy`
    스크래피 스파이더를 배포하고 원격 서버에서 구동.

:doc:`topics/autothrottle`
    부하(load)에 따라 동적으로 크롤링 속도 조정.

:doc:`topics/benchmarking`
    하드웨어에 따른 스크래피 성능 확인.

:doc:`topics/jobs`
    다수의 스파이더의 크롤링을 정지 및 재개하는 법.

.. _extending-scrapy_ko:

스크래피 확장
==============================

.. toctree::
   :caption: 스크래피 확장
   :hidden:

   topics/architecture
   topics/downloader-middleware
   topics/spider-middleware
   topics/extensions
   topics/api_ko
   topics/signals
   topics/exporters


:doc:`topics/architecture`
    스크래피 아키텍쳐의 이해.

:doc:`topics/downloader-middleware`
    페이지 요청 및 다운로드 과정의 커스터마이징.

:doc:`topics/spider-middleware`
    스파이더의 인풋과 아웃풋 커스터마이징.

:doc:`topics/extensions`
    커스텀 기능으로 스크래피 확장.

:doc:`topics/api_ko`
    확장 기능과 미들웨어를 사용하여 스크래피의 기능을 넓히기.

:doc:`topics/signals`
    스크래피의 시그널과 사용법.

:doc:`topics/exporters`
    스크랩된 아이템을 파일로 출력.


기타
============

.. toctree::
   :caption: 기타
   :hidden:

   news
   contributing
   versioning

:doc:`news`
    최신 스크래피 버전에서 변경된 점.

:doc:`contributing`
    스크래피 프로젝트에 기여하는 법.

:doc:`versioning`
    스크래피 버전관리와 API 안정성.
