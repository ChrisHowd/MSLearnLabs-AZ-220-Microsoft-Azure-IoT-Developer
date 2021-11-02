---
lab:
    title: 'Lab 07: Device Message Routing'
    module: 'Module 4: Message Processing and Analytics'
---

# Device Message Routing

## Lab Scenario

Contoso Management is impressed with your implementation of automatic device enrollment using DPS. They are now interested in having you develop an IoT-based solution related to product packaging and shipping.

The cost associated with packaging and shipping cheese is significant. To maximize cost efficiency, Contoso operates an on-premises packaging facility. The workflow is straightforward - cheese is cut and packaged, packages are assembled into shipping containers, containers are delivered to specific bins associated with their destination. A conveyor belt system is used to move the product through this process. The metric for success is the number of packages leaving the conveyor belt system during a given time period (typically a work shift).

The conveyor belt system is a critical link in this process and is visually monitored to ensure that the workflow is progressing at maximum efficiency. The system has three operator controlled speeds: stopped, slow, and fast. Naturally, the number of packages being delivered at the low speed is less than at the higher speed. However, there are a number of other factors to consider:

* the vibration level of the conveyor belt system is much lower at the slow speed
* high vibration levels can cause packages to fall from the conveyor
* high vibration levels are known to accelerate wear-and-tear of the system
* when vibration levels exceed a threshold limit, the conveyor belt must be stopped to allow for inspection (to avoid more serious failures)

In addition to maximizing throughput, your automated IoT solution will implement a form of preventive maintenance based on vibration levels, which will be used to detect early warning signs before serious system damage occurs.

> **Note**: **Preventive maintenance** (sometimes called preventative maintenance or predictive maintenance) is an equipment maintenance program that schedules maintenance activities to be performed while the equipment is operating normally. The intent of this approach is to avoid unexpected breakdowns that often incur costly disruptions.

It's not always easy for an operator to visually detect abnormal vibration levels. For this reason, you are looking into an Azure IoT solution that will help to measure vibration levels and data anomalies. Vibration sensors will be attached to the conveyor belt at various locations, and you will use IoT devices to send telemetry to IoT hub. The IoT hub will use Azure Stream Analytics, and a built-in Machine Learning (ML) model, to alert you to vibration anomalies in real time. You also plan to archive all of the telemetry data so that in-house machine learning models can be developed in the future.

You decide to prototype the solution using simulated telemetry from a single IoT device.

To simulate the vibration data in a realistic manner, you work with an engineer from Operations to understand a little bit about what causes the vibrations. It turns out there are a number of different types of vibration that contribute to the overall vibration level. For example, a "force vibration" could be introduced by a broken guide wheel or an especially heavy load placed improperly on the conveyor belt. There's also an "increasing vibration", that can be introduced when a system design limit (such as speed or weight) is exceeded. The Engineering team agrees to help you develop the code for a simulated IoT device that will produce an acceptable representation of vibration data (including anomalies).

The following resources will be created:

![Lab 7 Architecture](media/LAB_AK_07-architecture.png)

## In This Lab

In this lab, you will begin by reviewing the lab prerequisites and you will run a script if needed to ensure that your Azure subscription includes the required resources. You will then create a simulated device that sends vibration telemetry to your IoT hub. With your simulated data arriving at IoT hub, you will implement an IoT Hub Message Route and Azure Stream Analytics job that can be used to archive data. The lab includes the following exercises:

* Verify Lab Prerequisites

  * A script will be used to create any missing resources and a new device identity (sensor-v-3000) for this lab

* Write Code to generate Vibration Telemetry
* Create a Message Route to Azure Blob Storage
* Logging Route Azure Stream Analytics Job

## Lab Instructions

### Exercise 1: Verify Lab Prerequisites

This lab assumes that the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name |
| IoT Hub | iot-az220-training-{your-id} |
| Device ID | sensor-v-3000 |

To ensure these resources are available, complete the following tasks.

1. To create the required resources, open a new browser tab and enter the following address:

    [https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab07.json](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab07.json)

    ```url
    https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab07.json
    ```

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

The resources have now been created.

### Exercise 2: Write Code to generate Vibration Telemetry

Both long term and real-time data analysis are required to automate the monitoring of Contoso's conveyor belt system and enable predictive maintenance. Since no historical data exists, your first step will be to generate simulated data that mimics vibration data and data anomalies in a realistic manner. Contoso engineers have developed an algorithm to simulate vibration over time and embedded the algorithm within a code class that you will implement. The engineers have agreed to support any future updates required to adjust the algorithms.

