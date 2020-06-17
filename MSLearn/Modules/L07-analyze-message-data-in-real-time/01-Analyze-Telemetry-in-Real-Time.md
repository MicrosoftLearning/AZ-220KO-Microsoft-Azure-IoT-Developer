# Analyze Telemetry in Real Time

This lab follows on from Lab 7 - Device Message Routing.

## Create a Message Route to Analyze Telemetry in Real Time

The second message route we need to create uses an Azure Event Hub. This route is created for the IoT Hub, then added as an input to the Azure Stream Analytics job.

We need to update the job to handle two inputs and two outputs, and a more complex query.

The process of creating the second route follows a similar process to the first, though it diverges at the creation of an endpoint. An Event Hub is chosen as the endpoint for the telemetry route.

In the next exercise, you will create an Event Hubs *namespace*. You then have to create an *instance* of the namespace to complete the setting up of an Event Hub. You can then use this instance as the destination for the new message route.

After the route is created, we move on to updating the query.

### Make a Call to a Built-in ML Model

The built-in function we're going to call is `AnomalyDetection_SpikeAndDip`.

The `AnomalyDetection_SpikeAndDip` function takes a sliding window of data, and examines it for anomalies. The sliding window could be, say, the most recent two minutes of telemetry data. This sliding window keeps up with the flow of telemetry in close to real time. If the size of the sliding window is increased, generally the accuracy of anomaly detection will increase too. As will the latency.

As the flow of data continues, the algorithm establishes a normal range of values, then compares new values against those norms. The result is a score for each value, a percentage that determines the confidence level that the given value is anomalous. Low confidence levels are ignored, the question is what percentage confidence value is acceptable? In our query, we're going to set this tipping point at 95%.

There are always complications, like when there are gaps in the data (the conveyor belt stops for a while, perhaps). The algorithm handles voids in the data by imputing values.

Spikes and dips in telemetry data are temporary anomalies. However, as we're dealing with sine waves for vibration, we can expect a short period of "normal" values follow a high or low value that triggers an anomaly alert. The operator is looking for a cluster of anomalies occurring in a short time span. Such a cluster indicates something is wrong.

There are other built-in ML models, such as a model for detecting trends. We don't include these models as part of this module, but the student is encouraged to investigate further.

### Visualize data using Power BI

Visualizing numerical data, especially volumes of it, is a challenge in itself. How can we alert a human operator of the sequence of anomalies that infer something is wrong?

The solution we use in this module is to use some built-in functionality of Power BI. And the ability of Azure Stream Analytics to send data in a real-time format that Power BI can ingest.

We use the dashboard feature of Power BI to create a number of tiles. One tile contains the actual vibration measurement. Another tile is a gauge, showing from 0.0 to 1.0 the confidence level that the value is an anomaly. A third tile indicates if the 95% confidence level is reached. The main tile though shows the number of anomalies detected over the past hour. This tile makes it clear if a clutch of anomalies were detected in short succession.

The fourth tile includes time as the x-axis. This tile allows you to compare the anomalies with the red text in the telemetry console window. Is there a cluster of anomalies being detected when forced, or increasing, or both, vibrations are in action?

Let's create the Event Hub, create the second route, update the SQL query, create a Power BI dashboard, and let it all run!

## Restart Device Simulator App

As we need some real-time telemetry, you need to ensure the app from the previous lab is running.

1. In Visual Studio Code, to run the app in the terminal, enter the following command:

    ```bash
    dotnet run
    ```

   This command will run the **Program.cs** file in the current folder.

1. You should quickly see console output, similar to the following:

    ![Console Output](../../Linked_Image_Files/M99-L07-vibration-telemetry.png)

    > [!NOTE] Green text is used to show things are working as they should and red text when bad stuff is happening. If you don't get a screen similar to this image, start by checking your device connection string.

1. Watch the telemetry for a short while, checking that it is giving vibrations in the expected ranges.

1. You can leave this app running, as it's needed for the next section.
