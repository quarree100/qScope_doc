.. _frontend_programming:

Programming
###########

.. _frontend_initialization:

Initialization: Files, paths & data
***********************************

Config file
===========

To configure the frontend, the file ``q100viz/settings/config.py`` can be changed before running the application.

.. note:: Make yourself a copy of the file config.py and adjust it according to the context you want to use the setup. We had different scenarios for different workshop participants.

Here you can modify the path of files being imported/exported, :ref:`change the source file and the simulation time for the agent-based-model<simulation_setup>`, save information of the extents of the :ref:`grids<frontend_grid_setup>` and the :ref:`sliders<frontend_slider_setup>` here. The individual adjustments will be discussed in the according sections, respectively.

.. _session:

Session file
============

The file ``q100viz/session.py`` is a container for global variables. The file does some initial variable assignment, so that we won't stumble across any NoneType errors later on. Some config variables will be stored here with more accessible variable names (rather than having to use python dictionaries all the time). You'll basically find anything that needs to be referred to from different parts throughout the code here, such as :ref:`variables used for Debugging<devtools>`.
In the following graphics section, the pygame viewport is loaded and :ref:`keystone transformation<keystone_transformation>` is perfomed.
Furthermore, a ``buildings`` variable is created as an instance of the :ref:`Buildings class<buildings>`, which represents the houses the users can interact with. It is a DataFrame that is very commonly referred to in the project, as it stores polygon data and metadata of the GIS objects.

.. _environment:

Further contents of the code are:
- The ``environment`` variable that stores information about the machine state, like the current game mode and iteration, global scenarios regarding the energy prices, current state of houses connected to the heat grid. It is used mostly to transfer data to the infoscreen.

.. _scenario:

- A scenario for possible energy prices and further settings affecting the dynamics of the agent-based-model are set here as well. The source csv files for this lie in the :ref:`data folder<data>`.
- Initialization of  GIS objects, such as the geographic canvas extents and the basemap file are initialized
- Initialization of the :ref:`grid<grid>` objects. These are the cells representing the physical tiles on the table. They mirror the physical interaction and can be addressed by a grid object that is sent from the :ref:`The tag decoder software<cspy>` tag decoder at each interaction.
- Initialization of the _modes. The different game stages are stored in a variable called ``modes``.

.. _buildings:

Buildings & Metadata
====================

How are the buildings you see on the map colored and how can they be accessed? Most of the interaction in the QUARREE100 project is done via selection of buildings on an aerial map. The source data for these building polygons comes from a :ref:`Shapefile in the data folder<architecture>`

The ``Buildings`` class is basically one large DataFrame containing all metadata about the buildings taken from a shapefile. The source file contains information about the houses addresses, specific heat consumption, energy carrier, building type, etc.
Only existing buildings are regarded.
The buildings can be clustered in groups according to similar heat consumptions:

.. code-block:: python

      def make_clusters(self, start_interval):
        '''make groups of the selected buildings. group by standard deviation of energy consumption'''
        cluster_list = []
        for idx in range(len(self.df.index)):
            interval = start_interval  # standard deviation
            cluster = pandas.DataFrame()
            while len(cluster) < 2:  # make sure no building is alone
                cluster = self.df.loc[(
                        (self.df['energy_source'] == self.df.loc[
                            self.df.index[idx], 'energy_source'])
                        &
                        (self.df['spec_heat_consumption'] >= self.df.loc[self.df.index[idx],
                        'spec_heat_consumption'] - self.df['spec_heat_consumption'].std() * interval)
                        &
                        (self.df['spec_heat_consumption'] <= self.df.loc[self.df.index[idx],
                        'spec_heat_consumption'] + self.df['spec_heat_consumption'].std() * interval)
                        &
                        (self.df['spec_power_consumption'] >= self.df.loc[self.df.index[idx],
                        'spec_power_consumption'] - self.df['spec_power_consumption'].std() * interval)
                        &
                        (self.df['spec_power_consumption'] <= self.df.loc[self.df.index[idx],
                        'spec_power_consumption'] + self.df['spec_power_consumption'].std() * interval)
                    )]
                interval += 0.1  # increase range, try again if necessary

            cluster_list.append(cluster)
            devtools.print_verbose(
                "building {0} is in a group of to {1} buildings with similar specs:".format(self.df.index[idx], len(cluster)), session.VERBOSE_MODE, session.log)
            # devtools.print_verbose(cluster[['spec_heat_consumption', 'spec_power_consumption']].describe(), session.VERBOSE_MODE)

        return cluster_list

