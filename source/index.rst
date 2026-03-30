Welcome to the tutorial Writing a PyMoDAQ extension
===================================================

This tutorial covers writing a PyMoDAQ extension which can be used to turn measure absorption and emission spectra using a spectro-photometer starting from the templates provided by PyMoDAQ. A significant part of the tutorial deals with simulated hardware and can be reused for many other kinds of devices. Simulation helps also to avoid communication issues getting into the way while setting up and testing the basic things. Interfacing real hardware is not covered here. There is already quite some material available on that topic.

The code shown in this tutorial has been developped and tested under Linux. The code itself should be portable since everything is done in Python without relying on external libraries. Nonetheless, the methods for utilizing surrounding tools may vary on different operating systems. Setting up the module structure and similar is done on the Linux command line. Experienced Windows users and PyCharm experts are warmly welcome to add corresponding instructions to this tutorial to cover their beloved environments.

.. toctree::
   :maxdepth: 1
   :caption: Contents:

   preparation.rst
   hardware.rst
   viewer1d.rst
   move.rst
   extension.rst


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