During your initial prototype phase, you will implement a single IoT device that generates telemetry data. In addition to the vibration data, your device will create some additional values (packages delivered, ambient temperature, and similar metrics) that will be sent to Blob storage. This additional data simulates the data that will be used to develop machine learning modules for predictive maintenance.

In this exercise, you will:

* load the simulated device project
* update the connection string for your simulated device and review the project code
* test your simulated device connection and telemetry communications
* ensure that telemetry is arriving at your IoT hub

#### Task 1: Open your simulated device project

1. Open **Visual Studio Code**.

1. On the **File** menu, click **Open Folder**.

1. In the **Open Folder** dialog, navigate to the **07-Device Message Routing** folder.

    In _Lab 3: Setup the Development Environment_, you cloned the GitHub repository containing lab resources by downloading a ZIP file and extracting the contents locally. The extracted folder structure includes the following folder path:

    * Allfiles
        * Labs
            * 07-Device Message Routing
                * Starter
                    * VibrationDevice

1. Navigate to the **Starter** folder for Lab 7.

1. Click **VibrationDevice**, and then click **Select Folder**.

    You should see the following files listed in the EXPLORER pane of Visual Studio Code:

    * Program.cs
    * VibrationDevice.csproj

    > **Note**: If you are prompted to load required assets, you can do that now.

1. In the **EXPLORER** pane, click **Program.cs**.

    A cursory glance will reveal that the **VibrationDevice** application is very similar to those used in the preceding labs. This version of the application uses symmetric Key authentication, sends both telemetry and logging messages to the IoT Hub, and has a more complex sensor implementation.

1. On the **Terminal** menu, click **New Terminal**.

    Examine the directory path indicated as part of the command prompt to ensure that you are in the correct location. You do not want to start building this project within the folder structure of a previous lab project.

1. At the terminal command prompt, to verify that the application builds without errors, enter the following command:

    ```cmd
    dotnet build
    ```

    The output will be similar to:

    ```text
    ❯ dotnet build
    Microsoft (R) Build Engine version 16.5.0+d4cbfca49 for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 39.27 ms for D:\Az220-Code\AllFiles\Labs\07-Device Message Routing\Starter\VibrationDevice\VibrationDevice.csproj.
    VibrationDevice -> D:\Az220-Code\AllFiles\Labs\07-Device Message Routing\Starter\VibrationDevice\bin\Debug\netcoreapp3.1\VibrationDevice.dll

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:01.16
    ```

In the next task, you will configure the connection string and review the application.

#### Task 2: Configure connection and review code

The simulated device app that you will build in this task simulates an IoT device that is monitoring the conveyor belt. The app will simulate sensor readings and report vibration sensor data every two seconds.

1. Ensure that you have the **Program.cs** file opened in Visual Studio Code.

1. Near the top of the **Program** class, locate the declaration of the `deviceConnectionString` variable:

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. Replace `<your device connection string>` with the device connection string that you saved earlier.

    > **Note**: This is the only change that you are required to make to this code.

1. On the **File** menu, click **Save**.

1. Optionally, take a moment to review the code.

#### Task 3: Test your code to send telemetry

1. At the Terminal command prompt, to run the app, enter the following command:

    ```bash
    dotnet run
    ```

   This command will run the **Program.cs** file in the current folder.

1. Console output should be displayed that is similar to the following:

    ```text
    Vibration sensor device app.

    Telemetry data: {"vibration":0.0}
    Telemetry sent 10:29 AM
    Log data: {"vibration":0.0,"packages":0,"speed":"stopped","temp":60.22}
    Log data sent

    Telemetry data: {"vibration":0.0}
    Telemetry sent 10:29 AM
    Log data: {"vibration":0.0,"packages":0,"speed":"stopped","temp":59.78}
    Log data sent
    ```

    > **Note**:  In the Terminal window, green text is used to show things are working as they should and red text when bad stuff is happening. If you receive error messages, start by checking your device connection string.

1. Leave this app running for the next task.

    If you won't be continuing to the next task, you can enter **Ctrl-C** in the Terminal window to stop the app. You can start it again later by using the `dotnet run` command.

#### Task 4: Verify the IoT Hub is Receiving Telemetry

In this task, you will use the Azure portal to verify that your IoT Hub is receiving telemetry.