Further information such as paths for pre-generated graphics are added. The DataFrame will later comprise images exported by the :ref:`ABM<abm>` to be forwarded to and shown at the infoscreen.

.. note::

  "Behavior" data such as the connection to the QUARRE100-heat-grid, refurbishment of the house or energy-saving measures are pre-set in the following manner: ``false``, if house's energy_source (in source data) is not ``None``, else the house will come in pre-connected and refurbished.

Buildings can either be ``selected`` by a user or not. Selection is done if a cell is selected on the table (by placing a token physically). :ref:`The tag decoder software<cspy>` will detect any interaction with the table surface and forward the grid information to the frontend to be deciphered in the ``grid.py``: :ref:`read_scanner_data<read_scanner_data>` function.
The Buildings class contains additional functions, e.g. ``find_closest_heat_grid_line`` for graphical calculations and functions to organize, convert and export the DataFrame for specific needs.


.. _gis:

GIS: Shapes and Raster
======================

The file ``gis.py`` contains two classes:

1. **The GIS class** draws features from the source Shapefile, like polygons and lines, onto the canvas_. It provides functions to draw the whole polygon layer at once, color them in a certain style (e.g. according to heat grid connection status), etc.
2. **The Basemap class** initiates and warps the basemap image.

Positioning of the GIS layers is done during :ref:`initialization<session>` of the GIS class object, where the corner points of the ROI (region of interest) extent are set:

.. code-block:: python

  _gis = gis.GIS(
    config['CANVAS_SIZE'],
    # northeast          northwest           southwest           southeast
    [[1013631, 7207409], [1012961, 7207198], [1013359, 7205932], [1014029, 7206143]],
    viewport)

  basemap = gis.Basemap(
      config['CANVAS_SIZE'], config['BASEMAP_FILE'],
      # northwest          southwest           southeast           northeast
      [[1012695, 7207571], [1012695, 7205976], [1014205, 7205976], [1014205, 7207571]],
      _gis)

.. note::
  Some other ROIs we tested in QUARREE100 were:

  **smaller map extent:**

   _gis = session.gis = gis.GIS(canvas_size, [[1013578, 7207412], [1013010, 7207210], [1013386, 7206155], [1013953, 7206357]], viewport)

  **input area at left-hand side and placeholder for cameras at bottom:**

    _gis = session.gis = gis.GIS(canvas_size, [[1013554, 7207623], [1012884, 7207413], [1013281, 7206147], [1013952, 7206357]], viewport)

  **input area at right-hand side and placeholder for cameras at bottom:**

    gis = session.gis = gis.GIS(canvas_size, [[1013631, 7207409], [1012961, 7207198], [1013359, 7205932], [1014029, 7206143]], viewport)

.. _canvas:

Frontend Canvas & Display
*************************

The whole frontend was programmed using `pygame <pygame.org>`_ - a set of Python modules designed for writing video games. Pygame will create a graphical canvas, running in the loop, which will change its appearance according to user action.

.. _frontend_class:

Frontend Setup
==============

The frontend class itself is defined in ``q100viz/frontend.py``.
Upon initialization of the frontend class, the pygame environment is created. Things like the display framerate, window position etc can be set here.

.. _frontend_setup_window:

window position and size
------------------------

You can set the window's position using the os module:

.. code-block:: python
  :caption: frontend.py

    # window position (must be set before pygame.init!)
    if not run_in_main_window:
        os.environ['SDL_VIDEO_WINDOW_POS'] = "%d,%d" % (
            0, 2560)  # projection to the left

    # window size:
    canvas_size = session.config['CANVAS_SIZE']
    self.canvas = pygame.display.set_mode(canvas_size, NOFRAME)
    pygame.display.set_caption("q100viz")

For this setting, the monitors should be organized as follows:

.. image:: ../img/frontend_screen-position.png
  :align: center
  :alt: [Image of two schematic monitors, above each other and aligned left]

The canvas is masked by a layer that defines the margins of the region of interest (ROI). The following list of points defines the extent of a masking polygon:

.. code-block:: python
  :caption: frontend.py

    self.mask_points = [[0, 0], [85.5, 0], [85.5, 82], [0, 82], [0, -50],
                    [-50, -50], [-50, 200], [200, 200], [200, -50], [0, -50]]

