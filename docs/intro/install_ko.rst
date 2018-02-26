.. _intro-install:

==================
설치 가이드
==================

스크래피 설치
=========================

스크래피는 파이썬 2.7과 파이썬 3.4 이상에서 실행된다.

`Anaconda`_ 또는 `Miniconda`_\ 를 사용하고 있다면, 리눅스, 윈도우, OS X용
최신 패키지를 보유하고 있는 `conda-forge`_ 채널에서
설치할 수 있다.

``conda``\ 를 사용해서 설치하려면, 아래와 같이 실행한다.::

  conda install -c conda-forge scrapy

파이썬 패키지 설치에 익숙하다면 스크래피와 스크래피가 필요로 하는 패키지를
PyPI로 아래와 같이 설치할 수 있다::

    pip install Scrapy

이 경우에는 운영체제에 따라 스크래피가 의존하는 패키지를 컴파일할 때 문제가 발생 할 수 있다.
이 경우에는 :ref:`intro-install-platform-notes`\ 를 확인한다.

스크래피가 시스템 패키지와 충돌하는 것을 피하기 위해서
스크래피를 :ref:`전용 가상 환경 <intro-using-virtualenv>`\ 에
설치하는 것을 강하게 추천한다.

더 자세한 정보와 플랫폼에 대한 설명은 뒤에서 설명한다.


알아두면 좋은 것들
----------------------------

스크래피는 순수한 파이썬으로 작성되었으며 몇 가지 핵심 파이썬 패키지에 의존한다.:

* `lxml`_, 효율적인 XML, HTML 파서(parser)
* `parsel`_, lxml 기반으로 작성된 HTML/XML 데이터 추출 라이브러리
* `w3lib`_, URL 및 웹 페이지 인코딩을 위한 다용도 헬퍼(helper)
* `twisted`_, 비동기적 네트워킹 프레임워크
* `cryptography`_\ 과 `pyOpenSSL`_, 다양한 네트워크 수준 보안 요구사항 처리

스크래피는 다음 최저 사항에서 테스트하였다:

* Twisted 14.0
* lxml 3.4
* pyOpenSSL 0.14

이보다 오래된 버전에서도 작동할 수도 있지만
테스트가 되지 않았기 때문에 작동여부를 보장하지 않는다.

플랫폼에 따라서는 이 패키지중 일부가 파이썬이 아닌 패키지에 의존해서
추가적인 설치 과정을 요구할 수도 있다.
아래에 있는 :ref:`플랫폼 별 가이드 <intro-install-platform-notes>`\ 를 참고하라.

필수 패키지 설치에 문제가 있는 경우,
각 라이브러리의 설치 안내를 참고하기 바란다:

* `lxml 설치`_
* `cryptography 설치`_

.. _lxml 설치: http://lxml.de/installation.html
.. _cryptography 설치: https://cryptography.io/en/latest/installation/


.. _intro-using-virtualenv:

가상 환경 사용하기
-----------------------------------------

결론만 얘기하자면 플랫폼에 상관없이 가상 환경에 스크래피를 설치하는 것을
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

pip를 사용해서 스크래피를 설치하는 것도 가능하지만,
각종 설치상 문제를 피하기 위해
`Anaconda`_ 또는 `Miniconda`_\ 를 설치 한 후에 `conda-forge`_ 채널을 사용하여
스크래피를 설치할 것을 추천한다.

이미 `Anaconda`_ 또는 `Miniconda`_\ 을 설치했다면, 아래와 같이 스크래피를 설치하라::

  conda install -c conda-forge scrapy


.. _intro-install-ubuntu:

우분투(Ubuntu) 14.04 이상
--------------------------------------------------------

스크래피는 최신 버전의 lxml, twisted, pyOpenSSL로 테스트되었으며, 최신
우분투 패포 버전과 호환이 가능하다.
잠재적인 TLS 연결 문제가 있지만 14.04 같은 이전 버전의 우분투도 지원한다.

우분투에서 제공하는 ``python-scrapy`` 패키지는 너무 오래된 버전이므로 사용하면 **안** 된다.


스크래피를 우분투(또는 우분투 기반) 시스템에 설치하려면 아래의 필수 패키지를 설치해야 한다::

    sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

- ``python-dev``, ``zlib1g-dev``, ``libxml2-dev``, ``libxslt1-dev``\ 는
  ``lxml``\ 을 위해 필요하다
