A simple absorption spectrometer
================================

The goal of this chapter is to turn the bare spectro-photometer into an absorption spectrometer. A full absorption measurement needs the determination of the detector's dark signal, to be subtracted from each acquisition, and a reference of the incident light. The corresponding workflow is 

#. Determine the dark using the corresponding shutter.
#. Ask the user to insert a sample with pure solvent.
#. Perform the absorption measurement with the real sample.

The spectrometer should have three working modes, display raw detector data, display background subtracted detector data and display absorption. The corresponding modes of operation are defined in the preamble of the extension script.

.. code-block::
   :emphasize-lines: 3-5

    CLASS_NAME = 'Absorption'

    RAW             = 0
    WITH_BACKGROUND = 1
    ABSORPTION      = 2

Since the noise on the background and the reference data enter into each subsequent absorption measurement, it can be of advantage to accumulate these data with a larger number of samples. Therefore, an additional averaging parameter is introduced for each of these measurements.

.. code-block::
   :emphasize-lines: 3-5,9-21

    class Absorption(CustomExt):

        measurement_modes = { 'Raw': RAW, 'Background Subtracted': WITH_BACKGROUND,
                              'Absorption': ABSORPTION }
        mode_names = list(measurement_modes.keys())

        ...

        application_params = [
            {'name': 'measurement_mode', 'title': 'Measurement Mode',
             'type': 'list', 'limits': list(measurement_modes.keys()),
             'tip': 'Measurement Mode'},
            {'name': 'back_averaging', 'title': 'Background Averaging',
             'type': 'int', 'min': 1, 'max': 1000, 'value': 100,
             'tip': 'Background Software Averaging'},
            {'name': 'ref_averaging', 'title': 'Reference Averaging',
             'type': 'int', 'min': 1, 'max': 1000, 'value': 100,
             'tip': 'Reference Software Averaging'},
        ]

        params = application_params + [
        ...

To make the values of these settings persistent, their storage has to be updated as well

.. code-block::
   :emphasize-lines: 3-5,9-

        def read_settings(self, qt_settings):
            ...
            for param in self.application_params:
                self.settings[param['name']] = \
                    qt_settings.value(param['name'], param['value'])

        def write_settings(self, qt_settings):
            ...
            for param in self.application_params:
                qt_settings.setValue(name, self.settings[param['name']])

The GUI has also to be updated. It is always a good idea to display all data entering into the final result in some fashion.  A simple mockup ofthe GUI of our extension shows the general idea.

.. image:: extension-mockup.png
   :align: center


.. code-block::

    def setup_docks(self):
        ...
 
        # plot for raw spectrum data and reference 
        raw_data_dock = Dock('Raw Data')
        self.docks['raw-data'] = \
            self.dockarea.addDock(raw_data_dock, "bottom",
                                  self.docks['settings'])
        raw_data_widget = QWidget()
        self.raw_data_viewer = Viewer1D(raw_data_widget)
        self.raw_data_viewer.toolbar.hide()

        raw_data_dock.addWidget(raw_data_widget)

        # plot for background 
        background_dock = Dock('Background')
        self.docks['background'] = \
            self.dockarea.addDock(background_dock, "bottom",
                                  self.docks['raw-data'])
        background_widget = QWidget()
        self.background_viewer = Viewer1D(background_widget)
        background_dock.addWidget(background_widget)
        self.background_viewer.toolbar.hide()

Two new actions are needed

.. code-block::
   :emphasize-lines: 10-15,23-

    class Absorption(CustomExt):

    ...
 
        def setup_actions(self):
            self.add_action('acquire', 'Acquire', 'run2',
                            "Acquire", checkable=False, toolbar=self.toolbar)
            self.add_action('stop', 'Stop', 'stop2',
                            "Stop", checkable=False, toolbar=self.toolbar)
            self.add_action('background', 'Take Background', 'brightness_3',
                            "Take Background", checkable=False,
                            toolbar=self.toolbar)
            self.add_action('reference', 'Take Reference', 'lightbulb',
                            "Take Reference", checkable=False,
                            toolbar=self.toolbar)
        self._actions["stop"].setEnabled(False)

After deleting again the gui settings file, the extension should now look like

.. image:: absorption-extension.png

If you need icons which are not present in the icon library (:file:`pymodaq_gui/resources/icon_library`), you'll have to select suitable ones at https://fonts.google.com/icons. To be able to add them to PyMoDAQ's icon library you have to fork the PyMoDAQ repository, add the icons' names to the list in :file:`pymodaq_gui/resources/icons.toml` and follow the instructions in there and in :file:`pymodaq_gui/resources/check_icons_dev.py`. After a pull request, the additional icons will be available to all PyMoDAQ users. During development it is sufficient to install the pymodaq_gui package in editable mode within your work environment.

The newly defined actions do not yet trigger any real operations. 

.. code-block::

    class Absorption(CustomExt):

    ...

        def take_data(self, data: DataToExport):
            spectro_data = data.get_data_from_dim('Data1D')[0]
            self.n_samples = self.accumulate_data(spectro_data[0], self.n_samples)
            if self.n_samples < self.n_average:
                return

            if self.n_average < 2:
                self.spectrum_viewer.show_data(spectro_data)
                return

            current_mean, current_error = \
                self.average_data(self.sum_data, self.squares_data,
                                  self.n_samples)
            self.n_samples = 0

            if self.settings['measurement_mode'] == 'Raw':
                self.show_data(current_mean, current_error, 'raw')
                return

            mean_signal = current_mean - self.background
            error_signal = np.sqrt(current_error**2 + self.error_background**2)

            if self.settings['measurement_mode'] == 'Background Subtracted':
                self.show_data(mean_signal, error_signal, 'signal', current_mean)
            else: # self.settings['measurement_mode'] == ABSORPTION:
                valid_mask = \
                    np.logical_and(mean_signal > 0, self.reference_valid_mask)
                self.absorption = \
                    np.where(valid_mask,
                             -np.log10(mean_signal / self.reference), 0)
                self.error_absorption = \
                    1 / np.log(10) \
                    * np.sqrt((current_error / mean_signal)**2
                              + ((self.error_reference + self.error_background)
                                 / self.reference)**2
                              + (1 / mean_signal - 1 / self.reference)**2
                                * self.error_background)

                self.show_data(self.absorption, self.error_absorption, 'absorption',
                               current_mean, self.reference)

        def show_data(self, mean, error, name, raw=None, reference=None):
            dfp = DataFromPlugins(name=name, data=[mean, error], dim='Data1D',
                                  labels=[name, 'error'], axes=[self.x_axis])
            self.spectrum_viewer.show_data(dfp)
            if raw is not None:
                data = [raw]
                labels = ['raw signal']
                if reference is not None:
                    data.append(reference)
                    labels.append('reference')
                dfp = DataFromPlugins(name='raw', data=data, dim='Data1D',
                                      labels=labels, axes=[self.x_axis])
                self.raw_data_viewer.show_data(dfp)
            self.show_data(mean, error, 'raw')


.. code-block::

    class Absorption(CustomExt):

    ...

        def take_background(self):
            """Grab one background spectrum."""

            if hasattr(self.detector.controller, "set_shutter_value"):
                self.detector.controller.set_shutter_value('dark', 0)

            self.n_back = self.settings.child('back_averaging').value()
            for i in range(0, self.n_back):
                background,timestamp = self.detector.controller.grab_spectrum()
                self.accumulate_data(background, i)
            self.background, self.error_background = \
                self.average_data(self.sum_data, self.squares_data, self.n_back)
            self.have_background = True
            self.adjust_actions()
    # temp, move this to take data, recover wl from plugin data
            dfp = DataFromPlugins(name='Spectrograph',
                                  data=[self.background, self.error_background],
                                  dim='Data1D', labels=['background', 'error'],
                                  axes=[self.x_axis])
            self.spectrum_viewer.show_data(dfp)
            self.background_viewer.show_data(dfp)
            if hasattr(self.detector.controller, "set_shutter_value"):
                self.detector.controller.set_shutter_value('dark', 1200)

        def take_reference(self):
            """Grab one reference spectrum."""

            self.detector.controller.with_sample = False

            self.n_ref = self.settings.child('ref_averaging').value()

            for i in range(0, self.n_back):
                reference,timestamp = self.detector.controller.grab_spectrum()
                self.accumulate_data(reference, i)
            self.reference, self.error_reference = \
                self.average_data(self.sum_data, self.squares_data, self.n_ref)
            self.reference -= self.background

            self.reference_valid_mask = self.reference > 0
            self.have_reference = True
            self.adjust_actions()
            dfp = DataFromPlugins(name='Spectrograph',
                                  data=[self.reference, self.error_reference],
                                  dim='Data1D', labels=['reference', 'error'],
                                  axes=[self.x_axis])
            self.spectrum_viewer.show_data(dfp)
            dfp = DataFromPlugins(name='Spectrograph',
                                  data=[self.reference, self.error_reference],
                                  dim='Data1D', labels=['reference', 'error'],
                                  axes=[self.x_axis])
            self.raw_data_viewer.show_data(dfp)

            self.detector.controller.with_sample = True

.. code-block::

    class Absorption(CustomExt):

    ....

        def connect_things(self):
            self.connect_action('acquire', self.start_acquiring)
            self.connect_action('stop', self.stop_acquiring)
            self.connect_action('background', self.take_background)
            self.connect_action('reference', self.take_reference)


.. code-block::

    class Absorption(CustomExt):

    ....

        def adjust_actions(self):
            """Disable actions which need other actions to be performed first.
            A reference can only be taken when a background has been measured.
            Acquisition in absorption mode needs a reference (and therefore also
            a background).
            """
            if self.settings['measurement_mode'] == RAW:
                self._actions["acquire"].setEnabled(True)
                self._actions["background"].setEnabled(False)
                self._actions["reference"].setEnabled(False)
            if self.settings['measurement_mode'] == WITH_BACKGROUND:
                self._actions["acquire"].setEnabled(self.have_background)
                self._actions["background"].setEnabled(True)
                self._actions["reference"].setEnabled(False)
            if self.settings['measurement_mode'] == ABSORPTION:
                self._actions["acquire"].setEnabled(self.have_reference)
                self._actions["background"].setEnabled(True)
                self._actions["reference"].setEnabled(self.have_background)



.. code-block::

    class Absorption(CustomExt):

    ...
 
    def adjust_operation(self):
        """Stop acquisition if background / reference is missing but needed"""
        if self.measurement_mode < WITH_BACKGROUND:
            dock_title = "Raw Data"
        else:
            dock_title = "Absorption" if self.measurement_mode == ABSORPTION \
                else "Background Subtracted Data"
            if not self.have_background:
                self.detector.stop()
            elif self.measurement_mode == ABSORPTION and not self.have_reference:
                self.detector.stop()

        self.spectrum_label.setText(dock_title)

.. code-block::

    class Absorption(CustomExt):

    ...
 
        def adjust_operation(self):
            """Stop acquisition if background / reference is missing but needed"""
            if self.measurement_mode < WITH_BACKGROUND:
                dock_title = "Raw Data"
            else:
                dock_title = "Absorption" if self.measurement_mode == ABSORPTION \
                    else "Background Subtracted Data"
                if not self.have_background:
                    self.detector.stop()
                elif self.measurement_mode == ABSORPTION \
                  and not self.have_reference:
                    self.detector.stop()

            self.spectrum_label.setText(dock_title)

.. code-block::

    class Absorption(CustomExt):

    ...
 
        def connect_things(self):
            self.connect_action('save', self.save_current_data)
            self.connect_action('show', self.show_detector)
            self.connect_action('acquire', self.start_acquiring)
            self.connect_action('stop', self.stop_acquiring)
            self.connect_action('background', self.take_background)
            self.connect_action('reference', self.take_reference)

.. code-block::

    class Absorption(CustomExt):

    ...
 
        def setup_menu(self, menubar: QtWidgets.QMenuBar = None):
            file_menu = self.mainwindow.menuBar().addMenu('File')
            self.affect_to('save', file_menu)
            file_menu.addSeparator()
            #self.affect_to('quit', file_menu)
 
