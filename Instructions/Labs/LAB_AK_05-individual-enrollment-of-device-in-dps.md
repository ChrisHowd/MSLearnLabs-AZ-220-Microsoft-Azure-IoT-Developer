---
lab:
    title: 'Lab 05: Individual Enrollment of a Device in DPS'
    module: 'Module 3: Device Provisioning at Scale'
---

# Individual Enrollment of a Device in DPS

## Lab Scenario

Contoso management is pushing for an update to their existing Asset Monitoring and Tracking Solution. The update will use IoT devices to reduce the manual data entry work that is required under the current system and provide more advanced monitoring during the shipping process. The solution relies on the ability to provision IoT devices when shipping containers are loaded and deprovision the devices when the container arrives at the destination. The best option for managing the provisioning requirements appears to be the IoT Hub Device Provisioning Service (DPS).

The proposed system will use IoT devices with integrated sensors for tracking the location, temperature, and pressure of shipping containers during transit. The IoT devices will be placed within the existing shipping containers that Contoso uses to transport their cheese, and will connect to Azure IoT Hub using vehicle-provided WiFi. The new system will provide continuous monitoring of the product environment and enable a variety of notification scenarios when issues are detected. The rate at which telemetry is sent to IoT hub must be configurable.

In Contoso's cheese packaging facility, when an empty container enters the system it will be equipped with the new IoT device and then loaded with packaged cheese products. The IoT device will be auto-provisioned to IoT hub using DPS. When the container arrives at the destination, the IoT device will be retrieved and must be fully deprovisioned (disenrolled and deregistered). The recovered devices will be recycled and re-used for future shipments following the same auto-provisioning process.

You have been tasked with validating the device provisioning and deprovisioning process using DPS. For the initial testing phase you will use an Individual Enrollment approach.

The following resources will be created:

![Lab 5 Architecture](media/LAB_AK_05-architecture.png)

## In This Lab

In this lab, you will begin by reviewing the lab prerequisites and you will run a script if needed to ensure that your Azure subscription includes the required resources. You will then create a new individual enrollment in DPS that uses Symmetric Key attestation and specifies an initial Device Twin State (telemetry rate) for the device. With the device enrollment saved, you will go back into the enrollment and get the auto-generated Primary and Secondary keys needed for device attestation. Next, you create a simulated device and verify that device connects successfully with IoT hub and that the initial device twin properties are applied by the device as expected. To finish up, you will complete a deprovisioning process that securely removes the device from your solution by both disenrolling and deregistering the device (from DPS and IoT hub respectively). The lab includes the following exercises:

* Verify Lab Prerequisites
* Create new individual enrollment (Symmetric keys) in DPS
* Configure Simulated Device
* Test the Simulated Device
* Deprovision the Device

## Lab Instructions

### Exercise 1: Verify Lab Prerequisites

This lab assumes that the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name |
| IoT Hub | iot-az220-training-{your-id} |
| Device Provisioning Service | dps-az220-training-{your-id} |

To ensure these resources are available, complete the following tasks.

