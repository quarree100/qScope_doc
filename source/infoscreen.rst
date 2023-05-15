Infoscreen
##########

Infoscreen is a data visualization software of QUARRE100. It receives data from the Q-Scope Frontend and projects graphs on a screen.

Installation
****************************
Pre-requisit
=========================
To run the inforscreen, the machine needs a nodejs environment.

Installation
=========================

..  code-block:: bash

    :caption: Clone the repository to local
    git clone git@github.com:quarree100/qScope_infoscreen.git``
    
    :caption: install node modules
    cd qScope_infoscreen
    npm install
    
    :caption: setup a folder public/data as a symlink pointing to qScope/data
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

..  code-block:: bash

    :caption: run npm shortcut command
    npm start #equivalent to ``npm run start``, which is defined in package.json. "start" script runs ``npx nodemon q100_info.js``

Architecture
=========================
Infoscreen is primarily a node js application. q100_info.js runs a web application using expressjs, and communicate with Q-Scope Frontend via UDP messaging using socket.io. Rendering of views are done with html+css+javascript at public folder. 

Modes/Views
****************************

Buildings Interaction View
===========================
Tiles of information card shows the information of 4 different selected buildings.
In each tile, at the top the name of the building is stated. Under it three icons indicates the options that user selected (?) for the current round; the connection to the heat grid (triangle), refurbishment status (green plus), and energy consumption preference (leaf). The Heat consuption is indicated in a chart in 9 scale from A+ to H.
In the table these three options are displayed throughout rounds.

Individual Data View
=========================
The individual data view displayes the information of one selected building. In addition to the information in the buildings interaction view, emissoins over time and energy costs per household over time are shown in charts.

Total Data View
=========================
There are four different graphs to show the result of the simulation.
1. Energiekosten im Vergleich / Energy costs in comparison
shows the change of the energy cost over time per building.

2. Generalle Energiepreise nach Energietraeger / General energy prices by energy carrier
shows the change of price of energy prices per energy carrier.

3. Montaliche Emissionen im Vergleich / Montaliche emissions comparison
shows the change of monthly CO2 emissions over time per building.

4. Quartiersemissionen und Waermenetzanschluesse / Neighborhood emissions and heating network connections
shows the CO2 emissions of the whole neighborhood over time and the number of the connections to the heat grid over time.



developemnt mode
=========================

other stuff
=========================
- valuables 
- expected data