1. Open the [Azure Portal](https://portal.azure.com).

1. On your Resources tile, click **iot-az220-training-{your-id}**.

1. On the **Overview** pane, scroll down to view the metrics tiles.

1. Adjacent to **Show data for last**, change the time range to one hour.

    The **Device to cloud messages** tile should be plotting some current activity. If no activity is shown, wait a short while, as there's some latency.

    With your device pumping out telemetry, and your hub receiving it, the next step is to route the messages to their correct endpoints.

### Exercise 3: Create a Message Route to Azure Blob Storage

IoT solutions often require that incoming message data be sent to multiple endpoint locations, either dependent upon the type of data or for business reasons. Azure IoT hub provides the _message routing_ feature to enable you to direct incoming data to locations required by your solution.

The architecture of our system requires data be sent to two destinations: a storage location for archiving data, and a location for more immediate analysis.

Contoso's vibration monitoring scenario requires you to create two message routes:

* the first route will be to an Azure Blob storage location for data archiving
* the second route will be to an Azure Stream Analytics job for real-time analysis

Message routes should be built and tested one at a time, so this exercise will focus on the storage route. This route will be referred to as the "logging" route, and it involves digging a few levels deep into the creation of Azure resources.

One important feature of message routing is the ability to filter incoming data before routing to an endpoint. The filter, written as a SQL query, directs output through a route only when certain conditions are met.

One of the easiest ways to filter data is to evaluate a message property. You may recall adding message properties to your device messages in the previous exercise. The code that you added looked like the following:

```csharp
...
telemetryMessage.Properties.Add("sensorID", "VSTel");
...
loggingMessage.Properties.Add("sensorID", "VSLog");
```

You can now embed a SQL query within your message route that uses `sensorID` as a criteria for the route. In this case, when the value assigned to `sensorID` is `VSLog` (vibration sensor log), the message is intended for the storage archive.

In this exercise, you will create and test the logging route.

#### Task 1: Define the message routing endpoint

1. In the [Azure Portal](https://portal.azure.com/), ensure that your IoT hub blade is open.

1. On the left-hand menu, under **Hub settings**, click **Message routing**.

1. On the **Message routing** pane, ensure that the **Routes** tab is selected.

1. To add a new route, click **+ Add**.

    The **Add a route** blade should now be displayed.

1. On the **Add a route** blade, under **Name**, enter **vibrationLoggingRoute**

1. To the right of **Endpoint**, click **+ Add endpoint**, and then, in the drop-down list, click **Storage**.

    The **Add a storage endpoint** blade should now be displayed.

1. On the **Add a storage endpoint** blade, under **Endpoint name**, enter **vibrationLogEndpoint**

1. To display a list of Storage accounts associated with your subscription, click **Pick a container**.

    A list of the storage accounts already present in the Azure Subscription is listed. At this point you could select an existing storage account and container, however, for this lab you will create a new one.

1. To begin creating a storage account, click **+ Storage account**.

    The **Create storage account** blade should now be displayed.

1. On the **Create storage account** blade, under **Name**, enter **vibrationstore{your-id}**

    For example: **vibrationstorecah191211**

    > **Note**:  This field can only contain lower-case letters and numbers, must be between 3 and 24 characters, and must be unique.

1. In the **Account kind** dropdown, click **StorageV2 (general purpose v2)**.

1. Under **Performance**, ensure that **Standard** is selected.

    This keeps costs down at the expense of overall performance.

1. Under **Replication**, ensure that **Locally-redundant storage (LRS)** is selected.

    This keeps costs down at the expense of risk mitigation for disaster recovery. In production your solution may require a more robust replication strategy.

1. Under **Location**, select **@lab.CloudResourceGroup(ResourceGroup1).Location**.

1. Under **Minimum TLS version**, ensure **Version 1.2** is selected.

1. To create the storage account endpoint, click **OK**.

1. Wait until the request is validated and the storage account deployment has completed.

    Validation and creation can take a minute or two.

    Once completed, the **Create storage account** blade will close and the **Storage accounts** blade will be displayed. The Storage accounts blade should have auto-updated to show the storage account that was just created.

#### Task 2: Define the storage account container

1. On the **Storage accounts** blade, click **vibrationstore{your-id}**.

    The **Containers** blade should appear. Since this is a new storage account, there are no containers listed.

1. To create a container, click **+ Container**.

    The **New container** dialog should now be displayed.

1. On the **New container** dialog, under **Name**, enter **vibrationcontainer**

   Again, only lower-case letters and numbers are accepted.

1. Under **Public access level**, ensure that **Private (no anonymous access)** is selected.

1. To create the container, click **Create**.

    After a moment the **Lease state** for your container will update to display **Available**.

1. To choose this container for your solution, click **vibrationcontainer**, and then click **Select**.

    You should be returned to the **Add a storage endpoint** blade. Notice that the **Azure Storage container** has been set to the URL for the storage account and container you just created.

1. Leave the **Batch frequency** and **Chunk size window** fields set to their default values of **100**.

1. Under **Encoding**, notice that there are two options and that **AVRO** is selected.

    > **Note**:  By default IoT Hub writes the content in Avro format, which has both a message body property and a message property. The Avro format is not used for any other endpoints. Although the Avro format is great for data and message preservation, it's a challenge to use it to query data. In comparison, JSON or CSV format is much easier for querying data. IoT Hub now supports writing data to Blob storage in JSON as well as AVRO.

1. Take a moment to examine the value specified in **File name format** field.

    The **File name format** field specifies the pattern used to write the data to files in storage. The various tokens are replace with values as the file is created.

1. Under **Authentication type**, ensure **Key-based** is selected.


1. At the bottom of the blade, to create your storage endpoint, click **Create**.

    Validation and subsequent creation will take a few moments. Once complete, you should be located back on the **Add a route** blade.

    Notice that the **Endpoint** is now populated.

#### Task 3: Define the routing query

1. On the **Add a route** blade, under **Data source**, ensure that **Device Telemetry Messages** is selected.

1. Under **Enable route**, ensure that **Enable** is selected.

1. Under **Routing query**, replace **true** with the query below:

    ```sql
    sensorID = 'VSLog'
    ```

    This query ensures that only messages with the `sensorID` application property set to `VSLog` will be routed to the storage endpoint.

1. To save this route, click **Save**.

    Wait for the success message. Once completed, the route should be listed on the **Message routing** pane.

1. Navigate back to your Azure portal Dashboard.

#### Task 4: Verify Data Archival

1. Ensure that the device app you created in Visual Studio Code is still running.

    If not, run it in the Visual Studio Code terminal using `dotnet run`.

1. On your Resources tile, to open you Storage account blade, click **vibrationstore{your-id}**.

    If your Resources tile does not list your Storage account, click the **Refresh** button at the top of the resource group tile, and then follow the instruction above to open your storage account.

1. On the left-side menu of your **vibrationstore{your-id}** blade, click **Storage Explorer (preview)**.

    You can use the Storage Explorer to verify that your data is being added to the storage account.

    > **Note**: The Storage Explorer is currently in preview mode, so its exact mode of operation may change.

1. In **Storage Explorer (preview)** pane, expand **BLOB CONTAINERS**, and then click **vibrationcontainer**.

    To view the data, you will need to navigate down a hierarchy of folders. The first folder will be named for the IoT Hub.

    > **Note**: If no data is displayed, wait a few moments and try again.

1. In the right-hand pane, under **NAME**, double-click **iot-az220-training-{your-id}**, and then use double-clicks to navigate down into the hierarchy.

    Under your IoT hub folder, you will see folders for the Partition, then numeric values for the Year, Month, and Day. The final folder represents the Hour, listed in UTC time. The Hour folder will contain a number of Block Blobs that contain your logging message data.

1. Double-click the Block Blob for the data with the earliest time stamp.

1. Click the **Click here to begin download** link.

    A file named **{day_num}.avro** (i.e. **22.avro**) will be downloaded to the **Downloads** folder.

1. Open the downloaded file with **Visual Studio Code** and click **Do you want to open it anyway**.

    Although the data is not formatted in a way that is easy to read, you should be able to recognize it as your vibration messages.

1. Close the **Visual Studio Code** document containing your data.

1. Return to your Azure portal Dashboard.

### Exercise 4: Logging Route Azure Stream Analytics Job

In this exercise, you will create a Stream Analytics job that outputs logging messages to Blob storage. You will then use Storage Explorer in the Azure Portal to view the stored data.

This will enable you to verify that your route includes the following settings:

* **Name** - vibrationLoggingRoute
* **Data Source** - DeviceMessages
* **Routing query** - sensorID = 'VSLog'
* **Endpoint** - vibrationLogEndpoint
* **Enabled** - true

> **Note**: It may seem odd that in this lab you are routing data to storage, and then also sending your data to storage through Azure Stream Analytics. In a production scenario, you wouldn't have both paths long-term. Instead, it is likely that the second path that we're creating here would not exist. You will use it here, in a lab environment, as a way to validate that your routing is working as expected and to show a simple implementation of Azure Stream Analytics.

#### Task 1: Create the Stream Analytics Job

1. On the Azure portal menu, click **+ Create a resource**.

1. On the **New** blade, in the **Search the Marketplace** textbox, type **stream analytics** and then click **Stream Analytics job**.

1. On the **Stream Analytics job** blade, click **Create**.

    The **New Stream Analytics job** pane is displayed.

1. On the **New Stream Analytics job** pane, under **Name**, enter **vibrationJob**.

1. Under **Subscription**, choose the subscription you are using for the lab.

1. Under **Resource group**, select **@lab.CloudResourceGroup(ResourceGroup1).Name**.

1. Under **Location**, select **@lab.CloudResourceGroup(ResourceGroup1).Location**.

1. Under **Hosting environment**, ensure that **Cloud** is selected.

    Edge hosting will be discussed later in the course.

1. Under **Streaming units**, reduce the number from **3** to **1**.

    This lab does not require 3 units and this will reduce costs.

1. To create the Stream Analytics job, click **Create**.

1. Wait for the **Deployment succeeded** message, then open the new resource.

    > **Tip:** If you miss the message to go to the new resource, or need to find a resource at any time, select **Home/All resources**. Enter enough of the resource name for it to appear in the list of resources.

1. Take a moment to examine your new Stream Analytics job.

    Notice that you have an empty job, showing no inputs or outputs, and a skeleton query. The next step is to populate these entries.

#### Task 2: Create the Stream Analytics Job Input

1. On the left-side menu under **Job topology**, click **Inputs**.

    The **Inputs** pane will be displayed.

1. On the **Inputs** pane, click **+ Add stream input**, and then click **IoT Hub**.

    The **IoT Hub - New input** pane will be displayed.

1. On the **IoT Hub - New input** pane, under **Input alias**, enter `vibrationInput`.


1. Under **IoT Hub**, ensure that your **iot-az220-training-{your-id}** IoT hub is selected.

1. Ensure that **Select IoT Hub from your subscriptions** is selected.

1. Under **Subscription**, ensure that the subscription you used to create the IoT Hub earlier is selected.

1. Under **IoT Hub**, ensure that your **iot-az220-training-{your-id}** IoT hub is selected.

1. Under **Consumer group**, ensure that **$Default** is selected.

1. Under **Shared access policy name**, ensure that **iothubowner** is selected.

    > **Note**:  The **Shared access policy key** is populated and read-only.

1. Under **Endpoint**, ensure that **Messaging** is selected.

1. Leave **Partition key** blank.

1. Under **Event serialization format**, ensure that **JSON** is selected.

1. Under **Encoding**, ensure that **UTF-8** is selected.

    You may need to scroll down to see some of the fields.

1. Under **Event compression type**, ensure **None** is selected.

1. To save the new input, click **Save**, and then wait for the input to be created.

    The **Inputs** list should be updated to show the new input.

#### Task 3: Create the Stream Analytics Job Output

1. To create an output, on the left-side menu under **Job topology**, click **Outputs**.

    The **Outputs** pane is displayed.

1. On the **Outputs** pane, click **+ Add**, and then click **Blob storage/ADLS Gen2**.

    The **Blob storage/ADLS Gen2 - New output** pane is displayed.

1. On the **Blob storage/ADLS Gen2 - New output** pane, under **Output alias**, enter `vibrationOutput`.

1. Ensure that **Select storage from your subscriptions** is selected.

1. Under **Subscription**, select the subscription you are using for this lab.

1. Under **Storage account**, click **vibrationstore{your-id}**.

    > **Note**:  The **Storage account key** is automatically populated and read-only.

1. Under **Container**, ensure that **Use existing** is selected and that **vibrationcontainer** is selected from the dropdown list.

1. Under **Authentication Mode**, select **Connection string**

    > **Note** that the **Storage account key** is displayed.

1. Leave the **Path pattern** blank.

1. Leave the **Date format** and **Time format** at their defaults.

1. Under **Event serialization format**, ensure that **JSON** is selected.

1. Under **Encoding**, ensure that **UTF-8** is selected.

1. Under **Format**, ensure that **Line separated** is selected.

    > **Note**:  This setting stores each record as a JSON object on each line and, taken as a whole, results in a file that is an invalid JSON record. The other option, **Array**, ensures that the entire document is formatted as a JSON array where each record is an item in the array. This allows the entire file to be parsed as valid JSON.

1. Leave **Minimum rows** blank.

1. Under **Maximum time**, leave **Hours** and **Minutes** blank.

1. To create the output, click **Save**, and then wait for the output to be created.

    The **Outputs** list will be updated with the new output.

#### Task 4: Create the Stream Analytics Job Query

1. To edit the query, on the left-side menu under **Job topology**, click **Query**.

1. In the query editor pane, replace the existing query with the query below:

    ```sql
    SELECT
        *
    INTO
        vibrationOutput
    FROM
        vibrationInput
    ```

1. Directly above the query editor pane, click **Save Query**.

1. On the left-side menu, click **Overview**.

#### Task 5: Test the Logging Route

Now for the fun part. Does the telemetry your device app is pumping out work its way along the route, and into the storage container?

1. Ensure that the device app you created in Visual Studio Code is still running.

    If not, run it in the Visual Studio Code terminal using `dotnet run`.

1. On the **Overview** pane of your Stream Analytics job, click **Start**.

1. In the **Start job** pane, leave the **Job output start time** set to **Now**, and then click **Start**.

    It can take a few moments for the job to start.

1. On the Azure portal menu, click **Dashboard**.

1. On your Resources tile, click **vibrationstore{your-id}**.

    If your Storage account is not visible, use the **Refresh** button at the top of the resource group tile.

1. On the **Overview** pane of your Storage account, select the **Monitoring** section.

1. Under **Key metrics**, adjacent to **Show data for last**, change the time range to **1 hour**.

    You should see activity in the charts.

1. On the left-side menu, click **Storage Explorer (preview)**.

    You can use Storage Explorer for additional reassurance that all of your data is getting to the storage account.

    > **Note**:  The Storage Explorer is currently in preview mode, so its exact mode of operation may change.

1. In **Storage Explorer (preview)**, under **BLOB CONTAINERS**, click **vibrationcontainer**.

1. To view the data, select the json file and click **Download**, then click **Click here to download**.

1. Open the downloaded file with **Visual Studio Code**, and review the JSON data.

    ```json
    {"vibration":-0.025974767991863323,"EventProcessedUtcTime":"2021-10-22T22:03:10.8624609Z","PartitionId":3,"EventEnqueuedUtcTime":"2021-10-22T22:02:09.1180000Z","IoTHub":{"MessageId":null,"CorrelationId":null,"ConnectionDeviceId":"sensor-v-3000","ConnectionDeviceGenerationId":"637705296662649188","EnqueuedTime":"2021-10-22T22:02:08.7900000Z"}}
    {"vibration":-2.6574811793183173,"EventProcessedUtcTime":"2021-10-22T22:03:10.9718423Z","PartitionId":3,"EventEnqueuedUtcTime":"2021-10-22T22:02:11.1030000Z","IoTHub":{"MessageId":null,"CorrelationId":null,"ConnectionDeviceId":"sensor-v-3000","ConnectionDeviceGenerationId":"637705296662649188","EnqueuedTime":"2021-10-22T22:02:11.0720000Z"}}
    {"vibration":3.9654399589335796,"EventProcessedUtcTime":"2021-10-22T22:03:10.9718423Z","PartitionId":3,"EventEnqueuedUtcTime":"2021-10-22T22:02:13.3060000Z","IoTHub":{"MessageId":null,"CorrelationId":null,"ConnectionDeviceId":"sensor-v-3000","ConnectionDeviceGenerationId":"637705296662649188","EnqueuedTime":"2021-10-22T22:02:13.1500000Z"}}
    {"vibration":0.99447803871677132,"EventProcessedUtcTime":"2021-10-22T22:03:10.9718423Z","PartitionId":3,"EventEnqueuedUtcTime":"2021-10-22T22:02:15.2910000Z","IoTHub":{"MessageId":null,"CorrelationId":null,"ConnectionDeviceId":"sensor-v-3000","ConnectionDeviceGenerationId":"637705296662649188","EnqueuedTime":"2021-10-22T22:02:15.2120000Z"}}
    ```

1. Close the **Visual Studio Code** document containing your data.

1. Return to your Azure portal Dashboard.

1. Navigate back to your Dashboard.

1. On your Resources tile, click **vibrationJob**.

1. On the **vibrationJob** blade, click **Stop**, and then click **Yes**.

    You've traced the activity from the device app, to the hub, down the route, and to the storage container. Great progress! You will continue this scenario stream analytics in the next module when you take a quick look at data visualization.

1. Switch to the Visual Studio Code window.

1. At the Terminal command prompt, to exit the device simulator app, press **CTRL-C**.

> **IMPORTANT**: Do not remove these resources until you have completed the Data Visualization module of this course.
