
Grass GIS to extract River Profiles
===================================

A methodology on using GRASS GIS to extract river profiles from
digital elevation models.


Getting Started
---------------

These instructions will get you a copy of the documentation on your 
local machine if you wish to edit and contribute. 

An online version can be read here:

https://grass-gis-to-extract-river-profiles.readthedocs.io/en/latest/


Prerequisites
-------------

To build the documentation, the following packages need to be installed:

-  Python 2 or 3
-  Sphinx 2.1.1 - This can be installed with pip or conda depending on how you
   manage your Python packages

.. code:: bash

   $ sudo apt install python3-sphinx or sudo apt install python-sphinx
   $ sudo pip install sphinx

-  GRASS GIS 7.6

.. code:: bash

   $ sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
   $ sudo apt-get update
   $ sudo apt-get install grass


Build PDF Documentation
-----------------------

Install the project locally and run:

.. code:: bash

   $ make latexpdf

Contribute!
-----------

Please contribute! Use `Github Flow<https://guides.github.com/introduction/flow/index.html>`_ to suggest changes.

- Fork the repo and create your branch in master - send me pull requests.
- Documentation uses `Python-Sphinx<http://www.sphinx-doc.org/en/master/>`_ and `reStructuredText<http://docutils.sourceforge.net/rst.html>`_ syntax
