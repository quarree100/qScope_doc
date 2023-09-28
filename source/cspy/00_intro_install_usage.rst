.. _cspy:

Introduction, Installation and Usage
####################################

Introduction
************

The framework is based on `MIT's CityScope platform <https://cityscope.media.mit.edu/>`_ and was further developed by the `City Science Lab at HafenCity University in Hamburg <https://www.hcu-hamburg.de/research/csl>`_ (CSL_HCU) and the `Department of Resilient Energy Systems <https://www.uni-bremen.de/res>`_ at the University of Bremen.
The code we used is indeed a combination of the MIT cityscopy scanner and further enhancements by the CSL_HCU. The algorithm splits the webcam stream into a grid of cells that are again separated into four quarters. Each quarter of each cell shows a dot indicating that the containing quarter is below a certain lightness threshold (→ black dot) or above it (→ bright dot). This way, cspy can register a set of 2^4 = 16 tags (neglecting their rotation), or about 3.5 absolute rotation-variable tags.

.. image:: ../img/cspy_calibrated.png
    :align: center
    :alt: Still image of the webcam stream, separated into a grid of cells that will be observed. At the bottom, sliders can be seen that are detected, too. Black dots indicate that an overall black quarter was found in one of the cells. Some information on the left hand side of the window show user controls for the calibration of the detection.

Above you see an image of the output of cspy, with the camera stream split into a grid of cells that is to be monitored. Below, there are two sliders that can be observed, too. Some information on the left hand side of the window show user controls for the calibration of the detection.

.. _installing_cspy:

Installation
************

Make sure you have your :ref:`camera drivers<realsense>` installed properly and your cameras are ready for usage.
After that, the installation will hopefully just as easy as downloading `the decoder's repository <https://www.github.com/quarree100/cspy>`_ and doing ``pip3 install -r requirements.txt``.

.. note::
    You should copy the downloaded folder to be able to run two instances of the software, each using their own settings.
    .. TODO: this could be done in an easier way, using a startup flag to define which settings file to load from (and save to)

Usage
*****

Since we are working with two physical tables and, thus, with two cameras, two instances of the software have to be started, each with their own keystone-calibrated settings.

**1.** Run ``python3 run_scanner.py [settings_file]``.

([settings_file] can for example be ``settings/qscope_R.json`` for the table on the right). But first, the webcams will have to be calibrated.

.. _cspy_calibration:

Calibrating the Webcams
=======================

Keystoning
----------

The webcams have to be calibrated so that the *ROI* (region of interest) extends properly over the whole area of the table underside.

**1.** Run ``python3 run_keystone.py [settings_file]``.

.. image:: ../img/cspy_00_keystoning.png
    :align: center
    :alt: image of underside of table with the extent of the whole physical grid

This is for roughly setting the corner points of the ROI. You will see the full extent of the camera stream. The camera should be installed so that the whole physical grid can be seen.

**2.** Now Select 4 corner points by clicking on each of them in the order: up right, up left, bottom right, bottom left. (The program will quit once the initial keystone points were saved.)

**3.** Run ``python3 run_scanner.py [settings_file]``.

.. image:: ../img/cspy_01_calibration_blue.png
    :align: center
    :alt: distorted image of webcam stream, as set in run_keystone.py in the step before

You now see a close-up of the region of interest.

**4.** Press numbers ``1``, ``2``, ``3``, ``4``, to select the corner points.

**5.** Move the corner using ``WASD`` keys until the grid cells in the edges are placed neatly in the edges of the ROI and the white grid matches the grid of the physical table.

**6.** The step size for the displacing of the corner points can be toggled hitting `SPACEBAR`. A red arrow indicates slight movements, while a blue arrow indicates bigger steps.

**7.** Once everything is positioned nicely, press ``k`` to save the corner points to ``keystone.txt``. Corner points will be loaded from that file upon next start of the script.

.. hint::
    Placing non-white tokens in the very corners can be very helpful to set the absolute edges of the ROI.

.. _cspy_detection_settings:

Detection Settings
------------------

In total, more than the one webcam stream window is created with cspy - there are two additional windows that help finding the best detection conditions.

.. image:: ../img/cspy_all_windows.png
    :align: center
    :alt: Ideal calibration situation with three windows showing the original RGB webcam stream, the resulting image after brightness and threshold adjustments, and a greyscale overlay to even out uneven light distribution.

When running cspy, three windows are created: one showing the original RGB webcam stream, a second one showing the resulting (binary) image after brightness and threshold adjustments, and a third one showing a greyscale gradient overlay to even out uneven light distribution.

In an ideal calibration (like in the image above), the **binary image** should show the tags' black quarters as perfect white squares. No light should leak through the grid.

Setting up the best detection conditions requires a lot of fine-tuning of brightness and threshold levels. For this, a set of tools can be used via a keyboard:

**general:**

* ``+`` and ``-`` define whether your values will increase or decrease.
* ``SPACEBAR`` sets the magnitude of the change of the values.
* ``u``: toggle the gui interface

**1. lightness thresholds:**

* ``v``: change the overall pixels' luminance threshold.
* ``e``: (realsense-specific) adjust the camera's exposure
* ``g``: (realsense-specific) adjust the camera's gain

* ``q``: adjust the brightness threshold of the cell pixels' quantiles **hint: this will result in significant detection changes**

**2. gray gradient overlay:**

* ``5``: change the lower end of the grey gradient overlay
* ``6``: change the upper end of the grey gradient overlay

**3. sliders:**

* ``j``: toggle active slider through list of available sliders
* ``l``: change luminance threshold of active slider
* ``f``: change active slider's `a`-value
* ``b``: change active slider's `b`-value
* ``y``: change the y poisition of the active slider
* ``x``: change the left x position of the active slider
* ``c``: change the right x position of the active slider

After the calibration is complete, the values can be saved hitting ``k`` and will be written to the opened :ref:`settings file<cspy_settings>`. (You can select the settings file by appending it to the python execution command like so: ``python3 run_scanner.py settings/qscope_L.json``)

.. TODO: merge ``feature_export_calibration`` and ``beautifications`` to ``main``

.. hint:: Recommendation: place y-position of slider slightly ABOVE the slid, so you don't try to decode what's on the ceiling and other interferences with people.

.. attention:: Not all of these tools might work for you, since they are programmed specifically for the cameras we used. The exposure and gain controls only work for realsense cameras. For any other features you would have to implement your own functions.