1. Select **Deploy to Azure**:

    [![Deploy To Azure](media/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab05.json)

1. If prompted, login to the **Azure Portal**.

    The **Custom deployment** page will be displayed.

1. Under **Project details**, in the **Subscription** dropdown, ensure that the Azure subscription that you intend to use for this course is selected.

1. In the **Resource group** dropdown, select **@lab.CloudResourceGroup(ResourceGroup1).Name**.

    > **NOTE**: If **@lab.CloudResourceGroup(ResourceGroup1).Name** is not listed:
    >
    > 1. Under the **Resource group** dropdown, click **Create new**.
    > 1. Under **Name**, enter **@lab.CloudResourceGroup(ResourceGroup1).Name**.
    > 1. Click **OK**.

1. Under **Instance details**, in the **Region** dropdown, select the region closest to you.

    > **NOTE**: If the **@lab.CloudResourceGroup(ResourceGroup1).Name** group already exists, the **Region** field is set to the region used by the resource group and is read-only.

1. In the **Your ID** field, enter the unique ID you created in Exercise 1.

1. In the **Course ID** field, enter **az220**.

1. To validate the template, click **Review and create**.

1. If validation passes, click **Create**.

    The deployment will start.

1. Once the deployment has completed, in the left navigation area, to review any output values from the template,  click **Outputs**.

    Make a note of the outputs for use later:

    * connectionString
    * dpsScopeId

The resources have now been created.

### Exercise 2: Create new individual enrollment (Symmetric keys) in DPS

In this exercise, you will create a new individual enrollment for a device within the Device Provisioning Service (DPS) using _symmetric key attestation_. You will also configure the initial device state within the enrollment. After saving your enrollment, you will go back in and obtain the auto-generated attestation Keys that get created when the enrollment is saved.

#### Task 1: Create the enrollment

1. If necessary, log in to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. Notice that the **AZ-220** dashboard has been loaded and your Resources tile is displayed.

    You should see both your IoT Hub and DPS resources listed.

1. On the **@lab.CloudResourceGroup(ResourceGroup1).Name** resource group tile, click **dps-az220-training-{your-id}**.

1. On the left-side menu under **Settings**, click **Manage enrollments**.

1. At the top of the **Manage enrollments** pane, click **+ Add individual enrollment**.

1. On the **Add Enrollment** blade, in the **Mechanism** dropdown, click **Symmetric Key**.

    This sets the attestation method to use Symmetric key authentication.

1. Just below the Mechanism setting, notice that the **Auto-generate keys** option is checked.

    This sets DPS to automatically generate both the **Primary Key** and **Secondary Key** values for the device enrollment when it's created. Optionally, un-checking this option enables custom keys to be manually entered.

    > **Note**: The Primary Key and Secondary Key values are generated after this record is saved. In the next task you will go back into this record to obtain the values, and then use them within a simulated device app later in this lab.

1. In the **Registration ID** field, to specify the Registration ID to use for the device enrollment within DPS, enter **sensor-thl-1000**

    By default, the Registration ID will be used as the IoT Hub Device ID when the device is provisioned from the enrollment. When these values need to be different, you can enter the required IoT Hub Device ID in that field.

1. Leave the **IoT Hub Device ID** field blank.

    Leaving this field blank ensures that the IoT Hub will use the Registration ID as the Device ID. Don't worry if you see a default text value in the field that is not selectable - this is placeholder text and will not be treated as an entered value.

1. Leave the **IoT Edge device** field set to **False**.

    The new device will not be an edge device. Working with IoT Edge devices will be discussed later in the course.

1. Leave the **Select how you want to assign devices to hubs** field set to **Evenly weighted distribution**.

    As you only have one IoT Hub associated with the enrollment, this setting is somewhat unimportant.  In larger environments where you have multiple distributed hubs, this setting will control how to choose what IoT Hub should receive this device enrollment. There are four supported allocation policies:

    * **Lowest latency**: Devices are provisioned to an IoT hub based on the hub with the lowest latency to the device.
    * **Evenly weighted distribution (default)**: Linked IoT hubs are equally likely to have devices provisioned to them. This is the default setting. If you are provisioning devices to only one IoT hub, you can keep this setting.
    * **Static configuration via the enrollment list**: Specification of the desired IoT hub in the enrollment list takes priority over the Device Provisioning Service-level allocation policy.
    * **Custom (Use Azure Function)**: the device provisioning service calls your Azure Function code providing all relevant information about the device and the enrollment. Your function code is executed and returns the IoT hub information used to provisioning the device.

1. Notice that the **Select the IoT hubs this device can be assigned to** dropdown specifies the **iot-az220-training-{your-id}** IoT hub that you created.

    This field is used to specify the IoT Hub(s) that your device can be assigned to.

1. Leave the **Select how you want device data to be handled on re-provisioning** field set to the default value of **Re-provision and migrate data**.

    This field gives you high-level control over the re-provisioning behavior, where the same device (as indicated through the same Registration ID) submits a later provisioning request after already being provisioned successfully at least once. There are three options available:

    * **Re-provision and migrate data**: This policy is the default for new enrollment entries. This policy takes action when devices associated with the enrollment entry submit a new provisioning request. Depending on the enrollment entry configuration, the device may be reassigned to another IoT hub. If the device is changing IoT hubs, the device registration with the initial IoT hub will be removed. All device state information from that initial IoT hub will be migrated over to the new IoT hub.
    * **Re-provision and reset to initial config**: This policy is often used for a factory reset without changing IoT hubs. This policy takes action when devices associated with the enrollment entry submit a new provisioning request. Depending on the enrollment entry configuration, the device may be reassigned to another IoT hub. If the device is changing IoT hubs, the device registration with the initial IoT hub will be removed. The initial configuration data that the provisioning service instance received when the device was provisioned is provided to the new IoT hub.
    * **Never re-provision**: The device is never reassigned to a different hub. This policy is provided for managing backwards compatibility.

1. In the **Initial Device Twin State** field, to specify a property named `telemetryDelay` with the value of `"2"`, update the JSON object as follows:

    The final JSON will be like the following:

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "2"
            }
        }
    }
    ```

    This field contains JSON data that represents the initial configuration of desired properties for the device. The data that you entered will be used by the Device to set the time delay for reading sensor telemetry and sending events to IoT Hub.

1. Leave the **Enable entry** field set to **Enable**.

    Generally, you'll want to enable new enrollment entries and keep them enabled.

1. At the top of the **Add Enrollment** blade, click **Save**.

#### Task 2: Review Enrollment and Obtain Authentication Keys

1. On the **Manage enrollments** pane, to view the list of individual device enrollments, click **individual enrollments**.

    As you may recall, you will be using the enrollment record to obtain the Authentication keys.

1. Under **REGISTRATION ID**, click **sensor-thl-1000**.

    This blade enables you to view the enrollment details for the individual enrollment that you just created.

1. Locate the **Authentication Type** section.

    Since you specified the Authentication Type as Symmetric Key when you created the enrollment, the Primary and Secondary key values have been created for you. Notice that there is a button to the right of each textbox that you can use to copy the values.

1. Copy the **Primary Key** and **Secondary Key** values for this device enrollment, and then save them to a file for later reference.

    These are the authentication keys for the device to authenticate with the IoT Hub service.

1. Locate the **Initial device twin State**, and notice the JSON for the device twin Desired State contains the `telemetryDelay` property set to the value of `"2"`.

1. Close the **sensor-thl-1000** individual enrollment blade.

### Exercise 3: Configure Simulated Device

In this exercise, you will configure a Simulated Device written in C# to connect to Azure IoT using the individual enrollment created in the previous exercise. You will also add code to the Simulated Device that will read and update device configuration based on the device twin within Azure IoT Hub.

The simulated device that you create in this exercise represents an IoT device that will be located within a shipping container/box, and will be used to monitor Contoso products while they are in transit. The sensor telemetry from the device that will be sent to Azure IoT Hub includes Temperature, Humidity, Pressure, and Latitude/Longitude coordinates of the container. The device is part of the overall asset tracking solution.

> **Note**: You may have the impression that creating this simulated device is a bit redundant with what you created in the previous lab, but the attestation mechanism that you implement in this lab is quite different from what you did previously. In the previous lab, you used a shared access key to authenticate, which does not require device provisioning, but also does not give the provisioning management benefits (such as leveraging device twins), and it requires fairly large distribution and management of a shared key. In this lab, you are provisioning a unique device through the Device Provisioning Service.

#### Task 1: Create the Simulated Device

1. On the left-side menu of the **dps-az220-training-{your-id}** blade, click **Overview**.

1. In the top-right area of the blade, hover the mouse pointer over value assigned to **ID Scope**, and then click **Copy to clipboard**.

    You will be using this value shortly, so make note of the value if you are unable to use the clipboard. Be sure to differentiate between uppercase "O" and the number "0".

    The **ID Scope** will be similar to this value: `0ne0004E52G`

1. Open **Visual Studio Code**.

1. On the **File** menu, click **Open Folder** and then navigate to the Starter folder for Lab 5.

    The Lab 5 Starter folder is part of the lab resources files that you downloaded when setting up your development environment in lab 3. The folder path is:

    * Allfiles
      * Labs
          * 05-Individual Enrollment of a Device in DPS
            * Starter

1. In the **Open Folder** dialog, click **ContainerDevice**, and then click **Select Folder**.

    The ContainerDevice folder is a sub-folder of the Lab 5 Starter folder. It contains a Program.cs file and a ContainerDevice.csproj file.

    > **Note**: If Visual Studio Code prompts you to load required assets, you can click **Yes** to load them.

1. On the **View** menu, click **Terminal**.

    Verify that the selected terminal shell is the windows command prompt.

1. At the Terminal command prompt, to restore all the application NuGet packages, enter the following command:

    ```cmd/sh
    dotnet restore
    ```

1. In the Visual Studio Code **EXPLORER** pane, click **Program.cs**.

1. In the code editor, near the top of the Program class, locate the **dpsIdScope** variable.

1. Update the value assigned to **dpsIdScope** using the ID Scope that you copied from the Device Provisioning Service.

    > **Note**: If you don't have the value of ID Scope available to you, you can find it on the Overview blade of the DPS service (in the Azure portal).

1. Locate the **registrationId** variable, and update the assigned value using **sensor-thl-1000**

    This variable represents the **Registration ID** value for the individual enrollment that you created in the Device Provisioning Service.

1. Update the **individualEnrollmentPrimaryKey** and **individualEnrollmentSecondaryKey** variables using the **Primary Key** and **Secondary Key** values that you saved.

    > **Note**: If you don't have these Key values available, you can copy them from the Azure portal as follows -
    >
    > Open the **Manage enrollments** blade, click **Individual Enrollments**, click **sensor-thl-1000**. Copy the values and then paste as noted above.

1. On the Visual Studio Code **File** menu, click **Save**.

    Your simulated device will now use the device twin properties from Azure IoT Hub to set the delay between telemetry messages.

    > **Note**: The source code is extensively commented should you wish to dive into how the application connects via DPS, etc.

### Exercise 4: Test the Simulated Device

In this exercise, you will run the Simulated Device and verify that it's sending sensor telemetry to Azure IoT Hub. You will also change the rate at which telemetry is sent to Azure IoT Hub by updating the telemetryDelay device twin property for the simulated device within Azure IoT Hub.

#### Task 1: Build and run the device

1. Ensure that you have your code project open in Visual Studio Code.

1. On the **View** menu, click **Terminal**.

1. In the Terminal pane, ensure the command prompt shows the directory path for the `Program.cs` file.

1. At the command prompt, to build and run the Simulated Device application, enter the following command:

    ```cmd/sh
    dotnet run
    ```

    > **Note**: When the Simulated Device application runs, it will first write some details about it's status to the console (terminal pane).

1. Notice that the JSON output following the `Desired Twin Property Changed:` line contains the desired value for the `telemetryDelay` for the device.

    You can scroll up in the terminal pane to review the output. It should be similar to the following:

    ```text
    ProvisioningClient AssignedHub: iot-az220-training-{your-id}.azure-devices.net; DeviceID: sensor-thl-1000
    Desired Twin Property Changed:
    {"telemetryDelay":"2","$version":1}
    Reported Twin Properties:
    {"telemetryDelay":"2"}
    Start reading and sending device telemetry...
    ```

1. Notice that the Simulated Device application begins sending telemetry events to the Azure IoT Hub.

    The telemetry events include values for `temperature`, `humidity`, `pressure`, `latitude`, and `longitude`, and should be similar to the following:

    ```text
    11/6/2019 6:38:55 PM > Sending message: {"temperature":25.59094770373355,"humidity":71.17629229611545,"pressure":1019.9274696347665,"latitude":39.82133964767944,"longitude":-98.18181981142438}
    11/6/2019 6:38:57 PM > Sending message: {"temperature":24.68789062681044,"humidity":71.52098010830628,"pressure":1022.6521258267584,"latitude":40.05846882452387,"longitude":-98.08765031156229}
    11/6/2019 6:38:59 PM > Sending message: {"temperature":28.087463226675737,"humidity":74.76071353757787,"pressure":1017.614206096327,"latitude":40.269273772972454,"longitude":-98.28354453319591}
    11/6/2019 6:39:01 PM > Sending message: {"temperature":23.575667940813894,"humidity":77.66409506912534,"pressure":1017.0118147748344,"latitude":40.21020096551372,"longitude":-98.48636739129239}
    ```

    Notice the timestamp differences between telemetry readings. The delay between telemetry messages should be `2` seconds as configured through the device twin; instead of the default of `1` second in the source code.

1. Leave the simulated device app running.

    You will verify that the device code is behaving as expected during the next activities.

#### Task 2: Verify Telemetry Stream sent to Azure IoT Hub

In this task, you will use the Azure CLI to verify telemetry sent by the simulated device is being received by Azure IoT Hub.

1. Using a browser, open the [Azure Cloud Shell](https://shell.azure.com/) and login with the Azure subscription you are using for this course.

1. In the Azure Cloud Shell, enter the following command:

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id sensor-thl-1000
    ```

    _Be sure to replace the **{IoTHubName}** placeholder with the name of your Azure IoT Hub._

