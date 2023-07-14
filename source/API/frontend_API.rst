.. _frontend_api:

Frontend API
############

.. image:: ../img/Q-Scope_software_components.png
    :align: center
    :alt: Here you should see a schematic overview on the different software components the Q-Scope setup requires: cspy, frontend, infoscreen, abm

Purple Arrows in the image above indicate the UDP communication between the different software components of the Q-Scope framework.

Frontend UDP setup
******************

The incoming grid information from cspy (as described in the :ref:`previous chapter<cspy_communication>`) is deciphered in :ref:`grid.py<read_scanner_data>`

UDP communication is initialized in `frontend.py`. The frontend object has a thread running for each of the physical tables and links them to each of the :ref:`grids<frontend_grid>`, so the grids can receive messages from each of the according tag decoders. Additionally, it starts a third thread to function as an UDP server to receive messages from :ref:`GAMA<ABM>` and forward them to the infoscreen.

Upon initialization, the grids will be allocated a callback function, that is executed whenever a message is received from the tag decoders: read_scanner_data_. In this function, :ref:`the received message<cspy_grid_message>` will be interpreted so the grids can process the id and rotation of each grid cell. It is possible to infer the relative rotation from a previous state, which can be used as means of interaction (which, btw, is NOT done in the QUARREE100 project).

.. note::

  The actual UDPServer class can be found in `UDP.py`.

.. _read_scanner_data:

The grid is either updated when interacting with a computer mouse (left- right- or middle-click on the cells) or if the :ref:`tag decoder<cspy>` detects a change in the physical grid. In the latter case, a json-formatted string is sent to the frontend via UDP and decoded in the according grid.

.. code-block:: python
  :caption: the algorithm for deciphering the incoming grid data from :ref:`The tag decoder software<cspy>`:

      def read_scanner_data(self, message):
        try:
            msg = json.loads(message)
        except json.decoder.JSONDecodeError:
            print("Invalid JSON")
            return

        try:
            # update grid cells
            for y, row in enumerate(self.grid):
                for x, cell in enumerate(row):
                    cell.id, cell.rot = msg['grid'][y * self.x_size + x]

                    cell.selected = cell.id != 5  # any non-white object selects cells

                    # calculate relative rotation
                    if cell.rot == -1:  # an inactive cell has a rotation value of -1
                        cell.rel_rot = 0
                    elif cell.prev_rot != cell.rot:
                        cell.rel_rot = cell.rot - cell.prev_rot if cell.prev_rot > -1 else 0
                    cell.prev_rot = cell.rot

            session.flag_export_canvas = True
            session.active_mode.process_grid_change()

            # update slider values
            for slider_id in self.sliders.keys():
                if msg['sliders'][slider_id] is not None: self.sliders[slider_id].value = msg['sliders'][slider_id]
                self.sliders[slider_id].process_value()

        except TypeError as t:
            # pass
            print("type error", t)
        except IndexError:
            print("Warning: incoming grid data is incomplete")

Frontend and GAMA
*****************

The frontend can start a gama simulation in a subprocess by entering the :ref:`simulation mode<simulation_mode>`. GAMA will create csv files containing simulation results, which are used by the frontend to generate :ref:`graphs<graphs>`. These graphs are displayed on the infoscreen in the `data view modes<individual_data_view>`.

More information on the exported simulation results can be found in the according `ABM section<simulation_outputs>` of this documentation.

.. _frontend_UDP_message:

Frontend UDP messages
*********************

The messages sent from the frontend have the following generic structure:

.. code-block::
    :caption: UDP messages sent from the frontend to the infoscreen

    {
        "buildings_groups": {
            "group_0": {
                "buildings": [
                    {
                        "address": string,
                        "spec_heat_consumption": number,
                        "spec_power_consumption": number,
                        "cluster_size": number,
                        "emissions_graphs": string, #path to a GAMA generated image
                        "energy_prices_graphs": string, #path to a GAMA generated image
                        "CO2": number,
                        "connection_to_heat_grid": boolean,
                        "connection_to_heat_grid_prior": boolean,
                        "refurbished": boolean,
                        "refurbished_prior": boolean,
                        "save_energy": boolean,
                        "save_energy_prior": boolean,
                        "energy_source": string, #"Gas" or "Strom"
                        "cell": sting
                    },
                    ... #can have multiple buildings
                ],
                "connections": 0,
                "slider_handles": list #"save_energy", "connection_to_heat_grid", or/and "refurbished"
            },
            ...# should have group_0 to group_3
        }
    }

They are emitted each time a grid change is received from cspy. All UDP messages are composed using a set of tools from `q100viz/api.py`:

* ``send_message``: simple function to finally send a message via UDP. It should have json format for the infoscreen to process it properly.
* ``send_dataframe_as_json``: make a json struct from a pandas DataFrame object and send it via send_message()
* ``send_df_with_session_env``: translates a pandas DataFrame to json format and appends the session.environment dict
* ``send_session_env``: simply send the session.environment dict as json format
* ``forward_gama_message``: formats gama simulation status message to percentage and forwards it to the infoscreen via send_message()
* ``export_json``: Export a dataframe to JSON file. This is necessary to transform GeoDataFrames into a JSON serializable format.