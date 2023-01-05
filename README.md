# Topological Map Robot Control 2.0

_Development of the first assignment of experimental robot laboratoy._

Intriduction
==============
This assignment proposes to develop software architecture defined for the first assignment [tmrc1.0](https://github.com/mrhosseini75/tmrc.git).
In this assignment a Gazebo and Rviz were used to demonstrate robot movements on the map. In addition, in this development, the robot has a manipulator with a camera on board to detect the environment.
<p align="center">
<img width="600" src="https://user-images.githubusercontent.com/80394968/210667608-1dbdbb74-08cd-4158-9fc2-3ea1b981310e.jpg" alt="600">
</p>
<p align="center">
The map considered for the assignment
</p>


Tools
=======

To solve the following problem I used various tools, like:<br/>
1. *aRMOR*: it was used to `load` topological map, `add` and `manipulation` rooms, and get `query` from information.
2. *Move_it*: it was used to build urdf for robot model and make plan trajectory to move the robot arm_link.
3. *Move_base*: it was used to move robot base_link.
4. *Slam_gmapping*: it was to mapping enviroment detected by robot.
5. *SMACH viewer*: to view how the robot state 

Documentation
==============

For more information on nodes and sofar then refer to [Documentation](https://mrhosseini75.github.io/tmrc2/)

Software architecture
========================

<p align="center">
<img width="1000" src="https://user-images.githubusercontent.com/80394968/210672379-24db3928-c19b-42b4-b0d9-ebd34d164e6b.jpg" alt="1000">
</p>
<p align="center">
UML graph
</p>

The whole software structure is divided into 5 important parts:
  1. Python nodes
  2. Cpp nodes
  3. Helper scripts
  4. URDF
  5. Packages
 <br/>

/1. Python nodes/
---------------------
- **Finite State Machine(FSM):**

  FSM is a mother node of the whole architecture, it uses the **build_ontology_map.py** to import a class defined to ADD, MANIPULATION, and
  get QUERY from the information. Moreover imports three important services defined in **set_object_state.py** to set the state of the battery, 
  base movement, and arm movement. This node subscribes to *marker_publisher.cpp* node to get ``/image_id`` detected by the robot camera and this 
  id number is communicated with *marke_server.cpp* node to get information about id founded. When the robot wants to move after detecting all markers id
  published a Move Base Action to the *move_base* package to move base to the robot.
  Finally uses a service response to get Pose, Base movement state, and arm movement state from the *robot_states.py* node. 

- **Robote States:**

  Robot states subscribe to the ``/odom`` topic from robot urdf to get information odometry information.
  All services do set and get here, the services are: ``base_movement_state`` and ``battery_level_state``.
  Additionally ``Get_pose`` service done in this node and the I/O of the service communicates via **<response & requist>** with FSM node
  and helper scripts. 

/2. Cpp nodes/
----------------------
- **Marker Server:**

  All room proprieties are defined in this node like room id and room details (position, connection, door, last visit time).
  This node communicates with the FSM node via service (Req/Res) to send room information. 

- **My Moveit:**

  This node contains all necessary joint configurations to find boxes in the environment. Via service communicates with a helper script
  *set_object_state.py* to changes state of robot arm between *True* and *False*
  each time the arm state is *True* this node becomes active and sends joint position information to
  *robot_assignment.urdf*. Otherwise, if it is * False* the robot arm stop working.

/3. Helper scripts/
----------------------
*These scripts don't start any node they were used only to import essential functions used in node FSM and robot_state*

- **Build Ontology Map:**

  This script uses a service response generated from the *robot_state.py* node to get the current position
  of the robot in the map and update the **is_in** method. This method indicates the current location of
  the robot. Furthermore, this script is useful to load ontology_map defined in the armor package ad sync with 
  the resoner.

- **Set Object State:**

  Three important services for the state of the battery_state, base_movement_state, and arm_movement_state are defined here.
  This script sends a service request to the *robot_state.py* node to set the base movement state and set battery level 
  state. Additionally sends a service request also to *my_moveit.cpp* node for set arm movement state.

/4. URDF/
----------------------
- **Robot Assignment:**

  Its generated from * Moveit Setup Assistant* and contain a robot model and various topic useful 
  for moving the base and controlling the arm of the robot. Robot urdf subscribes to *move_base* to get 
  velocity uses ``/cmd_vel`` topic and publishes odometry information uses ``/odom`` topic to 
  *robot_state.py* node to get the current position of the robot in the ontology map, furthermore publishes 
  ``/robot_camera/image_box`` to *marker_publisher.cpp* node to send image detected via camera.

  Finally publishes base information using the ``/scan`` topic and  publishes tf transform tree [1] uses
  ``/tf`` topic to *slam_gmapping* to update robot situation in map.

  [1]
  tf is a package that lets the user keep track of multiple coordinate frames over time. tf maintains
  the relationship between coordinate frames in a tree structure buffered in time, and lets the user 
  transform points, vectors, etc between any two coordinate frames at any desired point in time.


/5. Packages/
------------------------
- **Aruco Ros:**

  Aruco library provides real-time marker-based 3D pose estimation using AR markers. In this 
  assignment used *marker_publisher.cpp* node to publishes ``/image_id`` detected from 
  ``/robot_camera/image_box`` and its published to *FSM* node.<br/>
   - For more information on Aruco package then refer to this [link](http://wiki.ros.org/aruco_ros).

- **SLAM Gmapping:**

  The gmapping package provides laser-based SLAM (Simultaneous Localization and Mapping),
  as a ROS node called slam_gmapping. Using slam_gmapping, you can create a 2-D occupancy
  grid map (like a building floorplan) from the laser and pose data collected by a mobile robot.
  Slam_gmapping used ``/scan`` and  ``tf`` topic data generated by robot urdf in order to update 
  the position of the **base_link** and **arm_link** in the map.<br/> 
   - For more information on the Slam package then refer to this [link](http://wiki.ros.org/gmapping).

- **Move Base:**

  The move_base package provides an implementation of an action that, given a goal in the world,
  will attempt to reach it with a mobile base. The *move_base* node links together a global and 
  local planner to accomplish its global navigation task. Move base to get an Action from *FSM* node 
  and publish velocity via ``/cmd_vel`` to robot urdf. <br/>
   - For more information on the Move base package then refer to this [link](http://wiki.ros.org/move_base).

- **Assignment Moveit:**

  After building the *urdf* or *xacro* file you can proceed in your terminal with the command 
  ``roslaunch moveit_setup_assistant setup_assistant.launch`` and run Moveit setup assistant.
  Now there is a possibility to define various properties for robot models and generate new
  urdf file plus package where there is an essential configuration with a launch file to be able 
  to use a robot model.<br/>
    - For more information on the Moveit package then refer to this [link](https://moveit.ros.org/).

- **aRMOR:**

  aRMOR is a powerful and versatile management system for single and multi-ontology architectures under ROS.
  It allows loading, querying, and modify ontology map. aRMOR add and manipulate room and door in ``ontological_map.owl``
  and send the query as feedback to the *FSM* node. Additionally, it works parallel with helper script ``build_ontology_map.py``
  to load the ontology map and synchronize with ``armor_client.py`` each time ontology is updated.<br/>
   - For more information on the armor package then refer to this [link](https://github.com/buoncubi/armor_py_api).


rqt graph
===========

<p align="center">
<img width="800" src="https://user-images.githubusercontent.com/80394968/210673602-a8dac878-3c91-4291-ae3e-4e21890535b5.png" alt="800">
</p>
</p>
<p align="center">
rqt graph with active nodes
</p>



Temporal diagram
=================
<p align="center">
<img width="1000" src="https://user-images.githubusercontent.com/80394968/210673863-602bac1a-ca7a-480f-9638-e3c3641b437e.jpg" alt="1000">
</p>
</p>
<p align="center">
temporal diagram 
</p>

At the beginning Launch file launch all the nodes and packages except ``finite_state_machine.py``.
The last one started working automatically with the first launch after 20 sec of delay, this delay is a 
necessary because simulation needs some time to add a robot model in the environment and make it able to execute 
function defined in *FSM* node. 

In the first step **SETUP_MAP**, slam_gmapping subscribe to ``/scan`` and ``/tf`` generated from robot urdf <Gazebo> and
also my_moveit.cpp subscribe to ``/tf`` information to uses for ``/arm_controller``. At the tip of the robot arm, there is a camera 
that helps the robot detect marker_id in the world. The image_od acquired by the camera publish to FSM, this node uses
a service(req & res) to get information about the room id detected. After finishing this step **GO_IN_ROOM** state started and FSM send 
an action to the move_base package, therefore move_base publishes ``/cmd_vel`` to robot urdf <Gazebo> to move the robot around the map.
The information related to odometry is sent to robot_state.py and the last node mentioned store information in ``Get_pose.srv``.

- N.B: There are 5 states in the FSM node, here demonstrated just two of them. At the beginning of each state  ``/arm_movement_state`` and
``/base_movement_state``are changed and ``/battery_level_state`` after the first state starts consuming.

  
Usage
========

- You have to creat a repository and named assignment_ws, to do that
  ```
  $ mkdir assignment_ws /src
  $ cd src
  ```
  
- Frok repository from my Github 
  ```
  $ git clone https://github.com/mrhosseini75/tmrc2.git
  ```
  
- You need to install following package in order to run assignment:
  
   - [Moveit](https://moveit.ros.org/install/docker/) package with all dependencies
   - [Move base](https://github.com/ros-planning/navigation.git)
   - [slam gmapping](https://github.com/ros-perception/slam_gmapping.git)
   - [SMACH viewer](https://www.oreilly.com/library/view/ros-robotics-projects/9781838649326/30a9e4d8-1df0-4db4-8607-b8341693a830.xhtml)

- Now you can go to the work space and build packages
  ```
  $ cd ..
  $ catkin_make
  ```
  
- After that you can launch assignment with following command
  ```
  $ roslaunch assignment2 assignment.launch 2> >(grep -v TF_REPEATED_DATA buffer_core)
  ```
  
- NB: There is an possibility to change battery level manually, use fallowing command
  ```
  $ rosservice call /state/set_battery_level "battery_level: <value>"
  ```
  
 **!!!WARNING!!! <br/>**
  
 1.When you launch the files it loses 20 seconds to start working, this delay is due to the time you add the robot in the environment and run the moveit package.
  
 2.In scripts, the system path has been specified in order to use the carried package. As following way:
  ```  
  import sys
  sys.path.append('/root/assignment_ws/src/tmrc2/assignment2')
  ```
 if you clone repository in diffrenet path, to have to update this line of code.
  
 3. If you try to run multiple times, the assignment generates too many logs and this makes your pc slower. This happens due usage of TF in the  assignment and to solve that you need to use the following command. 
  ```  
  $ rosclean purge
  ```
  
Video
========

- Rviz view state --> SETUP_MAP, GO_IN_ROOM, INSPECT_ROOM
 ------------------------------
  
https://user-images.githubusercontent.com/80394968/210685982-1223b2b8-5b33-4ca0-8ea1-b9e60b1342c4.mp4

- Gazebo view state --> GO_IN_ROOM, GO_TO_CHARGER
 ----------------------------------------------------
  
https://user-images.githubusercontent.com/80394968/210690781-7f23f2bb-d1d9-4d3e-9a2b-8a5bc590ba2b.mp4

- Gazebo view state --> GO_TO_CHARGER, WAIT FOR CHARGING, GO_TO_ROOM
 -----------------------------------------------------------------------------
  
https://user-images.githubusercontent.com/80394968/210691493-4793de64-bf74-477b-8a87-e161d82e2a41.mp4
  
- Marker_publisher/result 
 --------------------------

https://user-images.githubusercontent.com/80394968/210687196-a75ef8a5-3184-4f2f-be01-b68312185a12.mp4


Hypothesis and environment
===========================

**System’s features**

- The robot runs the arm to find the marker_id and starts patrolling on the map automatically.
- You can view what the robot sees via the camera that is on board the robot.
- Every time the robot moves to a new place the map updates and we can see via Rviz.

**System’s limitations**

- Sometimes robot fails to turn well and maybe hits the wall, this limitation is probably caused by not being optimized value angular velocity compared to linear velocity.

- The map updates slowly and this causes the robot to generate a wrong path, but after updating map generates a new correct path.

- Sometimes when he is exploring a room before finishing movement with the arm, the robot base starts to move. This is caused due to high control frequency.


**Possible technical Improvements**

- using an octomap to map the environment can be an interesting problem. On board, the robot is a 3D sensor, which saves information in  ``/head_mount_kinect/depth_registered/points ``. The last one can be used in octomap to cloud point there.
- Robot sometimes generates the wrong path and choose the wrong way to reach the target, but finally, find the correct one. The only thing that increases the time to get robot to a target. This may be the best thing to fix.


Autor and contact
==================

- Mohammad Reza Haji Hosseini
- email: <mrhhosseini75@gmail.com>
