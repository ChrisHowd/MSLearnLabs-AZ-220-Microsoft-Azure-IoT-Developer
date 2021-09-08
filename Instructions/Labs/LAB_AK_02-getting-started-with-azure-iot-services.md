---
lab:
    title: 'Lab 02: Getting Started with Azure IoT Services'
    module: 'Module 1: Introduction to IoT and Azure IoT Services'
---

# Getting Started with Azure IoT Services

## Lab Scenario

You are an Azure IoT Developer working for Contoso, a company that crafts and distributes gourmet cheeses.

You have been tasked with exploring Azure and the Azure IoT services that you will using to develop Contoso's IoT solution. You have already become familiar with the Azure portal and created a resource group for your project. Now you need to begin investigating the Azure IoT services.

## In This Lab

In this lab, you will create and examine an Azure IoT Hub and an IoT Hub Device Provisioning Service. The lab includes the following exercises:

* Explore Globally Unique Resource Naming Requirements
* Create an IoT Hub using the Azure portal
* Examine features of the IoT Hub service
* Create a Device Provisioning Service and link it to your IoT Hub
* Examine features of the Device Provisioning Service

## Lab Instructions

### Exercise 1: Explore Globally Unique Resource Naming Requirements

In labs 2-20 of this course, you will be creating Azure resources that are used to develop your IoT solution. To ensure consistency across the labs and to help in tidying up resources when you are finished with them, suggested resource names will be provided within the lab instructions. As much as possible, the suggested resource names will follow the naming guidelines recommended here: [Recommended naming and tagging conventions](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging). However, many of the resources that you will create during this course expose services that can be consumed across the web, which means that they must have globally unique names. To ensure that these resources satisfy the globally unique requirement, you will be adding a unique identifier to the end of the resource names when needed.

In this exercise, you will create your unique ID and review some examples that help to illustrate how you will use your unique ID during labs 2-20 of this course.

#### Task 1: Create Your Unique ID

1. Construct your unique ID by using your lower-case initials and the current date in the following pattern:

    ```text
    YourInitialsYYMMDD
    ```

    The first part of your unique ID will be your initials in lower-case. The second part will be the last two digits of the current year, the current numeric month, and the current numeric day. Here are some examples:

    ```text
    gwb200123
    bho200504
    cah201216
    dm200911
    ```

    Within the lab instructions, you will see `{your-id}` listed as part of the suggested resource name whenever you need to enter your unique ID. The `{your-id}` portion of the suggested resource name is a placeholder. You will replace the entire placeholder string (including the `{}`) with your unique value.

1. Make a note of your unique ID now and then **use the same value through the entire course**.

    > **Note**: Don't change the date portion of your unique ID each day. Use the same unique ID each day of the course.

#### Task 2: Review How and When to Apply Your Unique ID

Many of the resources that you create during the labs in this course will have publicly-addressable (although secured) endpoints and therefore must be globally unique. Examples of resources that require globally unique names include IoT Hubs, Device Provisioning Services, and Azure Storage Accounts.

As noted above, when you create these types of resources, you will be provided with a resource name that follows suggested guidelines and you will be instructed to include your unique ID as part of the resource name. To help clarify when you need to enter your unique ID, the suggested resource name will include a placeholder value for your unique ID. You will be instructed to replace the placeholder value, `{your-id}`, with your unique ID.

1. Review the following resource naming examples:

    If your Unique ID is: **cah191216**

    | Resource Type | Name Template | Example |
    | :--- | :--- | :--- |
    | IoT Hub | iot-az220-training-{your-id} | iot-az220-training-cah191216 |
    | Device Provisioning Service | dps-az220-training-{your-id} | dps-az220-training-cah191216 |
    | Azure Storage Account <br/>(name must be lower-case and no dashes) | az220storage{your-id} | az220storagecah191216 |

1. Review the following example for applying your unique ID within a Bash script:

    In some of the labs later in this course, you will be instructed to apply your unique ID value within a Bash script. The Bash script file, which is provided for you, might include code that is similar to the following:

    ```bash
    #!/bin/bash

    YourID="{your-id}"
    RGName="rg-az220"
    IoTHubName="iot-az220-training-$YourID"
    ```

    In the code above, if the value of your unique ID is `cah191216`, then the line containing `YourID="{your-id}"` should be updated to `YourID="cah191216"`.

    > **Note**: Notice that you do not change the `$YourID` value on the final code line. If it isn't `{your-id}` then don't replace it.