1. Notice that your IoT hub is receiving the telemetry messages from the sensor-thl-1000 device.

    Continue to leave the simulated device application running for the next task.

#### Task 3: Change the device configuration through its twin

With the simulated device running, the `telemetryDelay` configuration can be updated by editing the device twin Desired State within Azure IoT Hub. This can be done by configuring the Device in the Azure IoT Hub within the Azure portal.

1. Open the Azure portal (if it is not already open), and then navigate to your **Azure IoT Hub** service.

1. On the IoT Hub blade, on the left-side menu under **Explorers**, click **IoT devices**.

1. Under **DEVICE ID**, click **sensor-thl-1000**.

    > **IMPORTANT**: Make sure you select the device that you are using for this lab.

1. On the **sensor-thl-1000** device blade, at the top of the blade, click **Device Twin**.

    The **Device twin** blade provides an editor with the full JSON for the device twin. This enables you to view and/or edit the device twin state directly within the Azure portal.

1. Locate the JSON for the `properties.desired` object.

    This contains the desired state for the device. Notice the `telemetryDelay` property already exists, and is set to `"2"`, as was configured when the device was provisioned based on the Individual Enrollment in DPS.

1. To update the value assigned to the `telemetryDelay` desired property, change the value to `"5"`

    The value includes the quotes ("").

