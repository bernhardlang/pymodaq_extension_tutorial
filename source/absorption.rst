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
 
        def adjust_actions(self):
            """Disable actions which need other actions to be performed first.
            A reference can only be taken when a background has been measured.
            Acquisition in absorption mode needs a reference (and therefore also
            a background).
            """
            if self.measurement_mode == RAW:
                self._actions["acquire"].setEnabled(True)
                self._actions["background"].setEnabled(False)
                self._actions["reference"].setEnabled(False)
            if self.measurement_mode == WITH_BACKGROUND:
                self._actions["acquire"].setEnabled(self.have_background)
                self._actions["background"].setEnabled(True)
                self._actions["reference"].setEnabled(False)
            if self.measurement_mode == ABSORPTION:
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
 
