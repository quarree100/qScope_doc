.. _frontend_usage:

Usage
########

The Q-Scope frontend, as used in the project QUARRE100, was programmed using `pygame <pygame.org>`_ - a set of Python modules designed for writing video games.
In this chapter, we will first handle the frontend's installation and afterwards explain its features by going through the code (more or less linearly).

.. _frontend_installation:

Installation
************

After `cloning <https://github.com/git-guides/git-clone>`_ the frontend's repository, you can simply install all required python modules by executing ``pip install -r requirements.txt``. (If the requirements.txt file is incomplete, and you run into problems, just install whichever module is missing via ``pip3 install [MODULE]``.)
Simply run the script via ``python3 run_q100viz.py``

File Structure
==============

.. code-block::

  qScope_frontend
  └───q100viz
      └───graphics
      |   contains tools for graph creation and image handling
      └───interaction
      |   "gameplay" modes and interaction with sliders
      └───settings
  └───run_q100viz.py

Run
***
The main script is called ``run_q100viz.py``. You can start it from the qScope_frontend folder by running ``python3 run_q100viz.py``. Some flags can be set to enable debug options:

.. _frontent_startup_flags:

  * ``--select_random [NUMBER_OF_HOUSES]``: preselects n houses to opt in for a connection to the heat grid
  *  ``--verbose``: activates verbose mode to print more information to console
  *  ``--simulate_until [NUMBER_OF_YEAR]``: run the gama-simulation until the given year. last simulation step will be last day before entered year.
  *  ``--connect``: force buildings to opt in for "connection_to_heat_grid"
  *  ``--refurbish``: force buildings to opt in for "refurbish"
  *  ``--save_energy``: force buildings to opt in for "save_energy"
  *  ``--start_at``: force the frontend to start at a certain :ref:`mode <frontend_mode>`
  *  ``--test``:
  *  ``--main_window``: forces the projection canvas to pop up at the current monitor.
  *  ``--research_model``: There are two simulation files provided. If specified, the research model will be used for simulation (rather than the workshop model)

These flags are specified in the first section of the main script.
At the end of the main script, an instance of the :ref:`frontend<frontend_class>` is created and run in the loop.

.. _frontend_pygame_setup: