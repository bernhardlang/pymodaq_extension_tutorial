TODO Watching a photochemical reaction
======================================

The following sketch shows the extension of the previously used arrangement. Using two Y-fibres, the whitelight light from the lamp can be monitored without changing the cuvette.

.. image:: sketch-photochem.png

This permits to correct for drift of the lamp spectrum over the course of the experiment. However, it doesn't permit to automatically take a reference of the lamp spectrum :math:`R` beacuse the two beam paths are not identic. :math:`R` has still to be recorded by manually insterting a pure sample solvent. :math:`I_0`, the incident intensity monitored through the additional beam path, has to be recorded previous to the experiment as well. During the experiment and when needed, the shutters can then be switched such that :math:`I`, the incident intensity during the experiment, can be re-measured. The absorption is then obtained from

.. math::
   A(\lambda) = -\log_{10}\left(\frac SI\cdot\frac{I_0}R\right)

The intensity :math:`I` of the excitation light is attenuated in the sample according to Beer-Labert's law

.. math::
   dI = \log(10)\varepsilon I(l,t)c(t)dl

this corresponds to a number of excited molecules of

.. math::
   dn = \log(10)\frac{A\varepsilon}{h\nu}I(l,t)c(t)dldt

in a short amount of time :math:`dt` and :math:`A` is the area illuminated aby the excitation beam.

The number of molecules :math:`dn` absorbing a photon in an illuminated slice of thickness :math:`\mathrm d l` within a given amout of time is given by the intensity attenuation in that slice.

.. math::
   

The number of photons 
.. math::
   I(l) &= I_0 10^{-c\epsilon l} = I_0 e^{-c(t)\varepsilon l\log(10)}

   k_\mathrm{ex}(l) &= \Phi I(l)

   k(t) &= \int\limits_0^L k_\mathrm{ex}(l,t) \mathrm dl
     = \frac{\Phi I_0}{c(t)\varepsilon \log(10)}
       \left[1 - e^{c(t)\varepsilon L\log(10)}\right]

volumen! flaeche x dl, I -> Leistung -> P x t / hnu -> teilchen pro vol und zeit
