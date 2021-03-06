---
title: Week 1
---
# Week 1

---

## What are we doing today?
- Introductions
- Introduction to ROS
- Setting up the IGVC VM

---

## Initial Instructions
- We have a ~4 GB VM on a few flash drives
- Working with ROS 1 is best done on Linux
- This is a VM image containing Ubuntu, a distribution of Linux
- Easy way of running Linux without dual-booting
    - We do encourage dual-booting though

---

## ROS Training Instructors

----

## Meet Oswin
<p class="stretch"><img src="https://i.imgur.com/ILzYimh.jpg" alt="drawing" width="150"/></p>

- Oswin So
    - 2nd Year CS Major
    - Inside RoboJackets: IGVC Software Lead
    - Outside RoboJackets: There is no outside (Just kidding. I'm a pretty big anime fan.)
- Contacts
    - Slack: @oswinso
    - email: oswinso@gatech.edu

---

## IGVC
- Intelligient Ground Vehicle Competition
- Autonomous navigation competition
- Drive between two white lines dodging obstacles, driving to GPS waypoints

---

## What is ROS?
- Robotic Operating System
- Message passing framework for programming robots

---

## How does ROS work?
- Small programs, called **nodes** communicating with each other
- Each **node** only responsible for one aspect
    - Makes the stack cleaner
- Example Nodes:
    - Identifying Barrels
    - Localization (Figuring out where we are)
    - Mapping
    - Path Planning

---

## VM Setup

----

### Ensure you have virtualization turned on in your BIOS
[Guide](http://www.howtogeek.com/213795/how-to-enable-intel-vt-x-in-your-computers-bios-or-uefi-firmware/)
on how to do this

----

### 1. Go to `File -> Import Appliance`
<p class="stretch"><img src="https://i.imgur.com/keQmMy4.png"></p>

----

### 2. Select the `igvc.ova` file
<p class="stretch"><img src="https://i.imgur.com/3S2Pgt3.png"></p>

----

### 3. Increase the amoutn of memory and CPUs
- Allocate depending on your computer's specs

![](https://i.imgur.com/P8Adm2a.png)

----

### 4. Hit `Import`!

----

### Boot your new VM
- Double click the entry

---

## Build the training repo
- Open a terminal window

```bash
cd catkin_ws
catkin_make
```

---

## Running ROS Nodes

----

### roscore
- Before we can run any nodes, need to run roscore:
    ```bash
    roscore
    ```
- Don't worry about the details yet
- Think of it as something that handles ROS functionality

----

### Launching `buzzsim`
- Let's launch the simulator:
    ```bash
    rosrun igvc_buzzsim buzzsim
    ```
<img class="stretch" src="https://i.imgur.com/jsRD8i5.png">

----

### `rosrun` in detail:
```bash
rosrun {package name} {executable name}
```

- ROS package is an organizational unit under which we can put ROS nodes, libraries, scripts and more
- For example, `rosrun igvc_buzzsim buzzsim`
    - We are running the `buzzsim` executable which lives in the `igvc_buzzsim` package

---

## ROS Topics & Messages
- Let's try and move the turtle around
- We need to communicate to the simulator node with ROS **Topics** and ROS **Messages**

----

## ROS Topic & Messages
- Think of a ROS **topic** as an an address that messages are sent to
- The ROS **message** is the actual message sent on that topic

----

## Example: Buzzsim
- Buzzsim node listens to messages on the `/oswin/velocity` topic to make the turtle move.

<img class="stretch" src="https://drive.google.com/uc?export=download&id=1eMJvdkT4IcLWoMIsSyF7P_bzkNNJW8yU">

---

## Making the turtle move in Buzzsim
- We can send messages to the `/oswin/velocity` topic to control the turtle with our keyboard!
- Use someone else's node `teleop_twist_keyboard` to do that:
    ```bash
    rosrun teleop_twist_keyboard teleop_twist_keyboard.py cmd_vel:=/oswin/velocity
    ```
- We're launching the script `teleop_twist_keyboard.py` from the package `teleop_twist_keyboard`
- You should be able to control the turtle now!

----

## Making the turtle move in Buzzsim
- The `teleop_twist_keyboard` node **publishes** the velocity commands to the `/oswin/velocity` topic
- The `buzzsim` node **subscribes** to the `/oswin/velocity` topic and uses the received messages to move the turtle

<img class="stretch" src="https://drive.google.com/uc?export=download&id=1IMTA2zjoZJ-V2lv87u_clImAZ7xkhvEK">

---

## **Publishers** and  **Subscribers** in ROS
- `teleop_twist_keyboard` **publishes** to a topic, it's a ROS **Publisher**
- `buzzsim` **subscribes** to a topic, so it's a ROS **Subscriber**

ROS **Topics**, **Messages**, **Publishers** and **Subscribers** are the core parts of ROS

---

## `rostopic`
- To find out more information about ROS Topics, we can use `rostopic`:
```bash
rostopic
# rostopic is a command-line tool for printing information about ROS Topics.
# Commands:
# 	rostopic bw	display bandwidth used by topic
# 	rostopic delay	display delay of topic from timestamp in header
# 	rostopic echo	print messages to screen
# 	rostopic find	find topics by type
# 	rostopic hz	display publishing rate of topic    
# 	rostopic info	print information about active topic
# 	rostopic list	list active topics
# 	rostopic pub	publish data to topic
# 	rostopic type	print topic or field type
# 
# Type rostopic <command> -h for more detailed usage, e.g. 'rostopic echo -h'
```

----

### `rostopic list`
- Let's try out `rostopic list`
    ```bash
    rostopic list
    # /oswin/velocity
    # /rosout
    # /rosout_agg
    # /tf
    ```
- Notice that the `/oswin/velocity` topic is shown

----

### `rostopic info`
- We can get more information about the `/oswin/velocity` topic with `rostopic info`:
    ```bash
    rostopic info /oswin/velocity
    # Type: geometry_msgs/Twist
    #
    # Publishers:
    # * /teleop_twist_keyboard (http://robojackets:46001/)
    #
    # Subscribers: 
    #  * /buzzsimsim (http://robojackets:36899/)
    ```
- The *Type* of a topic is the type of message for that topic
- In this case, it is `geometry_msgs/Twist`

----

### Message types
- Since `geometry_msgs/Twist`, which comes from the package `geometry_msgs` package from ROS, we can
lookup the definition in the [ROS docs](http://docs.ros.org/melodic/api/geometry_msgs/html/msg/Twist.html)
(Google "geometry_msgs/Twist" and it should show up)

<img class="stretch" src="https://i.imgur.com/uovZmPe.png">

What about looking at what's getting published on a topic?

----

### `rostopic echo`
- We can use `rostopic echo` to look at the messages on a topic:

```bash
rostopic echo /oswin/velocity
# linear:
#   x: 0.5
#   y: 0.0
#   z: 0.0
# angular:
#   x: 0.0
#   y: 0.0
#   z: 0.0
# ---
```

----

### `rostopic pub`
- What if we wanted to publish to a topic from the command line?
    - We can use `rostopic pub`
- Close `teleop_twist_keyboard`
- Publish using `rostopic pub`
    ```bash
    rostopic pub /oswin/velocity geometry_msgs/Twist "linear:
      x: 1.0
      y: 0.0
      z: 0.0
    angular:
      x: 0.0
      y: 0.0
      z: 0.0"
    ```
  - Tab completion is really useful here. Type `rostopic pub /oswin/velocity ` and then keep tabbing
  - The identation matters (for those of you who know YAML, this is YAML)
- The turtle in the simulation should move
- Try playing around with the `x` component of `linear` and the `z` component of `angular`

---

## ROS messages

Now that we've seen how to use the `rostopic` tool, let's look more at the ROS **messages** themselves. So far, we've
seen the `geometry_msgs/Twist` message, but how do they work?

----

### `.msg` files
ROS Messages are defined using `.msg` files. For example, the
[`twist.msg`](https://github.com/ros/common_msgs/blob/jade-devel/geometry_msgs/msg/Twist.msg) looks like:
```.msg
# This expresses velocity in free space broken into its linear and angular parts.
Vector3  linear
Vector3  angular
```

----

Each line represents one field in the message, with the type on the left and the
name on the right. In this case, the `twist.msg` message has two fields:
"linear" of type `Vector3`, and "angular" of type `Vector3`.

----

The `Vector3` type that
this message refers to is another type that's defined
[somewhere else](https://github.com/ros/common_msgs/blob/jade-devel/geometry_msgs/msg/Vector3.msg):
```.msg
# This represents a vector in free space. 
# It is only meant to represent a direction. Therefore, it does not
# make sense to apply a translation to it (e.g., when applying a 
# generic rigid transformation to a Vector3, tf2 will only apply the
# rotation). If you want your data to be translatable too, use the
# geometry_msgs/Point message instead.

float64 x
float64 y
float64 z
```

----

In general, messages are described by other messages within them.

---

### Exercise: Writing our own ROS message
Let's try writing our own ROS message now. Imagine that want to write a path planner node that
publishes motor commands, and a motor controller node that subscribes to the motor commands.

----

We don't have a ROS message type for motor commands yet though, and so we need to make our own
custom ROS message.

Some requirements for our message:
1. Our robot has two motors, one for each wheel, so we need to be able to send each motor a separate command
2. We want to know **when** the message was sent, so we can tell if the message is old or not.
3. The command sent to the motor should be a float

----

Write your ROS message in the file `igvc_training_msgs/msg/motor_command.msg`.

Tips:
- Check out the [std_msgs](http://wiki.ros.org/std_msgs). It should contain everything you need
- Remember to `catkin_make` to compile the message!
- Try doing `rostopic pub` and `rostopic echo` to verify that your message works.

---

## Summary
This week, we learnt about:
- [Running ROS Nodes](#/9)
    + `rosrun <package-name> <node-name>`, ie. `rosrun igvc_buzzsim buzzsim`
- [ROS Topics and `teleop_twist_keyboard`](#/10)
    + A ROS topic is like an address that messages are sent to
    + A ROS node can **subscribe** and **publish** messages to any topic
    + Each topic has a message type.
- [The `rostopic` tool](#/13)
    + `rostopic list` to list available topics
    + `rostopic echo` to listen to topics
    + `rostopic pub` to publish to topics
- [ROS messages](#/13)
    + Described in a `.msg` file
    + Composed of other ROS messages
    + Wrote our own custom ROS message to communicate custom information between (hypothetical) nodes
    
----

## Summary
The presentation slides are available on github, under the [ros-training](github.com/RoboJackets/ros-training)
repository on the RoboJackets github.

There's also a link to the
[markdown version of the same content](https://github.com/RoboJackets/ros-training/blob/master/code/instructions/week1.md)
in the README of the repository.

---

## Next Week
We'll be learning how to write ROS **Publishers** and **Subscribers** in C++

---

See you next week!
