---
lab:
    title: 'Lab 09: Integrate IoT Hub with Event Grid'
    module: 'Module 5: Insights and Business Integration'
---

# Integrate IoT Hub with Event Grid

## Lab Scenario

Contoso management is impressed with the prototype solutions that you've created using Azure IoT services, and they feel comfortable assigning additional budget the capabilities that you have already demonstrated. They are now asking that you explore the integration of certain operational support capabilities. Specifically, they would like to see how the Azure tools support sending alert notifications to the managers who are responsible for specific work areas. Alert criteria will be defined by the business area managers. The telemetry data arriving at IoT hub will be evaluated to generate the notifications.

You've identified a business manager, Nancy, that you've had success working with in the past. You'll work with her during the initial phase of your solution.

Nancy informs you that her team of facility technicians is responsible for installing the new connected thermostats that will be used to monitor temperature across different cheese caves. The thermostat devices function as IoT devices that can be connected to IoT hub. To get your project started, you agree to create an alert that will generate a notification when a new device has been implemented.

To generate an alert, you will push a device created event type to Event Grid when a new thermostat device is created in IoT Hub. You will create a Logic Apps instance that reacts to this event (on Event Grid) and which will send an email to alert facilities when a new device has been created, specifying the device ID and connection state.

The following resources will be created:

![Lab 9 Architecture](media/LAB_AK_09-architecture.png)

## In This Lab

In this lab, you will complete the following activities:

* Verify that the lab prerequisites are met (that you have the required Azure resources)
* Create a Logic App that sends an email
* Configure an Azure IoT Hub Event Subscription
* Create new devices to trigger the Logic App

### Exercise 1: Verify Lab Prerequisites

This lab assumes the following Azure resources are available:

| Resource Type  | Resource Name          |
|----------------|------------------------|
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name              |
| IoT Hub        | iot-az220-training-{your-id} |

To ensure these resources are available, complete the following tasks.

1. Select **Deploy to Azure**:

    [https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab09.json](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab09.json)

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

The resources have now been created.

### Exercise 2: Create HTTP Web Hook Logic App that sends an email

Azure Logic Apps is a cloud service that helps you schedule, automate, and orchestrate tasks, business processes, and workflows when you need to integrate apps, data, systems, and services across enterprises or organizations.

In this exercise, you will create a new Azure Logic App that will be triggered via an HTTP Web Hook, then send an email using an Outlook.com email address.

#### Task 1: Create a Logic App resource in the Azure portal

