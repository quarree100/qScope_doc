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

..  code-block:: none

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


Modes/Views
=========================
todo:
- buildingsInteraction view
- individualDataview
- totalDataView


developemnt mode
=========================

other stuff
=========================
- valuables 
- expected data
