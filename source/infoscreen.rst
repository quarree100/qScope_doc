.. _infoscreen:

Infoscreen
##########

Overview
***********************

.. _installing_infoscreen:

Infoscreen Installation
=======================

Clone the repository to local:

..  code-block:: bash

    git clone git@github.com:quarree100/qScope_infoscreen.git``


Install node modules:

..  code-block:: bash

    cd qScope_infoscreen
    npm install


Setup a folder public/data as a symlink pointing to qScope/data:

..  code-block:: bash

    ln -s ../data/ ./public


File Structure
=========================

..  code-block::

    project qScope
    └───cspy
    └───data
    └───q100_abm
    └───qScope_frontend
    └───qScope_infoscreen
        └───node_modules
        └───public
            └───data #symlink to qScope/data
            └───img
            └───js #pure js files
            └───lib #js libraries such as jquery and socket.io
            index.html
            style.css
        q100_info.js
            nodejs file

Run the infoscreen
=========================

Run npm shortcut command:

..  code-block:: bash

    npm start #equivalent to ``npm run start``, which is defined in package.json. "start" script runs ``npx nodemon q100_info.js``

Modes/Views
****************************

.. _infoscreen_buildings_interaction:

Buildings Interaction View
===========================

.. image:: img/Infoscreen_01_buildingsInteraction.png
    :align: center
    :alt: [Image of the infoscreen, showing four colored squares, each of which contains the address of the selected houses. In each of the squares there is a table with information on energy saving measures.]

.. |icon_heat_grid| image:: img/infoscreen_icon_heat_grid_connection.png
    :height: 3ex

.. |icon_green_plus| image:: img/infoscreen_icon_green_plus.png
    :height: 3ex

.. |icon_save_energy| image:: img/infoscreen_icon_save_energy.png
    :height: 3ex

Tiles of information card shows the information of 4 different selected buildings.
In each tile, at the top the name of the building is stated. Under it three icons indicate the options that user selected for the current round, which itself is shown in the lower right corner.

|icon_heat_grid| the connection to the heat grid

|icon_green_plus| refurbishment status

|icon_save_energy| energy consumption preference.

The Heat consuption is indicated in a chart in 9 scale from A+ to H.

.. image:: img/infoscreen_energy_graph.png
    :align: center
    :alt: []
    :height: 4ex

In the table these three options are displayed throughout rounds.

.. _infoscreen_inidividual_data_view:

Individual Data View
=========================

.. image:: img/Infoscreen_03a_individualDataView.png
    :align: center
    :alt: [close-up of a green square showing information of one of the selected houses and two graphs with predicted CO2-emissions and energy costs up until the year 2045.]

The individual data view displays the information of one selected building. In addition to the information in the buildings interaction view, emissions over time and energy costs per household over time are shown in charts.


.. _infoscreen_individual_data_view:

Total Data View
=========================

