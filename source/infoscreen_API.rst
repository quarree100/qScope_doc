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
