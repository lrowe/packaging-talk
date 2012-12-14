A short introduction to Python packaging
++++++++++++++++++++++++++++++++++++++++

Laurence Rowe. December 13, 2012.

Some slides were cribbed from Ian Bicking's presentation:
http://svn.colorstudy.com/home/ianb/setuptools-presentation/setuptools-slides.txt

Notes on gh-pages
=================

First create an unrelated branch::

    $ git checkout --orphan gh-pages

* Commit the .gitignore and rm other files.
* Add and commit an empty ``.nojekyll`` file.

Updating the pages
==================

``bin/regen``:
    Regenerate the sphinx html and slides.

``bin/commit -m "my change"``:
    Commit changes to master branch; regen; commit changes to gh-pages branch; push from gh-pages checkout to parent checkout.