.. image:: img/Infoscreen_03b_totalDataView.png
    :align: center
    :alt: [Image of the infoscreen showing the neighborhood's cumulated data and the housesholds emissions and energy costs in comparison.]

There are four different graphs to show the result of the simulation.

1. Energiekosten im Vergleich / Energy costs in comparison
shows the change of the energy cost over time per building.

2. Generalle Energiepreise nach Energietraeger / General energy prices by energy carrier
shows the change of price of energy prices per energy carrier.

3. Montaliche Emissionen im Vergleich / Montaliche emissions comparison
shows the change of monthly CO2 emissions over time per building.

4. Quartiersemissionen und Waermenetzanschluesse / Neighborhood emissions and heating network connections
shows the CO2 emissions of the whole neighborhood over time and the number of the connections to the heat grid over time.


Architecture
****************************

Overview
=========================
Infoscreen is primarily a node js application. q100_info.js runs a web application using expressjs, and communicate with Q-Scope Frontend via UDP messaging using socket.io. Rendering of views are done with html+css+javascript at public folder.

js/main.js receives UDP message from the frontend, and injects data and renders respevtive views.

Message handling
===========================

`mode messages` (explained in infoscreen_API) received from the `frontend<frontend_udp_message>` are processed at main.js line 15 - 20:

.. code-block::  javascript
    
    if (json.hasOwnProperty('mode')) {
      if (json.mode != currentUserMode) {
        switchUserMode(json.mode);
        currentUserMode = json.mode;
      }
    }

switchUserMode function at render.js line 98 - 125 toggles the visibility of html elements which is relevant to the respective modes. 

.. code-block::  javascript

    const switchUserMode = function (mode) {
        // TODO: wrap in resetAnswer function
        grayoutAnswerYes()
        grayoutAnswerNo()
        if (mode == 'simulation') {
            displaySimulationMode()
        }
        else if (mode == 'input_scenarios') {
            displayInputEnvironmentMode()
        }
        else if (mode == 'buildings_interaction') {
            displayBuildingsInteractionMode()
            // displayDataViewIndividualMode()
        }
        else if (mode == 'questionnaire') {
            const question = questions[getRandomInt(5)] //TODO: get question number from UDP

            displayQuestionnaireMode(question)
        }
        else if (mode == 'individual_data_view') {
            displayDataViewIndividualMode()
            removeHouseholdCards()
            createHouseholdCards()
        }
        else if (mode == 'total_data_view') {
            displayDataViewTotalMode()
        }
    }

if a message contains 'current_iteration_round' property, at main.js line 21 - 31 it updates the global variable currentIterationRound which is stored at appData.js and refreshes the number of the columns in the table at the buildings interation view.

.. code-block::  javascript

    if (json.hasOwnProperty("current_iteration_round")) {
      if (currentIterationRound != json.current_iteration_round) {
        currentIterationRound = json.current_iteration_round;
        console.log("current_iteration_round = " + currentIterationRound);
        tableAddColumn(json.current_iteration_round);
        $('#currentRound').find('span').text(currentIterationRound + 1);
        if (currentIterationRound == 0) {
          location.reload(); // reload page
        }
      }
    }


if a message contains 'buildings_groups' property, at main.js line 46 - 58  renderHouseInfo funtion is triggered and it injects the data to the view for the buildings interaction view and the individual data view :

.. code-block::  javascript

    if (json.hasOwnProperty('buildings_groups')) {
        if ('group_0' in json.buildings_groups) renderHouseInfo(json.buildings_groups.group_0, "dataViewIndividualQuarter0");
        if ('group_1' in json.buildings_groups) renderHouseInfo(json.buildings_groups.group_1, "dataViewIndividualQuarter1");
        if ('group_2' in json.buildings_groups) renderHouseInfo(json.buildings_groups.group_2, "dataViewIndividualQuarter2");
        if ('group_3' in json.buildings_groups) renderHouseInfo(json.buildings_groups.group_3, "dataViewIndividualQuarter3");

        ...

        updateSelectedConnectionsNumber(json);
        updateIndividualData(json);
    }

if a message contains 'sliders' property, at main.js line x  78 -  80, it calls processSliderHandle function in render.js and changes the display of boarder around slider handle:

.. code-block::  javascript

    if (json.hasOwnProperty('sliders')) {
      processSliderHandle(json.sliders);
    }

.. code-block::  javascript

    function processSliderHandle(slider_data) {
        console.log(slider_data)
        if (slider_data.id == "slider0") {
            if (slider_data.handle == "scenario_energy_prices") {
            $("#current_energy_prices_scenario").css("border", "5px goldenrod solid"); // create thick golden border
            }
            else {
            $("#current_energy_prices_scenario").css("border", "1px goldenrod dotted"); // remove border
            }
            if (slider_data.handle == "num_connections") {
            $("#scenario_num_connections").css("border", "5px chartreuse solid"); // create thick blue border
            }
            else{
            $("#scenario_num_connections").css("border", "1px chartreuse dotted"); // create thick blue border
            }
        }
    }

Main.js line x 85 - 96 checks if a message contains 'data_view_neighborhood_data', 'neighborhood_images' or 'individual_data_view' property and calls respective functions at render.js to change the view in the individual data view:

.. code-block::  javascript

    if (json.hasOwnProperty("data_view_neighborhood_data")) {
      injectDataToDataView(json.data_view_neighborhood_data)
    }
    if (json.hasOwnProperty("neighborhood_images")) {
      renewResultsImages(json.neighborhood_images)
    }

    if (currentUserMode == 'individual_data_view') {
      if (json.hasOwnProperty("active_user_focus_data")) {
        focusActiveUserData(json.active_user_focus_data);
      }
    }

.. code-block::  javascript

    const injectDataToDataView = function (data) {
        console.log(data)
        data.forEach((data_per_iteration) => { // for each val in arr, exec func
            renewResultsImgSrcPath(data_per_iteration.iteration_round, data_per_iteration)
            console.log(GAMASimulationImgSrcPaths)
            renewDataViewGAMAImgsPerSection(data_per_iteration.iteration_round)
        });
        removeEmissionComparisonChart()
        createEmissionComparisonChart(data[0].emissions_data_paths)
    }

    const injectDataToDataView = function (data) {
        console.log(data)
        data.forEach((data_per_iteration) => { // for each val in arr, exec func
            renewResultsImgSrcPath(data_per_iteration.iteration_round, data_per_iteration)
            console.log(GAMASimulationImgSrcPaths)
            renewDataViewGAMAImgsPerSection(data_per_iteration.iteration_round)
        });
        removeEmissionComparisonChart()
        createEmissionComparisonChart(data[0].emissions_data_paths)
    }

    function focusActiveUserData(userNumber){
        // hide all quarterSections:
        for (var i = 0; i<4; i++){
            $("#dataViewIndividualQuarter" + i).css("display", "None");
        }

        // show only active quarterSection:
        let individualQuarter = $("#dataViewIndividualQuarter" + userNumber)
        individualQuarter.css("display", "grid");
    }

Main.js line 102 - 104 checks if a message contains 'final_step' and renews the global variables 'simulationFinalStep' at appData.js 
Main.js line 105 - 107 checks if a message contains 'step' property and calls updateSimulationProgress function at render.js and renews progress bar:

.. code-block::  javascript

    if (json.hasOwnProperty("final_step")) {
      simulationFinalStep = json.final_step;
    }
    if (json.hasOwnProperty("step")) {
      updateSimulationProgress(json.step)

.. code-block::  javascript

    const updateSimulationProgress = function (step) {
        const percentage = Math.ceil(step * 100 / simulationFinalStep)
        $("#simulationProgress").find('span').text(percentage)
    }

Buildings Interaction View
===========================

the function renderHouseInfo displays information of building groups that are conitaned in the incoming data.

.. code-block:: javascript

    const renderHouseInfo = function (groupData, quarterID) {
        const individualQuarter = $("#" + quarterID);
        buildings = groupData.buildings;

        if (groupData[0] == '') {
            individualQuarter.css("visibility", "hidden");
        }
        else {
            // show hidden elements:
            if (individualQuarter.css("visibility") == "hidden") {
            individualQuarter.css("visibility", "visible");
            }

            // get only first element of building list:
            const targetBuilding = buildings[buildings.length - 1];

            // update address:
            individualQuarter.find('.address').text(targetBuilding.address);

            // update building type
            target = "#" + quarterID + " > .nameAndTable > .houseInfo > .buildingType > span";
            if (targetBuilding.type == "MFH")
            $(target).text("Mehrfamilienhaus");
            if (targetBuilding.type == "EFH")
            $(target).text("Einfamilienhaus");

            // update consumption data:
            target = "#" + quarterID + " > .nameAndTable > .houseInfo > .heatConsumption > img";
            let heatConsumptionHandle = "default";
            if (targetBuilding.spec_heat_consumption > 250)
            heatConsumptionHandle = "h";
            if (targetBuilding.spec_heat_consumption < 250)
            heatConsumptionHandle = "g";
            if (targetBuilding.spec_heat_consumption < 200)
            heatConsumptionHandle = "f";
            if (targetBuilding.spec_heat_consumption < 160)
            heatConsumptionHandle = "e";
            if (targetBuilding.spec_heat_consumption < 130)
            heatConsumptionHandle = "d";
            if (targetBuilding.spec_heat_consumption < 100)
            heatConsumptionHandle = "c";
            if (targetBuilding.spec_heat_consumption < 75)
            heatConsumptionHandle = "b";
            if (targetBuilding.spec_heat_consumption < 50)
            heatConsumptionHandle = "a";
            if (targetBuilding.spec_heat_consumption < 30)
            heatConsumptionHandle = "aplus";
            $(target).attr("src", "img/qscope_energy_graph_triangle_" + heatConsumptionHandle + "_.png");

            // highlight selected decision:
            if (groupData.slider_handles.length > 0) {
            groupData.slider_handles.forEach(element => {
                individualQuarter.find("." + element).removeClass('highlightedRow')
                individualQuarter.find("." + element).addClass('highlightedRow')
            });
            }
            else {
            individualQuarter.find(".highlightedRow").removeClass('highlightedRow')
            }

            // update image:
            individualQuarter.find(".emissions_graphs img").attr("src", targetBuilding["emissions_graphs"]);
            individualQuarter.find(".energy_prices_graphs img").attr("src", targetBuilding["energy_prices_graphs"]);

        }
    }



Individual Data View
=========================


Individual data view shows detailed information of selected building group. In addition to what is displayed in the buildings interaction view,
the emissions graph and the energy prices graph

.. code-block::

    const updateIndividualData = function (data) {
        ...
        individualQuarter.find(".emissions_graphs img").attr("src", targetBuilding["emissions_graphs"]);
        individualQuarter.find(".energy_prices_graphs img").attr("src", targetBuilding["energy_prices_graphs"]);
        ...
    }




Total Data View
=========================

Total data view displayes four different graphs of GAMA simulation: energy_price, emissions_neighborhood_accu, emissions_groups, and energy_prices_groups.

.. code-block::

    const renewResultsImages = function(data){
        document.getElementById("energy_prices").src = data.energy_prices + "?update=" + new Date().getTime();
        document.getElementById("emissions_neighborhood_accu").src = data.emissions_neighborhood_accu + "?update=" + new Date().getTime();
        document.getElementById("emissions_groups").src = data.emissions_groups + "?update=" + new Date().getTime();
        document.getElementById("energy_prices_groups").src = data.energy_prices_groups + "?update=" + new Date().getTime();
    }




Developemnt tools
*****************

Developemnt tools (js/devTools.js) provides useful function for debugging.

* ``Space bar``: switch modes
* ``D``: show data view mode
* ``V``: show verbose (lines around html elements)
* ``T``: inject extra round data (a column will be added in the round information table)
* ``I``: inject sample data


Known Errors
************

``[nodemon] Internal watch failed: ENOSPC: System limit for number of file watchers reached, watch '/home/user/github/qScope/qScope_infoscreen/public/data/outputs/output_20221111_11-51-29/energy_prices/energy_prices_4.09.csv'`` can be fixed either by restarting the computer or by `increasing the system limit for number of file watchers <https://stackoverflow.com/questions/65300153/error-enospc-system-limit-for-number-of-file-watchers-reached-angular>`_.