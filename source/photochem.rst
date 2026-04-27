Watching a photochemical reaction
=================================

Description and simulation of the experiment
----------------------------

In this chapter we're extending the experiment to monitor a photo-isomerisation reaction. You may think about molecules like azobenzen or stilbene or similar. When left long enough in the dark, all molecules will be in their trans form. Upon illumination with blue light, a certain fraction undergoes an isomerisation to the cis form. The cis to trans back-reaction is thermally activated. The cycle of the reaction can be descibed by a set of coupled ordinary dirrerential equations in the two concentrations

.. math::
   :label: rates

   \dot c_\mathrm{trans}(t)
   &= - k_\mathrm{iso}(t)c_\mathrm{trans}(t)
      + k_\mathrm{back}c_\mathrm{cis}(t) \\
   \dot c_\mathrm{cis}(t)
   &= k_\mathrm{iso}(t)c_\mathrm{trans}(t)
      - k_\mathrm{back}c_\mathrm{cis}(t).

The isomerisation rate :math:`k_\mathrm{iso}(t)` depends on the light intensity and, through the sample absorption at the wavelength of the excitation light, also on the concentrations of the trans and cis isomers, :math:`c_\mathrm{trans}(t)` and :math:`c_\mathrm{cis}(t)`, respectively. The back-reaction rate, :math:`k_\mathrm{back}`, on the other hand, is truly a constant.

The reaction can be monitored by recording the absorption spectrum as a function of time while irradiating the sample whith exitation light. A very simple approach is to combine the two arrangements used so far, illuminating the sample with both excitation and probe light at the same time and record a sequence of absorption spectra. We'll use this approach as starting point and discuss the drawbacks and their improvements later.

Light passing through an absorbing sample is attenuated according to Beer-Lambert's law

.. math::
   I(l) = I_0 e^{-\varepsilon cl}

where :math:`\varepsilon` is the molar extinction coefficient of the absorbing species, :math:`c` its concentration in mol/liters and :math:`l` the length of the optical path in the sample in centimeters. The intensity of the excitation light as a function of optical path length in the sample solution is therefore given by

.. math::
   I(l) =
   I_0 e^{-(\varepsilon_\mathrm{trans}c_\mathrm{trans}
            + \varepsilon_\mathrm{cis}c_\mathrm{cis})l}


During a short amount of time :math:`dt` the number of photons absorbed by the sample is

.. math::
   dn = \frac{I_0}{h\nu}\left[1 - e^{\varepsilon c(t)L}\right]dt

where :math:`h\nu` is the energy of an excitation photon. The rate of isomerisation is therefore given by

.. math::
   k_\mathrm{iso}(t) = \Phi_\mathrm{iso}\dot n(t)\frac V{N_\mathrm A}
   = \frac{I_0V\Phi_\mathrm{iso}}{h\nu N_\mathrm A}
     \left[1 - e^{(\varepsilon_\mathrm{trans}c_\mathrm{trans}
           + \varepsilon_\mathrm{cis}c_\mathrm{cis})L}\right]\cdot
     \frac{\varepsilon_\mathrm{trans}c_\mathrm{trans}}
          {\varepsilon_\mathrm{trans}c_\mathrm{trans}
           + \varepsilon_\mathrm{cis}c_\mathrm{cis}}
      
where :math:`\Phi_\mathrm{iso}` is the quantum yield of isomerisation, :math:`N_\mathrm A` Avogadro's constant and :math:`V` the illuminated sample volume. The last term on the right hand side takes care of the fact that only part of the total absorption is due to molecules in the trans form. Since equations :eq:`rates` are nonlinear rate equations and form a closed loop they have to be solved numerically using an ODE solver.




A slightly more advanced measurement procedure involves switching off the excitation laser while measuring the absorption to avoid disturbing the spectrum by scattered excitation light, and switching off the probe light while not recording the absoption to avoid photo-isomerisation induced by probe light. As a further improvement we'll implement a way to take drifts of the probe light into account.

The following sketch shows the extension of the previously used arrangement. Using two Y-fibres, the whitelight light from the lamp can be monitored without changing the cuvette.

.. image:: sketch-photochem.png

This permits to correct for drifts of the lamp spectrum over the course of the experiment. However, it doesn't permit to automatically take a reference of the lamp spectrum :math:`R` beacuse the two beam paths are not identical. :math:`R` has still to be recorded by manually insterting a pure sample solvent. :math:`I_0`, the incident intensity monitored through the additional beam path, has to be recorded previous to the experiment as well. During the experiment and when needed, the shutters can then be switched such that :math:`I`, the incident intensity during the experiment, can be re-measured. Without drifts one should have :math:`I_0=I`. The absorption is then obtained from

.. math::
   A(\lambda) = -\log_{10}\left(\frac SI\cdot\frac{I_0}R\right)

Note that when dealing with absorption in optical density, the `decadic` molar extinction coefficient is typically used 

.. math::
   I(l) = I_0 10^{-\varepsilon_{10}cl} \qquad
   A(l) = -\log_{10} \left(\frac I{I_0}\right) = \varepsilon_{10}cl

We use the natural one :math:`\varepsilon=\varepsilon_{10}\log(10)` in this document so that the factor :math:`\log(10)` cluttering-up the equations can be dropped.