Finally, a seperate thread for UDP observation is started. Each table ("grid") has a seperate communication thread. More about how communication between tag decoder, frontend and infoscreen works in the :ref:`Communication <frontend_communication>` section.

.. _frontend_game_loop:

Frontend Projection
===================

After :ref:`initialization<frontend_class>`, the frontend will run in a loop to update the projection, evaluate keyboard input, handle the :ref:`game modes<modeselector>` and process :ref:`slider events<slider_events>`.
Finally, ``pygame.display.update()`` is called to actually do what it says, and a ``pygame.time.Clock`` is updated, using the defined framerate. We used a framerate of 12 FPS, because this is the maximum framerate used in the :ref:`tag decoder<cspy>`.

.. _key_events:

The following key events are implemented in the `QUARREE100 <https://www.quarree100.de>`_ example project:

- ``p`` toggle the display of GIS polygons
- ``m`` toggle basemap visibility
- ``g`` toggle visibility of grid outline and cell ID, rotation, coordinates
- ``n`` toggle visibility of the heat grid
- ``b`` toggle the black mask on viewport
- ``3`` start buildings_interaction_ mode
- ``4`` start simulation_mode_
- ``5`` start individual_data_view_ mode
- ``6`` start total_data_view_ mode

Canvas Composition
==================

The frontend image is composed of a set of layers, which are rendered ontop of each other in the following order:

#. draw polygons to _gis.surface
#. draw grid outĺine to grid.surface
#. draw mask to session.viewport
#. draw basemap to frontend.canvas
#. draw mode-specific surface (what does this do?)
#. render GIS layer: _gis.surface to frontend.canvas
#. slider: draw polygons, icons and text to slider.surface
#. draw grid.surface to frontend.canvas
#. draw session.viewport to frontend.canvas

.. note::
  More notes on how to use simple pygame features can be found in the :ref:`Frontend/pygame section! <simple_pygame_features>`

Drawing polygons and lines using GIS shapes
-------------------------------------------

The GIS shapes are drawn using the functions of the custom GIS class:

- ``draw_linestring_layer``: draws GIS features as lines - in our case, the heating grid is drawn using lines.

the following functions are used to draw polygons from geographical data (and color them according to the selected feature):

- ``draw_polygon_layer``: simply draw polygons and fill them with a provided color
- ``draw_polygon_layer_bool``: draw polygons and fill them either in color A or B (true/false)
- ``draw_polygon_layer_float``: draw polygons and fill them with a color gradient with the end points of a float between 0 and 1

Using these functions, we can color buildings on the map and fill them with a color according to a certain attribute, e.g. mapping their relative heat consumption to a color gradient between red and green, or color them either green or red, when connected to the heat grid, or not.
The functions always use the entire :ref:`buildings-dataset<buildings>` as an input parameter and draws all contained polygons at the same time. They are regularly called in the :ref:`loop function<frontend_game_loop>` of the frontend:

.. code-block:: python
  :caption: frontend.py

  # draw GIS layers:
  if session.show_polygons:
      session._gis.draw_linestring_layer(
          self.canvas, session._gis.nahwaermenetz, (217, 9, 9), 3)
      session._gis.draw_buildings_connections(
          session.buildings.df)  # draw lines to closest heat grid

      # fill and lerp:
      if session.VERBOSE_MODE:
          session._gis.draw_polygon_layer_float(
            self.canvas, session.buildings.df, 0,
            (96, 205, 21),
            (213, 50, 21),
            'spec_heat_consumption')
      else:
          session._gis.draw_polygon_layer_bool(
              self.canvas, session.buildings.df, 0,
              (213, 50, 21),
              (96, 205, 21),
              'connection_to_heat_grid')

      # stroke black:
      session._gis.draw_polygon_layer_bool(
          self.canvas, session.buildings.df, 1,
          (0, 0, 0),
          (0, 0, 0),
          'connection_to_heat_grid')

      # stroke according to connection status:
      session._gis.draw_polygon_layer_bool(
          surface=self.canvas, df=session.buildings.df,
          stroke=1,
          fill_false=(0, 0, 0),
          fill_true=(0, 168, 78),
          fill_attr='connection_to_heat_grid')

      # color buildings if connection is not -1:
      # session.gis.draw_polygon_layer_connection_year(
      #     session.buildings.df,
      #     stroke=0,
      #     fill_true=(96, 205, 21),
      #     fill_false=(213, 50, 21),
      #     fill_attr='connection_to_heat_grid')

