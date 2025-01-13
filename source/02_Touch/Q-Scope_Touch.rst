Q-Scope Touch
#############

The second iteration of the Q-Scope uses a TouchTable by Interactive Scape.

Specifications
**************

Device: ``Data Display AG P4KDSE 2160P`` providing a resolution up to 4096x2160 pixels. For the frontend application, we use a resolutoin of 3840x2160 pixels.

Usage
*****

In order to use the TouchTable with your/any computer, you have to set the switch on the bottom side of the TouchTable to "USB TOUCH". You can then use it as an input device for your machine. 

.. attention:: The touch input will most likely be mapped to your primary screen (tested with Linux/Ubuntu). By default, the touch input will neither be mapped to the resolution nor the orientation of your primary screen. 
    
**In order to get the touch events** where they belong (**onto the touch table**), you have to help yourself out using a command like this:

``xinput --map-to-output [touch_input_device] [touch_table_output_device]``, in our case: ``xinput --map-to-output "Advanced Silicon S.A. CoolTouch   System" "DP-1-7"``

.. hint:: You can find out about the input and output device names using ``xinput --list`` and ``xrandr``