1. At the top of the **Device twin** blade, click **Save**

    The `OnDesiredPropertyChanged` event will be triggered automatically within the code for the Simulated Device, and the device will update its configuration to reflect the changes to the device twin Desired state.

1. Switch to the Visual Studio Code window that you are using to run the simulated device application.

1. In Visual Studio Code, scroll to the bottom of the Terminal pane.

1. Notice that the device recognizes the change to the device twin properties.

    The output will show a message that the `Desired Twin Property Changed` along with the JSON for the new desired`telemetryDelay` property value. Once the device picks up the new configuration of device twin desired state, it will automatically update to start sending sensor telemetry every 5 seconds as now configured.

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":"5","$version":2}
    Reported Twin Properties:
    {"telemetryDelay":"5"}
    4/21/2020 1:20:16 PM > Sending message: {"temperature":34.417625961088405,"humidity":74.12403526442313,"pressure":1023.7792049974805,"latitude":40.172799921919186,"longitude":-98.28591913777421}
    4/21/2020 1:20:22 PM > Sending message: {"temperature":20.963297521678403,"humidity":68.36916032636965,"pressure":1023.7596862048422,"latitude":39.83252821949164,"longitude":-98.31669969393461}
    ```

1. Switch to the browser page where you are running the Azure CLI command in the Azure Cloud Shell.

    Ensure that you are still running the `az iot hub monitor-events` command. If it isn't running, re-start the command.

1. Notice that the telemetry events sent to Azure IoT Hub being received at the new interval of 5 seconds.

1. Use **Ctrl-C** to stop both the `az` command and the Simulated Device application.

1. Switch to your browser window for the Azure portal.

1. Close the **Device twin** blade.

1. Still in the Azure Portal, on the **sensor-thl-1000** device blade, click **Device Twin**.

1. Locate the JSON for the `properties.reported` object.

    This portion of the JSON contains the state reported by the device. Notice the `telemetryDelay` property exists here as well, and is also set to `5`.  There is also a `$metadata` value that shows you when the value was reported data was last updated and when the specific reported value was last updated.

1. Close the **Device twin** blade.

1. Close the simulated device blade, and then close the IoT Hub blade.

### Exercise 5: Deprovision the Device

In your Contoso scenario, when the shipping container arrives at it's final destination, the IoT device will be removed from the container and returned to a Contoso location. Contoso will need to deprovision the device before it can be tested and placed in inventory. In the future the device could be provisioned to the same IoT hub or an IoT hub in a different region. Complete device deprovisioning is an important step in the life cycle of IoT devices within an IoT solution.

In this exercise, you will perform the tasks necessary to deprovision the device from both the Device Provisioning Service (DPS) and Azure IoT Hub. To fully deprovision an IoT device from an Azure IoT solution it must be removed from both of these services.

#### Task 1: Disenroll the device from the DPS

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On your Resource group tile, to open your Device Provisioning Service, click **dps-az220-training-{your-id}**.

1. On the left-side menu under **Settings**, click **Manage enrollments**.

1. On the **Manage enrollments** pane, to view the list of individual device enrollments, click **Individual Enrollments**.

1. To the left of **sensor-thl-1000**, click the checkbox.

    > **Note**: You don't want to open the sensor-thl-1000 individual device enrollment, you just want to select it.

1. At the top of the blade, click **Delete**.

    > **Note**: Deleting the individual enrollment from DPS will permanently remove the enrollment. To temporarily disable the enrollment, you can set the **Enable entry** setting to **Disable** within the **Enrollment Details** for the individual enrollment.

1. On the **Remove enrollment** prompt, click **Yes**.

    The individual enrollment is now removed from the Device Provisioning Service (DPS). To complete the deprovisioning process, the **Device ID** for the Simulated Device also must be removed from the **Azure IoT Hub** service.

#### Task 2: Deregister the device from the IoT Hub

1. In the Azure portal, navigate back to your Dashboard.

1. On your Resource group tile, to open your Azure IoT Hub blade, click **iot-az220-training-{your-id}**.

1. On the left-side menu under **Explorers**, click **IoT devices**.

1. To the left of **sensor-thl-1000**, click the checkbox.

    > **IMPORTANT**: Make sure you select the device representing the simulated device that you used for this lab.

1. At the top of the blade, click **Delete**.

1. On the **Are you certain you wish to delete selected device(s)** prompt, click **Yes**.

    > **Note**:  Deleting the device ID from IoT Hub will permanently remove the device registration. To temporarily disable the device from connecting to IoT Hub, you can set the **Enable connection to IoT Hub** to **Disable** within the properties for the device.

Now that the Device Enrollment has been removed from the Device Provisioning Service, and the matching Device ID has been removed from the Azure IoT Hub, the simulated device has been fully retired from the solution.