- ``libssl-dev``, ``libffi-dev``\ 는 ``cryptography``\ 를 위해 필요하다

파이썬 3에 스크래피를 설치하고 싶으면, 파이썬 3의 개발용 헤더파일도 필요하다::

    sudo apt-get install python3 python3-dev

:ref:`가상 환경 <intro-using-virtualenv>`\ 에서
``pip``\ 로 스크래피를 설치할 수도 있다::

    pip install scrapy

.. note::
    Debian Jessie (8.0) 이상에서도 같은 필수 패키지를 사용한다.


.. _intro-install-macos:

맥(Mac) OS X
--------------------

스크래피가 필요로 하는 패키지를 설치하려면 C 컴파일러와 개발용 헤더파일을 필요하다.
OS X에서는 일반적으로 애플의 Xcode 개발 도구를 통해 제공된다. Xcode 커맨드 라인
도구를 설치하려면 터미널 창을 열고 다음을 실행하라::

    xcode-select --install

``pip``\ 가 시스템 패키지 업데이트를 하지 못하는 `문제 <https://github.com/pypa/pip/issues/2468>`_\ 가
있을 수 있는데 이 경우 스크래피와 필수 패키지 설치가 안될 수 있다. 이에 대한
해결책은 다음과 같다:

* *(추천)* 시스템 파이썬을 사용하지 **말고** 사용자의 시스템과 충돌을 일으키지 않는
  새 버전의 파이썬을 사용한다. `homebrew`_ 패키지 매니저를 사용해서
  다음처럼 설치할 수 있다.:

  * https://brew.sh/ 의 안내에 따라 `homebrew`_\ 를 설치한다.

  * ``PATH`` 변수를 시스템 패키지 전에 homebrew 패키지가 사용되도록 업데이트 한다.
    (기본 셸로 `zsh`_\ 를 사용하고 있으면 ``.bashrc``\ 를 ``.zshrc``\ 로 변경한다.)::

      echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

  * 변경 사항이 적용되도록 ``.bashrc``\ 를 다시 로드한다::

      source ~/.bashrc

  * 파이썬을 설치한다::

      brew install python

  * 최신 버전의 파이썬은 ``pip``\ 를 번들로 포함하고 있다. 따라서 따로 설치할 필요가 없다.
    그렇지 않은 경우에는 파이썬을 업그레이드하라::

      brew update; brew upgrade python

* *(선택)* 새 파이썬 환경에서 스크래피를 설치한다.

  이 방법은 위의 OS X 문제를 위한 해결방법일 뿐 아니라 의존성 관리를 위해서도 좋다.

  `virtualenv`_\ 는 파이썬에서 가상 환경을 생성하기 위해 사용하는 도구이다.
  http://docs.python-guide.org/en/latest/dev/virtualenvs/ 튜토리얼을 읽어볼 것을 추천한다.

모든 해결이 되면 스크래피를 설치한다::

  pip install Scrapy


PyPy
----

최신 PyPy 버전을 권장한다. 테스트를 한 버전은 5.9.0이다.
PyPy3는 리눅스에서만 테스트하였다.

CPython의 경우에는 스크래피가 의존하는 대부분의 패키지가 바이너리 파일로 구현되어 있지만
PyPy에서는 그렇지 않다. 따라서 설치시에 모든 필요 패키지를 직접 빌드해야 한다.

맥 OS X에서는 암호화 패키지를 빌딩할 때 문제가 있다.
해결 방법은
`here <https://github.com/pyca/cryptography/issues/2692#issuecomment-272773481>`_\ 에 있다.
``brew install openssl`` 명령으로 opensll을 설치하고
이 명령이 추천하는 옵션(스크래피 설치에만 필요하다.)을 익스포트해서 사용한다.
리눅스에서 설치할 때는 특별한 문제가 없다.
윈도우즈에서 PyPy 기반으로 스크래피를 사용하는 것은 테스트하지 않았다.

``scrapy bench`` 명령을 실행하면 스크래피가 잘 설치되었는지 확인할 수 있다.
만약 ``TypeError: ... got 2 unexpected keyword arguments`` 등의 오류가 나오면
setuptools가 PyPy의 관련 패키지를 설치하지 못했다는 뜻이다.
이 문제를 해결하려면 ``pip install 'PyPyDispatcher>=2.1.0'``\ 를 실행한다.


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
