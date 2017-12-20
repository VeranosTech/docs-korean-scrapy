.. _topics-item-pipeline:

=================================================
아이템 파이프라인(Item pipeline)
=================================================

스파이더(spider)로 아이템이 스크랩되면, 아이템은 아이템 파이프라인으로 보내지며
파이프라인에서는 연속적으로 실행되는 여러 구성요소를 통해 아이템을 처리한다.

각 아이템 파이프라인의 구성요소는 (때로는 단순히 "아이템 파이프라인"으로 불린다)
간단한 메서드를 구현한 파이썬 클래스다. 파이프라인은 아이템을 받아서 동작을 수행하며,
파이프라인을 지나서도 유지되야 하는지 또는 더이상 처리되지 않고 버려져야 하는지도 결정한다.

일반적인 아이템 파이프라인의 사용 목적은 다음과 같다:

* HTML 데이터 정리
* 스크랩 데이터 유효성 검사 (아이템이 특정 필드를 포함하고 있는지)
* 중복 확인 (중복 아이템 제거)
* 스크랩 아이템 데이터베이스에 저장


아이템 파이프라인 제작하기
==========================================

각 아이템 파이프라인 구성요소는 반드시 아래의 메서드를 구현한 파이썬 클래스여야 한다:

.. method:: process_item(self, item, spider)

   이 메서드는 모든 아이템 파이프라인 구성요소에 대해 호출된다. :meth:`process_item`\ 는
   반드시 데이터가 있는 딕셔너리나, :class:`~scrapy.item.Item` 객체, 또는 `Twisted Deferred`_\ 를
   반환하거나 :exc:`~scrapy.exceptions.DropItem` 예외를 발생시켜야 한다.
   드랍된 아이템은 이후의 파이프라인 구성요소로 처리되지 않는다.

   :param item: 스크랩 된 아이템
   :type item: :class:`~scrapy.item.Item` 객체 또는 딕셔너리

   :param spider: 아이템을 스크랩하는 스파이더
   :type spider: :class:`~scrapy.spiders.Spider` 객체

또한 다음 메서드도 구현할 수 있다:

.. method:: open_spider(self, spider)

   이 메서드는 스파이더가 열렸을 때 호출된다.

   :param spider: 열린 스파이더
   :type spider: :class:`~scrapy.spiders.Spider` 객체

.. method:: close_spider(self, spider)

   이 메서드 닫혔을 때 호출된다.

   :param spider: 닫힌 스파이더
   :type spider: :class:`~scrapy.spiders.Spider` 객체

.. method:: from_crawler(cls, crawler)

   존재할 경우, 이 클래스 메서드는 :class:`~scrapy.crawler.Crawler`\ 로 파이프라인 인스턴스를
   생성하기위해 호출된다. 반드시 새로운 파이프라인 인스턴스를 반환해야 한다.
   크롤러(Crawler) 객체는 설정정과 시그널(signal) 같은 모든 스크래피 핵심 구성요소에 접근할 수
   있게 해준다; 이것이 파이프라인이 구성요소에 접근하고 스크래피에 기능을 연결하는 방법이다.

   :param crawler: 이 파이프라인을 사용하는 크롤러
   :type crawler: :class:`~scrapy.crawler.Crawler` 객체


.. _Twisted Deferred: https://twistedmatrix.com/documents/current/core/howto/defer.html

아이템 파이프라인 예시
====================================

가격 유효성 검사 및 가격 미포함 아이템 드랍
--------------------------------------------------

VAT(``price_excludes_vat`` 속성)를 포함하지 않은 아이템의 ``price`` 속성을 수정하고 가격을 포함하지 않은
아이템을 버리는 가상의 파이프라인을 살펴보자::

    from scrapy.exceptions import DropItem

    class PricePipeline(object):

        vat_factor = 1.15

        def process_item(self, item, spider):
            if item['price']:
                if item['price_excludes_vat']:
                    item['price'] = item['price'] * self.vat_factor
                return item
            else:
                raise DropItem("Missing price in %s" % item)


아이템을 JSON 파일로 쓰기
--------------------------------------

아래의 파이프라인은 (모든 스파이더에서) 스크랩된 모든 아이템을 하나의 ``item.jl`` 파일에
저장한다. 하나의 아이템은 JSON 포맷으로 직렬화된 한 줄로 나타난다::

   import json

   class JsonWriterPipeline(object):

       def open_spider(self, spider):
           self.file = open('items.jl', 'w')

       def close_spider(self, spider):
           self.file.close()

       def process_item(self, item, spider):
           line = json.dumps(dict(item)) + "\n"
           self.file.write(line)
           return item