1. If necessary, log in to the [Azure portal](https://portal.azure.com) using the Azure account credentials that you are using for this course.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On the Azure portal menu, click **+ Create a resource**.

1. On the **Create a resource** blade, in the **Search services and marketplace** box, enter **logic app**

1. In the search results, click **Logic App**.

1. On the **Logic App** blade, click **Create**.

1. On the **Basics** tab, under **Project details**, select the **Subscription** that you are using for this course.

1. In the **Resource group** dropdown, under **Select existing**, click **@lab.CloudResourceGroup(ResourceGroup1).Name**.

1. Under **Instance details**, under **Type** select **Consumption**.

1. In the **Name** field, enter **logic-az220-training-{your-id}**

    For example: **logic-az220-training-cp191218**

    The name of your Azure Logic App must be globally unique because it is a publicly accessible resource that you must be able to access from any IP connected device.

1. In the **Region** dropdown, select the same Azure region that was used for the resource group.

1. Leave **Enable Log Analytics** set to **No**.

1. Click **Review + create**.

1. On the **Review + create** tab, click **Create**.

    > **Note**:  It will take a minute or two for the Logic App deployment to complete.

1. Navigate back to your Azure portal Dashboard.

#### Task 2: Configure Your Logic App

1. On your resource group tile, click the link to the Logic App resource that was just deployed.

    If the **logic-az220-training-{your-id}** Logic app is not displayed, refresh the resource group tile.

    > **Note**: When navigating to the **Logic App** for the first time, the **Logic Apps Designer** pane will be displayed. If this page doesn't come up automatically, click **Logic app designer** under the **Development Tools** section on the **Logic App** blade.

1. Under the **Start with a common trigger** section, click **When a HTTP request is received**.

    Starting out with one of the commonly used triggers is a convenient way to get started on your Logic App.

1. Notice that the visual designer opens with the **When a HTTP request is received** trigger selected. Click the trigger to open the details.

1. On the **When a HTTP request is received** trigger, under the **Request Body JSON Schema** textbox, click the **Use sample payload to generate schema** link.

    > **Note**: In the next step you will be adding the **DeviceCreated** sample event schema to the Request Body JSON Schema textbox. This sample, along with a couple of other event schema samples and some associated documentation, can be found at the following link for those who want to learn more: [Azure Event Grid event schema for IoT Hub](https://docs.microsoft.com/en-us/azure/event-grid/event-schema-iot-hub).

1. Use a copy-and-paste operation to add the following sample JSON to the **Enter or paste a sample JSON payload.** textbox, and then click **Done**.

    ```json
    [{
      "id": "56afc886-767b-d359-d59e-0da7877166b2",
      "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
      "subject": "devices/LogicAppTestDevice",
      "eventType": "Microsoft.Devices.DeviceCreated",
      "eventTime": "2018-01-02T19:17:44.4383997Z",
      "data": {
        "twin": {
          "deviceId": "LogicAppTestDevice",
          "etag": "AAAAAAAAAAE=",
          "deviceEtag": "null",
          "status": "enabled",
          "statusUpdateTime": "0001-01-01T00:00:00",
          "connectionState": "Disconnected",
          "lastActivityTime": "0001-01-01T00:00:00",
          "cloudToDeviceMessageCount": 0,
          "authenticationType": "sas",
          "x509Thumbprint": {
            "primaryThumbprint": null,
            "secondaryThumbprint": null
          },
          "version": 2,
          "properties": {
            "desired": {
              "$metadata": {
                "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
              },
              "$version": 1
            },
            "reported": {
              "$metadata": {
                "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
              },
              "$version": 1
            }
          }
        },
        "hubName": "egtesthub1",
        "deviceId": "LogicAppTestDevice"
      },
      "dataVersion": "1",
      "metadataVersion": "1"
    }]
    ```

    This sample JSON is an example of the JSON that Event Grid will POST to the Web Hook endpoint for the Logic App once it's created. This sample includes the IoT Hub Message Telemetry properties for the IoT Device that will be sending telemetry messages.

    > **NOTE**: The **Enter or paste a sample JSON payload** field is a rich editor that automatically inserts opening and closing braces, etc. In the LODS environment, if the "type text" option is used to copy the JSON above directly into the **Enter or paste a sample JSON payload** field, extra braces will be added and the content will be invalid. Instead, open **Notepad** within the LODS VM first, and then send the text to **Notepad**. From there, you can copy the text into the field without error.

1. Notice that the **Request Body JSON Schema** textbox is now populated with a JSON schema that was automatically generated based on the sample JSON that you provided.

1. Below the **When a HTTP request is received** trigger, click **+** and select **Add an action**.

1. Click **Choose an operation** and in the search textbox, enter **Outlook.com**

1. In the list of Actions, scroll down to the Send options, and then click **Send an email (V2)**.

    > **Note**:  These instructions walk through configuring the Logic App to send an email using an **Outlook.com** email address. Alternatively, the Logic App can also be configured to send email using the Office 365 Outlook or Gmail connectors as well.

1. On the **Outlook.com** connector, click **Sign in**, and then follow the prompts to authenticate with an existing Outlook.com account.

1. If prompted to **Let this app access your info**, click **Yes**.

1. On the **Send an email (V2)** action, in the **To** field, enter an email address to send email messages to.

    For the purposes of this lab, you will provide an email address where you can receive the email notification. You can enter the Outlook.com account used for this connector, or another email account that is easy for you to access.

    The Outlook.com account that was authenticated in the step above will be used to send the emails.

1. In the **Subject** field, enter **IoT Hub alert:**

1. In the **Body** field, enter the following message content:

    ```text
    This is an automated email to inform you that:

    {eventType} occurred at {eventTime}

    IoT Hub: {hubName}
    Device ID: {deviceID}
    Connection state: {connectionState}
    ```

1. Take a moment to examine the message body that you just entered.

    You may have realized that the curly-braces entries are intended to represent dynamic content. You will need to replace these placeholder entries with the actual Dynamic content values.

    > **Note**: If the Add dynamic content tools are not displayed to the right of the connector, click the **Add dynamic content** hyperlink located directly under the **Body** text box. If it doesn't list the fields you need, click **See more** in the Dynamic content pane to include the fields included in the message body that you entered.

    In the next step you will replace each placeholder value with the corresponding Dynamic content value.

1. For each dynamic content placeholder, delete the entry and then replace it with the corresponding Dynamic content field.

    When you add the first dynamic content value, because the input data schema is for an array, the Logic Apps Designer will automatically change the e-mail action to be nested inside of a **For each** action. When this happens, the **Send an email (V2)** action will collapse. To reopen your email message, click **Send an email (V2)**, and then continue editing the message body.

    When you have completed this step you should see a message body that is similar to the following:

    ![Fill out email information](./media/LAB_AK_09-email_content.png)

1. At the top of the designer, to save all changes to the Logic App Workflow, click **Save** .

1. To expand the _When a HTTP request is received_ trigger, click **When a HTTP request is received**.

1. Copy the value for the **HTTP POST URL** that is displayed.

    The **HTTP POST URL** will look similar to the following:

    ```text
    https://prod-87.eastus.logic.azure.com:443/workflows/b16b5556cbc54c97b063479ed55b2669/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=ZGqYl-R5JKTugLG3GR5Ir1FuM0zIpCrMw4Q2WycJRiM
    ```

    This URL is the Web Hook endpoint that is used to call the Logic App trigger via HTTPS. Notice the **sig** query string parameter and it's value. The **sig** parameter contains the shared access key that is used to authenticate requests to the Web Hook endpoint.

1. Save the URL for future reference.

### Exercise 3: Configure Azure IoT Hub Event Subscription

Azure IoT Hub integrates with Azure Event Grid so that you can send event notifications to other services and trigger downstream processes. You can configure business applications to listen for IoT Hub events so that you can react to critical events in a reliable, scalable, and secure manner. For example, build an application that updates a database, creates a work ticket, and delivers an email notification every time a new IoT device is registered to your IoT hub.

In this exercise, you will create an Event Subscription within Azure IoT Hub to setup Event Grid integration that will trigger a Logic App to send an alert email.

1. Navigate back to your Azure portal dashboard.

1. On your resource group tile, to navigate to your IoT Hub, click **iot-az220-training-{your-id}**.

1. On the **IoT Hub** blade, on the left side navigation menu, click **Events**.

1. On the **Events** pane, at the top, click **+ Event Subscription**.

1. On the Create Event Subscription blade, in the **Name** field, enter **MyDeviceCreateEvent**

1. Ensure that the **EventSchema** field is set to **Event Grid Schema**.

1. Under the **TOPIC DETAILS** section, in the **System Topic Name** field, enter **device-creation**.

    This section is informational and read-only

1. Under **EVENT TYPES**, open the **Filter to Event Types** dropdown, and then de-select all of the choices except **Device Created**.

1. Under **ENDPOINT DETAILS**, open the **Endpoint Type** dropdown, and then click **Web Hook**.

1. Under **ENDPOINT DETAILS**, click **Select an endpoint**.

1. In the **Select Web Hook** pane, under **Subscriber Endpoint**, paste the URL that you copied from your logic app, then click **Confirm Selection**.

    > **Important**: Do not click Create!

    You could save the event subscription here, and receive notifications for every device that is created in your IoT hub. However, for this lab we will use the optional fields to filter for specific devices.

1. At the top of the pane, click **Filters** .

    You will be using filters to filter for specific devices.

1. Under **ADVANCED FILTERS**, click **Add new filter**, and then fill in the fields with these values:

    * **Key**: Enter `Subject`

    * **Operator**: Select `String begins with`

    * **Value**:  Enter `devices/sensor-th`

    We will use this value to filter for device events associated with the Cheese Cave temperature and humidity sensors.

1. To save the event subscription, click **Create**.

### Exercise 4: Test Your Logic App with New Devices

Test your logic app by creating a new device to trigger an event notification email.

1. In your Azure portal, if needed, navigate to your IoT Hub blade.

1. On the left side navigation menu, under **Explorers**, click **IoT Devices**.

1. At the top of the IoT devices blade, click **+ New**.

1. In the **Device ID** field, enter **sensor-th-0050**

1. Leave all other fields at the defaults, and then click **Save**.

1. To test the event subscription filters, create additional devices using the following device IDs:

    * `sensor-v-3002`
    * `sensor-th-0030`
    * `sensor-v-3003`

    If you added the four examples total, your list of IoT devices should look like the following image:

    ![IoT Hub device list](./media/LAB_AK_09-iot-hub-device-list.png)

1. Once you've added a few devices to your IoT hub, check your email to see which ones triggered the logic app.
