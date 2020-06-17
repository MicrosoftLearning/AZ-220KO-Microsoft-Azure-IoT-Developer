# Automate IoT devices management with Azure IoT Hub

Azure IoT Hub is a cloud service designed to be your cloud gateway for IoT devices. It allows securely connect millions of devices and establish a bidirectional communication to not only collect data from sensors, but also allow for remote monitoring and management of the devices.

In this Lab, you'll learn how automate device management with IoT Hub to configure and manage IoT devices remotely at scale.

## Learn the scenario

Suppose you manage a company that offers a solution to maintain and monitor cheese caves' temperature and humidity at optimal levels. You have been working with gourmet cheese making companies for a long time and established long term trust with these customers who value the quality of your product.

Your solution consists in sensors and a climate system installed in the cave that report in real time on the temperature and humidity and an online portal customers can use to monitor and remotely operate their devices to adapt the temperature and humidity to the type of cheese they stored in their cave or to fine tune the environment for perfectly aging their cheese.

Your company is always enhancing the software running on the devices to better adapt to your customers different cheeses and diverse types of rooms they use to store their cheese. In addition to the features updates, you also want to make sure the devices deployed at customers locations have the latest security patches to ensure privacy and prevent hackers to take control of the system. In order to do this, you need to keep the devices up to date by remotely updating their firmware.

## In This Lab

In the lab, you will go through these steps:

- Setup an Azure IoT environment: and Azure IoT Hub instance and a device Id
- Write code for simulating the device that will implement the firmware update
- Test the firmware update process on a single device using Azure IoT Hub automatic device management

## Prerequisites

- An introductory knowledge of Azure IoT
- Ability to navigate the Azure IoT portal
- Ability to use the Azure CLI
- Ability to use C#, at the beginner level
- Experience using Visual Studio Code, at the beginner level