.. hint::
  Filling a polygon is done by applying a stroke width of 0.

When the buildings are set to be connected to the heat grid, a line is shown between the polygon and the closest heat grid line. This is basically a tangent to that heat grid line at the closest point and is calculated in the function ``GIS.draw_buildings_connections``.

Export Canvas to file
---------------------

We used to export the rendered canvas to a png file each frame (if changes are ready), to further use it on the :ref:`infoscreen<infoscreen>`. This is deprecated, but can be done via a code snippet like this:

.. code-block:: python
  :caption: insert this to frontend.py game loop to export the canvas to file:

  # export canvas:
    if session.flag_export_canvas:
        # create a cropped output canvas and export:
        temp = pygame.Surface((1460, 630))
        temp.blit(session.gis.surface, (0,0))
        temp = pygame.transform.rotate(temp, 270)
        pygame.image.save(temp, '../data/canvas.png')
        session.flag_export_canvas = False # has to be set True when changes are received from cspy

Drawing of Sliders
------------------

The sliders have a bool called ``show_text`` that, when ``True``, activates the display of the slider control texts. This variable can be used for the usage modes to define whether the slider controls shall be displayed.

.. _graphictools:

Graphic Tools
=============

Images
------

Pygame is able to load images onto Surface objects from PNG, JPG, GIF, and BMP image files. ``q100viz/graphics/graphictools.py`` contains an Image class that can be used to load and transform images according to the needs of the warped canvas.
Images just have to be initialized and warped, before they can be rendered to the surface.

.. code-block:: python
  :caption: example code for rendering images, see interface.py for comparison.

  # 1. load image:
  img = Image("images/piechart_disabled.tif")

  # 2. warp image:
  img.warp((1920, 1080))

  # 3. render image:
  surface.blit(img.image, (x,y))

.. _graphs:

Graphs
------

The frontend software creates graphs from the :ref:`simulation results<simulation_outputs>` using the matplotlib library. A toolkit is contained in ``q100viz/graphics/graphs.py``, providing the following functions:

* `export_individual_emissions`: exports specified column of csv-data-file for every iteration round to graph and exports png
* `export_individual_energy_expenses`: exports specified column of csv-data-file for every iteration round to graph and exports png
* `export_default_graph`: exports default data to graph with gray curve
* `export_compared_emissions`: exports all data for selected group buildings into one graph for total data view
* `export_neighborhood_emissions_connections`: creates a bar plot for the total number of connections to the heat grid with an overlaying line plot of total emissions
* `export_compared_energy_costs`: exports all data for selected group buildings into one graph for total data view
* `export_neighborhood_total_data`: exports specified column of csv-data-file for every iteration round to graph and exports png

and some helpful conversion functions used to get the right units:

* `GAMA_time_to_datetime`
* `grams_to_kg`
* `grams_to_tons`
* `rgb_to_hex`
* `rgb_to_float_tuple`


.. _grid:

Grid & Tiles
************

.. image:: ../img/grid_representations.png
  :align: center
  :alt: image of grid representations: photo of acrylic tiles, webcam stream from underneath, software representation in frontend

The grid objects are initialized in :ref:`frontend.py<frontend_communication>`. They are software representations of the physical grids' configuration and define how elements shown on the aerial map are to be displayed.

.. code-block:: python
  :caption: frontend.py

  for grid_, grid_udp in [[session.grid_1, grid_udp_1], [session.grid_2, grid_udp_2]]:
    udp_server = udp.UDPServer(*grid_udp, 4096)
    udp_thread = threading.Thread(target=udp_server.listen,
                                  args=(grid_.read_scanner_data,),
                                  daemon=True)
    udp_thread.start()

In the frontend code of our example, there are two grid objects, each representing a grid on one of the physical tables. Each of them starts a new thread to receive UDP messages with information on the grid cells' ids and their (absolute and relative) rotation

All cells have an ID that can be any number ranging from 0 (corresponding a tangible with a white underside) through 5 (codes on the underside). Once a cell gets an ID that is not 5 (white), it is considered to be "selected". As a result, :ref:`a broader frame<draw_simple_polygon_layer>` will be drawn around it (see image above). Then it can be addressed via one of the :ref:`sliders<frontend_slider_setup>`, information on the object will be displayed on the infoscreen, certain functions can be triggered upon selections, such as :ref:`mode <modeselector>` switching.

