# Firmware update mechanism

Before getting started with your first firmware update on an IoT device, let's discuss what it actually means to implement such an operation and how Azure IoT Hub helps making the process.

## What does updating an IoT device's firmware imply?

IoT devices most often are powered by optimized operating systems or even sometimes running code directly on the silicon (without the need for an actual operating system). In order to update the software running on this kind of devices the most common method is to flash a new version of the entire software package, including the OS as well as the apps running on it (called firmware).

Because each device has a specific purpose, its firmware is also very specific and optimized for the purpose of the device as well as the constrained resources available.

The process for updating a firmware is also something that can be very specific to the hardware itself and to the way the hardware manufacturer does things. This means that a part of the firmware update process is not generic and you will need to work with your device manufacturer to get the details of the firmware update process (unless you are developing your own hardware which means you probably know what the firmware update process).

While firmware updates can be and used to applied manually on devices, this is no longer possible considering the rapid growth in scale of IoT solutions. Firmware updates are now more commonly done over-the-air (OTA) with deployments of new firmware managed remotely from the cloud.

There is a set of common denominators to all over-the-air firmware updates for IoT devices:

1. Firmware versions are uniquely identified
1. Firmware comes in a binary file format that the device will need to acquire from an online source
1. Firmware is locally stored is some form of physical storage (ROM memory, hard drive,...)
1. Device manufacturer provide a description of the required operations on the device to update the firmware.

## Azure IoT Hub Automatic Device Management

Azure IoT Hub offers advanced support for implementing device management operations on a single and on collections of devices. The [Automatic Device Management](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config) feature allows to simply configure a set of operations, trigger them and then monitor their execution.

In the following exercise, you will create a simple simulator that will manage the device twin desired properties changes and will trigger a local process simulating a firmware update. The overall process would be exactly the same for a real device with the exception of the actual steps for the local firmware update. You will then use the Azure Portal to configure and execute a firmware update for a single device. IoT Hub will use the device twin properties to transfer the configuration change request to the device and monitor the progress as shown in this diagram:

:::image type="content" source="../../Linked_Image_files/M99-L16/FirmwareUpdateDiagram.png" alt-text="Firmware Update Configuration":::