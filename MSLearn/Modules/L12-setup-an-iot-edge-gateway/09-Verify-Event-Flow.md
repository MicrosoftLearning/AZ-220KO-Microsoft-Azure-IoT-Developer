# Verify Event Flow

In this unit, you will use the Azure CLI to monitor the events being sent to Azure IoT Hub from the downstream IoT Device through the IoT Edge Gateway. This will validate that everything is working correctly.

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. At the top of the Azure Portal click on the **Cloud Shell** icon to open up the **Azure Cloud Shell** within the Azure Portal. When the pane opens, choose the option for the **Bash** terminal within the Cloud Shell.

1. Within the **Cloud Shell**, run the following command to monitor the stream of events flowing to the Azure IoT Hub. This will verify that events from the simulate device, being sent to the IoT Edge Gateway, are being received by the Azure IoT Hub.

    ```cmd/sh
    az iot hub monitor-events -n <IoT-Hub-Name>
    ```

    Be sure to replace the `<IoT-Hub-Name>` placeholder for the `-n` parameter with the name of your Azure IoT Hub.

    The `az iot hub monitor-events` command enables you to monitor device telemetry & messages sent to an Azure IoT Hub.

1. With everything working correctly, the output from the `az iot hub monitor-events` command will look similar to the following:

    ```cmd/sh
    chris@Azure:~$ az iot hub monitor-events -n AZ-220-HUB-1119
    Starting event monitor, use ctrl-c to stop...
    {
        "event": {
            "origin": "DownstreamDevice1",
            "payload": "{\"temperature\":30.931512529929872,\"humidity\":78.70672198883571}"
        }
    }
    {
        "event": {
            "origin": "DownstreamDevice1",
            "payload": "{\"temperature\":30.699204018199445,\"humidity\":78.04910910224966}"
        }
    }
    ```

Once you have completed this lab and verified the event flow, exit the console application by pressing **CTRL+C**.