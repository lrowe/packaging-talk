A short introduction to Python packaging
++++++++++++++++++++++++++++++++++++++++

Laurence Rowe. December 13, 2012.

.. slides::

    * View as `single page`_

.. notslides::

    * View as `slides`_

Some slides were cribbed from Ian Bicking's presentation:
http://svn.colorstudy.com/home/ianb/setuptools-presentation/setuptools-slides.txt

.. _single page: ../
.. _slides: slides/

A very brief history of Python packaging
========================================

Distutils
=========

* ``distutils`` in the standard library
* Old way of packaging and installing packages
* No support for specifying requirements

Setuptools
==========

* Added support for requirements
* easy_install to download and install from PyPI

Distribute
==========

* Community developed fork of setuptools
  - Setuptools author did not have time to maintain it

* Work on Setuptools seems to have resumed now...
  - Install one or the other not both!
  - Most people seem to have moved to distribute

Distutils 2
===========

* Attempt to create a new, cleaned up packaging system for the standard library
* Seems to have stalled...

Creating a Package
==================

Lay your files out like this::

    MyPackage/
        MANIFEST.in
        setup.py
        src/
            mypackage/
                __init__.py
                other_stuff.py
                data/
                  mydata.xml
                tests/
        docs/
        README.rst
        CHANGES.rst

Package Layout
==============

* Your "distribution" has a name: ``MyPackage``
* Not to be confused with your "package": ``mypackage`` (of course, probably will be confused)
* Packages (and modules) all lower-case by convention
* A distribution can contain multiple top level packages.
* I prefer to put the packages under "src" to maintain a clean Python path when developing.
* Putting tests within the package path lets you test

Package Layout
==============

* Documentation and (usually) tests go outside the package
* ``src/mypackage/`` is all that really gets "installed"
* ``setup.py`` describes the package
* ``MANIFEST.in``, specifies which files to include in the package

setup.py
========

A typical ``setup.py``::

    import os
    from setuptools import setup, find_packages

    here = os.path.abspath(os.path.dirname(__file__))
    README = open(os.path.join(here, 'README.rst')).read()
    CHANGES = open(os.path.join(here, 'CHANGES.rst')).read()

    install_requires = [
        'SQLAlchemy>=0.7',
        'pyramid>=1.4b1',
        'setuptools',
        ]

    tests_require = [
        'WebTest',
        ]

    ...

setup.py: continued
===================

::

    ...

    setup(
        name='MyPackage',
        version='0.1dev',
        description='An example web app.',
        long_description=README + '\n\n' + CHANGES,
        author='Laurence Rowe',
        author_email='lrowe@stanford.edu',
        url="http://example.com/mypackage",
        license="MIT",

    ...

setup.py: continued
===================

::

    ...

        install_requires=install_requires,
        packages=find_packages('src'),
        package_dir={'': 'src'},
        include_package_data=True,
        zip_safe=False,
        tests_require=tests_require,
        extras_require={
            'test': tests_require,
            },
        entry_points='''
            [paste.app_factory]
            main = mypackage:main
            ''',
        )

setup.py: an explanation
========================

* All the metadata goes in ``setup()``
* Some of this is used to install the package
* Some is used to create an archive of the package
* Some is used for dependencies
* Some is used for PyPI

setup.py: the arguments
=======================

``name``:
    The name of your distribution.  Don't put spaces in it.  Becomes the name of your archive.

``version``:
    The version.  Suffixes like ``a1`` and ``pre5`` are sorted as you'd expect.

``description``, ``long_description``:
    For use by PyPI.  ``long_description`` is in restructured-text format. I like to use the README and CHANGES for this.

``author``, ``author_email``, ``url``, ``license``:
    Also used by PyPI.

setup.py: more arguments
========================

``packages``:
    You list *all* the packages that should be installed, including subpackages, like ``['mypackage', ...]``.  

    ``find_packages()`` does this for you.  The ``exclude`` argument keeps it from auto-detecting things that look like packages.

``include_package_data``:
    If set to True, this tells setuptools to automatically include any data files it finds inside your package directories, that are either under CVS or Subversion control, or which are specified by your MANIFEST.in file.

``zip_safe``:
    A boolean (True or False) flag specifying whether the project can be safely installed and run from a zip file.

setup.py: requirements
======================

``install_requires``:
    This is a list of requirements for this package.  Each is a
    package a string like you would give to ``easy_install.py``.

``tests_require``:
    This is a list of requirements for running the tests with ``setup.py test``. You normally want a better test runner.