1. Review the following example for applying your unique ID within C# code:

    In some of the labs later in this course, you will be instructed to apply your unique ID value within C# source files. The C# source code, which will be provided to you, might include a code section that looks similar to the following:

    ```csharp
    private string _yourId = "{your-id}";
    private string _rgName = "rg-az220";
    private string _iotHubName = $"iot-az220-training-{_yourId}";
    ```

    In the code above, if the value of your unique ID is `cah191216`, then the line containing `private string _yourId = "{your-id}";` should be updated to `private string _yourId = "cah191216";`

    > **Note**: Notice that you do not change the `_yourId` value on the final code line. Once again, if it isn't `{your-id}` then don't replace it.

1. Notice that not all resource names require you to apply your unique ID.

    As you may have already considered, the Resource Group that you created in the previous lab did not include your unique ID value.

    Some resources, like the Resource Group, must have a unique name within your subscription, but the name does not need to be globally unique. Therefore, each student taking this course can use the resource group name: **rg-az220**. Of course this is only true if each student uses their own subscription, but that should be the case.

1. Apply an additional `01` or `02` if it turns out that your unique ID isn't so unique.

    It could happen that two or more people with the same initials start the course on the same day. You may not know, unless the person is sitting next to you, until you create your IoT Hub in the next exercise. Azure will let you know if the suggested resource name, including your unique ID, isn't globally unique. In that case you will be instructed to update your unique ID by appending an additional `##` value. For example, if the value of your unique ID is `cah191216`, your updated unique ID value could become:

    ```text
    cah20121600
    cah20121601
    cah20121602
    ...
    cah20121699
    ```

    If you do have to update your unique ID, try to use it consistently.

### Exercise 2: Create an IoT Hub using the Azure portal

The Azure IoT Hub is a fully managed service that enables reliable and secure bidirectional communications between IoT devices and Azure. The Azure IoT Hub service provides the following:

* Establish bidirectional communication with billions of IoT devices
* Multiple device-to-cloud and cloud-to-device communication options, including one-way messaging, file transfer, and request-reply methods.
* Built-in declarative message routing to other Azure services.
* A queryable store for device metadata and synchronized state information.
* Secure communications and access control using per-device security keys or X.509 certificates.
* Extensive monitoring for device connectivity and device identity management events.
* SDK device libraries for the most popular languages and platforms.

There are several methods that you can use to create an IoT Hub. For example, you can create an IoT Hub resource using the Azure portal, or you can create an IoT Hub (and other resources) programmatically. During this course you will be investigating various methods that can be used to create and manage Azure resources, including Azure CLI and Bash scripts.

In this exercise, you will use the Azure portal to create and configure your IoT Hub.

#### Task 1: Use the Azure portal to create a resource (IoT Hub)

1. Login to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.  You may find it easiest to use an InPrivate / Incognito browser session to avoid accidentally using the wrong account.

1. Notice that the AZ-220 dashboard that you created in the previous lab has been loaded.

    You will be adding resources to your dashboard as the course continues.

1. On the Azure portal menu, click **+ Create a resource**.

    The **New** blade that opens is a front-end to the Azure Marketplace, which is a collection of all the resources you can create in Azure. The marketplace contains resources from both Microsoft and the community.

1. In the Search textbox, type **IoT Hub** and then press **Enter**.

    The **Marketplace** blade will open to display the available services matching your search criteria.

    > **Note**: Marketplace services provided by private contributors may include a cost that is not covered by a Microsoft Azure Pass or other Microsoft Azure free credit offerings. You will be using Microsoft provided resources during the labs in this course.

1. On the **Marketplace** blade, click the **IoT Hub** search result.

    > **Note**:
    > A **Create** action is shown at the bottom of the **IoT Hub** search result - that will navigate directly to the IoT Hub creation view. In normal use you may chose to click this - for the purpose of the tutorial, click anywhere in the main body of the **IoT Hub** search result.

