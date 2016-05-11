Using sphinx with lagopus-book
==============================

Sphinx is used to generate lagopus-books in multiple formats, like html/pdf/epub. It will convert reStructuredText (reST: *.rst) format to target formats with simple command like $ make html.

 Refer to http://www.sphinx-doc.org/en/stable/ for more details about Sphinx.

This document first describes how to install Sphinx and build html/pdf/epub from lagopus-book source code downloaded from github. (Sphinx related files required are already in lagopus-book source tree)

It will also describe how to generate and modify Sphinx related files from scrach, in case you need to rebuild the environment.

Software Versions
-----------------

Ubuntu Desktop 15.10 was used to confirm steps described in this document.
However, it is expected to work on Ubuntu 14.04 LTS as well.

 Ubuntu Desktop download page: http://www.ubuntu.com/download/desktop

Sphinx installation
-------------------

 Refer to "First Steps with Sphinx" for more details: http://www.sphinx-doc.org/en/stable/tutorial.html

Install required packages and set path.

::
  
  Basic packages:
  $ sudo apt-get update
  $ sudo apt-get install python-pip python-dev
  $ pip install Sphinx

  For latex/pdf
  $ sudo apt-get install texlive texlive-latex-extra
  (Add texlive-lang-japanese if you want to enable Japanese)

Set PATH to .local/bin/ by adding below to end of ~/.profile
::

  $ vi .profile
  if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin/:$PATH"
  fi
  $ sudo reboot


Building html/pdf/epub documents
--------------------------------

Download source code and run lagopus-book/en$ make <target>

 Validated targets: html, latexpdf, epub

::

  $ git clone https://github.com/lagopus/lagopus-book.git
  $ cd lagopus-book/en
  lagopus-book/en$ make html
  lagopus-book/en$ make latexpdf
  lagopus-book/en$ make epub

Documents will be stored under en/build/<target> directory.
::

  lagopus-book/en/build/html/index.html
  lagopus-book/en/build/latex/lagopus-book.pdf
  lagopus-book/en/build/epub/lagopus-book.epub

Below files are Sphinx related files automatically created while running sphinx-quickstart.

* Edit conf.py to change layout and view of the document.
* Edit index.rst to add pages and change orderings.

::

  lagopus-book/en/Makefile
  lagopus-book/en/source/index.rst
  lagopus-book/en/source/conf.py

More details are described in `Steps to build Sphinx environment from scrach`_.

 Note:
 
 * Some times reference link will not be generated correctly whenyou repeat make html
 * Try make html clean; make html when you face such an issue.

Conventions related to Sphinx
-----------------------------

reStructuredText (.rst)
^^^^^^^^^^^^^^^^^^^^^^^

reStructuredText is used to document lagopus-book.

Pages below are referenced during documentation to be compatible with Sphinx.

* reStructuredText Primer (Sphinx official page)
  * English: http://www.sphinx-doc.org/en/stable/rest.html
  * Japanese: http://docs.sphinx-users.jp/rest.html

External Links
^^^^^^^^^^^^^^

As described in above mentioned "reStructuredText Primer (Sphinx official page)", you can write external link in two ways, "One line" and "Separate". (external = outside of lagopus-book)

``click this link`` is the text shown, and ``<http://TargetURL>`` is target URL of the link.
::

  One line:
  `click this link <http://TargetURL>`_

  Separate:
  `click this link`_.
  .. _click this link: http://<TargetURL>/

Cross document references (Internal Links)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Instead of External Links, "Cross document references" are used to create link to another page in lagopus-book to make sure links are set properly for non html documents.

Follow below steps to set Cross document references.

* Add .. _ref_<filename>: + <empty line> to the first/second (1st/2nd) line of each document as an anchor.
* At where you want to place a link, use :ref:`ref_<filename>`
* <filename> is name of the file without extention (.rst).
* Note _ref (with underscore) is used for reference anchor and ref (without underscore) is used for the link.

Example:

* On first line of "installation-dpdk.rst"

::

  .. _ref_installation-dpdk:
  <blank line>

* At where you want to place a link:

::

  :ref:`ref_installation-dpdk`


Steps to build Sphinx environment from scrach
---------------------------------------------

Steps to build Sphinx environment:

* Install Sphinx
* Run sphinx-quickstart
* Edit configuration files: ``index.rst``, ``conf.py``

Install Sphinx
^^^^^^^^^^^^^^

Refer to `Sphinx installation`_ in this page for steps to install Sphinx.

Run sphinx-quickstart
^^^^^^^^^^^^^^^^^^^^^

