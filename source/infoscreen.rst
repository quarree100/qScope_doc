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
In each tile, at the top the name of the building is stated. Under it three icons indicates the options that user selected (?) for the current round;

|icon_heat_grid| the connection to the heat grid

|icon_green_plus| refurbishment status

|icon_save_energy| energy consumption preference.

The Heat consuption is indicated in a chart in 9 scale from A+ to H.

.. image:: img/infoscreen_energy_graph.png
    :align: center
    :alt: []
    :height: 4ex

In the table these three options are displayed throughout rounds.

Individual Data View
=========================

.. image:: img/Infoscreen_03a_individualDataView.png
    :align: center
    :alt: [close-up of a green square showing information of one of the selected houses and two graphs with predicted CO2-emissions and energy costs up until the year 2045.]


The individual data view displayes the information of one selected building. In addition to the information in the buildings interaction view, emissoins over time and energy costs per household over time are shown in charts.

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
=========================

Developemnt tools (js/devTools.js) provides useful function for debugging.

Space bar: switch modes

D: show data view mode

V: show verbose (lines around html elements)

T: inject extra round data (a column will be added in the round information table)

I: inject sample data


UDP communication API with the Frontend application
=====================================================

a message to switch modes should follow the following structure:

.. code-block::

    {
        "mode" : string # "buildings_interaction", "individual_data_view" or "total_data_view"
    }


a message which contains building data should follow the following structure:

.. code-block::

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

