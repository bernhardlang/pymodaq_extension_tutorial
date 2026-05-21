Introduction
============

This tutorial guides through the coding of a measurement application in form of a PyMoDAQ dashboard extension. It covers mainly advanced material beyond writing instrument plugins, but still starting step by step from scratch. The reader should be familar with the use of PyMoDAQ, its dashboard and the standard extensions like the DAQ scan or the data logger. Knowing how to programm in Python is also pretty helpful. The tutorial does not rely on any special hardware (besides a computer). All instrumental aspects are simulated.


The Experiment
--------------

The simulated experiment used for this tutorial involves mesauring the absorption of a liquid solution. Let's assume that we have dissolved some dye molecules in some solvents which absorb light aroud a certain wavelength. The aim of the experiment is to determine the corresonding absorption spectrum. The intens ity of the light transmitted to through the sample, :math:`I_\mathrm{trans}`, is given by

.. math::
   I_\mathrm{trans} = I_0 10^{-\varepsilon cl}

where :math:`I_0` is the incident intensity, :math:`\varepsilon` the molar extinction coefficient of the dissolved dye molecules, :math:`c` their concentration and :math:`l` the length of the optical path in the sample cell. The absorption of interest is then given by

.. math::
   A(\lambda) = -\log_{10}\frac{I(\lambda)}{I_0(\lambda)}
   = \varepsilon c(\lambda)l


The detector of the spectro-photometer used to record the intensity of the transmitted light as a function of wavelength exhibits a certain thermally induced dark signal which has to be subtracted from the obtained signal. In consequence, this signal has to be determined beforehand by closing a shutter in front of the entrance of the spectro-phhotometer and recording the signal with no incident light.

The experiment in its base version consists of a whitelight lamp, a sample cuvette, a shutter and a spectro-photometer. The shutter is used to record the dark signal of the photometer's detector.

.. image:: sketch-experiment.png

In a second step, the incident light intensity has to be determined. At the same time, the effect of reflections and scatter on the surfaces of the sample cell and in the solvent, which induce some additional, featureless absorption, can be taken into account. To this end, the sample cell is filled first with solvent only, and the transmitted light intensity is recorded as reference :math:`I_0(\lambda)`.

In later chapters, the experiment will be extended to measure fluoresence spectra and finally to record the course of a photochemical reaction by measuring a series of absorption spectra as a function of time.


The Coded Machinery
-------------------

To interface and controll this experiment, we'll need to control the spectrophotometer and the shutter. This involves writing a PyMoDAQ plugin for each of these. The flow of the experiment, measuring first the 
