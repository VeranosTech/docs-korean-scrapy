.. _topics-items:

=========
아이템
=========

.. module:: scrapy.item
   :synopsis: Item and Field classes

스크랩핑의 주 목적은 구조화되지 않은 자료, 주로 웹페이지에서 구조화된 자료를 추출하는
것이다. 스크래피 스파이더(spider)는 추출된 데이터를 파이썬(Python) 딕셔너리로
반환할 수 있다. 이는 편리하고 친숙하지만, 파이썬 딕셔너리는 구조가 부족하다:
특히 많은 스파이더를 사용하는 거대한 프로젝트에서는 필드 이름에 오타를 내거나 일관성
없는 데이터를 반환하기 쉽다.

공통적인 출력 데이터 포맷을 정의하기 위해서 스크래피는 :class:`Item` 클래스를 제공한다.
:class:`Item` 객체는 스크랩된 데이터를 모으기 위한 간단한 컨테이너다.
이 객체는 `딕셔너리 형`_ API에 사용할 수 있는 필드를 편리하게 선언할 수 있는 신택스(syntax)를 제공한다.

다양한 스크래피 구성요소는 아이템이 제공하는 추가적인 정보를 사용한다:
익스포터(exporter)sms 내보낼 컬럼(column)을 알아내기 위해 선언된 필드를 확인하며,
직렬화(serialization)는 아이템 필드의 메타데이터를 사용해 커스터마이즈할 수 있고,
:mod:`trackref`\ 는 메모리 누수를 감지를 돕기 위해 아이템 인스턴스를 추척한다
(:ref:`topics-leaks-trackrefs`\ 를 참고하라).

.. _딕셔너리 형: https://docs.python.org/2/library/stdtypes.html#dict

.. _topics-items-declaring:

아이템 선언
===============

아이템은 간단한 클래스 정의 신택스와 :class:`Field` 객체를 사용해 선언된다.
다음은 선언의 예시이다::

    import scrapy

    class Product(scrapy.Item):
        name = scrapy.Field()
        price = scrapy.Field()
        stock = scrapy.Field()
        last_updated = scrapy.Field(serializer=str)

.. note:: `장고(Django)`_\ 와 익숙한 사람은 스크래피 아이템이
   다른 필드 타입이 없기 때문에 훨씬 단순하다는 사실을 제외하면
   `장고 모델`_\ 과 유사하게 선언되는 것을 알아차렸을 것이다.

.. _장고(Django): https://www.djangoproject.com/
.. _장고 모델: https://docs.djangoproject.com/en/dev/topics/db/models/

.. _topics-items-fields:

아이템 필드
==================

:class:`Field` 객체는 각 필드의 메타데이터를 지정하기 위해 사용된다.
위의 예시에서 묘사된 ``last_updated`` 필드를 위한 직렬화 함수가 그 예이다.

사용자는 각 필드에 대한 모든 종류의 메타데이터를 지정할 수 있다.\
:class:`Field` 객체가 받아들이는 값에 대해 제한은 없다.
같은 이유 때문에, 사용 가능한 메타데이터 키의 참조 리스트는 없다.
:class:`Field` 객체에서 정의된 각 키는 다른 구성요소에서 사용될 수 있고,
구성요소만이 그것에 대해 알고 있다.
또한 필요에 맞게 프로젝트에 다른 :class:`Field`\ 를 정의하고 사용할 수 있다.
:class:`Field`\ 객체의 주 목표는 한 장소에 모든 필드 메타데이터를 정의하는
방법을 제공하는 것이다. 일반적으로 동작이 각 필드에 의존하는 구성요소는 그 동작을 구성하는
특정 필드 키를 사용한다. 어떤 메타데이터 키가 각 구성요소에 사용되는지를 보려면
해당 문서를 반드시 참고해야 한다.

아이템을 선언하기 위해 사용된 :class:`Field` 객체는 클래스 속성으로 할당된 채
남아있지 않는다는 사실을 명심하라. 대신, :attr:`Item.fields` 속성을 통해서
접근할 수 있다.

아이템으로 작업하기
===============================