1. On the **IoT Hub** blade, click **Usage Information + Support**

    Under **Useful Links**, notice the list of resource links.

    There is no need to explore these links now, but it's worth noting that they are available. The _Documentation_ link, for example, takes you to the root page for IoT Hub resources and documentation. You can use this page to review the most up-to-date Azure IoT Hub documentation and explore additional resources that are outside the scope of this course. You will be referred to the docs.microsoft.com site throughout this course for additional reading on specific topics.

    If you opened one of the links, close it now and use your browser to navigate back to the Azure portal tab.

#### Task 2: Create an IoT Hub with required property settings

1. To begin the process of creating your new IoT Hub, click **Create**.

    > **Tip:** In the future, there are two other ways to get to the _Create_ experience of any Azure resource type:
    >
    >    1. If you have the service in your Favorites, you can click the service to navigate to the list of instances, then click the _+ Add_ button at the top.
    >    2. You can search for the service name in the _Search_ box at the top of the portal to get to the list of instances, then click the _+ Add_ button at the top.

    The following steps walk you through the settings required to create your IoT Hub, explaining each of the fields as you fill them in.

1. On the **IoT hub** blade, in the **Subscription** dropdown, ensure that the Azure subscription that you intend to use for this course is selected.

    The _Basics_ tab that is selected initially contains uninitialized fields that you are required to fill in, but there are settings on other tabs that you will need to be familiar with as well.

1. To the right of **Resource group**, open the dropdown, and then click **rg-az220**

    This is the resource group that you created in the previous lab. You will be grouping the resources that you create for this course together in the same resource group. It is best practice to group related resources in this way, and will help you to clean up your resources when you no longer need them.

1. To the right of **Region**, open the drop-down list and select the same region that you selected for your resource group.

    > **Note**: One of the upcoming labs will use Event Grid. To support this future lab, you need to select a Region that supports Event Grid. For the current list of regions that support Event Grid, see the following link: [Products available by region](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=event-grid&regions=all)

    As you saw previously, Azure is supported by a series of datacenters that are placed in regions all around the world. When you create something in Azure, you deploy it to one of these datacenter locations.

    > **Note**:  When picking a region to host your resources, keep in mind that picking a region close to your end users will decrease load/response times. In a production environment, if you are on the other side of the world from your end users, you should not be picking the region nearest you.

1. To the right of **IoT hub name**, enter a globally unique name for your IoT Hub as follows:

    To provide a globally unique name, enter **iot-az220-training-{your-id}** (remember to replace **{your-id}** with the unique ID you created in Exercise 1).

    For example: **iot-az220-training-cah191216**

    The name of your IoT Hub must be globally unique because it is a publicly accessible resource that you must be able to access from any of your IP enabled IoT devices.

    Consider the following when you specify a unique name for your new IoT Hub:

    * The value that you apply to **IoT hub name** must be unique across all of Azure. This is true because the value assigned to the name will be used in the IoT Hub's connection string. Since Azure enables you to connect devices from anywhere in the world to your hub, it makes sense that all Azure IoT hubs must be accessible from the Internet using the connection string and that connection strings must therefore be unique. You will explore connection strings later in this lab.

    * The value that you assign to **IoT hub name** cannot be changed once your resource has been created. If you do need to change the name, you'll need to create a new IoT Hub with the desired name, re-register your devices from the original hub and register them with the new one, and delete your old IoT Hub.

    * The **IoT hub name** field is a required field.

    > **Note**:  Azure will ensure that the name you enter is unique. If the name that you enter is not unique, Azure will display a message below the name field as a warning. If you see the warning message, you should update your unique ID. Try appending your unique ID with '**00**', or '**01**', or '**02**, 'etc. as necessary to achieve a globally unique name.

    > **Note**: Some resource names do not allow extended characters like the dash (-) or underscore (_), so stick with numeric digits when updating your unique ID.

1. At the top of the blade, click **Management**.

    Take a minute to review the fields and other information presented on this tab.

