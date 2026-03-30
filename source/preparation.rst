Preparing the stage
===================

Following a brief introduction to the essential tools, this initial section is devoted to a recap on how to set up the module's folder structure.

Used tools
----------

A couple of tools will be used in this tutorial. To avoid cluttering-up the text, no special introduction will be given to these tools here.

* Git and github
* Conda / venv
* No special editor or IDE is needed. PyCharm is a good choice since PyMoDAQ developers use it a lot.
* pde (the built-in Python Debugger) or the debugger built into PyCharm


Setting up the plugin skeleton
------------------------------

In this section we prepare the module structure which is needed for PyMoDAQ to recognise the instrument plugins and the extension. We start by cloning the plugin template repository on github. Note that the route taken here does not build a fork of an existing repository. We're going to develop something new, right?::

  $ cd /path/to/extension-plugin-code

Then clone the template repository::

  $ git clone https://github.com/PyMoDAQ/pymodaq_plugins_template.git
  Cloning into 'pymodaq_plugins_template'...

We'll call the extension 'Absorption' and give the repository the corresponding name::

  $ mv pymodaq_plugins_template pymodaq_plugins_absorption
  $ cd pymodaq_plugins_absorption
  $ mv src/pymodaq_plugins_template src/pymodaq_plugins_absorption

Furthermore, we're not going to need the templates for 0D and 2D viewer plugins, neither for custom applications and models. Let's delete the corresponding template files because they would show up in the dashboard's plugin list. However, don't delete the folders and the __init__.py files in there because at some later point you may want to add real plugins there. The now deleted template code can easily be retrieved from github in case.

Being a git clone of a template repository, it still is linked to the history of that template module which is probably not what we want. Its is better to make a fresh start by deleting git's repository folder and re-initialising git::

  $ rm -rf .git
  $ git init
  $ git add -A
  $ git commit -m "initial repository commit"

If you plan to develop multiple plugins, you may also tar or zip the project folder right after having deleted the .git folder and keep the material for re-use later on.

My preferred code editor (being a keyboard player) makes backup copies of files being changed by appending a tilde to the file name. I therefore add the line::

  *~

at the bottom of .gitignore. Yo may add similar matter to prevent git considering temporal files which are not supposed to enter into the repository.

Let's inspect the root folder::

  $ ls
  hatch_build.py	icon.ico  LICENSE  pyproject.toml  README.rst  src  tests

A couple of files in there should be modified.

* LICENSE: You may want to add your name to the Copyright (c) statement.
* pyproject.toml: Here goes main information for the installation procedure. The code is pretty self-explaining. Just fill in the blanks. The package-url depends on whether you want to host your package on your own repository or you plan to get it hosted on the PyMoDAQ repository. Don't worry too much about it at this time. It is a good idea to delete the comments telling what to do once you did it. We'll leave the features as they are for the moment, announcing only instruments, because that is what we are going to do as a first step. The extension comes later.
* README.rst: This file is intended to give the user of your plugin the necessary information to install and to run it. Have a look into other plugin repositories to get an idea of what is typically provided here and how.

That's it already for the root folder and the book-keeping matter in there. We can go on working on python sources now.

Preparing the environment
.........................

As a last step before diving into coding we have to set up a virtual environment which we are going to use during the development of our plugin. Due to the fast evolution of Python packages, keeping track of compatible versions is a quite important issue. Virtual environments are used as containers which isolate parallel installations using different and potentially incompatible versions of packages. In the past, conda was used by the PyMoDAQ community. Licence issues have changed that. Until things have settled, the Python-onboard package venv can be used::

  $ python -m venv /path/to/environment
  $ source /path/to/environment/bin/activate
  $ pip install pymodaq pyside6

As path you should choose a sensible name for your project. The subfolders created in the environment folder contain links to the python interpreter to be used and will receive all packages installed by pip once the environment is activated. Some prefer to have a separate evironment for each package / plugin to be developped. Different environments are needed at least when working with different versions of some packages in parallel. But don't worry, new environments can always be set up later in case of incompatibilites are coming up. 

The activation will add the environment's bin directory to the head of the current search path so that the binaries therein will be found before any others with the same name. If you whish to use a specific pythom version, say 3.12, call::

  $ python3.12 -m venv /path/to/environment

That version must of course have been previously installed on your computer. After the activation, don't worry when Python complains about missing packages which you know are being already present on your system. Placing the python interpreter into an environment actually makes it looking like a pristine installation of the bare interpreter without any additional packages. The third of the above lines makes sure that all packages needed for PyMoDAQ are getting installed in the needed versions.

To try whether things work as supposed, let's start the dashboard::

  $ python -m pymodaq.dashboard

The dashboard should start up. However, in the preset manager, no new plugins are visible. That is all right because we haven't installed the plugin module yet. This has to happen in the so-called editable mode. To this end, change to the root directory of your plugin, where e.g. the file pyproject.toml is to be found and type::

  $ pip install -e .

This tells the python installer to incorporate the module under development into the local site-packages. Watch out for the dot after the -e. Depending on the version of pip coming with your operating system, it might be necessary to first update pip::

  $ python -m pip install pip --upgrade

When we fire up the dashbord now and start the preset-manager, a new item 'DAQ0D/Template' should appear in the Detectors-Add drop-down list.

.. image:: preset-tutorial.png
	   
A few words on the conda environments may be worth mentioning here.

* A commen problem in Linux environments is that python comes already with the system installation and is used for various tasks. However, no environments are set up by default. This may screw up things when python packages are installed as superuser but not using the system's package manager (like apt for Debian). The latter and the work of the distribution maintainers take care of version issues. However, when using sudo pip install ... you are on your own and may not have been aware of possible problems when having done such in the past. 
* PyCharm seems to have some difficulties in coping with already existing conda environments. After setting everything according to the instructions on the JetBrains website, the prompt in the terminal may show that instead of the chosen environment, PyCharm has actually activated the base environment. Supposedly missing packages are a good indication that this happened. Creating new environments from inside PyCharm seems to work more reliably.