이 섹션에는 :ref:`위에서 선언된 <topics-items-declaring>` ``Product`` 아이템을 사용해서 수행하는
일반적인 작업의 예시가 있다. API가 `딕셔너리 API`_\ 와 매우 유사하다는 사실을 알게 될 것이다.

아이템 생성
---------------------

::

    >>> product = Product(name='Desktop PC', price=1000)
    >>> print product
    Product(name='Desktop PC', price=1000)

필드 값 얻기
-----------------------

::

    >>> product['name']
    Desktop PC
    >>> product.get('name')
    Desktop PC

    >>> product['price']
    1000

    >>> product['last_updated']
    Traceback (most recent call last):
        ...
    KeyError: 'last_updated'

    >>> product.get('last_updated', 'not set')
    not set

    >>> product['lala'] # getting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'lala'

    >>> product.get('lala', 'unknown field')
    'unknown field'

    >>> 'name' in product  # is name field populated?
    True

    >>> 'last_updated' in product  # is last_updated populated?
    False

    >>> 'last_updated' in product.fields  # is last_updated a declared field?
    True

    >>> 'lala' in product.fields  # is lala a declared field?
    False

필드 값 설정
----------------------

::

    >>> product['last_updated'] = 'today'
    >>> product['last_updated']
    today

    >>> product['lala'] = 'test' # setting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'

Accessing all populated values
------------------------------

입력된 모든 값에 접근하려면, 일반적인 `딕셔너리 API`_\ 를 사용하라::

    >>> product.keys()
    ['price', 'name']

    >>> product.items()
    [('price', 1000), ('name', 'Desktop PC')]

다른 일반 작업
-----------------------

아이템 복사::

    >>> product2 = Product(product)
    >>> print product2
    Product(name='Desktop PC', price=1000)

    >>> product3 = product2.copy()
    >>> print product3
    Product(name='Desktop PC', price=1000)

아이템으로 딕셔너리 생성::

    >>> dict(product) # create a dict from all populated values
    {'price': 1000, 'name': 'Desktop PC'}

딕셔너리로 아이템 생성::

    >>> Product({'name': 'Laptop PC', 'price': 1500})
    Product(price=1500, name='Laptop PC')

    >>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'

아이템 확장하기
=============================

원본 아이템의 상속클래스를 선언해서 아이템을 확장(필드를 추가하거나
필드의 메타데이터를 변경)할 수 있다.

예::

    class DiscountedProduct(Product):
        discount_percent = scrapy.Field(serializer=str)
        discount_expiration_date = scrapy.Field()

또한 이전 필드의 메타데이터를 사용해서 값을 더 추가하거나 기존의 값을 변경시켜서 필드
메타데이터를 확장할 수 있다::::

    class SpecificProduct(Product):
        name = scrapy.Field(Product.fields['name'], serializer=my_serializer)

위 코드는 ``name`` 필드를 위한 ``serializeer`` 메타데이터 키를 추가 (또는 교체) 시키면서
기존의 메타데이터 값은 유지시켰다.

아이템 객체
=================

.. class:: Item([arg])

    주어진 인자로 선택적으로 초기화된 새로운 아이템을 반환한다.

    아이템은 생성자(constructor)를 포함해, 기본 `딕셔너리 API`_\ 를 복제했다.
    아이템으로 제공되는 유일한 추가 속성은 아래와 같다:

    .. attribute:: fields

        이 아이템에 입력된 것뿐만 아니라 *선언된 모든 필드*\ 를 포함하는 사전.
        키는 필드 이름이고 값은 :ref:`아이템 선언 <topics-items-declaring>` 내에서 사용되는 :class:`Field` 객체다.

.. _딕셔너리 API: https://docs.python.org/2/library/stdtypes.html#dict

필드 객체
=============

.. class:: Field([arg])

    :class:`Field` 클래스는 단순히 내장 `딕셔너리`_ 클래스에 대한 알리아스(alilas)이며
    추가적인 기능이나 속성을 제공하지 않는다. 즉, :class:`Field` 객체는 평범한 파이썬
    딕셔너리다. 별도의 클래스는 클래스 속성을 기반으로 하는 :ref:`아이템 선언 신택스 <topics-items-declaring>`\ 를
    지원하는 데 사용된다

.. _딕셔너리: https://docs.python.org/2/library/stdtypes.html#dict


