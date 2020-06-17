# Sending and Receiving Telemetry

Telemetry is the output from sensors. There are many different types of sensors. Acceleration, humidity, location, pressure, temperature, and velocity are the most commonly used in commercial applications. Other sensors include radiation, motion-sensitivity, acoustics, air quality, heart rate, and so on. And they all pump out telemetry for some other process to consume.

The frequency of telemetry output is an important factor. A temperature sensor in a refrigeration unit may only have to report every minute, or less. An acceleration sensor on an aircraft may have to report at least every second.

An IoT device may contain one or more sensors, and have some computational power. There may be LED lights, and even a small screen, on the IoT device. However, the device isn't intended for direct use by a human operator. An IoT device is designed to receive its instructions from the cloud.

## Control a Cheese Cave Device

In this module, we assume the IoT cheese cave monitoring device has temperature and humidity sensors. The device has a fan capable of both cooling or heating, and humidifying or de-humidifying. Every few seconds, the device sends current temperature and humidity values to the IoT Hub. This rapid frequency is unrealistic for a cheese cave (maybe every 15 minutes, or less, would be granular enough), except during code development when we want rapid activity!

We assume, in the next unit, that the fan can be in one of three states: on, off, and failed. The fan is initialized to the off state. In a later unit, the fan is turned on by use of a direct method.

Another feature of our IoT device is that it can accept desired values from the IoT Hub. The device can then adjust its fan to target these desired values. These values are coded in this module using a feature called device twins. Desired values will override any default settings for the device.

## Coding the Sample

The coding in this module is broken down into three parts: sending and receiving telemetry, sending and receiving a direct method, and managing device twins.

Let's start by writing two apps: one for the device to send telemetry, and one back-end service to run in the cloud, to receive the telemetry. You'll be able to select your preferred language (Node.js or C#), and development environment (Visual Studio Code, or Visual Studio).