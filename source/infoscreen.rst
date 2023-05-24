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


Buildings Interaction View
===========================


Individual Data View
=========================

Total Data View
=========================

developemnt mode
=========================

UDP communication API with the Frontend application
=====================================================