1. To the right of **Pricing and scale tier**, ensure that **S1: Standard tier** is selected.

    Azure IoT Hub provides several tier options depending on how many features you require and how many messages you need to send within your solution per day. The _S1_ tier that you are using in this course allows a total of 400,000 messages per unit per day and provides the all of the services that are required in this training. You won't actually need 400,000 messages per unit per day, but you will be using features provided by the Standard tier, such as _Cloud-to-device commands_, _Device management_, and _IoT Edge_. IoT Hub also offers a Free tier that is meant for testing and evaluation. It has the same capabilities as the Standard tier, but limited messaging allowances. It is important to note that you cannot upgrade from the Free tier to either Basic or Standard. The Free tier allows 500 devices to be connected to the IoT hub and up to 8,000 messages per day. Each Azure subscription can create one IoT Hub in the Free tier.

    > **Note**:  The _S1 - Standard_ tier has a cost of $25.00 USD per month per unit. You will be specifying 1 unit. For details about the other tier options, see [Choosing the right IoT Hub tier for your solution](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-scaling).

1. To the right of **Number of S1 IoT hub units**, ensure that **1** is selected.

    As mentioned above, the pricing tier that you choose establishes the number of messages that your hub can process per unit per day. To increase the number of messages that your hub can process without moving to a higher pricing tier, you can increase the number of units. For example, if you want your IoT hub to support ingress of up to 800,000 messages per day, you could specify *two* S1 tier units. For this course you will be using just 1 unit.

