.. _communication:

Communication
#############

TODO: api

API: Backend
************

TODO: grid, decoder-side

Incoming grid status is deciphered in ``grid.py``: read_scanner_data_

.. _frontend_communication:

TODO: grid, frontend side

API: Frontend
*************
JSON and CSV constructs used for the communication between GAMA, the Q-Scope-infoscreen and -frontend.


Starting the GAMA Simulation via XML
------------------------------------
When moving from Input Mode to Simulation Mode (by Placing a Token on the according ModeSelector_), an xml file is composed containing all the global environment data. The general structure looks like this:

.. code-block:: xml

  <Experiment>
    <Parameter name="year">0</Parameter>
    <Parameter name="foerderung">0</Parameter>
    <Parameter name="CO2-Preis">0</Parameter>
    <Parameter name="CO2-emissions">0</Parameter>
    <Parameter name="versorgung">0</Parameter>
    <Parameter name="investment">0</Parameter>
    <Parameter name="anschluss">0</Parameter>
    <Parameter name="connection_speed">0</Parameter>
  </Experiment>


Composing the xml struct is done via ``stats.to_xml`` and receives single rows of a dataframe.

.. code-block:: python

  def to_xml(row):
    xml = ['<Experiment>']
    for field in row.index:
        xml.append('  <Parameter name="{0}">{1}</Parameter>'.format(field, row[field]))
    xml.append('</Experiment>')
    return '\n'.join(xml)

and then in `input_mode.py`:

.. code-block:: python

    # enter simulation mode:
  elif x == int(session.grid_settings['ncols'] * 2 / 3 + 2):
      session.active_mode = session.simulation
      grid.deselect(int(session.grid_settings['ncols'] * 2 / 3), len(grid.grid) - 1)
      print(session.active_mode)

      # compose dataframe to start
      df = pd.DataFrame(session.environment, index=[0])
      xml = '\n'.join(df.apply(stats.to_xml, axis=1))
      print(xml)
      f = open('../data/simulation_df.xml', 'w')
      f.write(xml)
      f.close()

API: Infoscreen
***************

TODO: UDP frontend -> infoscreen