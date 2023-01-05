Temporal diagram
=================



.. image:: image/temporal_diagram.jpg
  :width: 1000
  :align: center
  :alt: temporal_diagram.jpg
   
At the beginning Launch file ``topological_map_robot_control.launch``,
starts six nodes and one armor service. The service works directly
with the ``FSM`` node and the ``topological_map.owl`` map. The service
allows the ``FSM`` node to modify, check, and load the ``topological_map.owl`` map.

In addition Launch file sets the size of the environment at the ``planner``
node and the initial position of the robot in the environment at the ``controller_client``
node. Parameters were used to set the values.

``FSM`` node publish ``\target_point`` to ``planner_client``. Planner make a path to use
different point to reach goal and publish ``\path`` to ``controller_client`` node.

Nodes in pairs like MOTION PLANNER and MOTION CONTROLLER start working with
each other through ``SimpleAction``. The first node has role of server and the second 
node has role of client, client uses ``CALLBACK`` to connect to server and server uses
``FEEDBACK`` to control status of client.

