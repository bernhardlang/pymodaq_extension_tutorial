Welcome to the tutorial Writing a PyMoDAQ extension
===================================================

This tutorial guides through writing a PyMoDAQ extension starting from the templates provided by PyMoDAQ. From the point of view of the application, a (simulated) spectro-photometer is turend into an absorption spectrometer. Writing the corresponding plugin is covered for the sake of completeness. Advanced chapters treat incorporating configurator, scanner and data mixer.

The code shown in this tutorial has been developped and tested under Linux. The code itself should be portable since everything is done in Python without relying on external libraries. Nonetheless, the surrounding tools may vary on different operating systems. Setting up the module structure and similar is done on the Linux command line (or git-shell). Experienced Windows users and PyCharm experts are warmly welcome to add corresponding instructions to this tutorial to cover their beloved environments.

.. toctree::
   :maxdepth: 1
   :caption: Contents:

   intro.rst
   preparation.rst
   hardware.rst
   spectro-photometer.rst
   shutter.rst
   dashboard.rst
   extension.rst
   absorption.rst
   state-manager.rst
   fluo.rst
   data-mixer.rst
   photochem.rst
   sequencer.rst
   step-scan.rst


Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
