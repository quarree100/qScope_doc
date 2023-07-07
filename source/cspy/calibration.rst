.. _cspy_calibration:

Calibrating the Webcams
***********************

The webcams have to be calibrated so that the *ROI* (region of interest) extends properly over the whole area of the table underside.

#. Run ``python3 run_keystone.py``.
#. Select 4 corner points by clicking on each of them in the order: up right, up left, bottom right, bottom left. (The program will quit once the initial keystone points were saved.)
#. Run ``python3 run_scanner.py``.
#. Press numbers ``1``, ``2``, ``3``, ``4``, to select the corner points.
#. Move the corner using ``WASD`` keys until the grid cells in the edges are placed neatly in the edges of the ROI and the white grid matches the grid of the physical table.
#. The step size for the displacing of the corner points can be toggled hitting `SPACEBAR`. A red arrow indicates slight movements, while a blue arrow indicates bigger steps.
#. Once everything is positioned nicely, press ``k`` to save the corner points to ``keystone.txt``. Corner points will be loaded from that file upon next start of the script.

.. hint::
    Placing non-white tokens in the very corners can be very helpful to set the absolute edges of the ROI.