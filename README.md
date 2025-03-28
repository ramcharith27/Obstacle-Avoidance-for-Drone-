# Obstacle Avoidance Drone Using Pixhawk and Ultrasonic Sensors
# Overview
This project implements an obstacle avoidance system for a drone controlled by a Pixhawk flight controller. The avoidance mechanism is based on ultrasonic sensors, which detect obstacles and adjust the drone's movement accordingly.

#Features
Uses an ultrasonic sensor to detect obstacles within a 30 cm range.
Neutralizes pitch, roll, and yaw when an obstacle is detected.
Reads input signals from the receiver channels (Throttle, Pitch, Roll, Yaw).
Generates a PPM signal to control the Pixhawk.
Outputs real-time data via the Serial Monitor for debugging.

# Hardware Requirements
Pixhawk Flight Controller
HC-SR04 Ultrasonic Sensor
Arduino 
RC Transmitter and Receiver
Drone Frame, Motors, and ESCs

# Code Explanation
The Arduino script does the following:
Measures distance using the ultrasonic sensor.
Reads RC input signals from the receiver.
Adjusts control signals when an obstacle is detected.
Generates a PPM signal to send commands to the Pixhawk.

# Installation and Usage
Upload the provided Arduino script to the microcontroller.
Connect the ultrasonic sensor and receiver inputs as per the wiring diagram.
Power the Pixhawk and ensure proper PPM signal reception.
Observe obstacle avoidance behavior in flight.
