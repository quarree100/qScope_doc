.. _devtools:

Debugging and Devtools
######################

There is a set of debugging- and developer tools that can facilitate the programming of frontend elements in ``q100viz/devtools.py``.
To begin with, there is a bunch of :ref:`startup flags<frontend_startup_flags>` and :ref:`key input events<key_events>` can be used to switch modes and change the slider values.

.. _verbose_mode:

Verbose Mode
************

There is a verbose mode, that will display additional information right on the interface, and print more information to the console. The verbose mode can be activated using the startup flag ``-v`` or toggled by pressing ``v``. Its status is stored in the ``session.VERBOSE_MODE`` bool.
In verbose mode, buildings will be colored according to their heat consumption, not by their connections status. You'll see the slider values displayed next to the slider and an 'x' indicating the position where the physical slider should be (if not, it is not :ref:`positioned properly<frontend_slider_setup>`). When verbose mode is on, API messages sent to the infoscreen will be printed to the console.

Functions
*********

* ``print_verbose``: if this function is used to print information to the console, the information will be written to the log_, too.
* ``mark_random_buildings_for_simulation`` takes the global list of buildings and selects N random buildings to force-connect them to the heat grid, set a date for refurbishment or enable energy saving options. This function can be enabled using the ``--select_random`` startup flag. It will then be called shortly before the simulation is run.
* ``mark_buildings_for_simulation``: like above, but options can be set for specific buildings, chosen by their ID
* ``print_full_df``: prints a full pandas DataFrame, not only its head and tail (pandas default)

.. _log:

Log
***

``session.log`` is a string that logs some Runtime Errors and API messages. When the frontend application terminates, the log file is written to the frontend folder.

.. important::
  The log file will only be written, if the program is closed properly (e.g. Alt+F4, do not close the application using Ctrl+C KeyboardInterrupt.)

.. _interactive_programming:

Interactive Programming
***********************

In the ``q100viz/sandbox`` folder there are some files for testing specific parts of the code, and scripts to pre-calculate data (to compare them with the simulation results, for example). For interactive programming we used some Jupyter Notebook Scripts and load the ``q100viz.session`` so the Buildings data can be processed without the pygame running in the loop.

In ``q100viz/sandbox/simulation_demox.ipynb``, a batch of simulations is conducted, without loading the pygame frontend:

.. code-block:: python
  :caption: q100viz/sandbox/simulation_demox.ipynb

  # --------------- import q100viz libraries we want to use: ----------
  path_to_frontend = '..'
  os.chdir(path_to_frontend)
  print('current working directory:', os.getcwd()) # shall be q100viz root folder
  import q100viz.session as session
  import q100viz.devtools as devtools
  import q100viz.graphics.graphs as graphs

  # ------------------------- prepare buildings: ----------------------
  session.buildings.df['selected'] = False
  session.buildings.df['group'] = -1

  # ------------------------- prepare simulation: ---------------------
  devtools.mark_random_buildings_for_simulation(session.buildings.df, 4, connection_to_heat_grid=random.randint(2020,2040), refurbished=True)

  # ------------------------- start simple simulation: ----------------
  session.simulation.setup(input_max_year=2045)
  session.simulation.run()

  # ------------------------- test batch-simulation: ------------------
  session.VERBOSE_MODE = True

  # 1. 2 specific + 2 random buildings, no decisions
  devtools.mark_buildings_for_simulation(session.buildings.df, ['5.11', '6.06'])
  devtools.mark_random_buildings_for_simulation(
              session.buildings.df, 2)
  session.simulation.setup(input_max_year=2022)
  session.simulation.run()

  # 2. same buildings, connection=2022
  devtools.mark_buildings_for_simulation(
              session.buildings.df[session.buildings.df['selected'] == True], 4, connection_to_heat_grid=2021)
  session.simulation.setup(input_max_year=2022)
  session.simulation.run()

  # 2. same buildings, connection=2022 + refurbished
  devtools.mark_buildings_for_simulation(
              session.buildings.df[session.buildings.df['selected'] == True], 4, connection_to_heat_grid=2021, refurbished=True)
  session.simulation.setup(input_max_year=2022)
  session.simulation.run()

  # 2. same buildings, connection=2022 + save_energy
  session.buildings.df['refurbished'] = False  # reset refurbishment

  devtools.mark_buildings_for_simulation(
              session.buildings.df[session.buildings.df['selected'] == True], 4, connection_to_heat_grid=2021, save_energy=True)
  session.simulation.setup(input_max_year=2022)
  session.simulation.run()

