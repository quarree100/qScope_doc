Frontend API
############

#. TODO: combine these sections to one page
#. TODO: general overview section with schematic image of message flows between the frameworks

Frontend UDP setup
******************

UDP communication is initialized in `frontend.py`. The frontend object has a thread running for each of the physical tables and links them to each of the :ref:`grids<grid>`, so the grids can receive messages from each of the according tag decoders. Additionally, it starts a third thread to function as an UDP server to receive messages from :ref:`GAMA<ABM>` and forward them to the infoscreen.
Upon initialization, the grids will be allocated a callback function, that is executed whenever a message is received from the tag decoders: read_scanner_data_. In this function, :ref:`the received message<cspy_grid_message>` will be interpreted so the grids can process the id and rotation of each grid cell. It is possible to infer the relative rotation from a previous state, which can be used as means of interaction (which is NOT done in the QUARREE100 project).

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
            # TODO: this causes type error when no slider value provided in cspy → provide 0 by default?
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

JSON and CSV constructs used for the communication between GAMA, the Q-Scope-infoscreen and -frontend.

.. _frontend_UDP_transmitter:

Frontend UDP transmitter
************************

TODO: API functions that send messages to infoscreen.