``sphinx-quickstart`` is a wizard which will generate configuration files and directory structure by answering to questions. You should run this once per Sphinx project when first creating it.

Source code (*.rst) of lagopus-book is stored under ``lagopus-book/en/source/``. Since we want to keep ``en/source/`` clean, we separate source and build directory like below.
::

  lagopus-book/en/source
  lagopus-book/en/build

To make structure above, run ``sphinx-quickstart`` under ``lagopus-book/en/``.

* Answers you should explicitly enter are listed in example below.
* Press "return" and use default for other questions.

::

  $ cd lagopus-book/en
  lagopus-book/en$ sphinx-quickstart
  > Separate source and build directories (y/n) [n]: y
  > Project name: lagopus-book
  > Author name(s): NIPPON TELEGRAPH AND TELEPHONE CORPORATION
  > Project version: 0.2
  > Project release [0.2]: 0.2.3
  > Do you want to use the epub builder (y/n) [n]: y
  > autodoc: automatically insert docstrings from modules (y/n) [n]: y

As a result, below files will be created.
::

  lagopus-book/en$ ls
  build  make.bat  Makefile  source
  lagopus-book/en$ ls source/
  conf.py  index.rst  _static  _templates

* Do git add if you want to manage them with git.

::

  lagopus-book$ git add en/Makefile
  lagopus-book$ git add en/source/index.rst
  lagopus-book$ git add en/source/conf.py
  (make.bat is not used)

Copy source code under ``lagopus-book/en/source/``.
You could also first download the source code and then run ``sphinx-quickstart``.

Edit configuration files: ``index.rst`` ``conf.py``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit ``index.rst`` to modify document structures and title.

* Change document title to anything you like. For exmaple:

::

  (original)
  Welcome to lagopus-book's documentation!
  ========================================

  (Edit to make it a bit short)
  Welcome to lagopus-book !!
  ==========================

* Add files to the doucment by list file names under .. toctree::

::

  ~/lagopus-book/en/source$ vi index.rst
  .. toctree::
     :maxdepth: 2

     introduction.rst
     installation-rawsocket.rst
     installation-dpdk.rst
     installation-kvm.rst
     cmd-options.rst
     lagosh.rst
     datastore.rst
     sample-ryu-simplesw.rst
     virtualbox.rst

* If not required, remove ``Indices and tables`` below.

::

  Indices and tables
  ==================

  * :ref:`genindex`
  * :ref:`modindex`
  * :ref:`search`

Change HTML theme by editing ``conf.py``.

* For example, comment out ``html_theme = 'alabaster'`` and add line ``html_theme = 'classic'``.

::

  lagopus-book/en/source$ vi conf.py
  #html_theme = 'alabaster'
  html_theme = 'classic'

Edit latex/pdf settings by editing ``conf.py``.

 Using tartget=latexpdf, Sphinx will first convert source (*.rst) to latex format and then to pdf.
 Edit ``preamble`` in ``latex_elements`` for following customizations.

* Enable word wrap in code block.

  * By default, long line in code block will simply go out of the margin.
  * Add below two lines of configuration to enable line break.
  * Refer to github issue for more details: https://github.com/sphinx-doc/sphinx/issues/2304

::
  
  \\usepackage[draft]{minted}
  \\fvset{breaklines=true}

* Change color of code block.
  
  * PDF file created by ``make latefpdf`` with default LaTeX preambles will have code block with single black line as border and no background color. However, the border line would disapper depending on zoom in rate when viewing the pdf file on Windows Adobe Acrobat Reader DC (version 2015.0.10.20059).
  * To workaround this, the color of code-block border line and background should be modified to same color, ex: "LightGray".
  * Add below two lines of configuration to change color of code block.
  * FYI: RGB color chart : http://lowlife.jp/yasusii/static/color_chart.html

::
  
  \\definecolor{VerbatimColor}{rgb}{0.83, 0.83, 0.83}
  \\definecolor{VerbatimBorderColor}{rgb}{0.83, 0.83, 0.83}

* Summary of changes related to latex/pdf settings.

  * After applying above changes, ``preamble`` in ``conf.py`` should be replaced like below.
::

  lagopus-book/en/source$ vi conf.py

  #'preamble': '',
  'preamble': '''
  \\usepackage[draft]{minted}
  \\fvset{breaklines=true}
  \\definecolor{VerbatimColor}{rgb}{0.83, 0.83, 0.83}
  \\definecolor{VerbatimBorderColor}{rgb}{0.83, 0.83, 0.83}
  '''

