.. _abm:

ABM
###

In `QUARREE100<quarree>` an agent-based model was developed using `GAMA <https://gama-platform.org/download>`_. The model itself is called `TREND model <https://www.github.com/quarree100/q100_abm>` (and can be found on GitHub). It models the behavior transformation of citizens involved in the district's heat transition.

Usually GAMA provides a user-friendly GUI with interactive variable input possibilities and graphical outputs, but from within our :ref:`frontend<frontend_usage>`, it is run in "headless mode" - the simulation is run "invisibly" in the background.

.. _installing_gama:

Installing GAMA
***************

`GAMA <https://gama-platform.org/download>`_ will be executed in a subprocess of the Q-Scope in :ref:`simulation mode <simulation_mode>`. It is recommended to install the Version with JDK.

.. attention:: We used GAMA 1.8.2 to develop the code; it might not necessarily work with different versions (we experienced some issues when switching between versions)!

.. _gama_headless_mode:

Starting GAMA in headless mode
******************************

Instead of starting GAMA with its GUI (like in every usual execution), it can be run from the terminal like this: ``/path/to/gama-headless.sh /path/to/simulation.xml /path/to/output_folder``.

.. _simulation_xml:

The GAMA headless simulation takes an input xml file to initialize variables used in the model with desired values like below:

.. code-block:: xml
    :caption: arbitrary values are taken from scenario_A.csv

    <Experiment_plan>
     <Simulation experiment="agent_decision_making" sourcePath="/home/dunland/github/qScope/q100_abm_qscope-workshop/q100/models/qscope_ABM.gaml" finalStep="730" seed="1.0">
      <Parameters>
       <Parameter name="timestamp" type="STRING" value="20230509_14-22-50" var="timestamp"/>
       <Parameter name="Alpha scenario" type="STRING" value="Static_mean" var="alpha_scenario"/>
       <Parameter name="Carbon price scenario" type="STRING" value="B - Moderate" var="carbon_price_scenario"/>
       <Parameter name="Energy prices scenario" type="STRING" value="Prices_2021" var="energy_price_scenario"/>
       <Parameter name="Q100 OpEx prices scenario" type="STRING" value="12 ct / kWh (static)" var="q100_price_opex_scenario"/>
       <Parameter name="Q100 CapEx prices scenario" type="STRING" value="1 payment" var="q100_price_capex_scenario"/>
       <Parameter name="Q100 Emissions scenario" type="STRING" value="Constant_Zero_emissions" var="q100_emissions_scenario"/>
       <Parameter name="Carbon price for households?" type="BOOLEAN" value="false" var="carbon_price_on_off"/>
      </Parameters>
      <Outputs>
       <Output id="0" name="neighborhood" framerate="729" />
       <Output id="1" name="households_employment_pie" framerate="729" />
       <Output id="2" name="Charts" framerate="729" />
       <Output id="3" name="Modernization" framerate="729" />
       <Output id="4" name="Monthly Emissions" framerate="729" />
       <Output id="5" name="Emissions cumulative" framerate="729" />
      </Outputs>
     </Simulation>
    </Experiment_plan>

The final shell command that is used in a :ref:`subprocess of the frontend<simulation_setup>` looks like this: ``/opt/gama-platform/headless/gama-headless.sh /home/USER/qScope/data/outputs/output_20230704_13-51-36/simulation_parameters_20230704_13-51-36.xml /home/USER/qScope/data/outputs/output_20230704_13-51-36``

Both xml file and the command are created in the Q-Scope's :ref:`simulation mode<simulation_mode>`. Another important file for the execution of the simulation is the ``buildings_clusters_YYYYmmdd_HH-MM-SS.csv`` file dropped to the simulation output folder (described below).

.. csv-table::
    :header: id,	spec_heat_consumption,	spec_power_consumption,	energy_source,	connection_to_heat_grid,	refurbished,	save_energy,	group

    7.14	158.35729	29.34483	Strom	False	2025	False	1
    7.21	112.92122	14.60804	Gas	2025	False	False	0

This information tells GAMA which buildings are selected have preset options should to be considered upon the start of the simulation.

.. _simulation_outputs:

Simulation Outputs
******************

The :ref:`simulation setup algorithm<simulation_setup>` logs the simulation start time and defines the output path to export the results in the following manner:

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

* ``output_[timestamp]``: contains simulation results of the specific :ref:`game iteration round<game_iterations>`
* ``connections`` contains ``connections_export.csv`` providing a list of the amount of agents connected to the heat grid (in percent and absolute)
* in ``emissions``, there are building-specific timelines with calculated CO2-emissions and graphs, depicting these lists, :ref:`created in the frontend code<graphs>`.
* ``energy_prices`` stores building-specific, caluclated energy prices as lists and graphs
* ``snapshot`` is the place GAMA exports its native graphical output to.

.. hint:: When wanting to use and display gama-exported-images on infoscreen, try this:

  .. code-block:: python
    :caption: should by implemented in simulation_mode.py right before starting the simulation via self.make_xml
        # compose image paths as required by infoscreen
        session.gama_iteration_images[session.environment['current_iteration_round']] = [
            str(os.path.normpath('data/outputs/output_{0}/snapshot/Chartsnull-{1}.png'.format(
                self.timestamp, str(self.final_step - 1)))),
            str(os.path.normpath('data/outputs/output_{0}/snapshot/Emissions cumulativenull-{1}.png'.format(
                self.timestamp, str(self.final_step - 1)))),
            str(os.path.normpath('data/outputs/output_{0}/snapshot/Monthly Emissionsnull-{1}.png'.format(
                self.timestamp, str(self.final_step - 1)))),
            str(os.path.normpath('data/outputs/output_{0}/snapshot/households_employment_pienull-{1}.png'.format(
                self.timestamp, str(self.final_step - 1)))),
            str(os.path.normpath('data/outputs/output_{0}/snapshot/Modernizationnull-{1}.png'.format(
                self.timestamp, str(self.final_step - 1)))),
            str(os.path.normpath('data/outputs/output_{0}/snapshot/neighborhoodnull-{1}.png'.format(
                self.timestamp, str(self.final_step - 1))))
        ]

        # send final_step to infoscreen:
        session.api.send_dataframe_as_json(pandas.DataFrame(data={"final_step": [self.final_step]}))
