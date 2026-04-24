TODO Introduction
=================

This tutorial guides through the coding of a measurement application in form of a PyMoDAQ dashboard extension. It covers advanced material in a step by step manner. The reader should be familer with the use of PyMoDAQ, its dashboard and the extensions like the DAQ scan or the data logger. Knowledge

The experiment in its base version consists of a whitelight lamp, a sample cuvette, a shutter and a spectro-photometer. The shutter is used to record the dark signal of the photometer's detector.

.. image:: sketch-experiment.png

Later on, the experiment will be extended. By means of two Y-fibres and another shutter, half of the light is guided around the sample so that temporal drifts in the light intensity can be monitored and be corrected for. An additional LED shining into the sample at right angle with respect to the whitelight is used to trigger a photochemical reaction.
