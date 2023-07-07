.. _abm:

ABM
###


.. _installing_gama:

Installing GAMA
***************

You will have to have [GAMA](https://gama-platform.org/download) installed. It's best to choose the Version with JDK.

Understanding GAMA
******************

Simulation
**********

.. _simulation_outputs:

Simulation Outputs
------------------

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

TODO: explain contents of the folders


.. _gama_headless_mode:

Starting GAMA in headless mode.
-------------------------------

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

The final shell command that is used in a :ref:`subprocess of the frontend<simulation_setup>` looks like this: ``/opt/gama-platform/headless/gama-headless.sh /home/USER/qScope/data/outputs/output_20230704_13-51-36/simulation_parameters_20230704_13-51-36.xml /home/USER/qScope/data/outputs/output_20230704_13-51-36``,

or in short: ``/path/to/gama-headless.sh /path/to/simulation.xml /path/to/output_folder``


TODO: buildings clusters are used in GAMA to define selected/focused buildings

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
