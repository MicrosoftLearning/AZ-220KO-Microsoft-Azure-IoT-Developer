# Test firmware update on a single device

In this unit we will use the Azure portal to create a new device management configuration and apply it to our single simulated device.

## Start device simulator

In the same terminal you setup the application for the simulated device, start the simulator typing the following command (replacing \<device connection string\> with the device connection string you got at the end of task 2):

``` 
dotnet run "<device connection string>" 
```

You should see the following output in the terminal (where "mydevice" is the device ID you used when creating the device identity):

``` 
    mydevice: Device booted
    mydevice: Current firmware version: 1.0.0
```
> [!NOTE]
> Make sure to put "" around your connection string. For example: "HostName=AZ-220-HUB-{YourID}.azure-devices.net;DeviceId=SimulatedSolutionThermostat;SharedAccessKey={}="

## Create the device management configuration

1. Sign into the [Azure portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true).
1. Go to the IoT Hub blade. You can find your IoT Hub by typing in the search bar (on top) the name you used when creating it in task 2.
1. In the IoT Hub, find the **Automatic Device Management** blade and select **IoT Device Configuration**, then select **Add Device Configuration**

    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-1.png" alt-text="IoT Devive Configuration":::
    
1. Enter an ID for the configuration such as **firmwareupdate** then click on **Next: Twins settings >** on the bottom.

    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-2.png" alt-text="Configuration ID":::
    
1. For the **Device Twin Property** field, enter the following:

    ```
    properties.desired.firmware
    ```
    
1. In the **Device Twin Property Content** field type the following. Then click on **Next: Metrics >**

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```


    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-3.png" alt-text="Device Twin configuration":::
    
1. In the **Metrics** blade we will define a custom metric to track the firmware update was effective. Create a new custom metric called **"fwupdated"** and type in the below criteria, then click on **Next: Target devices >**

    ``` SQL
        SELECT deviceId FROM devices
            WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```
    
    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-4.png" alt-text="Metrics":::
    
1. In the **Priority** field, type **"10"** and in the **Target Condition** field, type in the following query, replacing "\<your device id\>" with the device Id you used to create the device in task 2, then click on **Next: Review + Create >**

    ``` SQL
        deviceId='<your device id>'
    ```

    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-5.png" alt-text="Target Devices":::
    
1. On the next blade you should see the validation succeed for your new configuration. Click on **Create**.

    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-6.png" alt-text="Validation":::

1. Once the configuration has been created you will see it in the **Automatic Device Management** blade:

    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-7.png" alt-text="New Configuration added":::
        
## See the magic happen
At this point IoT Hub will look for devices matching the configuration's target devices criteria, and will apply the firmware update configuration automatically.
On the terminal you started the device on, you should see the following traces indicating the progress of the firmware update process on the device itself:

    :::image type="content" source="../../Linked_Image_files/M99-L16/simulateddevice1.png" alt-text="Simulated Device executing FW update":::
    
And on the Device Management Configuration you should see the following:

    :::image type="content" source="../../Linked_Image_files/M99-L16/devicemanagement-8.png" alt-text="Configuration applied and successful":::
    
You have validated that the firmware update process on your simulated device works. You can stop the device simulator by simply pressing the "Enter" key in the terminal.