.. _frontend_calibration:

Calibration
***********

The projector is positioned above the table and will cast a distorted image onto the physical table. Thus, some calibration has to be done for the projection to match the table dimensions.

Frontend Calibration
====================

#. The calibration mode can be entered using the ``c`` key.
#. Four corner points are shown as white rectangles, one of which is filled white and , by that, marked active.
#. The active corner point can be moved using the arrow keys. Make sure these rectangles are positioned at the edges of the physical table, and that the line connecting these points does not leave the table extents.
#. The next corner point can be selected using ``TAB``.
#. The magnitude of moving the corner points can be toggled pressing ``SPACEBAR``. The lines connecting the corner points will be blue for big steps and red for small steps.
#. Eventually, the settings can be saved to pressing ``s``. This stores a ``keystone.save`` file to the frontend folder that will automatically be loaded next time upon startup.

.. hint::
  Combining the calibration view with the grid view (``g``) can be helpful to also check, if the physical grid is placed properly.

.. note::
  Make sure the :ref:`backend is calibrated<cspy_calibration>` properly, too!

.. _keystone_transformation:

keystone transformation
=======================

.. note::

  general information on image transformation using opencv:

  `tutorial_py_geometric_transformations <https://docs.opencv.org/3.4/da/d6e/tutorial_py_geometric_transformations.html>`_

  `using cv.perspectiveTransform for vectors <https://docs.opencv.org/3.4/d2/de8/group__core__array.html#gad327659ac03e5fd6894b90025e6900a7>`_
  and `cv.warpPerspective for images <https://docs.opencv.org/3.4/da/d54/group__imgproc__transform.html#gaf73673a7e8e18ec6963e3774e6a94b87>`_

Transformation Example
----------------------

**adding a new surface, draw on it and transform it:**

.. code-block:: python
  :caption: q100viz/keystone.py

  class SomeClass:
    # session.canvas_size = 1920, 1080
    self.surface = keystone.Surface(session.canvas_size, pygame.SRCALPHA)

    # x_size, y_size = 22, 22
    self.surface.src_points = [[0, 0], [0, y_size], [x_size, y_size], [x_size, 0]]
    self.surface.dst_points = [
        [config['X1'], config['Y1']],
        [config['X1'], config['Y2']],
        [config['X2'], config['Y2']],
        [config['X2'], config['Y1']]]
    # where e.g. X1 = 0, X2 = 50, Y1 = 0, Y2 = 81.818 (%)

    def draw(self, viewport):

      pygame.draw.polygon(self.surface, pygame.Color(255, 255, 255), [[20, 70], [20, 20], [80, 20], [80, 70]])  # render polygon

      viewport.blit(self.surface, (0,0))  # cast it to viewport

slider transformation
---------------------

* slider uses the transformation of the :ref:`grid<grid>`
* **drawing of polygons and values** should be done via ``self.surface.blit(...)``. Slider surface is rendered and "blitted" to main canvas.

``print(slider.coords_transformed)`` returns:

.. code-block::
  :caption: slider.coords_transformed

  [[860.9641723632812, 915.1583862304688],
  [863.9833984375, 614.8511352539062],
  [1228.917724609375, 622.6510009765625],
  [1226.5196533203125, 923.7374267578125]]

with ``[[bottom-left[x], bottom-left[y]], [upper-left[x], upper-left[y]], [upper-right[x], upper-right[y]], [bottom-right[x], bottom-right[y]]]``
