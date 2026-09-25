Interlude I: Custom Application
===============================

As an interlude we'll have a look into writing a custom application instead of a custom extension. The principal difference between the correspoding Python classes is that :code:`CustomApp` does not have access to the functionality of the dashboard. Using a custom application makes sense when controlling a dedicated instrument or arrangement which is used by people not being familar with the wealth of PyMoDAQ's functionality. It can be seen as a standalone extension.
