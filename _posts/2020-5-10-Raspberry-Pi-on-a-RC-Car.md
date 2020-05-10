---
layout: post
title: Raspberry Pi on a RC Car
---

Instead of building an entire mobile robot from scratch, I chose to modify a current off the shelf RC car. My goal for this project 
was gaining a better understanding of the ROS software libraries commonly used in robots.

One nice thing about ROS is that we have access to hundreds of plugins that we can customize and adapt to our project's needs. For example, we use the teleop_twist_keyboard package for converting keyboard inputs into a Twist message. ROS supports many different types of messages out of the box and we can also add custom messages if needed. A Twist message expresses velocity in free space broken into its linear and angular parts. As our mobile robot can only move forwards/backwards and can turn left/right, we only will use part of the message space. Thus, the first change I made was I updated the keyboard input script so that it only takes input from the 'wasd' keys mapping to standard 2D movement. This simplified script is located [here](https://github.com/rishabhkjain/self-driving-car/blob/master/teleop_twist_keyboard/teleop_simple_keyboard.py). 

So now, in our ROS project, we have a node that takes in keyboard input and publishes the corresponding message to the /cmd_vel topic. Now, we need to build a node that will run on the Raspberry Pi that subscribes to the /cmd_vel topic and commands the RC car to the message's instructions. Before writing the code for this, we need to wire up our Pi so that it can control the car. Taking apart the car, we find that the drivetrain consists of two motors connected to a H-Bridge. Initially, I planned on connecting the Raspberry Pi directly to the H-Bridge for controlling the motors. However, without the board schematic and an electronic bench, it would have been challenging finding the right places for adding connections to the H-Bridge. 

Instead, we will utilize the RC car's remote for sending the movement instructions. The remote is designed so that whenever a push button is depressed, the car will move in that coressponding direction. Further probing of the remote reveals that the button's signals are active low meaning the button shorts the signal wire to ground when depressed. Thus, we can use the Pi's GPIO pins to directly set the signal wire to low/high. After adding jumper's to the four signal wires, we can get started with writing the Pi's controller. 

We use the RPI GPIO library for easy access to the GPIO pins. This controller node is a simple adaptation of the simple subscriber from the ROS tutorial where it parses the message received and accordingly sets the GPIO pins to move the car in the given direction. The source code for this node is available [here](https://github.com/rishabhkjain/self-driving-car/blob/master/rc_control/rpi_rc_controller.py) 

That's all the code we need to control the car remotely! Next, we need to install and configure ROS on the Raspberry Pi. I followed [this](http://thomas-messmer.com/index.php/14-free-knowledge/howtos/86-ros-melodic-on-raspberry-pi-3) tutorial to install ROS. Next, we need to change the ROS master URI on the Pi so that it uses the same ROS master as is running on my laptop. My network was having issues with hostname resolution so I ended up having to use IP addresses instead for identifying my laptop and Pi. If you use IP addresses like I did, you also need to change the ROS_IP and ROS_HOSTNAME parameters to properly sync the two devices together. Since I was not using Raspbian, I ran into some permission issues with my non-root account accessing the GPIO pins. In order to fix it, I created a new usergroup named gpio and added my user to that group. Next, I ran the following two commands to give the group permissions to the GPIO pins.  
```bash
sudo chown root.gpio /dev/gpiomem
sudo chmod g+rw /dev/gpiomem
```

Lastly, we can run both nodes and control our car remotely. Although this implementation is very simple, it provides
a nice foundation to build autonomous capabilities into. Next, I want to try implementing basic object following capabilities into this car. As I do not have much testing space in my apartment, I will first try to implement the code in Gazebo and then port it to my RC car. 