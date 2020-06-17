# Monitor Simulated Device

You'll first create a dashboard showing all the capabilities of the device template. Next, you'll create a real device, and record the connection settings needed for the remote device app.

## Create a rich dashboard

1. Click on the **Views** menu option, then on **Visualizing the device**.

    [![Screenshot showing how to create a new view](../../Linked_Image_files/M99-L20/refrigerated-trucks-add-view.png)](../../Linked_Image_files/M99-L20/refrigerated-trucks-add-view.png#lightbox)

1. You should now see a list of all the **Telemetry**, **Properties**, and **Commands** you created, each with a check box.

1. Click the **Location** check box, then **Add tile**. Dashboards are made up of tiles. The reason we choose the location tile first, is that we want to expand it from its default size. Drag the lower right-hard corner of the tile, so that the tile is at least twice the default size. This tile is the most fun, it will show the location of the truck on a map of the world.

1. Before adding more tiles, change the **View name** to something more specific, "Truck view", or something similar.

1. Now, click each of the rest of the telemetry and properties capabilities in turn, starting at the top, and **Add tile**. We are going for function over form here, we can prettify the dashboard later. For now, we just want a dashboard that will confirm all the telemetry being sent from our remote device. There's no need to add the commands to the dashboard, though that option does exist.

1. When you've added all the tiles, scroll around a bit on your dashboard, and check out the wording in the tiles.

    [![Screenshot showing some sample dashboard tiles](../../Linked_Image_files/M99-L20/refrigerated-trucks-sample-tiles.png)](../../Linked_Image_files/M99-L20/refrigerated-trucks-sample-tiles.png#lightbox)

1. You can drag tiles around, and the portal will try to rearrange them neatly.

1. When you are satisfied with your dashboard, click **Save**, then click **Publish**. You'll now notice that in the dialog that appears, that the **Views** entry is **Yes**. Click **Publish** in the dialog.

You can create as many views as you want to, giving each a friendly name. For this lab though, one dashboard will work well.

The next step is to create a device.

## Create a real device

By "real" IoT Central understands that there's a remote app running. The app can be in a real device, taking input from real sensors, or running a simulation. Both options are treated as a connection to a _real_ device.

1. Click **Devices** in the left-hand menu.

1. Click **RefrigeratedTruck** in the **Devices** menu, to ensure the device we create uses this device template. The device template you select will be shown in bold text.

1. Click **+ New**. Verify in the dialog that the device name includes the **RefrigeratedTruck** text. If it doesn't, you've not selected the right device template.

    [![Screenshot showing the dialog to create a device](../../Linked_Image_files/M99-L20/refrigerated-trucks-create-device.png)](../../Linked_Image_files/M99-L20/refrigerated-trucks-create-device.png#lightbox)

1. Change the **Device ID** to a friendlier name, say "RefrigeratedTruck1".

1. Change the **Device name** to a friendlier name, say "RefrigeratedTruck - 1".

1. Leave the **Simulated** setting at **Off**. We are going to be building a real truck here. Well, a simulated _real_ truck! Setting this value to **On** instructs IoT Central to pump out random values for our telemetry. These random values can be useful in validating a device template.

1. Click **Create**. Wait a few seconds, then your device list should be populated with a single entry. Note the **Device status** is **Registered**. Not until the device status is **Provisioned** will the IoT Central app accept a connection to the device. The coding task that follows shows how to provision a device.

1. Click on the **RefrigeratedTruck - 1** name, and you'll see the live dashboard, with lots of **Waiting for data** messages.

1. Click on the **Commands** entry in the bar that includes **Truck view**. Notice that the two commands you entered are ready to be run.

The next step is to create the keys that will allow a remote device to communicate with this app.

### Record the connection keys

1. Click **Connect** in the top-right menu. Do _not_ click **Connect to gateway**.

1. In the **Device connection** dialog that follows, carefully copy the **ID scope**, **Device ID**, and **Primary key** to a text file. Typically, use a tool like Notepad, and save the file with a meaningful name, say "Truck connections.txt".

1. Leave the **Connect method** as **Shared access signature (SAS)**.

1. When you've saved off the IDs and key, click **Close** on the dialog.

Leave the IoT portal open in your browser, waiting as it is.

