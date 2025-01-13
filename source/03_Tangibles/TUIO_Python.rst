
TUIO and Python
===============

We use the python package pythontuio to set up a Tuio Client  in a subthread to listen for messages:

.. code-block:: python

    # tuio: 
    tuio_listener = Tuio_Listener()
    
    tuio_client = TuioClient('10.42.0.1', 3333) # these settings usually come from config.py, so go there to edit them!
    tuio_thread = threading.Thread(target=tuio_client.start, daemon=True)
    tuio_client.add_listener(tuio_listener)
    
    tuio_thread.start()

.. hint:: If you want to use the script with the TUIOSimulator script, you should set the hostname to 'localhost' (instead of the IP shown above).

The general Tangible interaction is handled in the script `Tangibles.py <https://github.com/quarree100/qScope_frontend/blob/main_tangibles/q100viz/interaction/Tangibles.py>`_. Here we define a Tangible class to process position and rotation changes.  We override the standard functions provided by the pythontuio module to trigger our custom logics:

.. code-block:: python

    import pythontuio

    class Tuio_Listener(pythontuio.TuioListener):

        # cursors (touch events) are not used in the Q-Scope Tangibles version:
        def add_tuio_cursor(self, cursor: pythontuio.Cursor):
            pass  # no action defined

        def update_tuio_cursor(self, cursor: pythontuio.Cursor):
            pass

        def remove_tuio_cursor(self, cursor: pythontuio.Cursor):
            pass

        def add_tuio_object(self, object: pythontuio.Object):
            devtools.print_verbose(
                f"Neues Tangible hinzugefügt: ID={object.class_id}, X={object.position[0]}, Y={object.position[1]}"
                )

            if object.class_id >= 0:
                session.tangibles[object.class_id] = Tangible(object) # create new Tangible Object

        def update_tuio_object(self, object: pythontuio.Object):
            if object.class_id < 0: return

            # create object if tangible was already on surface:
            if not object.class_id in session.tangibles.keys():
                session.tangibles[object.class_id] = Tangible(object)

            session.tangibles[object.class_id].update(object)

        def remove_tuio_object(self, object: pythontuio.Object):
            devtools.print_verbose(
                f"pythontuio.Object entfernt: ID={object.class_id}"
                )
            if not session.tangibles[object.class_id]: return

            session.tangibles[object.class_id].destroy_me = True  # set flag to delete Tangible Object in main thread
        

TUIO Simulator
***************
There is a great tool for developing TUIO-borne applications (if you are using TUIO 1.0): The TUIO_Simulator! It is a small java application I included in the :ref:`install script collection<bundle-installation>`.
You can start it using `java -jar ./TuioSimulator.jar` from your terminal inside the script's folder. Then you can 

* click and move your mouse to move an object ON the table
* shift-click and move your mouse to move an object OVER the table
* right-click and move the mouse to rotate an object 
