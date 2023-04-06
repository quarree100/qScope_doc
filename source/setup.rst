Setup
=====
Setting up the physical table, combining all the necessary components (cspy, infoscreen, frontend)

Hardware Overview
*****************

.. image:: img/CAD_closeUp_annotated.png
    :align: center

Tangibles
^^^^^^^^^

TODO: different tags, usable vs. unusable

Software Overview
*****************

.. image:: img/Q-Scope_software_components.png
    :align: center
    :alt: Here you should see a schematic overview on the different software components the Q-Scope setup requires: cspy, frontend, infoscreen, abm

The Q-Scope setup consists of a table with a grid of tangible tiles that will be scanned and decoded by cspy. The software then sends the grid information to the frontend, which will project interaction information onto the table accordingly and send information on the machine state to the infoscreen to display metadata. A GAMA Agent-Based-Model (ABM) can be executed via interaction on the table. The data it outputs is stored locally and loaded by the infoscreen to display comprehensive graphs.

Installation
************

go to github and download the following repositories:

I would recommend putting them all into one project folder like so:

.. code-block::

    project qScope
    └───cspy_L
    │   Token Tag Decoder (for left table)
    └───cspy_R
    |   Tag Decoder for the right table
    └───data
    └───q100_abm
    └───qScope_infoscreen
    │   infoscreen (NodeJS/ JavaScript)
    └───qScope_frontend
        projection (Python)

where:

* cspy_L and cspy_R: https://github.com/quarree100/cspy (you'll need this script twice - one for each table)
* data: has to be linked from Seafile server as discussed :ref:`below <Data>`.
* GAMA: https://github.com/quarree100/q100_abm
* qScope_infoscreen: https://github.com/quarree100/qScope_infoscreen
* qScope_frontend: https://github.com/quarree100/qScope_frontend

.. _Data:

Data
****

TODO: