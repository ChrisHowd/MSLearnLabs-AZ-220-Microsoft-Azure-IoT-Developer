---
lab:
    title: 'Lab 16: Automate IoT Device Management with Azure IoT Hub'
    module: 'Module 8: Device Management'
---

# Automate IoT Device Management with Azure IoT Hub

IoT devices often use optimized operating systems or even run code directly on the silicon (without the need for an actual operating system). In order to update the software running on devices like these the most common method is to flash a new version of the entire software package, including the OS as well as the apps running on it (called firmware).

Because each device has a specific purpose, its firmware is also very specific and optimized for the purpose of the device as well as the constrained resources available.

The process for updating firmware can also be specific to the hardware and to the way the hardware manufacturer created the board. This means that a part of the firmware update process is not generic and you will need to work with your device manufacturer to get the details of the firmware update process (unless you are developing your own hardware which means you probably know what the firmware update process).

While firmware updates used to be applied manually on individual devices, this practice no longer makes sense considering the number of devices used in typical IoT solutions. Firmware updates are now more commonly done over-the-air (OTA) with deployments of new firmware managed remotely from the cloud.

There is a set of common denominators to all over-the-air firmware updates for IoT devices:

1. Firmware versions are uniquely identified
1. Firmware comes in a binary file format that the device will need to acquire from an online source
1. Firmware is locally stored is some form of physical storage (ROM memory, hard drive,...)
1. Device manufacturer provide a description of the required operations on the device to update the firmware.

Azure IoT Hub offers advanced support for implementing device management operations on single devices and on collections of devices. The [Automatic Device Management](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config) feature enables you to simply configure a set of operations, trigger them, and then monitor their progress.

## Lab Scenario

The automated air processing system that you implemented in Contoso's cheese caves has helped the company to raise their already high quality bar. The company has more award-winning cheeses than ever before.

Your base solution consists of IoT devices that are integrated with sensors and a climate control system to provide real-time control of temperature and humidity within a multi-chamber cave system. You also developed a simple back-end app that demonstrated the ability to manage devices using both direct methods and device twin properties.

Contoso has extended the simple back-end app from your initial solution to include an online portal that operators can use to monitor and remotely manage the cave environment. With the new portal, operators can even customize the temperature and humidity within the cave based on the type of cheese or for a specific phase within the cheese aging process. Each chamber or zone within the cave can be controlled separately.

The IT department will be maintaining the back-end portal that they developed for the operators, but your manager has agreed to manage the device-side of the solution.

For you, this means two things:

1. The Operations team at Contoso is always looking for ways to make improvements. These improvements often lead to requests for new features in the device software.

1. The IoT devices that are deployed to cave locations need the latest security patches to ensure privacy and to prevent hackers from taking control of the system. In order to maintain system security, you need to keep the devices up to date by remotely updating their firmware.

You plan to implement features of IoT Hub that enable automatic device management and device management at scale.

The following resources will be created:

![Lab 16 Architecture](media/LAB_AK_16-architecture.png)

## In this lab

In this lab, you will complete the following activities:

* Verify that the lab prerequisites are met (that you have the required Azure resources)
* Write code for a simulated device that will implement a firmware update
* Test the firmware update process on a single device using Azure IoT Hub automatic device management

## Lab Instructions

### Exercise 1: Verify Lab Prerequisites

This lab assumes the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name |
| IoT Hub | iot-az220-training-{your-id} |
| IoT Device | sensor-th-0155 |

To ensure these resources are available, complete the following tasks.

