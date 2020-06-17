# Open IoT Edge Gateway Device Inbound Ports using Azure CLI

In this unit, you will use the Azure CLI to configure the Network Security Group (NSG) that secures access to the Azure IoT Edge Gateway from the Internet. The necessary ports for MQTT, AMQP, and HTTPS communications need to be opened so the downstream IoT device(s) can communicate with the gateway.

For the Azure IoT Edge Gateway to communicate with Child IoT Devices, the TCP/IP port for the devices protocol must be open for **Inbound** communication. The device could use one of three supported protocols to communicate with the IoT Gateway.

These are the TCP/IP port numbers for the supported protocols:

| Protocol | Port Number |
| --- | --- |
| MQTT | 8883 |
| AMQP | 5671 |
| HTTPS<br/>MQTT + WS (Websocket)<br/>AMQP + WS (Websocket) | 443 |

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. Within the **Azure portal**, navigate to the **IoTEdgeGateway** resource group.

1. Make note of the resource name of the **Network security group** (NSG) that was created for the **IoTEdgeGateway** virtual machine.

    The name of the NSG should match the format of `{vm-name}-nsg`.

1. Open the Azure Cloud Shell by clicking the **Terminal** icon within the top header bar of the Azure portal, and select the **Bash** shell option.

1. Within the **Azure Cloud Shell**, to find the name of the Network Security Group (NSG) in use by the Azure ioT Edge Gateway VM, enter the following command:

    ```bash
    az network nsg list --resource-group AZ-220-IoTEdgeResources -o table

    Location    Name                     ProvisioningState    ResourceGroup            ResourceGuid
    ----------  -----------------------  -------------------  -----------------------  ------------------------------------
    westus2     AZ220EdgeVM{YOUR-ID}NSG  Succeeded            AZ-220-IoTEdgeResources  <GUID> 
    ```

1. Within the **Azure Cloud Shell**, run the following commands to add **Inbound rules** to the NSG for MQTT, AMQP, and HTTPS communication protocols:

    ```cmd/sh
    az network nsg rule create --name MQTT --nsg-name AZ220EdgeVM{YOUR-ID}NSG --resource-group AZ-220-IoTEdgeResources --destination-port-ranges 8883 --priority 101
    az network nsg rule create --name AMQP --nsg-name AZ220EdgeVM{YOUR-ID}NSG --resource-group AZ-220-IoTEdgeResources --destination-port-ranges 5671 --priority 102
    az network nsg rule create --name HTTPS --nsg-name AZ220EdgeVM{YOUR-ID}NSG --resource-group AZ-220-IoTEdgeResources --destination-port-ranges 443 --priority 103
    ```

    Be sure to replace the placeholders with the appropriate values before running the commands:

    | Placeholder | Value to replace |
    | :--- | :--- |
    | `{nsg-name}` | Enter the name of the **Network Security Group**.
    | `{resource-group}` | Enter the name of the **Resource group** for the virtual machine.

    > [!NOTE] In production, it's best practice to only open inbound communication to the communication protocol(s) in use by your IoT devices. If your devices only use MQTT, then only open inbound communication for that port. This will help limit the surface attack area of open ports that could be exploited.

1. With the **Inbound rules** added to the **Network Security Group** (NSG), the Child IoT Device will be allowed to communicate with the IoT Edge Gateway virtual machine.
