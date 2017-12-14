.. _intro-install:

==================
설치 가이드
==================

스크래피(Scrapy) 설치
=========================

스크래피는 파이썬 2.7과 파이썬 3.3 이상에서 실행된다.

`Anaconda`_ 또는 `Miniconda`_\ 를 사용하고 있다면, 리눅스, 윈도우, OS X를 위한
최신 패키지를 보유하고 있는 `conda-forge`_ 채널로부터 패키지를
설치할 수 있다.

``conda``\ 를 사용해서 설치하려면, 아래와 같이 실행하라::

  conda install -c conda-forge scrapy

파이썬 패키지 설치에 익숙하다면 스크래피와 의존성으로
PyPI로 아래와 같이 설치할 수 있다::

    pip install Scrapy

종종 운영체제에 따라서 스크래피 의존성에 대한 컴파일 문 해결을 요구하는 경우도 있다.
그러면 :ref:`intro-install-platform-notes`\ 를 확인해 보아라.

당신의 시스템 패키지와 충돌하는 것을 피하기 위해서
스크래피를 :ref:`전용 가상 환경 <intro-using-virtualenv>`\ 에
설치하는 것을 강하게 추천하는 바이다.제

더 자세한 정보와 플랫폼 세부 안내는 앞으로 설명할 것이다.


알아두면 좋은 것들
----------------------------

스크래피는 순수한 파이썬으로 작성되었으며 몇 가지 핵심 파이썬 패키지에 의존한다 (다른 것들 중에서는):

* `lxml`_, 효율적인 XML, HTML 파서(parser)
* `parsel`_, lxml 기반으로 작성된 HTML/XML 데이터 추출 라이브러리
* `w3lib`_, URL 및 웹 페이지 인코딩을 위한 다용도 헬퍼(helper)
* `twisted`_, 비동기적 네트워킹 프레임워크
* `cryptography`_\ 과 `pyOpenSSL`_, 다양한 네트워크 수준 보안 요구사항 처리

스크래피가 테스트된 최저 버전은 다음과 같다:

* Twisted 14.0
* lxml 3.4
* pyOpenSSL 0.14

스크래피는 오래된 버전의 패키지에서도 작동할 수 있지만
이전 버전에 대해서는 테스트가 되지 않았기 때문에 계속 작동을 할지는 보장되지 않는다.

플랫폼에 따라서는 이 패키지중 일부가 파이썬이 아닌 패키지에 의존해서
추가적인 설치 과정을 요구할 수도 있다.
아래에 있는 :ref:`플랫폼 별 가이드 <intro-install-platform-notes>`\ 를 참고하라.

이 의존성에 관해서 문제가 있는 경우,
각 라이브러리의 설치 안내를 참고하기 바란다:

* `lxml 설치`_
* `cryptography 설치`_

.. _lxml 설치: http://lxml.de/installation.html
.. _cryptography 설치: https://cryptography.io/en/latest/installation/


.. _intro-using-virtualenv:

가상 환경 사용하기 (추천)
-----------------------------------------

TL;DR: 우리는 플랫폼에 상관없이 가상 환경에 스크래피를 설치하는 것을
추천한다.

파이썬 패키지는 전역(시스템 전체 범위)에 설치될 수도 있고 사용자 공간에 설치될 수도 있다.
우리는 스크래피를 시스템 전체 범위에 설치하는 것을 추천하지 않는다.

대신, 소위 "가상 환경"(`virtualenv`_)이라 불리는 곳에 설치할 것을 추천한다.
가상 환경은 이미 설치된 파이썬 시스템 패키지와 충돌(시스템 도구나 스크립트를 망가뜨릴 수 있다)하지 않게 만들어 주며
(``sudo`` 같은 것 없이) ``pip``\ 로 정상적으로 설치할 수 있게 한다.

가상 환경에 대해 알아보려면 `가상 환경 설치 안내`_\ 를 참고하라.
전역으로 설치하려면 (여기서는 전역으로 설치하는 것이 도움이 된다),
다음을 실행하면 된다::

    $ [sudo] pip install virtualenv

가상 환경 생성은 `사용자 가이드`_\ 를 참고하라.

.. note::
    Linux 또는 OS X를 사용하는 경우, `virtualenvwrapper`_\ 를 사용해서 쉽게 가상 환경을 생성할
    수 있다.

가상 환경을 생성하고 나면, 가상 환경 안애 다른 패키지와 마찬가지로 ``pip``\ 로 스크래피를
설치할 수 있다.
(먼저 설치할 필요가 있는 비파이썬 의존성에 대해서는 아래의
:ref:`플랫폼 별 가이드 <intro-install-platform-notes>`\ 를 참고하라.)

파이썬 가상환경은 파이썬 2나 3을 기본으로 생성할 수 있다.

* 스크래피를 파이썬 3 용으로 설치하고 싶다면, 파이썬 3 가상 환경에 스크래피를 설치하면 된다.
* 만약 파이썬 2 용으로 설치하고 싶다면, 파이썬 2 가상 환경에 설치하면 된다.

.. _virtualenv: https://virtualenv.pypa.io
.. _가상 환경 설치 안내: https://virtualenv.pypa.io/en/stable/installation/
.. _virtualenvwrapper: https://virtualenvwrapper.readthedocs.io/en/latest/install.html
.. _사용자 가이드: https://virtualenv.pypa.io/en/stable/userguide/


.. _intro-install-platform-notes:

플랫폼 별 설치 안내
====================================

