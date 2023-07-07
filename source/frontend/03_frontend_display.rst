.. _canvas:

Canvas & Display
################

The whole frontend was programmed using `pygame <pygame.org>`_ - a set of Python modules designed for writing video games. Pygame will create a graphical canvas, running in the loop, which will change its appearance according to user action.

.. _frontend_class:

Frontend Setup
**************

The frontend class itself is defined in ``q100viz/frontend.py``.
Upon initialization of the frontend class, the pygame environment is created. Things like the display framerate, window position etc can be set here.

.. _frontend_setup_window:

window position and size
========================

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
*******************

After :ref:`initialization<frontend_class>`, the frontend will run in a loop to update the projection, evaluate keyboard input, handle the :ref:`game modes<modeselector>` and process :ref:`slider changes<sliders>`.
Finally, ``pygame.display.update()`` is called to actually do what it says, and a ``pygame.time.Clock`` is updated, using the defined framerate. We used a framerate of 12 FPS, because this is the maximum framerate used in the :ref:`tag decoder<cspy>`.

.. _key_events:

The following key events are implemented in the `QUARREE100 <https://www.quarree100.de>`_ example project:

- ``p`` toggle the display of GIS polygons
- ``m`` toggle basemap visibility
- ``g`` toggle visibility of grid outline and cell ID, rotation, coordinates
- ``n`` toggle visibility of the heat grid
- ``b`` toggle the black mask on viewport
- ``3`` start :ref:`Buildings Interaction Mode<buildings_interaction>`
- ``4`` start :ref:`Simulation Mode<simulation_mode>`
- ``5`` start :ref:`Individual Data View Mode<individual_data_view>`
- ``6`` start :ref:`Total Data View Mode<total_data_view>`

Canvas Composition
******************

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
===========================================

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
=====================

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
==================

The sliders have a bool called ``show_text`` that, when ``True``, activates the display of the slider control texts. This variable can be used for the usage modes to define whether the slider controls shall be displayed.

.. _graphictools:

Graphic Tools
**************

Images
======

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
======

The frontend software creates graphs from the :ref:`simulation results<simulation_outputs>` using the matplotlib library. A toolkit is contained in ``q100viz/graphics/graphs.py``, providing the following functions:

* ``export_individual_emissions``: exports specified column of csv-data-file for every iteration round to graph and exports png
* ``export_individual_energy_expenses``: exports specified column of csv-data-file for every iteration round to graph and exports png
* ``export_default_graph``: exports default data to graph with gray curve
* ``export_compared_emissions``: exports all data for selected group buildings into one graph for total data view
* ``export_neighborhood_emissions_connections``: creates a bar plot for the total number of connections to the heat grid with an overlaying line plot of total emissions
* ``export_compared_energy_costs``: exports all data for selected group buildings into one graph for total data view
* ``export_neighborhood_total_data``: exports specified column of csv-data-file for every iteration round to graph and exports png

and some helpful conversion functions used to get the right units:

* ``GAMA_time_to_datetime``
* ``grams_to_kg``
* ``grams_to_tons``
* ``rgb_to_hex``
* ``rgb_to_float_tuple``

TODO: image of a graph and explain the lines, round iterations and pre-calculated grey lines

.. attention::
  If :ref:`verbose mode<verbose_mode>` is activated, house addresses and absolte consumption data will be added to the graphs!
