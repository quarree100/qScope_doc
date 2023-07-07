
.. _frontend_initialization:

Initialization: Files, paths & data
###################################

Config file
***********

To configure the frontend, the file ``q100viz/settings/config.py`` can be changed before running the application.

.. note:: Make yourself a copy of the file config.py and adjust it according to the context you want to use the setup. We had different scenarios for different workshop participants.

Here you can modify the path of files being imported/exported, :ref:`change the source file and the simulation time for the agent-based-model<simulation_setup>`, save information of the extents of the :ref:`grids<frontend_grid_setup>` and the :ref:`sliders<frontend_slider_setup>` here. The individual adjustments will be discussed in the according sections, respectively.

.. _session:

Session file
************

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
********************

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

  "Behavior" data such as the connection to the QUARRE100-heat-grid, refurbishment of the house or energy-saving measures are pre-set in the following manner: ``False``, if house's energy_source (in source data) is not ``None``, else the house will come in pre-connected and refurbished.

Buildings can either be ``selected`` by a user or not. Selection is done if a cell is selected on the table (by placing a token physically). :ref:`The tag decoder software<cspy>` will detect any interaction with the table surface and forward the grid information to the frontend to be deciphered in the ``grid.py``: :ref:`read_scanner_data<read_scanner_data>` function.
The Buildings class contains additional functions, e.g. ``find_closest_heat_grid_line`` for graphical calculations and functions to organize, convert and export the DataFrame for specific needs.


.. _gis:

GIS: Shapes and Raster
**********************

The file ``gis.py`` contains two classes:

1. **The GIS class** draws features from the source Shapefile, like polygons and lines, onto the :ref:`canvas<canvas>`. It provides functions to draw the whole polygon layer at once, color them in a certain style (e.g. according to heat grid connection status), etc.
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
