Infoscreen API
####################

Overview
***********************

This section explains data structre of UDP messaging between the frontend application and the infoscreen.


UDP communication API with the Frontend application
=====================================================

a message to switch modes should follow the following structure:

.. code-block::

    {
        "mode" : string # "buildings_interaction", "individual_data_view" or "total_data_view"
    }

TODO: :ref:`messages received from the frontend<frontend_udp_message>` are processed using the following functions: