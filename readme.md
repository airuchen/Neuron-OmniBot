﻿# Neuron Omni Bot
This is a demo package of Neuron Omni Bot's autonomous and fast integration capability using ROS
### Features
- Fully integration between SEMA library and ROS2
- Three omni-directional poly wheels with **TODO**watts electrical motor on each wheel
- Quadrature hall effect encoder provide 390 pulses (1560 count) per wheel revolution
- Fully customized moving controller and motor driver using STM32F103 chip
- Fully customized ROS motor driver node that utilize standard ROS convension
- MPU6050 6-axis IMU and it's corresponding ROS publisher with standard ROS state estimator
- Easy to use ROS launch file with maximum custimization capabilities
- Finely tuned ROS navigation, guidance, and control algorithms with full list of parameters provided
- Multiple user interfaing methods using the SEMA library


# Getting Started
These instructions will get you a copy of this demo and running on your local machine. If you're unfamiliar with ROS operation, there are some usefull trick you may want to try in the [section below](#useful-tricks) For multiple machine remote control, please visit [multi-machines](#multiple-machines) section below.
## Prerequisites
1. Neuron hardware and SEMA library  
You'll need the ADLINK SEMA library and a compatible motherboard to run this example. You can conatct **TODO: SOMEONE** for more information.
2. **TODO** Download the source of this project to your ROS2 workspace  
    ```
    cd catkin_ws/src
    git clone https://github.com/EwingKang/neuron_demo_gpio.git
    ```  
3. Some electronics hardware: **TODO** 4 LEDs, 2 switches(one tactile, one contact)
4. Laser scanner with its corresponding ROS node properly installed.
5. Wires properly connected (see **TODO some section** below)

## Installing
### hardware setup 
1. STM32 3-Motor driver
2. Laser scanner (for YDLidar)
3. SEMA GPIOs
4. Other recommandations
    *  It is recommanded you have your robot's sharp edges wraped
    * DO NOT obstruct the view of laser scanner
    * Your two wifi antennas should be pointing perpendicular(i.e. 90 degrees) to each other

### software setup
1. Install ADLINK SEMA
Your Neuron Bot should already have proper SEMA installed. Please go to **TODO: EMPTY PROJECT** if you have any questions.

2. Install ROS kinetic and setup workspace
	Your Neuron Bot should already have ROS set. If not, you mar refer to [the install guide](http://wiki.ros.org/kinetic/Installation/Ubuntu), and [catkin_ws setup guide](http://wiki.ros.org/catkin/Tutorials/create_a_workspace). Make sure you have environmental path add to .bashrc to save time

3.  Install packages (ubuntu software):
    * ROS stuff
     ```
    #robot localization
    sudo apt-get install ros-kinetic-robot-localization 
    
    #laser slam
	sudo apt-get install ros-kinetic-gmapping ros-kinetic-scan-tools\ ros-kinetic-navigation # laser slam
    
    #navigation and planning
    sudo apt-get install ros-kinetic-teb-local-planner ros-kinetic-teb-local-planner-tutorials\ ros-kinetic-eband-local-planner
     ```
    * Recommanded
        * KATE: text editor (very similar to Notepad++)
        `sudo apt-get install kate`
        * htop: a low-cost system monitor
        `sudo apt-get install htop`
        * serial port terminoa (GUI monitor) 
        `sudo apt-get install gtkterm`
        * OBS
        * SSH server
  

4. Set up SEMA soft link
Change to any node that uses SEMA library, find the SEMA include library header location, and run the auto shell command file. 
    ```
    cd ${project_root}/lib
        e.g.: cd ros2_ws/src/ros2/neuron_demo_gpio/lib
    ./setlink.sh
    ```  
    Note: If you get some error like _`error: no such file as...`_, you'll need to make the setlink.sh executable by `chmod +x setlink.sh` after you've changed the command prompt to that directory.      

5. Compile the source code  
Now, we'll use the Catkin, the ROS build management tool to build our nodes. We'll need root access for library linking for anything that uses SEMA. Root access is gained by the second step below. Great power comes with great responsibility, **it is strongly recommanded you to exit root mode** since you can to terrible stuff with that much of power.
    ```
    cd ~/catkin_ws
    sudo -sE
    catkin_make
    exit   #exit root mode
    ```  
6. Setup Laser scanner port (from [YDLidar github](https://github.com/EAIBOT/ydlidar))
    ```
    roscd ydlidar/startup
    sudo chmod 777 ./*
    sudo sh initenv.sh
    ```

7.  Add serial access
    `sudo adduser ros dialout`

## Run the demo
The Neuron Omnibot demo can be divided into four part:
* [Omnibot IO](#omnibot-driver): motor controller, laser scanner, LED indecators, and servos
* [SLAM](#laser-slam): simultaneous localization and mapping
* [Localization](#robot-localization): after we build our 2D map
* [Move!](#navigation): Obstacle detecting, planning, trajectory generation, and vehicle control  

Each of the above function is wraped as a single ROS launch file for user's easy execution. We'll have a step-by-step tutorial below. For each launch file, we'll open a new terminal. You can do that by pressing _`ctrl + alt + t`  
### OmniBot driver
In this section, we'll start our ROS omnibot driver. The driver includes all the IO and sensory device including motor controller, encoder odometry, laser scanner, and IMU state estimation.
![](graphwith links) 
1. roscore  
roscore is the core of the ROS as its name suggest. We encourage you to manually start the core on a seperate window because it gives user the power and responsibility to control everything.
    ```
    roscore
    ```

2. Robot Base driver
    This launch file include multiple node. It launches the communication between STM32 motor controller, laser slam, as well as all the robot TF definition. Please note that if you're ending the node by `ctrl + c`, you only have to hit once and give it a seconds for it to shutdown automatically. The LaserScan node requies some time to shutdown the serial port.
    ```
    roslaunch omni_base_driver omni_localization.launch
    ```
3. Keyboard controller node
We use [teleop_twist_keyboard](http://wiki.ros.org/teleop_twist_keyboard) as our manual driver. The default command is a little too fast, so use `x` and `c` to reduce velocity to around 0.3. 
    ```
    rosrun teleop_twist_keyboard teleop_twist_keyboard.py
    ```
    Note: because Neuron OmniBot is a holonomic robot, you can hold `shift`+moving around command to do translational moving.
 4. RVIZ monitoring
    RVIZ stands for ROS-VIsualiZation, which is a powerfull 3-D visualization environment. We can launch RVIZ simply by:
    ```
     rviz
    ```
    Now, use the "File -> Open Config" or `ctrl + o` to  open base  visualization settings located at `$(omni_base_driver)/rviz_config/omni_driver_laser.rviz`. One should note that despite it is easier to simply use the gui provided to open the rviz config file, it is possible to use command line by adding **absolute path**:
    ```
     rviz -d "/home/ros/catkin_ws/src/omni_base_slam/rviz_config/omni_slam.rviz"
     ```

### Laser Slam  
In this section, we'll build our map with our 2D laser scanner. 
![](graphwith links)
 
1. Make sure you have everything in the [base driver](#omni-bot-driver)  launched. This includes all the robot TF, motor driver, and laser scanner. 
3. Setup rviz correctly so we can see everything:
   you can open `($ omni_base_slam)/rviz_config/omni_slam.rviz` manually, or by running the following command:
   ```
   rviz -d "/home/ros/catkin_ws/src/omni_base_slam/rviz_config/omni_slam.rviz"
   ```
4. Let's start the laser localization and mapping procedure with [gmapping](http://wiki.ros.org/gmapping) by the following command:
    ```
    roslaunch omni_base_slam omni_gmapping.launch
    ```
5.  Drive around using keyboard driver introduced in [base driver](#omni-bot-driver).
6.  After you map the whole place, remember to save the map **before** closing the gmapping:
    ```
    rosrun map_server map_saver -f map_file_name
    ```
    A map file(.x)  and a config file (.xxx) will be saved under your user home `~/`, make sure to move both of these files to `($ omni_base_slam)/map/`
7. Stop the gmapping by `ctrl + c` on the gmapping terminal (terminal of the step.4).

### Robot Localization

![](graphwith links) 

1.  Make sure you have everything in the [base driver](#omni-bot-driver)  launched. This includes all the robot TF, motor driver, and laser scanner. 
2. Setup rviz correctly so we can see everything:
   you can open `($ omni_base_nav)/rviz_config/omni_amcl.rviz` with rviz gui, or by running the following command:
       ```
     rviz -d "/home/ros/catkin_ws/src/omni_base_nav/rviz_config/omni_amcl.rviz"
     ```
3. Put the map file and its config file to `($ omni_base_slam)/map/` as stated in the previous section. Modify line x of `($ omni_base_nav)/omni_lxxxxx.launch`  to reflect the correct file name.
3. Now, we'll start our localization package [amcl](http://wiki.ros.org/amcl) with our pre-define settings:
    ```
    roslaunch omni_base_nav omni_localize.launch
    ```
    By default, the localization package will initialize the robot at (x,y)=(0,0), i.e. same as the starting pose when we started the mapping process. However, we can manually assign the starting position by using "set 2D pose estimation" function in the RVIZ if it's not the case. Select the tool, click on the position and drag the arrow for its initial heading.
4. Now we've initialize the robot pose, we can see many arrows in the RVIZ world. These arrows are the particles used to localize the robot. Because of how the Monte Carlo method (AMCL) works, those poses will not update nor converge if the robot remains still. With that being said, we can still ask the localization to try to update by:
    ```
    rosservice call /request_nomotion_update
    ```
    And you'll see the arrows become more unison and your laser scaning pattern will gradually match with the map.
5. It is often possible for robot to identify it's initial location without manually set the initiali pose. You can call this service so the localization package will evenly distribute the pose particle. After the global initialization, you can perform multiple no-motion-update mentioned above  and the package will localize itself.
    ```
    rosservice call /request_xxxxxxxxx
    ```


### Path Finding
![](graphwith links) 
1. [base driver](#omni-bot-driver) is started
2. [localization](#robot-loclization) is initilized
3. RVIZ is set to `($ omni_base_nav)/rviz_config/omni_nav.rviz`
4. There are three different popular local planning package for you to choose: the [Dynamic Window Approach](http://wiki.ros.org/dwa_local_planner), the [Timed Elastic Band](http://wiki.ros.org/teb_local_planner) planner, and the [Elastic Band](http://wiki.ros.org/eband_local_planner) planner. You can choose from one of the following command:
    ```
    roslaunch omni_base_nav omni_nav_dwa.launch
    roslaunch omni_base_nav omni_nav_teb.launch
    roslaunch omni_base_nav omni_nav_eband.launch
    ```
5. After the planner is started, you should see something like the graph below in the RVIZ. You can choose the "2D nav goal" tools on the top banner of the RVIZ. Click on the map and drage to specify the target orientation. The robot should drive toward the goal by itself.
![](graphwith links) 
The background gray map is the global map, while the colorful blue-pink-red one is the 1.5x1.5 meter local map. The global path is drawn in xxx while the local planner is draw in green. Different local planner will have different representation.

# Multiple Machines

# Useful tricks
### Linux/Ubuntu terminal
* `ctrl + shift + T` to open new terminal tab
* `ctrl + PageUp/PageDn` to switch between tabs

### ROS tools
* `rqt` ROS - QT, ROS information visualization tools
* `rostopic`: ROS topic server functions
    * `rostopic list`  to list all topics 
    * `rostopic echo /SOME_TOPIC` to print the topic directly.
* `rosservice`  : ROS service server functions

### TBDDBTTBDDBT What should happened?
Besides the information you see on each terminal, proper ROS2 topic is published. 
* **neuron_demo_gpio** node  
Subscribes to topic _neuron_gpio_cmd_ from _talker_  
Publishes topic _neuron_gpio_data_ of type _std_msgs::String_  
* **neuron_omni_io** node  
Subscribes to topic _neuron_gpio_cmd_ from _talker_  
Controls the LEDs and read from switches  
**INSERT NODE GRAPH HERE**


# Code explained
**TBD**
**ANY** kind of std_msgs data from **TOPIC_CMD** will trigger the node to
publish the next hardware monitor data to **TOPOC_DATA** topic.
**1.** The setting of **TOPIC_CMD** and **TOPIC_DATA** can be found in ***reuronHwmNode.hpp***



# More Info
## Notice
1. This application **MUST** be running under **SUDO -E** since the SEMA driver need both root privilege and the user-exported variables.


## Version
0.1.0

## Authors
* **Alan Chen** - *Initial library* - (alan.chen@adlinktech.com)
* **Ewing Kang** - *Demo implementation* - (https://github.com/EwingKang)

## License
This project is licensed under the Apache License, Version 2.0

## Acknowledgments

## Running the tests


  
========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    ========================================================    
    




![](https://pandao.github.io/editor.md/images/logos/editormd-logo-180x180.png)

![](https://img.shields.io/github/stars/pandao/editor.md.svg) ![](https://img.shields.io/github/forks/pandao/editor.md.svg) ![](https://img.shields.io/github/tag/pandao/editor.md.svg) ![](https://img.shields.io/github/release/pandao/editor.md.svg) ![](https://img.shields.io/github/issues/pandao/editor.md.svg) ![](https://img.shields.io/bower/v/editor.md.svg)


**Table of Contents**

[TOCM]

[TOC]

#H1 header
######H6 header
#Heading 1 link [Heading link](https://github.com/pandao/editor.md "Heading link")
######Heading 6 link [Heading link](https://github.com/pandao/editor.md "Heading link")

##Headers (Underline)

H1 Header (Underline)
=============

H2 Header (Underline)
-------------

###Characters
                
----

~~Strikethrough~~ <s>Strikethrough (when enable html tag decode.)</s>
*Italic*      _Italic_
**Emphasis**  __Emphasis__
***Emphasis Italic*** ___Emphasis Italic___

Superscript: X<sub>2</sub>，Subscript: O<sup>2</sup>

**Abbreviation(link HTML abbr tag)**

The <abbr title="Hyper Text Markup Language">HTML</abbr> specification is maintained by the <abbr title="World Wide Web Consortium">W3C</abbr>.

###Blockquotes

> Blockquotes

Paragraphs and Line Breaks
                    
> "Blockquotes Blockquotes", [Link](http://localhost/)。

###Links

[Links](http://localhost/)

[Links with title](http://localhost/ "link title")

`<link>` : <https://github.com>

[Reference link][id/name] 

[id/name]: http://link-url/

GFM a-tail link @pandao

###Code Blocks (multi-language) & highlighting

####Inline code

`$ npm install marked`

####Code Blocks (Indented style)

Indented 4 spaces, like `<pre>` (Preformatted Text).

    <?php
        echo "Hello world!";
    ?>
    
Code Blocks (Preformatted text):

    | First Header  | Second Header |
    | ------------- | ------------- |
    | Content Cell  | Content Cell  |
    | Content Cell  | Content Cell  |

####Javascript　

```javascript
function test(){
			return this;
		},
    window.box =box;
})();

var testBox = box();
testBox.add("jQuery").remove("jQuery");
```

####HTML code

```html
<!DOCTYPE html>
<html>
    <head>
        <h1>Hello world!</h1>
    </body>
</html>
----

###Lists

####Unordered list (-)

- Item A
- Item B
- Item C
     
####Unordered list (*)

* Item A
* Item B
* Item C

####Unordered list (plus sign and nested)
                
+ Item A
+ Item B
    + Item B 1
    + Item B 2
    + Item B 3
+ Item C
    * Item C 1
    * Item C 2
    * Item C 3

####Ordered list
                
1. Item A
2. Item B
3. Item C
                
----
                    
###Tables
                    
First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell 

| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |

| Function name | Description                    |
| ------------- | ------------------------------ |
| `help()`      | Display the help window.       |
| `destroy()`   | **Destroy your computer!**     |

| Item      | Value |
| --------- | -----:|
| Computer  | $1600 |
| Phone     |   $12 |
| Pipe      |    $1 |

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
                
----

####HTML entities

&copy; &  &uml; &trade; &iexcl; &pound;
&amp; &lt; &gt; &yen; &euro; &reg; &plusmn; &para; &sect; &brvbar; &macr; &laquo; &middot; 

X&sup2; Y&sup3; &frac34; &frac14;  &times;  &divide;   &raquo;

18&ordm;C  &quot;  &apos;

##Escaping for Special Characters

\*literal asterisks\*

##Markdown extras

###GFM task list

- [x] GFM task list 1
- [x] GFM task list 2
- [ ] GFM task list 3
    - [ ] GFM task list 3-1
    - [ ] GFM task list 3-2
    - [ ] GFM task list 3-3
- [ ] GFM task list 4
    - [ ] GFM task list 4-1
    - [ ] GFM task list 4-2

###Emoji mixed :smiley:

> Blockquotes :star:

####GFM task lists & Emoji & fontAwesome icon emoji & editormd logo emoji :editormd-logo-5x:

- [x] :smiley: @mentions, :smiley: #refs, [links](), **formatting**, and <del>tags</del> supported :editormd-logo:;
- [x] list syntax required (any unordered or ordered list supported) :editormd-logo-3x:;
- [x] [ ] :smiley: this is a complete item :smiley:;
- [ ] []this is an incomplete item [test link](#) :fa-star: @pandao; 
- [ ] [ ]this is an incomplete item :fa-star: :fa-gear:;
    - [ ] :smiley: this is an incomplete item [test link](#) :fa-star: :fa-gear:;
    - [ ] :smiley: this is  :fa-star: :fa-gear: an incomplete item [test link](#);
            
###TeX(LaTeX)
   
$$E=mc^2$$

Inline $$E=mc^2$$ Inline，Inline $$E=mc^2$$ Inline。

$$\(\sqrt{3x-1}+(1+x)^2\)$$
                    
$$\sin(\alpha)^{\theta}=\sum_{i=0}^{n}(x^i + \cos(f))$$
                
###FlowChart

```flow
st=>start: Login
op=>operation: Login operation
cond=>condition: Successful Yes or No?
e=>end: To admin

st->op->cond
cond(yes)->e
cond(no)->op
```

###Sequence Diagram
                    
```seq
Andrew->China: Says Hello 
Note right of China: China thinks\nabout it 
China-->Andrew: How are you? 
Andrew->>China: I am good thanks!
```