1. Select **Deploy to Azure**:

    [![Deploy To Azure](media/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2fARM%2fAllfiles%2FARM%2Flab16.json)

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
    * deviceConnectionString
    * devicePrimaryKey

The resources have now been created.

### Exercise 2: Examine code for a simulated device that implements firmware update

In this exercise, you will review a simulated device that manages the device twin desired property changes and will trigger a local process simulating a firmware update. The process that you implement for launching the firmware update will be similar to the process used for a firmware update on a real device. The process of downloading the new firmware version, installing the firmware update, and restarting the device is simulated.

You will use the Azure Portal to configure and execute a firmware update using the device twin properties. You will configure the device twin properties to transfer the configuration change request to the device and monitor the progress.

#### Task 1: Examine the device simulator app

In this task, you will use Visual Studio Code to review the console app.

1. Open **Visual Studio Code**.

1. On the **File** menu, click **Open Folder**

1. In the Open Folder dialog, navigate to the lab 16 Starter folder.

    In _Lab 3: Setup the Development Environment_, you cloned the GitHub repository containing lab resources by downloading a ZIP file and extracting the contents locally. The extracted folder structure includes the following folder path:

    * Allfiles
      * Labs
          * 16-Automate IoT Device Management with Azure IoT Hub
            * Starter
              * FWUpdateDevice

1. Click **FWUpdateDevice**, and then click **Select Folder**.

    You should see the following files listed in the EXPLORER pane of Visual Studio Code:

    * FWUpdateDevice.csproj
    * Program.cs

1. In the **EXPLORER** pane, click the **FWUpdateDevice.csproj** file to open it, and note the referenced NuGet packages:

    * Microsoft.Azure.Devices.Client -Device SDK for Azure IoT Hub
    * Microsoft.Azure.Devices.Shared - Common code for Azure IoT Device and Service SDKs
    * Newtonsoft.Json - Json.NET is a popular high-performance JSON framework for .NET

1. In the **EXPLORER** pane, click **Program.cs**.

#### Task 2: Review the application code

In this task, you will review the code for simulating a firmware update on the device in response to an IoT Hub generated request.

1. Ensure that you have the **Program.cs** file open in Visual Studio Code.

1. Locate the `Global Variables` comment.

    In this simple example, a device connection string, the device ID and the current firmware version are tracked.

1. In the code editor, locate the following line of code:

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. Replace the **\<your device connection string\>** with the device connection string that you saved earlier.

    This is the only code change required.

1. Locate the **Main** method.

    This method is similar to the device simulators used earlier - the **deviceConnectionString** is used to create a **DeviceClient** instance to connect to IoT Hub, etc. and the device twin property changed callback is configured.

    The **InitDevice** method is new and merely simulates the bootup cycle of a device and reports the current firmware by updating the device twin via the **UpdateFWUpdateStatus** method.

    The app then loops, waiting for a device twin update that will trigger a firmware update.

1.Locate the **UpdateFWUpdateStatus** method and review the code:

    This method creates a new **TwinCollection** instance, populates it with the provided values, and then updates the device twin.

1. Locate the **OnDesiredPropertyChanged** method and review the code:

    This method is invoked as the callback when a device twin update is received by the device. If a firmware update is detected, the **UpdateFirmware** method is called. This method simulate the download of the firmware, updating the firmware and then rebooting the device.

### Exercise 3: Test firmware update on a single device

In this exercise, you will use the Azure portal to create a new device management configuration and apply it to our single simulated device.

#### Task 1: Start device simulator

1. If necessary, open your **FWUpdateDevice** project in Visual Studio Code.

1. Ensure that you have the Terminal pane open.

    The folder location of the command prompt be the `FWUpdateDevice` folder.

1. To run the `FWUpdateDevice` app, enter the following command:

    ``` bash
    dotnet run "<device connection string>"
    ```

    > **Note**: Remember to replace the placeholder with the actual device connection string, and be sure to include "" around your connection string.
    >
    > For example: `"HostName=iot-az220-training-{your-id}.azure-devices.net;DeviceId=sensor-th-0155;SharedAccessKey={}="`

1. Review the contents of the Terminal pane.

    You should see the following output in the terminal:

    ``` bash
        sensor-th-0155: Device booted
        sensor-th-0155: Current firmware version: 1.0.0
    ```

#### Task 2: Create the device management configuration

1. If necessary, log in to your [Azure portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On your Azure portal Dashboard, click **iot-az220-training-{your-id}**.

    Your IoT Hub blade should now be displayed.

1. On the left side navigation menu, under **Automatic Device Management**, click **IoT device configuration**.

1. On the **IoT device configuration** pane, click **+ Add Device Configuration**.

1. On the **Create Device Twin Configuration** blade, under **Name**, enter **firmwareupdate**

    Ensure that you enter `firmwareupdate` under the the required **Name** field for the configuration, not under **Labels**.

1. At the bottom of the blade, click **Next: Twins Settings >**.

1. Under **Device Twin Settings**, in the **Device Twin Property** field, enter **properties.desired.firmware**

1. In the **Device Twin Property Content** field, enter the following:

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```

1. At the bottom of the blade, click **Next: Metrics >**.

    You will be using a custom metric to track whether the firmware update was effective.

1. On the **Metrics** tab, under **METRIC NAME**, enter **fwupdated**

1. Under **METRIC CRITERIA**, enter the following:

    ``` SQL
    SELECT deviceId FROM devices
        WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```

1. At the bottom of the blade, click **Next: Target devices >**.

1. On the **Target Devices** tab, under **Priority**, in the **Priority (higher values ...)** field, enter **10**.

1. Under **Target Condition**, in the **Target Condition** field, enter the following query:

    ``` SQL
    deviceId='<your device id>'
    ```

    > **Note**: Be sure to replace `'<your device id>'` with the Device ID that you used to create the device. For example: `'sensor-th-0155'`

1. At the bottom of the blade, click **Next: Review + Create >**

    When the **Review + create** tab opens, you should see a "Validation passed" message for your new configuration.

1. On the **Review + create** tab, if the "Validation passed" message is displayed, click **Create**.

    If the "Validation passed" message is displayed, you will need to go back and check your work before you can create your configuration.

1. On the **IoT device configuration** pane, under **Configuration Name**, verify that your new **firmwareupdate** configuration is listed.

    Once the new configuration is created, IoT Hub will look for devices matching the configuration's target devices criteria, and will apply the firmware update configuration automatically.

1. Switch to the Visual Studio Code window, and review the contents of the Terminal pane.

    The Terminal pane should include new output generated by your app that lists the progress of the firmware update process that was triggered.

1. Stop the simulated app, and close Visual Studio Code.

    You can stop the device simulator by simply pressing the "Enter" key in the terminal.
