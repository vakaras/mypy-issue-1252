Set up Docker (just to make sure that it is not global configuration problem):

::

    sudo docker run --rm -ti -v "$(pwd):/tmp/test" ubuntu:15.10
    apt-get update && apt-get install -y wget python3-dev git

Reproduce bug
=============

::

    cd /tmp/test
    wget -c \
        https://pypi.python.org/packages/source/v/virtualenv/virtualenv-14.0.5.tar.gz \
        -O virtualenv.tar.gz
    tar -xvf virtualenv.tar.gz
    python3 virtualenv-14.0.5/virtualenv.py env
    env/bin/python bootstrap.py -c broken.cfg
    bin/buildout -U -c broken.cfg
    bin/mypy test.py

Prints::

    test.py:1: error: Could not find builtins


Build package
=============

::

    cd /tmp
    git clone --recurse-submodules https://github.com/vakaras/mypy.git
    cd mypy/
    python3 setup.py sdist
    mkdir /tmp/dist
    mv dist/mypy-lang-0.3.1.tar.gz /tmp/dist/

Testing Fix
===========

::

    cd /tmp/test
    rm -rf .installed.cfg bin eggs/mypy_lang-0.3.1-py3.4.egg
    env/bin/python bootstrap.py -c fixed.cfg
    bin/buildout -c fixed.cfg
    bin/mypy test.py

Prints nothing.

Regression Testing
==================

::

    mkdir /tmp/test2
    cd /tmp/test2
    wget -c \
        https://pypi.python.org/packages/source/v/virtualenv/virtualenv-14.0.5.tar.gz \
        -O virtualenv.tar.gz
    tar -xvf virtualenv.tar.gz
    python3 virtualenv-14.0.5/virtualenv.py env
    env/bin/pip install --no-index --find-links=/tmp/dist mypy-lang
    env/bin/mypy /tmp/test/test.py

Prints nothing.