.. _intro-install-windows:

윈도우(Windows)
-----------------------

윈도우에 pip를 사용해서 스크래피를 설치하는 것이 가능하지만,
대부분의 설치 문제를 피할 수 있기 때문에
`Anaconda`_ 또는 `Miniconda`_\ 를 설치 한 후에 `conda-forge`_ 채널을 사용해서
스크래피를 설치할 것을 추천한다.

이미 `Anaconda`_ 또는 `Miniconda`_\ 을 설치했다면, 아래와 같이 스크래피를 설치하라::

  conda install -c conda-forge scrapy


.. _intro-install-ubuntu:

우분투(Ubuntu) 12.04 이상
--------------------------------------------------------

스크래피는 최신 버전의 lxml, twisted, pyOpenSSL로 테스트되었으며, 최신
우분투 패포 버전과 호환이 가능하다.
하지만 잠재적인 TLS 연결 문제가 있지만 12.04 같은 이전 버전의 우분투도 지원한다.

일반적으로 너무 오래된 버적이고 최신 버전을 따라잡는데 느리기 때문에
우분투에서 제공하는 ``python-scrapy`` 패키지를 사용하면 **안** 된다.


스크래피를 우분투(또는 우분투 기반) 시스템에 설치하려면 아래의 의존성을 설치해야 한다::

    sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

- ``python-dev``, ``zlib1g-dev``, ``libxml2-dev``, ``libxslt1-dev``\ 는
  ``lxml``\ 을 위해 필요하다
- ``libssl-dev``, ``libffi-dev``\ 는 ``cryptography``\ 를 위해 필요하다

파이썬 3에 스크래피를 설치하고 싶으면, 파이썬 3 개발 헤더도 필요하다::

    sudo apt-get install python3 python3-dev

:ref:`가상 환경 <intro-using-virtualenv>`\ 에서
``pip``\ 로 스크래피를 설치할 수 있다::

    pip install scrapy

.. note::
    Debian Wheezy (7.0) 이상에서 동일한 비파이썬 의존성이 스크래피 설치를 위해 사용될 수 있다.


.. _intro-install-macos:

맥(Mac) OS X
--------------------

스크래피의 의존성을 설치는 C 컴파일러와 개발 헤더를 필요로 하는데.
OS X에서는 일반적으로 애플의 Xcode 개발 도구를 통해 제공된다.Xcode 커맨드 라인
도구를 설치하려면 터미널 창을 열고 다음을 실행하라::

    xcode-select --install

``pip``\ 의 시스템 패키지 업데이트를 막는 `문제 <https://github.com/pypa/pip/issues/2468>`_\ 가
있다. 이는 스크래피와 의존성 설치를 성공적으로 하기 위해서 해결되어야 한다.
제안되는 해결첵은 다음과 같다:

* *(추천)* 시스템 파이썬을 사용하지 **말고** 사용자의 시스템과 충돌을 일으키지 않는
  업데이트 된 새로운 버전을 설치하라. `homebrew`_ 패키지 매니저를 사용해서
  설치하는 법은 아래와 같다:

  * https://brew.sh/ 안내에 따라 `homebrew`_\ 를 설치하라

  * ``PATH`` 변수를 시스템 패키지 전에 homebrew 패키지가 사용되도록 업데이트 하라.
    (기본 셸로 `zsh`_\ 를 사용하고 있으면 ``.bashrc``\ 를 ``.zshrc``\ 로 변경하라.)::

      echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

  * 변경이 제대로 되도록 ``.bashrc``\ 를 다시 로드하라::

      source ~/.bashrc

  * 파이썬을 설치하라::

      brew install python

  * 최신 버전의 파이썬은 ``pip``\ 를 번들로 포함하고 있다. 따라서 따로 설치할 필요가 없다.
    그렇지 않은 경우에는 파이썬을 업그레이드하라::

      brew update; brew upgrade python

* *(선택)* 분리된 파이썬 환경에 스크래피를 설치하라.

  이 방법은 위의 OS X 문제를 위한 해결방법이지만, 의존성 관리를 위한 전반적인 좋은
  방법이며, 첫 번째 방법을 보완할 수 있습니다.

  `virtualenv`_\ 는 파이썬에서 가상 환경을 생성하기 위해 사용하는 도구이다.
  우리는 http://docs.python-guide.org/en/latest/dev/virtualenvs/ 튜토리얼을 읽어볼 것을 추천한다.

해결이 되었으면 스크래피를 설치한다::

  pip install Scrapy


.. _Python: https://www.python.org/
.. _pip: https://pip.pypa.io/en/latest/installing/
.. _lxml: http://lxml.de/
.. _parsel: https://pypi.python.org/pypi/parsel
.. _w3lib: https://pypi.python.org/pypi/w3lib
.. _twisted: https://twistedmatrix.com/
.. _cryptography: https://cryptography.io/
.. _pyOpenSSL: https://pypi.python.org/pypi/pyOpenSSL
.. _setuptools: https://pypi.python.org/pypi/setuptools
.. _AUR Scrapy package: https://aur.archlinux.org/packages/scrapy/
.. _homebrew: https://brew.sh/
.. _zsh: https://www.zsh.org/
.. _Scrapinghub: https://scrapinghub.com
.. _Anaconda: https://docs.anaconda.com/anaconda/
.. _Miniconda: https://conda.io/docs/user-guide/install/index.html
.. _conda-forge: https://conda-forge.org/
