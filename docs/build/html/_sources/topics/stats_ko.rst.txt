.. _topics-stats:

================
통계 수집
================

스크래피는 키/밸류 형태로 통계를 수집하는 기능을 제공한다.
여기에서 밸류는 보통 카운터(counter) 값이다.
이 기능을 통계 수집기(Stats Collector)라고 한다.
:ref:`topics-api-crawler`\ 의 :attr:`~scrapy.crawler.Crawler.stats` 속성을
이용해서 접근할 수 있다.
:ref:`topics-stats-usecases` 섹션에서 자세히 설명한다.

통계 수집 기능이 활성화되어있는지에 상관없이
통계 수집기는 언제나 모듈에서 임포트하고 API를 사용할 수 있다.
비활성화되어 있어도 API를 사용할 수는 있으나 실제로는 아무것도 수집하지 않는다.
이는 통계 수집기 사용을 단순화하기 위해서이다.
스파이더나 스크래피 확장 모듈, 그밖의 모든 코드에서 통계를 수집하려면
한 줄 이상의 코드를 써야 한다.

통계 수집기의 또다른 특징은 (활성화되어 있을 때) 아주 효율적이고
비활성화되어 있을 때도 극도로 효율적이라는 점이다.

통계 수집기는 스파이더가 열릴 때 한 스파이더당 하나의 통계 테이블을 자동으로 열고
스파이더가 닫히면 테이블도 닫는다.

.. _topics-stats-usecases:

통계 수집기의 일반적 사용법
======================================================

:attr:`~scrapy.crawler.Crawler.stats` 속성을 통해 통계 수집기에 접근한다.
다음은 통계 접근용 확장 모듈의 예이다.::

    class ExtensionThatAccessStats(object):

        def __init__(self, stats):
            self.stats = stats

        @classmethod
        def from_crawler(cls, crawler):
            return cls(crawler.stats)

통계 값 설정::

    stats.set_value('hostname', socket.gethostname())

통계 값 증가::

    stats.inc_value('custom_count')

값이 이전 값보다 클 때만 갱신::

    stats.max_value('max_items_scraped', value)

값이 이전 값보다 작을 때만 갱신::

    stats.min_value('min_free_memory_percent', value)

통계 값 얻기::

    >>> stats.get_value('custom_count')
    1

모든 통계 값 얻기::

    >>> stats.get_stats()
    {'custom_count': 1, 'start_time': datetime.datetime(2009, 7, 14, 21, 47, 28, 977139)}

사용가능한 통계 수집기
====================================================

기본적인 :class:`StatsCollector` 이외에도 기능을 확장한 다른 통계 수집기도 있다.
:setting:`STATS_CLASS` 설정값을 사용하여 통계 수집기를 선택할 수 있다.
디폴트는 :class:`MemoryStatsCollector`\ 이다.


.. module:: scrapy.statscollectors
   :synopsis: Stats Collectors

MemoryStatsCollector
--------------------

.. class:: MemoryStatsCollector

    마지막으로 스크래핑을 실행했을 때의 통계를 메모리에 유지하고 스파이터가 닫히면 닫히는 간단한 통계 수집기.
    :attr:`spider_stats` 속성으로 접근할 수 있으며 키 값은 스파이더 도메인 이름이다.

    스파이더에서 사용되는 기본 통계 수집기이다.

    .. attribute:: spider_stats

       스파이더 이름을 키로 가지는 딕셔너리의 딕셔너리.
       각 스파이더의 마지막 스크래핑 실행시의 통계를 가지고 있다.

DummyStatsCollector
-------------------

.. class:: DummyStatsCollector

    아무런 작업을 하지 않는 통계 수집기.
    :setting:`STATS_CLASS` 설정으로 이 수집기를 선택하면 실제로는 통계 수집을 비활성화하는 것과 같다.
    통계 수집으로 인한 성능 저하는 페이지 파싱 등의 다른 스크래피 작업에 무시할 정도이다.