Some cells can be programmed to trigger additional events, like leaving the current :ref:`game mode<mode>`. This is done via tables in ``q100viz/settings/``. Read more on how to program link functions to cells :ref:`here<programming_cell_functions>`.

.. hint::
  The grid display can be toggled using the ``g`` key. In the upper left corner of each cell, the cell's ID is displayed. The number in the upper right corner represents the cell's current rotation.

.. _frontend_grid_setup:

Grid Setup
==========

The grid objects contain lists of cells, which can be addressed using enumeration routines:

.. code-block:: python
  :caption: access cells by iterating the grids

  # iterate grid:
  for grid in [session.grid_1, session.grid_2]:
      for y, row in enumerate(grid.grid):
          for x, cell in enumerate(row):
            # do cell operation

.. _grid_coordinates:

grid coordinates:
-----------------

The positions of the cells are stored in ``grid.rects_transformed``. This variable contains coordinates of the absolute pixel positions **after** their keystone-transformation on the canvas.

.. code-block:: python

  for i, (cell, coords) in enumerate(session.grid_1.rects_transformed):
      print("{0}: ({1}|{2}): {3}".format(i, cell.x, cell.y, coords))

  # returns:
  '''
  0: (0|0): [[134.9009246826172, 4.38118839263916], [134.4179229736328, 37.4811897277832], [167.75010681152344, 38.0572509765625], [168.22642517089844, 4.963389873504639]]
  1: (1|0): [[168.22642517089844, 4.963389873504639], [167.75010681152344, 38.0572509765625], [201.06971740722656, 38.633094787597656], [201.53933715820312, 5.545371055603027]]
  2: (2|0): [[201.53933715820312, 5.545371055603027], [201.06971740722656, 38.633094787597656], [234.37672424316406, 39.20872497558594], [234.8396759033203, 6.127132415771484]]
  3: (3|0): [[234.8396759033203, 6.127132415771484], [234.37672424316406, 39.20872497558594], [267.6711730957031, 39.78413391113281], [268.12744140625, 6.708674430847168]]
  4: (4|0): [[268.12744140625, 6.708674430847168], [267.6711730957031, 39.78413391113281], [300.9530334472656, 40.35932922363281], [301.4026184082031, 7.28999662399292]]
  5: (5|0): [[301.4026184082031, 7.28999662399292], [300.9530334472656, 40.35932922363281], [334.2223205566406, 40.934303283691406], [334.6652526855469, 7.871099472045898]]

  '''

Grid Interaction
================

The grid is either updated when interacting with a computer mouse (left- right- or middle-click on the cells) or if the :ref:`tag decoder<cspy>` detects a change in the physical grid. In the latter case, a json-formatted string is sent to the frontend via UDP and decoded in the according grid. Take a look at the code :ref:`here<read_scanner_data>`
In either case, the function `gis.get_intersection_indexer` is called from ``grid.get_intersection``, checking for overlapping polygons with the selected cell.

.. _modeselector:

ModeSelector
------------

A ModeSelector is a specific cell on the grid, which, when selected via token, activates a certain Mode.

TODO: explain how a mode is triggered and hint: do not use mode.activate() but session.active_mode = mode

.. _sliders:

Sliders
-------

.. _frontend_slider_setup:

TODO: how to define and setup the sliders.

.. _slider_events:

TODO: how to use the sliders, what happens if you use them

.. _devtools:

.. _programming_cell_functions:

Programming Cell Functions
--------------------------

In order to create a new game mode or make a cell "do something" upon selection/interaction, functions can be allocated to cells by adjusting the tables in ``q100viz/settings/``. All .csv files are used to assign functionality to grid cells by combining the cell's coordinates with a certain handle and color.

A table can look like this:

.. csv-table:: grid initialization - q100viz/settings/buildings_interaction_grid_1.csv
  :header: "x", "y", "handle", "color"

  0,18,connection_to_heat_grid,#0075b4
  2,18,refurbished,#0075b4
  4,18,save_energy,#0075b4
  11,18,connection_to_heat_grid,#fdc113
  13,18,refurbished,#fdc114
  15,18,save_energy,#fdc115

The handles for game mode switching have to match one of the strings defined in ``session.MODE_SELECTOR_HANDLES``.: ``'start_individual_data_view', 'start_total_data_view', 'start_buildings_interaction', 'start_simulation'``. You can find more on how these "Mode Selectors" work in :ref:`the according section<modeselector>`.