1. To the right of **Defender for IoT**, ensure that **Off** is selected.

    Azure Defender for IoT is a unified security solution for identifying IoT/OT devices, vulnerabilities, and threats. It enables you to secure your entire IoT/OT environment, whether you need to protect existing IoT/OT devices or build security into new IoT innovations.

    > **TIP**:
    **Azure Defender for IoT** was formerly known as **Azure Security Center** and you may still see places in Azure, and in this content, where the name has not yet been updated.

    Azure Defender for IoT is on by default because security is important to your IoT solution. You will be exploring Azure Defender for IoT in Lab 19 of this course. Disabling it for now ensures that the Lab 19 instructions work as expected.

    Currently, you can enable Azure Defender at the subscription level, through the Azure portal. Azure Defender is free for the first 30 days. Any usage beyond 30 days will be automatically  charged as per the pricing information detailed [here](https://azure.microsoft.com/en-us/pricing/details/azure-defender/).

1. Expand **Advanced Settings**, and then ensure that **Device-to-cloud partitions** is set to **4**.

    The number of partitions relates the device-to-cloud messages to the number of simultaneous readers of these messages. Most IoT hubs will only need four partitions, which is the default value. For this course you will create your IoT Hub using the default number of partitions.

1. Within the **Transport Layer Security (TLS)** section, ensure the **Minimum TLS Version** is set to **1.0**.

    IoT Hub uses Transport Layer Security (TLS) to secure connections from IoT devices and services. Three versions of the TLS protocol are currently supported, namely versions 1.0, 1.1, and 1.2.

    > [!Important]
    > The **Minimum TLS Version** property cannot be changed once your IoT Hub resource is created. It is therefore essential that you properly test and validate that all your IoT devices and services are compatible with TLS 1.2 and the recommended ciphers in advance. You can learn more about IoT Hub and TLS below:
    > * [Transport Layer Security (TLS) support in IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-tls-support)

1. At the top of the blade, click **Review + create**.

    Take a minute to review the settings that your provided.

1. At the bottom of the blade, to finalize the creation of your IoT Hub, click **Create**.

    Deployment can take a minute or more to complete. You can open the Azure portal Notification pane to monitor progress.

1. Notice that after a couple of minutes you receive a notification stating that your IoT Hub was successfully deployed to your **rg-az220** resource group.

1. On the Azure portal menu, click **Dashboard**, and then click **Refresh**.

    You should see that your resource group tile lists your new IoT Hub.

### Exercise 3: Examine the IoT Hub Service

As you have already learned, IoT Hub is a managed service, hosted in the cloud, that acts as a central message hub for bi-directional communication between your Azure IoT services and your connected devices.

IoT Hub's capabilities help you build scalable, full-featured IoT solutions such as managing industrial equipment used in manufacturing, tracking valuable assets in healthcare, monitoring office building usage, and many more scenarios. IoT Hub monitoring helps you maintain the health of your solution by tracking events such as device creation, device failures, and device connections.

In this exercise, you will examine some of the features that IoT Hub provides.

#### Task 1: Explore the IoT Hub Overview information

1. If necessary, log in to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. Verify that your AZ-220 dashboard is being displayed.

1. On the **rg-az220** resource group tile, click **iot-az220-training-{your-id}**

    When you first open your IoT Hub blade, the **Overview** information will be displayed. As you can see, the area at the top of this blade provides some essential information about your IoT Hub service, such as datacenter location and subscription. But this blade also includes tiles that provide information about how you are using your hub and recent activities. Let's take a look at these tiles before exploring further.

1. At the bottom-left of your IoT Hub blade, notice the **IoT Hub Usage** tile.

    > **Note**:  The tiles positions are based upon the width of the browser window, so the layout may be a little different than described.

    This tile provides a quick overview of what is connected to your hub and message count. As you add devices and start sending messages, this tile will provide nice "at-a-glance" information.

1. To the right of the **IoT Hub Usage** tile, notice the **Number of messages used** tile and the **Device to cloud messages** tile.

    The **Device to cloud messages** tile provides a quick view of the incoming messages from your devices over time. You will be registering a device and sending messages to your hub during a module in the next module, so you will begin to see information on these tiles pretty soon.

    The **Number of messages used** tile can help you to keep track of the total number of messages used.

#### Task 2: View features of IoT Hub using the left-side menu

1. On the IoT Hub blade, take a minute to scan the left-side menu options.

    As you would expect, these menu options are used to open panes that provide access to properties and features of your IoT Hub. For example, some panes provides access to devices that are connected to your hub.

1. On the left-side menu, under **Explorers**, click **IoT devices**

    This pane can be used to add, modify, and delete devices registered to your hub. You will get pretty familiar with this pane by the end of this course.

1. On the left-side menu, near the top, click **Activity log**

    As the name implies, this pane gives you access to a log that can be used to review activities and diagnose issues. You can also define queries that help with routine tasks. Very handy.

1. On the left-side menu, under **Settings**, click **Built-in endpoints**

    IoT Hub exposes "endpoints" that enable external connections. Essentially, an endpoint is anything connected to or communicating with your IoT Hub. You should see that your hub already has two endpoints defined:

    * _Events_
    * _Cloud to device messaging_

1. On the left-side menu, under **Messaging**, click **Message routing**

    The IoT Hub message routing feature enables you to route incoming device-to-cloud messages to service endpoints such as Azure Storage containers, Event Hubs, and Service Bus queues. You can also create routing rules to perform query-based routes.

1. At the top of the **Message routing** pane, click **Custom endpoints**.

    Custom endpoints (such as Service Bus queue and Storage) are often used within an IoT implementation.

1. Take a minute to scan through the menu options under **Settings**

    > **Note**:  This lab exercise is only intended to be an introduction to the IoT Hub service and get you more comfortable with the UI, so don't worry if you feel a bit overwhelmed at this point. You will be configuring and managing your IoT Hub, devices, and their communications as this course continues.

### Exercise 4: Create a Device Provisioning Service using the Azure portal

The Azure IoT Hub Device Provisioning Service is a helper service for IoT Hub that enables zero-touch, just-in-time provisioning to the right IoT hub without requiring human intervention. The Device Provisioning Service provides the following:

* Zero-touch provisioning to a single IoT solution without hardcoding IoT Hub connection information at the factory (initial setup)
* Load balancing devices across multiple hubs
* Connecting devices to their owner's IoT solution based on sales transaction data (multitenancy)
* Connecting devices to a particular IoT solution depending on use-case (solution isolation)
* Connecting a device to the IoT hub with the lowest latency (geo-sharding)
* Reprovisioning based on a change in the device
* Rolling the keys used by the device to connect to IoT Hub (when not using X.509 certificates to connect)

There are several methods that you can use to create an instance of the IoT Hub Device Provisioning Service. For example, you can use the Azure portal, which is what you will do in ths task. But you can also create a DPS instance using Azure CLI or an Azure Resource Manager Template.

#### Task 1: Use the Azure portal to create a resource (Device Provisioning Service)

1. If necessary, log in to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On the Azure portal menu, click **+ Create a resource**.

    As you saw previously, the **New** blade provides you with the capability to search the Azure Marketplace for services.

1. In the Search textbox, type **Device Provisioning Service** and then press Enter.

1. On the **Marketplace** blade, click **IoT Hub Device Provisioning Service** search result.

    > **Note**:
    > A **Create** action is shown at the bottom of the **IoT Hub Device Provisioning Service** search result - that will navigate directly to the IoT Hub Device Provisioning Service creation view. In normal use you may chose to click this - for the purpose of the tutorial, click anywhere in the main body of the **IoT Hub Device Provisioning Service** search result.

1. On the **IoT Hub Device Provisioning Service** blade, click **Usage Information + Support**.

    Under **Useful links**, notice the list of resource links.

    Again, there is no need to explore this documentation now, but is is good to know that it is available. The IoT Hub Device Provisioning Service Documentation page is the root page for DPS. You can use this page to explore current documentation and find tutorials and other resources that will help you to explore activities that are outside the scope of this course. You will be referred to the docs.microsoft.com site throughout this course for additional reading on specific topics.

    If you opened one of the links, close it now and use your browser to navigate back to the Azure portal tab.

#### Task 2: Create a Device Provisioning Service with required property settings

1. To begin the process of creating your new DPS instance, click **Create**.

    Next, you need to specify information about the Hub and your subscription. The following steps walk you through the settings, explaining each of the fields as you fill them in.

1. Under **Name**, enter a globally unique name for your IoT Hub Device Provisioning Service as follows:

    To provide a globally unique name, enter **dps-az220-training-{your-id}** (remember to replace **{your-id}** with the unique ID you created in Exercise 1).

    For example: **dps-az220-training-cah191216**

1. Under **Subscription**, ensure that the subscription you are using for this course is selected.

1. Under **Resource Group**, open the dropdown, and then click **rg-az220**

    You will be grouping the resources that you create for this course together in the same resource group. It's a best practice to group related resources in this way, and will help you to clean up your resources when you no longer need them.

1. Under **Location**, open the drop-down list and select the same region that you selected for your resource group.

    > **Note**: When picking a datacenter to host your resources, keep in mind that picking a datacenter close to your end users will decrease load/response times. If you are on the other side of the world from your end users, you should not be picking the datacenter nearest you.

1. At the bottom of the blade, click **Create**.

    Deployment can take a minute or more to complete. You can open the Azure portal Notification pane to monitor progress.

1. Notice that after a couple of minutes you receive a notification stating that your IoT Hub Device Provisioning Service instance was successfully deployed to your **rg-az220** resource group.

1. On the Azure portal menu, click **Dashboard**, and then click **Refresh**.

    You should see that your resource group tile lists your new IoT Hub Device Provisioning Service.

#### Task 3: Link your IoT Hub and Device Provisioning Service.

1. Notice that the AZ-220 dashboard lists both your IoT Hub and DPS resources.

    You should see both your IoT Hub and DPS resources listed - (you may need to hit **Refresh** if the resources were only recently created)

1. On the **rg-az220** resource group tile, click **dps-az220-training-{your-id}**.

1. On the **Device Provisioning Service** blade, under **Settings**, click **Linked IoT hubs**.

1. At the top of the blade, click **+ Add**.

    You will use the **Add link to IoT hub** blade to provide the information required to link your Device Provisioning service instance to an IoT hub.

1. On the **Add link to IoT hub** blade, ensure that the **Subscription** dropdown is displaying the subscription that you are using for this course.

    The subscription is used to provide a list of the available IoT hubs.

1. Open the IoT hub dropdown, and then click **iot-az220-training-{your-id}**.

    This is the IoT Hub that you created in the previous exercise.

1. In the Access Policy dropdown, click **iothubowner**.

    The _iothubowner_ credentials provide the permissions needed to establish the link with the specified IoT hub.

1. To complete the configuration, click **Save**.

    You should now see the selected hub listed on the Linked IoT hubs pane. You might need to click **Refresh** to show Linked IoT hubs.

1. On the Azure portal menu, click **Dashboard**.

### Exercise 5: Examine the Device Provisioning Service

The IoT Hub Device Provisioning Service is a helper service for IoT Hub that enables zero-touch, just-in-time provisioning to the right IoT hub without requiring human intervention, enabling customers to provision millions of devices in a secure and scalable manner.

#### Task 1: Explore the Device Provisioning Service Overview information

1. If necessary, log in to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. Verify that your AZ-220 dashboard is being displayed.

1. On the **rg-az220** resource group tile, click **dps-az220-training-{your-id}**

    When you first open your Device Provisioning Service instance, it will display the Overview information. As you can see, the area at the top of the blade provides some essential information about your DPS instance, such as status, datacenter location and subscription. This blade also provides the _Quick Links_ section, which provide access to:

    * [Azure IoT Hub Device Provisioning Service Documentation](https://docs.microsoft.com/en-us/azure/iot-dps/)
    * [Learn more about IoT Hub Device Provisioning Service](https://docs.microsoft.com/en-us/azure/iot-dps/about-iot-dps)
    * [Device Provisioning concepts](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service)
    * [Pricing and scale details](https://azure.microsoft.com/en-us/pricing/details/iot-hub/)

    When time permits, you can come back and explore these links.

#### Task 2: View features of Device Provisioning Service using the navigation menu

1. Take a minute to scan the left-side menu options.

    As you might expect, these options open panes that provide access to activity logs, properties and feature of the DPS instance.

1. On the left-side menu, near the top, click **Activity log**

    As the name implies, this pane gives you access to a log that can be used to review activities and diagnose issues. You can also define queries that help with routine tasks. Very handy.

1. On the left-side menu, under **Settings**, click **Quick Start**.

    This pane lists the steps to start using the Iot Hub Device Provisioning Service, links to documentation and shortcuts to other blades for configuring DPS.

1. On the left-side menu, under **Settings**, click **Shared access policies**.

    This pane provides management of access policies, lists the existing policies and the associated permissions.

1. On the left-side menu, under **Settings**, click **Linked IoT hubs**.

    Here you can see the linked IoT Hub from earlier. The Device Provisioning Service can only provision devices to IoT hubs that have been linked to it. Linking an IoT hub to an instance of the Device Provisioning service gives the service read/write permissions to the IoT hub's device registry; with the link, a Device Provisioning service can register a device ID and set the initial configuration in the device twin. Linked IoT hubs may be in any Azure region. You may link hubs in other subscriptions to your provisioning service.

1. On the left-side menu, under **Settings**, click **Certificates**.

    Here you can manage the X.509 certificates that can be used to secure your Azure IoT hub using the X.509 Certificate Authentication. You will investigate X.509 certificates in a later lab.

1. On the left-side menu, under **Settings**, click **Manage enrollments**.

    Here you can manage the enrollment groups and individual enrollments.

    Enrollment groups can be used for a large number of devices that share a desired initial configuration, or for devices all going to the same tenant. An enrollment group is a group of devices that share a specific attestation mechanism. Enrollment groups support both X.509 as well as symmetric. All devices in the X.509 enrollment group present X.509 certificates that have been signed by the same root or intermediate Certificate Authority (CA). Each device in the symmetric key enrollment group present SAS tokens derived from the group symmetric key. The enrollment group name and certificate name must be alphanumeric, lowercase, and may contain hyphens.

    An individual enrollment is an entry for a single device that may register. Individual enrollments may use either X.509 leaf certificates or SAS tokens (from a physical or virtual TPM) as attestation mechanisms. The registration ID in an individual enrollment is alphanumeric, lowercase, and may contain hyphens. Individual enrollments may have the desired IoT hub device ID specified.

1. Take a minute to review some of the other menu options under **Settings**

   > **Note**:  This lab exercise is only intended to be an introduction to the IoT Hub Device Provisioning Service and get you more comfortable with the UI, so don't worry if you feel a bit overwhelmed at this point. You will be covering DPS in much more detail as the course continues.
