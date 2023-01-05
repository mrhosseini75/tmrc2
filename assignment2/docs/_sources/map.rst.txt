Map
==========

Environment
------------

The general mapping was built in this form. I considered an environment **10x10** and for each 
room I considered a point so the planner starts making a connection between points to go from
one room to the other. The initial position of the robot is considered in room E and every time
the battery robot is low it must return to this room and recharge itself.

.. image:: image/map.jpg
  :width: 1000
  :align: center
  :alt: map.jpg


SMACH viewer
------------

After launching **smach viewer**, you can see robot status changing between rooms and wait for a 
few seconds in each room.

.. image:: image/smach.jpg
  :width: 1000
  :align: center
  :alt: smach.jpg