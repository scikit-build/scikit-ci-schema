scikit-ci-schema
================

YAML schema and examples for scikit-ci format.

`scikit-ci` format describes how to list commands to execute on multiple
continous integration service in one file.


Validation
----------

The `kwalify <http://www.kuwata-lab.com/kwalify/>`_ validator should be used.


::

  $ kwalify -lf scikit-ci-schema.yml scikit-ci.yml
  scikit-ci.yml#0: valid.


To facilitate the installation of the kwalify tool implemented in ruby, the docker
image `jcfr/dock-kwalify https://microbadger.com/images/jcfr/dock-kwalify`_
can be used: ::

  curl https://raw.githubusercontent.com/jcfr/dock-kwalify/master/kwalify.sh \
  -o ~/bin/kwalify && \
  chmod +x ~/bin/kwalify


.. note:

  Since the current version of `scikit-ci` schema makes use of `anchors and aliases <http://www.kuwata-lab.com/kwalify/ruby/users-guide.02.html#tips-anchor>`_,
  the python implementation of the validator `pykwalify <https://github.com/Grokzen/pykwalify>`_ can not yet be used.


Example
-------

::

  schema_version: "0.5.0"

  before_install:

    environment:
      PYTHON: python
      RUN_ENV: ""

    appveyor:
      environment:
        PYTEST_ADDOPTS: "-m \"not fortran\""
        PYTHON: $<PYTHON_DIR>\\python.exe
        RUN_ENV: .\\..\\ci\\appveyor\\run-with-visual-studio.cmd
      commands:
        - python ../ci/appveyor/patch_vs2008.py
        - python ../ci/appveyor/install_visual_studio_wrapper.py
        - python ../ci/appveyor/tweak_environment.py

    travis:
      osx:
        environment:
          PYTEST_ADDOPTS: "-m \"not fortran\""
          RUN_ENV: ../ci/travis/run-with-pyenv.sh
        commands:
          - python ../ci/travis/install_pyenv.py

  install:

    commands:
      - python ../ci/install_cmake.py 3.6.2
      - $<RUN_ENV> $<PYTHON> -m pip install --disable-pip-version-check --upgrade pip
      - $<RUN_ENV> $<PYTHON> -m pip install -r requirements.txt
      - $<RUN_ENV> $<PYTHON> -m pip install -r requirements-dev.txt

    circle:
      commands:
        - sudo apt-get update
        - sudo apt-get install gfortran

  before_build:
    commands:
      - $<RUN_ENV> $<PYTHON> -m flake8 -v

  build:
    commands:
      - $<RUN_ENV> $<PYTHON> setup.py build

  test:

    appveyor:
      environment:
        PATH: $<PYTHON_DIR>\\Scripts;$<PATH>

    commands:
      - $<RUN_ENV> $<PYTHON> setup.py test

  after_test:

    commands:
      - $<RUN_ENV> codecov -X gcov --required --file ./tests/coverage.xml
      - $<RUN_ENV> $<PYTHON> setup.py bdist_wheel

    appveyor:
      commands:
        - $<RUN_ENV> $<PYTHON> setup.py bdist_wininst
        - $<RUN_ENV> $<PYTHON> setup.py bdist_msi


License
-------

This project is maintained by Jean-Christophe Fillion-Robin from Kitware Inc.

Materials in this repository are distributed under the following licenses:

- Unless specified otherwise, all Works of Art are licensed under the Creative Commons by Attribution 4.0 License. See LICENSE_CC_BY_40 file for details.