``extras_require``:
    Optional requirements, installed with ``MyPackage[extra_name]``.

Setuptools does not expose ``tests_require``, so make them available as an 'extra' too.

setup.py: package data
======================

``include_package_data``:
    If set to True, this tells setuptools to automatically include any data files it finds inside your package directories, that are either under CVS or Subversion control, or which are specified by your MANIFEST.in file.

With Git or Mercurial, you should include a MANIFEST.in.

MANIFEST.in
===========

::

    recursive-include src *
    recursive-include docs *
    include *.rst *.txt *.ini *.cfg *.py *.rb
    global-exclude *.pyc
    global-exclude .DS_Store
    exclude .installed.cfg .mr.developer.cfg
    prune src/*.egg-info
    prune docs/build

Isolation
=========

Be careful not to change any of your system installed packages or things can break!

Virtualenv
==========

Creates an independent python for you.

::

    $ sudo easy_install virtualenv
    $ virtualenv myenv --no-site-packages --distribute
    $ cd myenv
    $ source bin/activate  # or just use bin/python directly

You can choose whether to include any system installed packages.

Pip
===

The new ``easy_install``, included by default with virtualenv.

Usage::

    $ pip install MyPackage
    $ pip freeze > requirements.txt
    $ pip install -r requirements.txt

Freezing requirements gives you a simple way to create a repeatable installation.

Buildout
========

* Like Virtualenv and Pip on steroids (though it predates them.)
* Defines a number of ``parts`` to build
* Each 'part' is built to a recipe
  - A recipe is simply a python package
  - In the simplest case it just installs some python distributions.
  - More recipes can be found on PyPI by searching for "buildout recipe"
  - Can even download and build other software.

buildout.cfg: A simple example
==============================

::

    [buildout]
    # Ignore system installed packages
    include-site-packages = false
    parts = mypackage

    # Build this part using the ``zc.recipe.egg`` recipe.
    [mypackage]
    recipe = zc.recipe.egg
    eggs = MyPackage
    interpreter = py

Installing a buildout
=====================

Download bootstrap.py::

    $ cd newproject
    $ wget https://raw.github.com/buildout/buildout/1.6.x/bootstrap/bootstrap.py

Run bootstrap with your desired version of Python::

    $ python2.7 bootstrap.py --distribute

Run the buildout::

    $ bin/buildout

buildout.cfg: A more complex example
====================================

::

    [buildout]
    # Extensions are one of t
    extensions =
        buildout.dumppickedversions
        mr.developer
    extends = versions.cfg
    parts =
        encoded
        rubygems
        compile-css
        test
        behave

    # Use the package source in this directory
    develop = .

    ...

buildout.cfg: continued
=======================

::

    ...
    # The mr.developer extension automates checkout of dependent packages.
    sources-dir = develop
    auto-checkout =
        behave
        behaving

    [sources]
    # Use my fork until https://github.com/jeamland/behave/pull/102 is merged
    behave = git https://github.com/lrowe/behave.git
    behaving = git https://bitbucket.org/ggozad/behaving.git

    ...

buildout.cfg: continued
=======================

::

    ...
    [encoded]
    recipe = zc.recipe.egg
    eggs =
        encoded
        pyramid
        waitress
    interpreter = py

    # Install Ruby gems:
    [rubygems]
    recipe = rubygemsrecipe
    gems =
        sass
        compass

    # Run a shell command on install
    [compile-css]
    recipe = collective.recipe.cmd
    on_install = true
    on_update = true
    cmds =
        ${buildout:directory}/bin/compass compile

    ...

buildout.cfg: continued
=======================

::

    ...

    # The ``test`` extra dependencies are installed here
    [test]
    recipe = zc.recipe.egg
    eggs =
        encoded[test]
        pytest
    # Rename the testrunner to ``test``
    scripts =
        py.test=test

    [behave]
    recipe = zc.recipe.egg
    eggs =
        behave
        behaving


versions.cfg
============

Equivalent to Pip's requirements.txt::

    [buildout]
    versions = versions

    [versions]
    Chameleon = 2.11
    Mako = 0.7.3
    MarkupSafe = 0.15
    PasteDeploy = 1.5.0
    SQLAlchemy = 0.7.9
    WebOb = 1.2.3
    WebTest = 1.4.3
    collective.recipe.cmd = 0.6
    ...

Resources
=========

* http://distribute.readthedocs.org/
* http://pypi.python.org/pypi/virtualenv
* http://www.virtualenv.org/
* http://www.pip-installer.org/
* http://www.buildout.org/