valid handles are:

**household-individual handles:** are set in ``session.VALID_DECISION_HANDLES``: ``'connection_to_heat_grid', 'refurbished', 'save_energy'``

**mode selection handles**: ``'start_individual_data_view', 'start_total_data_view', 'start_buildings_interaction', 'start_simulation'``

**colors** can be set using strings from this list: https://www.pygame.org/docs/ref/color_list.html

.. _frontend_mode:
.. _mode:

Game Modes
**********

All game-specific surfaces (a.k.a. "Game Modes") are stored in the folder ``q100viz/interaction``. Each game mode is represented as a custom class with similar generic functions. Each class is initiated to an object in :ref:`session.py<session>` to be able to globally access it.

.. image:: ../img/Q-Scope_game-stages.png
  :align: center
  :alt: [Schematic overview on the different game stages with information on what's being displayed on frontend and infoscreen, and explanations of possible user interaction]

* In the :ref:`QUARREE100 use case<quarree>` there are different machine states, defined by the files in ``q100viz/interaction/`` → these are the modes the program is running at (per time)
* implemented modes are:
    * :ref:`Interaction <buildings_interaction>`
    * :ref:`Simulation <simulation_mode>`
    * :ref:`Data View <data_view>`
    * (:ref:`Calibration<calibration_mode>` - for Debugging)

**Game Mode Generic Functions:**

* The ``__init__`` function is seldomly used, since it will be run in the beginning of the script (in ``session.py``), before the variables (e.g. ``grid``) are initialized.
* The ``activate`` function is called automatically in the game loop, when `session.active_mode` changes to this object in the :ref:`game loop <frontend_game_loop>`. **It should not be called manually!**. This function can be used to define which graphical parts shall be displayed, by setting ``session.show_polygons`` etc to true or false. The same can be done for each slider individually.
* ``process_event`` is a function that takes care of mouse events (for debugging purposes, it is possible to select and alter the buildings via mouse L/M/R clicks).
* ``process_grid_change`` is the most important function in each of the game mode classes, as it takes care of the interaction possibilities: It is called each time a :ref:`grid change<read_scanner_data>` is received from cspy. See the :ref:`buildings interaction mode<buildings_interaction>` code as an example.

Some cells can be programmed to trigger additional events, like leaving the current game mode. This is done via tables in ``q100viz/settings/``. Read more on how to program link functions to cells :ref:`here<programming_cell_functions>`.

.. hint::

  For debugging purposes, the Game Modes can be switched using input keys:

  - ``3`` activates the :ref:`Buildings Interaction<buildings_interaction>`
  - ``4`` starts the simulation
  - ``5`` enters the :ref:`individual data view<individual_data_view>`
  - ``6`` enters the :ref:`total (neighborhood) data view<total_data_view>`
  - ``C`` starts the :ref:`Calibration Mode<calibration_mode>`

.. _buildings_interaction:

1. Buildings Interaction Mode
=============================
In the Input Mode, users can set household-, buildings- global parameters. They can leave the mode placing a token on the "simulation mode" selector.

Buildings Interaction
---------------------

The ``process_grid_change`` function of this mode make sure that, after each incoming grid change, the whole grid is iterated using the following routine:

  #. check for intersections with selected (non-white) cells and polygons
  #. according to the rotation of the cell, set the selection of an overlapping building to true and set its ``cell`` value to the ID of the cell. (IDs of the building will later be used for the grouping of buildings - and to allocate them to the users)
  #. for slider handles: update the selected feature of the building with the current slider value
  #. for mode selectors: enable countdown timer for next mode to start
  #. for global/scenario handles: connect additional buildings to the heat grid. There is a dedicated dataframe for these additionally selected buildings called ``session.scenario_selected_buildings`` that excludes all user-selected buildings, so they can be specifically referred to. These buildings will be set 'selected'.
  #. Finally, environment- and buildings-information will be :ref:`sent via UDP to the infoscreen<frontend_UDP_transmitter>`.

Buildings Mode Display
----------------------

.. image:: ../img/frontend_full.png
  :align: center
  :alt: Image of the Frontend in Buildings Mode.

The Buildings Interaction Mode is the most feature-rich display. It shows the basemap with buildings polygons and the heat grid on top. Selected buildings are highlighted by the user-specific color (according to the :ref:`ID<programming_tangibles>` of the token used for selection). On the right, there is a global section containing some functional cells to force-connect a selectable number of buildings to the heat grid.
It contains interaction possibilites for the change of the game modes and sliders for individual setting of the buildings' decision features.

.. _simulation_mode:

1. Simulation
=============

The "Simulation Mode" is the second mode to be run, once all users have selected and specified their households. In this mode, the frontend will start :ref:`GAMA<installing_gama>` in headless mode (no GUI) a subprocess to run the agent-based-model. The users will have to wait until the simulation finished, and the only thing the frontend does is forwarding status information via UDP from GAMA to the infoscreen.

The Simulation can be started by either placing a token on the specified cell on the right side of the frontend (or using the ``S`` key). It will generate an experiment API file for GAMA according to this scheme: https://gama-platform.org/wiki/Headless#simulation-output and run the provided model file using the ``gama-headless.sh``. These two files are to be set up in ``config.py``:

.. code-block:: python
  :caption: config.py

  'GAMA_HEADLESS_FOLDER' : '/home/qscope/GAMA/headless/',
  'GAMA_MODEL_FILE' : '../q100_abm/q100/models/qscope_ABM.gaml',

**ATTENTION**: make sure to set the user rights of ``gama-headless.sh`` executable via ``chmod u+x gama-headless.sh``

.. _simulation_setup:

Setting up the simulation
-------------------------

Upon initialization of this game stage, a new thread is started for the gama simulation to run (later), so the rendering of the pygame canvas will not be stopped when the subprocess begins.

In order to start the simulation, it first has to be set up, using the ``simulation.setup()`` function. Only after that it can be started by setting ``session.active_mode = simulation``.

The function accepts the following **Input Parameters**:
* ``input_max_year`` (int): until which year should the simulation run? providing "2045" will make the simulation run up until 2044-12-31.
* ``export_neighborhood_graphs`` (bool) disable export of individual graphs, for debugging purposes

The **simulation setup algorithm** logs the simulation start time and defines the output path to export the results in the following manner:

1. Each time the frontend software is started, a new output folder is created: ``qScope/data/outputs/output_YYYYmmdd_HH_MM_SS``

.. code-block::
  :caption: tree view of output folder

    project qScope
    └───data
        └───outputs
            └───output_YYYYmmdd_HH-MM-SS
            |   └───connections
            |   └───emissions
            |   └───energy_prices
            |   └───snapshot
            └───buildings_clusters_YYYYmmdd_HH-MM-SS.csv
            └───buildings_power_suppliers.csv
            └───console-outputs-null.txt
            └───simulation_parameters_YYYYmmdd_HH-MM-SS.xml
            └───simulation_outputsnull.xml

Read more about the simulation results in the :ref:`ABM section<simulation_outputs>`.

2. An xml file necessary to start the simulation in headless mode is created from the ``session.environment`` parameters. Here we store initial values for certain variables in GAMA. These parameters are:
   * Alpha scenario
   * Carbon price scenario
   * Energy prices scenario
   * Q100 OpEx prices scenario
   * Q100 CapEx prices scenario
   * Q100 Emissions scenario
   * Q100 Emissions scenario
   * Carbon price for households?

The xml struct is created in a function called ``make_xml`` and saved in the output folder using the time stamp of the simulation start. ``simulation_parameters_YYYYmmdd_HH-MM-SS.xml``

A set of different scenarios can be found in the data folder in the scenario_X.csv files. These all regard different energy price scenarios under which the model can be investigated.

In short, the **Input Data** for the simulations in QUARREE100 are defined in the ``qScope/data/`` folder via the files ``scenario_X.csv``. Which one of these files is taken, will be defined in ``session.py`` in the ``environment['active_scenario_handle']`` entry. The file this entry points to, will be read in ``simulation.setup()``, and transfered to an xml file to eventually set the simulation's input data accordingly.

.. csv-table:: example of a simulation scenario to serve as input data for GAMA variables.
  :widths: 20, 15, 20, 30, 20
  :header: name,type,value,name_human_readable,value_human_readable

  alpha_scenario,string,Dynamic_moderate,Anteil monatlicher Energieausgaben am Haushaltsbudget,Moderat ansteigend (ca. 7 - 10 %)
  carbon_price_scenario,string,B - Moderate,CO2-Bepreisung,Moderat ansteigend (ca. 25 - 350 € / Tonne CO2)
  energy_price_scenario,string,Prices_2022 1st half,Energiepreisentwicklung (ohne politische Maßnahmen), Preisentwicklung 2022
  q100_price_opex_scenario,string,12 ct / kWh (static),Q100-Wärmeversorgung: Betriebskosten, Dauerhaft 12 ct / kWh
  q100_price_capex_scenario,string,5 payments,Q100-Wärmeversorgung: Investitionskosten,Ratenzahlung (5 x 1000 €)
  q100_emissions_scenario,string,Declining_Linear,Q100-Wärmeversorgung: Emissionsverlauf, Jährlich abnehmend (100 - 0 g / kWh)

.. note::
  For debugging purposes, some random N buildings can be marked as selected and force-connected to the heat grid for the simulation by starting the frontend script with the input flag ``--select_random N`` (int). :ref:`See more about the frontend startup flags here<frontend_startup_flags>`.

3. After setting up the simulation input data, ``simulation.running`` will be set ``True``, which causes the simulation to actually be executed (once) in the dedicated thread via ``run_script()`` within ``simulation.run()``. The process of the latter function will sty on hold until the subprocess is done.

4. Once the subprocess is done, :ref:`matplotlib graphs are created<graphs>` from the output data and the paths of these files will be send via UDP to the infoscreen to be displayed there in the next game stage.

Simulation Mode View
--------------------

There is no possibility for user interaction in this mode. The frontend only forwards information on the in simulation process in percent to the infoscreen, where that number is displayed.

.. _data_view:
.. _individual_data_view:

3a. Individual Data View
========================

This game stage is used to focus on single selected buildings and show the created graphs on heat consumption, CO2-emission and energy costs of that building :ref:`on the infoscreen<infoscreen_individual_data_view>`.

The interaction surface is constrained to the :ref:`mode selectors<modeselector>` and to four cells highlighted in the colors of the users - these can be used to focus on the building of the according user. The general basemap can not be used for interaction.

.. _total_data_view:

3b. Total Data View
===================

The total data view mode is used to :ref:`display all neighborhood data on the infoscreen<infoscreen_individual_data_view>` displaying charts that compare the selected houses energy costs and emissions, as well as the overall energy cost development and the total amount of buildings connected to the heat grid.

All possible interaction is constrained to the :ref:`game mode selectors<modeselector>`.

Debugging and Devtools
**********************

TODO: session.log
TODO: session.VERBOSE_MODE
TODO: debug_num_of_random_buildings, debug_connection_date,debug_force_refurbished, debug_force_save_energy

TODO: refer to key_events_

.. _calibration_mode:

Calibration
===========

.. _keystone_transformation:

keystone transformation
=======================

general information on image transofrmation using opencv:

`tutorial_py_geometric_transformations <https://docs.opencv.org/3.4/da/d6e/tutorial_py_geometric_transformations.html>`_

`using cv.perspectiveTransform for vectors <https://docs.opencv.org/3.4/d2/de8/group__core__array.html#gad327659ac03e5fd6894b90025e6900a7>`_
and `cv.warpPerspective for images <https://docs.opencv.org/3.4/da/d54/group__imgproc__transform.html#gaf73673a7e8e18ec6963e3774e6a94b87>`_

**adding a new surface, draw on it and transform it:**

.. code-block:: python

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
    # where e.g. X1 = 0, X2 = 50, Y1 = 0, Y2 = 81.818

    def draw(self, viewport):

      pygame.draw.polygon(self.surface, pygame.Color(255, 255, 255), [[20, 70], [20, 20], [80, 20], [80, 70]])  # render polygon

      viewport.blit(self.surface, (0,0))  # cast it to viewport

in file ``q100viz/keystone.py``

frontend representation
-----------------------

* slider uses the transformation of the :ref:`grid<grid>`
* **drawing of polygons and values** should be done via ``self.surface.blit(...)``. Slider surface is rendered and "blitted" to main canvas.

``print(slider.coords_transformed)`` returns:

.. code-block::

  [[860.9641723632812, 915.1583862304688],
  [863.9833984375, 614.8511352539062],
  [1228.917724609375, 622.6510009765625],
  [1226.5196533203125, 923.7374267578125]]

with ``[[bottom-left[x], bottom-left[y]], [upper-left[x], upper-left[y]], [upper-right[x], upper-right[y]], [bottom-right[x], bottom-right[y]]]``
