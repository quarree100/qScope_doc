Usage
=====
Let's start by learning what this is all about.

Step-by-step
************

You'll need Python to start the **frontend**. The frontend will be started by navigating to the  project folder and do ``python3 run_q100viz.py``. A window will open and show the contents that will be projected onto the table. This is the frontend the users can see and interact with:

.. image:: img/frontend_full.png
    :align: center
    :width: 600
    :alt: You should see an image of the frontend here. It is basically a black canvas with a map on it and some buttons on the side.

You see a lot of the black canvas around a slightly distorted map. This is due to the "keystoning", the adjustment of the image for the angles the projector produces with respect to the table. By casting an appropriately distorted image onto the table, the distortion will even out, geometrically. ✨

Now we want to interact with the things we see on the canvas - the buttons, the sliders and the map. For this, we'll need cspy, which serves as the **backend**, decoding the configuration of tangibles on the table.
Start the script for each table individually by navigating to the cspy folders and do ``python3 run_keystone.py``. A window will show up to define the Region of Interest and do the keystone calibration. After doing this once, the adjustment will be saved and this step can be skipped next time.
The scanning will be started with ``python3 run_scanner.py``. The decoder will send interaction data now to the frontend script, which will react by altering the projection.

In order for the **infoscreen** to receive and process information, it has to be started by executing ``npm start`` in the q100_info folder.

You can start each program individually, but be aware that, for the handshake between the programs to succeed, it is recommended to follow :ref:`a certain order <Run all>`.

How to use the Frontend
***********************

TODO: how to use the frontend as a user, once everything is set up

Backend
*******

Infoscreen
**********

.. _Run all:

Running it altogether
*********************

TODO: get script from Q-Scope and show the order here