.. _auh2:

AuH2
----

In the folder `Inelastica/Examples/AuH2` you will find a folder `L9.68` and a Python script ``AuH2-setup.py``. The folder contains two subfolders

* *CGrun* is a standard SIESTA geometry optimization run for a H2 molecule in a bridge configuration connected to 1D Au wires.

* *TSrun* is a standard `TranSIESTA`_ setup with the H2 molecule between two semi-infinite Au leads.

In both cases the folder name `L9.68` refers to an electrode separation (between fixed Au atoms) of 9.68 Ang.

To demonstrate the methods of Inelastica you can try to run the following jobs in the listed order.

geom2geom
~~~~~~~~~

Go into the `L9.68/CGrun` folder and try:

.. code-block:: bash

    geom2geom RUN.fdf struct3.xyz -x 3 -y 2 -z 1

This will convert the geometry from the *fdf* format to an *xyz* file repeated 3x2x1 times. ``geom2geom`` can handle *xyz*, *fdf*, *ANI*, *XV* etc files.

The command:

.. code-block:: bash

    geom2geom --help

tells you how to use the script.


CGrun
~~~~~

The first task is to stretch the geometry from `L9.68` to a new electrode separation of 10.0 Ang (e.g., with a folder name `L10.00`).

To do this make sure that

.. code-block:: bash

    CG = True
    FC = False
    OS = False
    TS = False

in ``AuH2-setup.py``. Now simply type

.. code-block:: bash

    python AuH2-setup.py

and the script will

* make a new `L10.00/CGrun` folder
* read the *L9.68/CGrun/Chain.XV* geometry and stretch to 10.0 Ang
* write ``RUN.pbs`` script and submit to queue


FCrun/OSrun
~~~~~~~~~~~

The next task is to perform finite displacements of a selected group of atoms to determine vibrational frequencies, modes, and e-ph couplings.

To do this make sure that

.. code-block:: bash

    CG = False
    FC = True
    OS = True
    TS = False

in ``AuH2-setup.py``. Now simply type

.. code-block:: bash

    python AuH2-setup.py


TSrun
~~~~~

To run `TranSIESTA`_ for the stretched wire geometry (`L10.00`) use

.. code-block:: bash

    CG = False
    FC = False
    OS = False
    TS = True


PHrun
~~~~~

To use the data from the `FCrun/OSrun` folders to determine the vibrational frequencies, modes, and e-ph couplings, you can execute on the command line:

.. code-block:: bash

    Phonons -c -F 9 -L 14 --FCfirst=11 --FClast=12 PHrun

To learn about the different flags, try

.. code-block:: bash

    Phonons -h


EigenChannels
~~~~~~~~~~~~~

Go into the `TSrun` folder:

.. code-block:: bash

    EigenChannels EC

will create a new subdirectory `EC` and calculate the eigenchannel scattering states. They are saved in the macu file format which can be read by molekel (old version?). Other file formats may be specified with the `--cube=...` option, see ``EigenChannels --help``. Support for more file formats can be found by downloading the `last release <releases_>`_. In addition, you will get *.xmgr* files that can be plotted in xmgrace. Also try:

.. code-block:: bash

    EigenChannels --help

to find more options.


IETS calculations
~~~~~~~~~~~~~~~~~

Go into the `TSrun` folder:

.. code-block:: bash

    Inelastica -F 10 -L 15 -p ../PHrun/Output.nc --LOEscale=0.0 INrun

will create a new subdirectory `INrun` and calculate the IETS spectra. The results are saved in a netCDF file which can be examined by the ``ncdump`` command or plotted in xmgrace (compiled with netCDF support).

To learn about the different flags, try

.. code-block:: bash

    Inelastica -h

You can compare your results to :download:`this netCDF output file <results/AuH2_L10.00_Chain.EC.Tot.IN.nc>`.
