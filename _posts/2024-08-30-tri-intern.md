---
title: "TRI Intern"
permalink: /tri-intern/
date: 2024-08-30
share: false
excerpt: Integrated & implemented the brake-by-wire system, and developed a ROS2 Python package for a load-cell handbrake
---

Timeline: December 2023 - August 2024<br>
Location: Los Altos, CA, USA<br>
Skills: MATLAB, Simulink, ROS2, Python, dSPACE

I was an intern on the Platform Research team in Human Interactive Driving at Toyota Research Institute. I integrated mechatronic components to build a new off-road & high-torque battery electric vehicle.

### Brake-by-wire
<figure>
<img src= "/assets/images/tri-brake-ecu-to-actuator-wire.png">
<figcaption> A wire I fabricated to connect the brake ECU to the actuator, thereby setting up the vehicle's brake-by-wire technology</figcaption>
</figure>
I implemented a reliable brake-by-wire architecture with 4 individually controllable caliper pressures, thereby providing higher autonomous control capabilities.

I integrated the brake pedal position sensor to require lesser driver pedal application pressure than traditional brake systems, thereby reducing stopping distances.

I also established communication between the main computer, brake ECU (Engine Control Unit), pedal sensor, and stroke sensor via Simulink & CAN (Controller Area Network) bus communication to allow for shared control capabilities between the brake pedal & an autonomous braking system.

Finally, I utilized the dSPACE ControlDesk experiment software to verify the brake-by-wire software implementation amongst the internal actuators & solenoids, brake ECU firmware, and the onboard Simulink computer. This was done to ensure optimum conversion of lower-level ECU values to more interpretable values for the vehicle's shared control.

### Load-cell handbrake
I deeveloped a ROS2 (Robot Operating System 2) Python package to interface a pressure-sensitive load-cell USB handbrake onto a Linux computer for the vehicle. This removed the need for a traditional cable-based mechanical handbrake.

My package architecture utilizes publisher/subscriber nodes & multithreading with a thread-safe queue. The implementation will allow drivers to drift by locking the vehicle’s rear wheels.

### Other explorations
I started the development & integration of an HMI (Human-Machine Interface) keypad and digital dash display to provide drivers with vital component health metrics

I also utilized Solidworks CAD to design an improved computer shelf system that would allowing for easier access of the vehicle's computers (remove all at once and debug outside the vehicle). This new design significantly improves on previous vehicles in terms of wiring management and will reduce communication errors.

Additionally, I explored a 48V DC Brushless motor (with an integrated controller) to independently steer a wheel for future battery electric vehicles through a university collaboration. I supported the system design through a custom Simulink state machine implementing NMT (Network Management) Protocol to control the CANOpen motor’s state & identify error codes through PDOs (Process Data Objects) & SDOs (Service Data Objects).


​[Toyota Research Institute: Human Interactive Driving](https://www.tri.global/our-work/human-interactive-driving)