.. note:: JsonWriterPipeline의 목적은 아이템 파이프라인을 어떻게 작성하는지
   소개하기 위해서다. 만약 정말로 스크랩된 모든 데이터를 JOSN 파일로 저장하려면
   :ref:`Feed exports <topics-feed-exports>`\ 를 사용해야 한다.

MongoDB에 아이템 작성하기
-----------------------------------

아래의 예에서 우리는 `pymongo_`\ 를 사용해서 `MongoDB_`\ 에 아이템을 작성할 것이다.
MongoDB 주소와 데이터베이스 이름은 스크래피 설정에서 지정되었다;
MongoDB 집합의 이름은 아이템 클래스에서 따라 지었다.

이 예의 주요 포인트는 :meth:`from_crawler` 메서드를 사용하는 방법과
리소스를 적절하게 정리하는 방법을 보여주는 것이다::

    import pymongo

    class MongoPipeline(object):

        collection_name = 'scrapy_items'

        def __init__(self, mongo_uri, mongo_db):
            self.mongo_uri = mongo_uri
            self.mongo_db = mongo_db

        @classmethod
        def from_crawler(cls, crawler):
            return cls(
                mongo_uri=crawler.settings.get('MONGO_URI'),
                mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
            )

        def open_spider(self, spider):
            self.client = pymongo.MongoClient(self.mongo_uri)
            self.db = self.client[self.mongo_db]

        def close_spider(self, spider):
            self.client.close()

        def process_item(self, item, spider):
            self.db[self.collection_name].insert_one(dict(item))
            return item

.. _MongoDB: https://www.mongodb.org/
.. _pymongo: https://api.mongodb.org/python/current/


아이템의 스크린샷 찍기
------------------------------------

이 예제는 :meth:`process_item` 메서드에서 `Deferred`_\ 를 반환하는 방법을 설명한다.
이 메서드는 아이템 url의 스크린샷을 렌더링하기 위해 `Splash`_\ 를 사용한다.파이프라인은
로컬에서 `Splash`_\ 의 인스턴스를 실행하도록 요청한다.
리퀘스트가 다운로드된 후에 지연 콜백이 일어나면, 아이템을 파일에 저장하고 파일명을 아이템에
추가한다::

    import scrapy
    import hashlib
    from urllib.parse import quote


    class ScreenshotPipeline(object):
        """Pipeline that uses Splash to render screenshot of
        every Scrapy item."""

        SPLASH_URL = "http://localhost:8050/render.png?url={}"

        def process_item(self, item, spider):
            encoded_item_url = quote(item["url"])
            screenshot_url = self.SPLASH_URL.format(encoded_item_url)
            request = scrapy.Request(screenshot_url)
            dfd = spider.crawler.engine.download(request, spider)
            dfd.addBoth(self.return_item, item)
            return dfd

        def return_item(self, response, item):
            if response.status != 200:
                # Error happened, return item.
                return item

            # Save screenshot to file, filename will be hash of url.
            url = item["url"]
            url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
            filename = "{}.png".format(url_hash)
            with open(filename, "wb") as f:
                f.write(response.body)

            # Store filename in item.
            item["screenshot_filename"] = filename
            return item

.. _Splash: https://splash.readthedocs.io/en/stable/
.. _Deferred: https://twistedmatrix.com/documents/current/core/howto/defer.html

중복 필더
-----------------

중복된 아이템을 찾고, 이미 처리된 아이템을 드랍하는 필터.
아이템이 교유한 id를 가지고 있지만 스파이더가 동일한 id를 가진 다수의 아이템을 반환한다고
가정하자::


    from scrapy.exceptions import DropItem

    class DuplicatesPipeline(object):

        def __init__(self):
            self.ids_seen = set()

        def process_item(self, item, spider):
            if item['id'] in self.ids_seen:
                raise DropItem("Duplicate item found: %s" % item)
            else:
                self.ids_seen.add(item['id'])
                return item


아이템파이프라인 구성요소
=====================================

아이템 파이프라인 구성요소를 활성화하려면 반드시 그 클래스를 아래 예시처럼 :setting:`ITEM_PIPELINES` 세팅에 추가해야 한다::

   ITEM_PIPELINES = {
       'myproject.pipelines.PricePipeline': 300,
       'myproject.pipelines.JsonWriterPipeline': 800,
   }

이 세팅에서 클래스에 할당한 정수값은 실행되는 순서를 결정한다:
아이템은 낮은 값에서부터 높은 값을 가진 클래스를 빠져나간다.
0-1000 범위로 정의하는 것이 일반적이다.
