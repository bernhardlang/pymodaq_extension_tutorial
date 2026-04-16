Simulating a spectro-photometer
===============================

Let us begin this section with a brief comment on simulating devices. Simulating things is a useful starting point here because we avoid communication issues with real devices for now. On top of that, no hardware is needed and the development can take place anywhere where an internet connection is available (the latter for installing missing stuff and searching for help when it doesn't-doesn't). However, it can be very useful to keep simulation capabilities in place even when everything works with real hardware.

* There might be only a single and expensive device around and your colleagues are using it at present for a longer measurement campain. With a simulated device you may happily continue coding.
* Suppose that the experiment has a surprising result. Being able not only to simulate some default values but the outcome of an entire experiment while using the whole chain from the simulated device over PyMoDAQ's functionality to data treatment software may be a very helpful quality control. Is the result surprising because a mistake happended during the experiment? during data acquision? during treatment? *or did we actually discover something new??*
* You may learn a lot about your experiment and its instrumentation by making sure that at the tail of the chain you are actually getting back exactly what you have put into the simulation at its head. Remember that the only way a sample can talk to you in a spectroscopic experiment is by means of signal photons it emits. These and the carried information pass by a multitude of optical elements, detectors, ADC converters, data treatment of all sorts etc. Quite some Chinese whispers to master.

We start with a controller situated in the hardware directory of the plugin (:file:`src/pymodaq_plugins_tutorial_extensions/hardware`). When using real hardware, an instance of the controller is managing the communication with the device. We use the same structure here because a couple of simulated intstruments will have to work together, sharing common "knowledge" about the state of the simulated experiment. We use here a :code:`dataclass` to avoid a lot of boiler plate code at initialisation.

.. code-block:: python

  import numpy as np
  from dataclasses import dataclass


  @dataclass
  class MockSpectrograph:

      integration_time: float = 50
      n_pixels: int = 1024
      readout_noise: int = 4
      dark_level: float = 150
      light_level: float = 500
      pe_per_lsb: float = 18.3
      adc_bits: int = 16
      wl_from: float = 300
      wl_to: float = 900
      absorption: float = 0.3

      def __post_init__(self):
	  self.calculate_base_data()

      def calculate_base_data(self):
	  n_pix = self.n_pixels
	  self.wavelengths = \
	      np.linspace(self.wl_from, self.wl_to, n_pix)
	  self.pixels = np.linspace(0, n_pix - 1, n_pix)
	  self.spectrum = \
	      np.exp(-((self.pixels - n_pix / 2) / (n_pix / 3))**4)
	  self.absorption = \
	      self.absorption \
	      * np.exp(-((self.pixels - n_pix / 4) / (n_pix / 8))**2)

The method :code:`__post_init__` is called from within the dataclass' :code:`__init__` method. :code:`calculate_base_data` initialises the part of the simulation which does not change with the state of the experiment. It should be called each time one of the parameters used in there have changed.

To test whether this works correctly we add some code at the end of the file which will be executed only when the file is directly called in a Python interpreter. It uses :code:`matplotlib` to visualise the calculated data.

.. code-block:: python

  if __name__ == '__main__':
      import matplotlib.pyplot as plt
      spectrograph = MockSpectrograph()

      plt.plot(spectrograph.wavelengths, spectrograph.spectrum)
      plt.plot(spectrograph.wavelengths, spectrograph.absorption)
      plt.legend(['light spectrum', 'absorption'])
      plt.show()

The result should look like

.. image:: simu-spect-abs.png

Next comes the method which is used to generate realistic spectroscopic data.
Without light exposure detectors show some dark signal which is mostly coming from thermal noise. Its amplitude is typically proportional to the integration time. It comes in units LSB (least significant bit), i.e. in increments of the ADC. In case the signal gets too strong, the ADC signal saturates at the highest possible value (e.g. 65535 for a 16 bit ADC). The simulated signal is therefore cut to that level. Together with the data we send a time stamp.

.. code-block:: python

    def simulate_spectrum(self, shutter_open: bool, sample: bool):
        data = np.random.normal(loc=self.dark_level * self.integration_time,
                                scale=self.readout_noise, size=self.n_pixels)

        max_adc = (1 << self.adc_bits) - 1
        data = np.where(data < max_adc, np.floor(data), max_adc)
        return data, time.time()

    ...

        dark, time_stamp = \
        spectrograph.simulate_spectrum(shutter_open=False, sample=False)
	plt.plot(dark)
	plt.title('dark')
	plt.show()


.. image:: background.png

Once the probe light passes through the sample cell, but with only solvent in it, it induces a photo current in the detector. Its level is given in number of collected photo electrons which is in turn porportional to the integration time. The property :code:`light_level` is in units of ADC LSB. The detected signal will exhibit a fluctuation due to counting statistics. Other types of fluctuations like a variation of the overall amplitude due to correlated thermal noise could be implemented here as well. For the purpose of demonstration we'll leave it at the level of counting (Poisson) statistics.

.. code-block:: python
   :emphasize-lines: 4-7

    def simulate_spectrum(self, shutter_open: bool, sample: bool):
        data = np.random.normal(loc=self.dark_level * self.integration_time,
                                scale=self.readout_noise, size=self.n_pixels)
        if shutter_open:
            light = self.spectrum * self.light_level * self.integration_time \
                * self.pe_per_lsb
            data += np.random.poisson(light) / self.pe_per_lsb
        max_adc = (1 << self.adc_bits) - 1
        data = np.where(data < max_adc, np.floor(data), max_adc)

        return data, time.time()

    ...

        data, time_stamp = \
	    spectrograph.simulate_spectrum(shutter_open=True, sample=False)
	plt.plot(data)
	reference = data - dark
	plt.plot(reference)
	plt.legend(['raw', 'dark subtracted'])
	plt.show()
	


.. image:: reference.png

Finally, after having inserted an absorbing sample, the intensity of the light transmitted through the sample is reduced according to 

.. math::
   A(\lambda) = -\log_{10}\frac{I(\lambda)}{I_0(\lambda)}

where :math:`A(\lambda)` is the absorption measured as optical density, :math:`I(\lambda)` is the light transmitted througth the sample and :math:`I_0(\lambda)` the intensity of the light transmitted through pure solvent.

.. code-block:: python
   :emphasize-lines: 7,8

    def simulate_spectrum(self, shutter_open: bool, sample: bool):
        data = np.random.normal(loc=self.dark_level * self.integration_time,
                                scale=self.readout_noise, size=self.n_pixels)
        if shutter_open:
            light = self.spectrum * self.light_level * self.integration_time \
                * self.pe_per_lsb
            if sample:
                light *= np.pow(10, -self.absorption)
            data += np.random.poisson(light) / self.pe_per_lsb
        max_adc = 1 << self.adc_bits
        data = np.where(data < max_adc, np.floor(data), max_adc)

        return data, time.time()

    ...

	data, time_stamp = \
	    spectrograph.simulate_spectrum(shutter_open=True, sample=True)
	plt.plot(data)
	signal = data - dark
	plt.plot(data - dark)
	plt.legend(['raw through sample', 'dark subtracted'])
	plt.show()

The resulting spectra should look like follows

.. image:: raw.png

And finally, calculating the absorption from these data, the resuét should look like

.. code-block:: python

	plt.plot(-np.log10(signal / reference))
	plt.title('absorption')
	plt.show()

.. image:: absorption.png
