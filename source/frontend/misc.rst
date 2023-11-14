Miscellaneous
#############

.. _simple_pygame_features:

Simple Pygame Features
**********************

Displaying Text
===============

.. code-block:: python

  # 1. define font:
  font = pygame.font.SysFont('Arial', 20)
  # 2. use font to write to canvas:
  canvas.blit(font.render(str(mouse_pos), True, (255,255,255)), (200,700))

Polygons
========

Draw polygons on specific surface
---------------------------------

.. code-block:: python

  # general:
  # points = [[x1, y1], [x1, y2], [x2, y1], [x2, y2]]
  #          [[bottom-left], [top-left], [bottom-right], [top-right]]
  # points_transformed = reference_surface.transform(points)
  # pygame.draw.polygon(reference_surface, color, points_transformed)

  # example:
  points = [[20, 70], [20, 20], [80, 20], [80, 70]]  # percentage relative to surface
  points_transformend = session.grid_1.surface.transform(points)

  #                   surface,   color,      coords_transformed
  pygame.draw.polygon(viewport, (255, 0, 0), viewport.transform(rect_points))

.. _draw_simple_polygon_layer:

Draw Polygon Layer (simple)
---------------------------

.. code-block:: python
  :caption: gis.py - simply draw polygons on a specific surface with a specific stroke and color. Note: when stroke is 0, the polygon will be filled.

  def draw_polygon_layer(self, surface, df, stroke, fill):
    '''draw polygon layer, do not lerp'''
    try:
        for polygon in df.to_dict('records'):
            if fill:
                fill_color = pygame.Color(*fill)

            points = self.surface.transform(polygon['geometry'].exterior.coords)
            pygame.draw.polygon(self.surface, fill_color, points, stroke)

    except Exception as e:
        session.log += "\n%s" % e
        print("cannot draw polygon layer: ", e)

Draw polygon layer and lerp color using bool
--------------------------------------------

.. code-block:: python
  :caption: gis.py - draw polygons on a specific surface with certain stroke; lerp color according to bool

  def draw_polygon_layer_bool(self, surface, df, stroke, fill_false, fill_true=None, fill_attr=None):
    '''draw polygon layer, lerp using bool value'''
    try:
        for polygon in df.to_dict('records'):
            if fill_false:
                fill_color = pygame.Color(*fill_false)

                if fill_true:
                    fill_color = pygame.Color(fill_true) if polygon[fill_attr] else fill_color

            points = self.surface.transform(polygon['geometry'].exterior.coords)
            pygame.draw.polygon(self.surface, fill_color, points, stroke)

    except Exception as e:
        session.log += "\n%s" % e
        print("cannot draw polygon layer: ", e)

**draw polygon layer and lerp colors according to float:**

.. code-block:: python
  :caption: gis.py - draw polygons on a specific surface with certain stroke; lerp color according to float values

    def draw_polygon_layer_float(self, surface, df, stroke, fill, lerp_target=None, lerp_attr=None):
      '''draw polygon layer and lerp using float'''
      try:
          for polygon in df.to_dict('records'):
              if fill:
                  fill_color = pygame.Color(*fill)

                  if lerp_target:
                      target_color = pygame.Color(lerp_target)
                      fill_color = fill_color.lerp(target_color, polygon[lerp_attr] / df[lerp_attr].max())

              points = self.surface.transform(polygon['geometry'].exterior.coords)
              pygame.draw.polygon(self.surface, fill_color, points, stroke)

      except Exception as e:
          session.log += "\n%s" % e
          print("cannot draw polygon layer: ", e)

Images
======

Pygame is able to load images onto Surface objects from PNG, JPG, GIF, and BMP image files.

.. code-block:: python

  image = pygame.image.load("images/scenario_progressive.bmp")
  canvas.blit(image, (0,0))

See more about the usage of pygame images under :ref:`graphictools<graphictools>`.

.. _creating_your_own_project:

Creating your own project
*************************

In order to transfer the project's focus to another neighborhood, you'd need to conduct the following steps:

* input basemap raster file
    * in our example project, we have a basemap file stored in ``.jpg``-format. This file is provided by the :ref:`config.py<frontend_config>` and should be set there.
* input shapefile for polygons and raster data:
    * further geodata can be loaded via Shapefiles using geopandas. In :ref:`our example project<quarree>`, we have implemented a custom function in ``gis.py`` called ``read_shapefile()`` (evoked in ``buildings.py/load_data()``) that handles the readout of the shapefile's metadata, transfers the data into a pandas DataFrame and adds more custom data categories.
* define corner coordinates / ROI (using GIS) and rotation of map. :ref:`The basemap is essentially initialized<gis>` in ``session.py`` with the corner coordinates of the image:

.. code-block:: python
  :caption: ``session.py:120``

  ############################## INITIALIZATION #########################
  #--------------------------------- gis --------------------------------
  # Initialize geographic viewport and basemap
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
  basemap.warp()


*  Take care to crop the basemap image properly in the frontend section:

.. code-block:: python
  :caption: frontend.py:216 ``crop_width`` and ``crop_height`` define how much the basemap has to be cropped.

  # basemap
  if session.show_basemap:
      crop_width = 4644
      crop_height = 800
      self.canvas.blit(session.basemap.image, (0, 0),
                        (0, 0, crop_width, crop_height))

