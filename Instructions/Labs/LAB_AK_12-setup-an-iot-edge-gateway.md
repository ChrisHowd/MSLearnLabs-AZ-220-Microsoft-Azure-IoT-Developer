---
lab:
    title: 'Lab 12: Setup an IoT Edge Gateway'
    module: 'Module 6: Azure IoT Edge Deployment Process'
---

# Setup an IoT Edge Gateway

## Lab Scenario

This lab is theoretical and will walk you through how an IoT Edge device can be used as a gateway.

There are three patterns for using an IoT Edge device as a gateway: transparent, protocol translation, and identity translation:

**Transparent** – Devices that theoretically could connect to IoT Hub can connect to a gateway device instead. The downstream devices have their own IoT Hub identities and are using any of the MQTT, AMQP, or HTTP protocols. The gateway simply passes communications between the devices and IoT Hub. The devices are unaware that they are communicating with the cloud via a gateway, and a user interacting with the devices in IoT Hub is unaware of the intermediate gateway device. Thus, the gateway is transparent. Refer to Create a transparent gateway for specifics on using an IoT Edge device as a transparent gateway.

**Protocol translation** – Also known as an opaque gateway pattern, devices that do not support MQTT, AMQP, or HTTP can use a gateway device to send data to IoT Hub on their behalf. The gateway understands the protocol used by the downstream devices, and is the only device that has an identity in IoT Hub. All information looks like it is coming from one device, the gateway. Downstream devices must embed additional identifying information in their messages if cloud applications want to analyze the data on a per-device basis. Additionally, IoT Hub primitives like twins and methods are only available for the gateway device, not downstream devices.

**Identity translation** - Devices that cannot connect to IoT Hub can connect to a gateway device, instead. The gateway provides IoT Hub identity and protocol translation on behalf of the downstream devices. The gateway is smart enough to understand the protocol used by the downstream devices, provide them identity, and translate IoT Hub primitives. Downstream devices appear in IoT Hub as first-class devices with twins and methods. A user can interact with the devices in IoT Hub and is unaware of the intermediate gateway device.

The following resources will be created:

![Lab 12 Architecture](media/LAB_AK_12-architecture.png)

## In This Lab

In this lab, you will complete the following activities:

* Verify that the lab prerequisites are met (that you have the required Azure resources)
* Deploy an Azure IoT Edge Enabled Linux VM as an IoT Edge Device
* Generate and Configure IoT Edge Device CA Certificates
* Create an IoT Edge Device Identity in IoT Hub using Azure Portal
* Setup the IoT Edge Gateway Hostname
* Connect an IoT Edge Gateway Device to IoT Hub
* Open IoT Edge Gateway Device Ports for Communication
* Create the Downstream Device Identity in IoT Hub
* Connect a Downstream Device to IoT Edge Gateway
* Verify Event Flow

## Lab Instructions

### Exercise 1: Verify Lab Prerequisites

This lab assumes that the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name |
| IoT Hub | iot-az220-training-{your-id} |

To ensure these resources are available, complete the following tasks.

1. Select **Deploy to Azure**:

    [![Deploy To Azure](media/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2fARM%2fAllfiles%2FARM%2Flab12.json)

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

### Exercise 2: Deploy and configure a Linux VM  as an IoT Edge Gateway

In this exercise, you will deploy an Ubuntu Server VM and configure it as an IoT Edge Gateway.

#### Task 1: Create IoT Edge Gateway Device Identity

In this task, you will use Azure IoT Hub to create a new IoT Edge device identity that you will use for the IoT Edge Transparent Gateway (your IoT Edge VM).

1. If necessary, log in to your Azure portal using your Azure account credentials, and then navigate to your Azure Dashboard.

1. On the **@lab.CloudResourceGroup(ResourceGroup1).Name** resource group tile, to open your IoT hub, click **iot-az220-training-{your-id}**.

1. On the **iot-az220-training-{your-id}** blade, on the left-side menu under **Automatic Device Management**, click **IoT Edge**.

    The IoT Edge pane allows you to manage the IoT Edge devices connected to the IoT Hub.

1. At the top of the pane, click **Add an IoT Edge device**.

1. On the **Create a device** blade, in the **Device ID** field, enter **vm-az220-training-gw0001-{your-id}**.

    Be sure to replace {your-id} with the value that you created at the beginning of the course. This is the device identity will be used for authentication and access control.

1. Under **Authentication type**, ensure that **Symmetric key** is selected, and leave the **Auto-generate keys** box checked.

    This will have IoT Hub automatically generate the symmetric keys for authenticating the device.

1. Leave the other settings at the default values, and then click **Save**.

    After a moment, the new IoT Edge Device will added to the list of IoT Edge devices.

1. Under **Device ID**, click **vm-az220-training-gw0001-{your-id}**.

1. On the **vm-az220-training-gw0001-{your-id}** blade, copy the **Primary Connection String**.

    A copy button is provided to the right of the value.

1. Save the value of the **Primary Connection String** to a file, making a note about which device it is associated with.

1. On the **vm-az220-training-gw0001-{your-id}** blade, notice that the list of **Modules** is limited to **\$edgeAgent** and **\$edgeHub**.

    The IoT Edge Agent (**\$edgeAgent**) and IoT Edge Hub (**\$edgeHub**) modules are a part of the IoT Edge Runtime. The Edge Hub is responsible for communication, and the Edge Agent deploys and monitors the modules on the device.

1. At the top of the blade, click **Set Modules**.

    The **Set module on device** blade can be used to add additional modules to an IoT Edge Device. For now though, you will be using this blade to ensure the message routing is configured correctly for the IoT Edge Gateway device.

1. At the top of the **Set module on device** blade, click **Routes**.

    Under **Routes**, the editor displays a configured default route for the IoT Edge Device. At this time, it should be configured with a route that sends all messages from all modules to Azure IoT Hub. If the route configuration doesn't match this, then update it to match the following route:

    * **NAME**: `route`
    * **VALUE**: `FROM /* INTO $upstream`

    The `FROM /*` part of the message route will match all device-to-cloud messages or twin change notifications from any module or leaf device. Then, the `INTO $upstream` tells the route to send those messages to the Azure IoT Hub.

    > **Note**:  To learn more about configuring message routing within Azure IoT Edge, reference the [Learn how to deploy modules and establish routes in IoT Edge](https://docs.microsoft.com/azure/iot-edge/module-composition#declare-routes#declare-routes) documentation article.

1. At the bottom of the blade, click **Review + create**.

    This tab of the **Set module on device** blade displays the deployment manifest for your Edge device. You should see a message at the top of the blade that indicates "Validation passed"

1. Take a moment to review the deployment manifest.

1. At the bottom of the blade, click **Create**.

#### Task 2: Provision IoT Edge VM

In this task, you will use an ARM (Azure Resource Manager) Template to provision a Linux VM, install the IoT Edge runtime, configure the connection to IoT Hub, generate X509 certificates for encrypting device to gateway communication, and add them to the IoT Edge runtime configuration.

> **Information**: To learn more about the steps that have been automated, review the following resources:
>
> * [Install or uninstall Azure IoT Edge for Linux](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2020-11)
> * [Manage certificates on an IoT Edge device](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manage-device-certificates?view=iotedge-2020-11)
> * [Create demo certificates to test IoT Edge device features](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-create-test-certificates?view=iotedge-2020-11)

1. Select **Deploy to Azure**:

    [![Deploy To Azure](media/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2fARM%2fAllfiles%2FARM%2Flab12a.json)

1. If prompted, login to the **Azure Portal**.

    The **Custom deployment** page will be displayed.

1. Under **Project details**, in the **Subscription** dropdown, ensure that the Azure subscription that you intend to use for this course is selected.

1. In the **Resource group** dropdown, select  create and enter **@lab.CloudResourceGroup(ResourceGroup1).Namevm**.

1. In the **Region** field, enter the same location you have used earlier.

1. In the **Virtual Machine Name** textbox, enter **vm-az220-training-gw0001-{your-id}**

1. In the **Device Connection String** field, enter the connection string value from the previous exercise.

1. In the **Virtual Machine Size** field, ensure **Standard_DS1_v2** is entered.

1. In the **Ubuntu OS Version** field, ensure that **18.04-LTS** is entered.

1. In the **Admin Username** field, enter a username.

1. In the **Authentication Type** field, ensure **Password** is selected.

1. In the **Admin Password Or Key** field, enter the password you wish to use.

1. In the **Allow SSH** field, ensure **true** is selected.

1. To validate the template, click **Review and create**.

1. If validation passes, click **Create**.

    > **Note**:  Although the deployment may complete quickly, the configuration of the VM continues in the background.

1. Once the template has completed, navigate to the **Outputs** pane and make a note of the following:

    * Public FQDN
    * Public SSH

####
 Task 3: Open IoT Edge Gateway Device Ports for Communication

Standard IoT Edge devices don't need any inbound connectivity to function, because all communication with IoT Hub is done through outbound connections. Gateway devices are different because they need to receive messages from their downstream devices. If a firewall is between the downstream devices and the gateway device, then communication needs to be possible through the firewall as well. For the Azure IoT Edge Gateway to function, at least one of the IoT Edge hub's supported protocols must be open for inbound traffic from downstream devices. The supported protocols are MQTT, AMQP, and HTTPS.

The IoT communication protocols supported by Azure IoT Edge have the following port mappings:

| Protocol | Port Number |
| --- | --- |
| MQTT | 8883 |
| AMQP | 5671 |
| HTTPS<br/>MQTT + WS (Websocket)<br/>AMQP + WS (Websocket) | 443 |

The IoT communication protocol chosen for your devices will need to have the corresponding port opened for the firewall that secures the IoT Edge Gateway device. In the case of this lab, an Azure Network Security Group (NSG) is used to secure the IoT Edge Gateway, so Inbound security rules for the NSG will be opened on these ports.

In a production scenario, you will want to open only the minimum number of ports for your devices to communicate. If you are using MQTT, then only open port 8883 for inbound communications. Opening additional ports will introduce addition security attack vectors that attackers could take exploit. It is a security best practice to only open the minimum number of ports necessary for your solution.

In this task, you will configure the Network Security Group (NSG) that secures access to the Azure IoT Edge Gateway from the Internet. The necessary ports for MQTT, AMQP, and HTTPS communications need to be opened so the downstream IoT device(s) can communicate with the gateway.

1. If necessary, log in to your Azure portal using your Azure account credentials.

1. On your Azure dashboard, locate the **@lab.CloudResourceGroup(ResourceGroup1).Namevm** resource group tile.

    Notice that your resource group tile includes a link to the associated Network security group.

1. On the **@lab.CloudResourceGroup(ResourceGroup1).Namevm** resource group tile, click **nsg-vm-az220-training-gw0001-{your-id}**.

1. On the **Network security group** blade, on the left-side menu under **Settings**, click **Inbound security rules**.

1. At the top of the **Inbound security rules** pane, click **Add**.

1. On the **Add inbound security rule** pane, ensure **Source** is set to **Any**.

    This allows traffic from any source - in production you may wish to limit this to specific addresses, etc.

1. Under **Destination**, ensure **Destination** is set to **Any**.

    This ensures outgoing traffic can be routed to any location. In production you may wish to limit the addresses.

1. Under **Destination port ranges**, change the value to **8883**.

    This is the port for the MQTT protocol.

1. Under **Protocol**, click **TCP**.

    MQTT uses TCP.

1. Under **Action**, ensure **Allow** is selected.

    As this rule is intended to allow outgoing traffic, **Allow** is selected.

1. Under **Priority**, a default value is supplied - in most cases this will be **1010** - it **must** be unique.

    Rules are processed in priority order; the lower the number, the higher the priority. We recommend leaving gaps between rules - 100, 200, 300, etc. - so that it's easier to add new rules without having to edit existing rules.

1. Under **Name**, change the value to **MQTT**

1. Leave all other settings at the default, and then click **Add**.

    This will define an inbound security rule that will allow communication for the MQTT protocol to the IoT Edge Gateway.

1. After the MQTT rule is added, to open ports for the **AMQP** and **HTTPS** communication protocols, add two more rules with the following values:

    | Destination port ranges | Protocol | Name |
    | :--- | :--- | :--- |
    | 5671 | TCP | AMQP |
    | 443 | TCP | HTTPS |

   > **Note**: You may need to use the **Refresh** button in the toolbar at the top of the pane to see the new rules appear.

1. With these three ports open on the Network Security Group (NSG), the downstream devices will be able to connect to the IoT Edge Gateway using either MQTT, AMQP, or HTTPS protocols.

### Exercise 3: Download Device CA Certificate

In this exercise, you will explore the **vm-az220-training-gw0001-{your-id}** Virtual Machine that you just created and download the generated test certificates to the cloud shell.

#### Task 1: Connect to the VM

1. Verify that the IoT Edge virtual machine has been deployed successfully.

    You can check the Notification pane in the Azure portal.

1. Verify that your **@lab.CloudResourceGroup(ResourceGroup1).Namevm** resource group has been pinned to your Azure dashboard.

    To pin your resource group to the dashboard, navigate to your Azure dashboard, and then complete the following:

    * On the Azure portal menu, click **Resource groups**.
    * On the **Resource groups** blade, under **Name**, locate the **@lab.CloudResourceGroup(ResourceGroup1).Namevm** resource group.
    * On the **@lab.CloudResourceGroup(ResourceGroup1).Namevm** row, on the right side of the blade, click **...** and then click **Pin to dashboard**.

    You may want to edit your dashboard to make the RG tiles and listed resources more accessible.

1. On the Azure portal toolbar, click **Cloud Shell**

1. At the Cloud Shell command prompt, paste the **ssh** command that you noted in the earlier task, similar to **ssh vmadmin@vm-az220-training-edge0001-dm080321.centralus.cloudapp.azure.com**, and then press **Enter**.

1. When prompted with **Are you sure you want to continue connecting?**, type **yes** and then press **Enter**.

    This prompt is a security confirmation since the certificate used to secure the connection to the VM is self-signed. The answer to this prompt will be remembered for subsequent connections, and is only prompted on the first connection.

1. When prompted to enter the password, enter the administrator password that you created when the Edge Gateway VM was provisioned.

1. Once connected, the terminal will change to show the name of the Linux VM, similar to the following. This tells you which VM you are connected to.

    ``` bash
    username@vm-az220-training-gw0001-{your-id}:~$
    ```

1. To determine the Virtual Machines public IP address, enter the following command:

    ```bash
    nslookup vm-az220-training-gw0001-{your-id}.centralus.cloudapp.azure.com
    ```

    The output will be similar to:

    ```bash
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    Name:   vm-az220-training-gw0001-{your-id}}.centralus.cloudapp.azure.com
    Address: 168.61.181.131
    ```

    The public IP of the VM is the final **Address** value - in this case **168.61.181.131**.

    > **Important**: Make a note of this IP address - you will need it later. The IP Address will usually change every time the VM is restarted.

#### Task 2: Explore the IoT Edge configuration

During the initial launch of the VM, a script was executed that configured IoT Edge. This script performed the following operations:

* Installed **aziot-identity-service** package
* Installed **aziot-edge** package
* Downloaded an initial version of **config.toml** (the config file for IoT Edge) to **/etc/aziot/config.toml**
* Added the device connection string supplied when the ARM template was executed to **/etc/aziot/config.toml**
* Cloned the [Iot Edge git repository](https://github.com/Azure/iotedge.git) to **/etc/gw-ssl/iotedge**
* Created a directory **/tmp/lab12** and copied the IoT Edge gateway SSL test tools from **/etc/gw-ssl/iotedge**
* Generated the test SSL certs in **/tmp/lab12** and copied them to **/etc/aziot**
* Added the certs to the **/etc/aziot/config.toml**
* Applied the updated **/etc/aziot/config.toml** to the IoT Edge runtime

1. To determine the version of IoT Edge that was installed, enter the following command:

    ```bash
    iotedge --version
    ```

    The version installed at the time of writing is `iotedge 1.2.3`

1. To view the IoT Edge configuration, enter the following command:

    ```bash
    cat /etc/aziot/config.toml
    ```

    The output will be similar to:

    ```s
        [provisioning]
    source = "manual"
    connection_string = "HostName=iot-az220-training-dm080221.azure-devices.net;DeviceId=sensor-th-0067;SharedAccessKey=2Zv4wruDViwldezt0iNMtO1mA340tM8fnmxgoQ3k0II="

    [agent]
    name = "edgeAgent"
    type = "docker"

    [agent.config]
    image = "mcr.microsoft.com/azureiotedge-agent:1.2"

    [connect]
    workload_uri = "unix:///var/run/iotedge/workload.sock"
    management_uri = "unix:///var/run/iotedge/mgmt.sock"

    [listen]
    workload_uri = "fd://aziot-edged.workload.socket"
    management_uri = "fd://aziot-edged.mgmt.socket"

    [moby_runtime]
    uri = "unix:///var/run/docker.sock"
    network = "azure-iot-edge"

    trust_bundle_cert = 'file:///etc/aziot/azure-iot-test-only.root.ca.cert.pem'

    [edge_ca]
    cert = 'file:///etc/aziot/iot-edge-device-ca-MyEdgeDeviceCA-full-chain.cert.pem'
    pk = 'file:///etc/aziot/iot-edge-device-ca-MyEdgeDeviceCA.key.pem'
    ```

    During the setup, the **connection_string**, **trust_bundle_cert**, **cert** and **pk** values were updated.

1. To ensure the IoT Edge daemon is running, enter the following command:

    ```bash
    sudo iotedge system status
    ```

    This command will display output similar to:

    ```bash
    System services:
        aziot-edged             Running
        aziot-identityd         Running
        aziot-keyd              Running
        aziot-certd             Running
        aziot-tpmd              Ready

    Use 'iotedge system logs' to check for non-fatal errors.
    Use 'iotedge check' to diagnose connectivity and configuration issues.
    ```

1. To verify the IoT Edge runtime has connected, run the following command:

    ```bash
    sudo iotedge check
    ```

    This runs a number of checks and displays the results. For this lab, ignore the **Configuration checks** warnings/errors. The **Connectivity checks** should succeed and be similar to:

    ```bash
    Connectivity checks (aziot-identity-service)
    --------------------------------------------
    √ host can connect to and perform TLS handshake with iothub AMQP port - OK
    √ host can connect to and perform TLS handshake with iothub HTTPS / WebSockets port - OK
    √ host can connect to and perform TLS handshake with iothub MQTT port - OK

    Configuration checks
    --------------------
    ** entries removed for legibility **

    Connectivity checks
    -------------------
    √ container on the default network can connect to IoT Hub AMQP port - OK
    √ container on the default network can connect to IoT Hub HTTPS / WebSockets port - OK
    √ container on the default network can connect to IoT Hub MQTT port - OK
    √ container on the IoT Edge module network can connect to IoT Hub AMQP port - OK
    √ container on the IoT Edge module network can connect to IoT Hub HTTPS / WebSockets port - OK
    √ container on the IoT Edge module network can connect to IoT Hub MQTT port - OK
    ```

    If the connection fails, double-check the connection string value in **config.toml**.

1. To exit the VM shell, enter the following command:

    ```bash
    exit
    ```

    The connection to the VM should close and the cloud shell prompt should be displayed.

#### Task 3: Download SSL certs from VM to Cloud Shell

Next, you need to "download" the **MyEdgeDeviceCA** certificate from the **vm-az220-training-gw0001-{your-id}** virtual machine so that it can be used to encrypt communications between a leaf device and the IoT Edge gateway.

1. At the Cloud Shell command prompt, to download the **/tmp/lab12** directory from the **vm-az220-training-gw0001-{your-id}** virtual machine to the **Cloud Shell** storage, enter the following commands:

    ```bash
    mkdir lab12
    scp -r -p {username}@{FQDN}:/tmp/lab12 .
    ```

    >**Important**: The command above has a `space` followed by a period `.` at the end.

    > **Note**: Replace the **{username}** placeholder with the username of the admin user for the VM, and replace the **{FQDN}** placeholder with the fully qualified domain name for the VM. Refer to the command that you used to open the SSH session if needed.
    > `scp -r -p vmadmin@vm-az220-training-edge0001-dm080321.centralus.cloudapp.azure.com:/tmp/lab12 .`

1. Enter the Admin password for the VM when prompted.

    Once the command has executed, it will have downloaded a copy of the **/tmp/lab12** directory with the certificate and key files over SSH to the Cloud Shell storage.

1. To verify that the files have been downloaded, enter the following commands:

    ```bash
    cd lab12
    ls
    ```

    You should see the following files listed:

    ```bash
    certGen.sh  csr        index.txt.attr      index.txt.old  openssl_root_ca.cnf  serial
    certs       index.txt  index.txt.attr.old  newcerts       private              serial.old
    ```

    Once the files are copied to Cloud Shell storage from the **vm-az220-training-gw0001-{your-id}** virtual machine, you will be able to easily download any of the IoT Edge Device certificate and key files to your local machine as necessary. Files can be downloaded from the Cloud Shell using the `download <filename>` command. You will do this later in the lab.

### Exercise 4: Create a Downstream Device

In this exercise, a downstream device will be created and connected to IoT Hub via the gateway.

#### Task 1: Create Device Identity in IoT Hub

In this task, you will create a new IoT device identity in Azure IoT Hub for the downstream IoT device. This device identity will be configured so that the Azure IoT Edge Gateway is a parent device for this downstream device.

1. If necessary, log in to your Azure portal using your Azure account credentials.

1. On your Azure dashboard, to open your IoT Hub, click **iot-az220-training-{your-id}**.

1. On the **iot-az220-training-{your-id}** blade, on the left-side menu under **Explorers**, click **IoT devices**.

    This pane of the IoT Hub blade allows you to manage the IoT Devices connected to the IoT Hub.

1. At the top of the pane, to begin configuring a new IoT device, click **+ New**.

1. On the **Create a device** blade, under **Device ID**, enter **sensor-th-0072**

    This is the device identity used for authentication and access control.

1. Under **Authentication type**, ensure that **Symmetric key** is selected.

1. Under **Auto-generate keys**, leave the box checked.

    This will have IoT Hub automatically generate the Symmetric keys for authenticating the device.

1. Under **Parent device**, click **Set a parent device**.

    You will be configuring this downstream device to communicate with IoT Hub through the IoT Edge Gateway device that you created earlier in this lab.

1. On the **Set an Edge device as a parent device** blade, under **Device ID**, click **vm-az220-training-gw0001-{your-id}**, and then click **OK**.

1. On the **Create a device** blade, to create the IoT Device identity for the downstream device, click **Save**.

1. On the **IoT devices** pane, at the top of the pane, click **Refresh**.

1. Under **Device ID**, click **sensor-th-0072**.

    This will open the details view for this device.

1. On the IoT Device summary pane, to the right of the **Primary Connection String** field, click **Copy**.

1. Save the connection string for later reference.

    Be sure to note that this connection string is for the sensor-th-0072 child device.

#### Task 2: Download device x509 xertificate

In this task, you will configure the connection between a pre-built downstream device and your Edge gateway device.

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On the Azure portal toolbar, click **Cloud Shell**.

    Ensure that the environment is set to **Bash**.

    > **Note**: If Cloud Shell was already open and you are still connected to the Edge device, use an **exit** command to close the SSH session.

1. At the Cloud Shell command prompt, to download the root CA X.509 certificate for the IoT Edge Gateway virtual machine, enter the following command:

    ```bash
    download lab12/certs/azure-iot-test-only.root.ca.cert.pem
    ```

    The Azure IoT Edge Gateway was previously configured in the **/etc/aziot/config.toml** file to use this root CA X.509 certificate for encrypting communications with any downstream devices connecting to the gateway. This X.509 certificate will need to be copied to the downstream devices so they can use it to encrypt communications with the gateway.

1. Copy the **azure-iot-test-only.root.ca.cert.pem** X.509 certificate file to the **/Starter/DownstreamDevice** directory where the source code for the downstream IoT device is located.

    > **Important**: Make sure the file has that exact name. It may have a different name (for example, with **(1)** added) from previous labs, so rename it after copying it if necessary.

#### Task 3: Create hosts file entry

In earlier versions of this lab, the FQDN would be used as the value from the **GatewayHostName** in the device connectionstring, however the test x509 certificate generated by the current versions of the test scripts no longer supports this. Instead, only the hostname is used and an entry must be created in the local machine's **hosts** file to resolve the hostname to an IP Address. Complete the following steps to add the required entry to the hosts file.

1. Open Visual Studio Code.

1. On the **File** menu, click **Open File**.

1. Navigate to the following folder **c:\\Windows\\System32\\Drivers\\etc\\** file, and open the **hosts** file.

    > **Note**: the **hosts** file has no extension.

1. Add the following line to the **hosts** file, followed by an empty line:

    ```text
    {VM Public IP Address} vm-az220-training-gw0001-{your-id}
    {blank line}
    ```

    For example,

    ```text
    168.61.181.131 vm-az220-training-gw0001-dm090821

    ```

1. Save the file - when prompted that the save failed, click **Retry as Admin...** and in the **User Account Control** dialog, click **Yes**.

The local machine can now resolve the VM name to the appropriate IP Address.

#### Task 4: Connect Downstream Device to IoT Edge Gateway

1. Open Visual Studio Code.

1. On the **File** menu, click **Open Folder**.

1. In the **Open Folder** dialog, navigate to the Starter folder for lab 12, click **DownstreamDevice**, and then click **Select Folder**.

    You should see the azure-iot-test-only.root.ca.cert.pem file listed in the EXPLORER pane along with the Program.cs file.

    > **Note**: If you see messages to restore dotnet and/or load the C# extension, you can complete the installs.

1. In the EXPLORER pane, click **Program.cs**.

    A cursory review will reveal that this app is a variant of the **CaveDevice** application that you worked on in previous labs.

1. Locate the declaration for the **connectionString** variable, and then replace the placeholder value with the Primary Connection String for the **sensor-th-0072** IoT device.

1. Append the assigned **connectionString** value with a **GatewayHostName** property, and then set the value of GatewayHostName to be the full DNS name for your IoT Edge gateway device.

    The full DNS name of your Edge gateway device is the Device ID, **vm-az220-training-gw0001-{your-id}**, appended with your specified region and the Azure commercial cloud domain name, for example: **.westus2.cloudapp.azure.com**.

    The completed connection string values should match the following format:

    ```text
    HostName=<IoT-Hub-Name>.azure-devices.net;DeviceId=sensor-th-0072;SharedAccessKey=<Primary-Key-for-IoT-Device>;GatewayHostName=<HostName-for-IoT-Edge-Device>
    ```

    > **Important**: In the earlier version of the IoTEdge runtime, the **GatewayHostName** was the full DNS name

    Be sure to replace the placeholders shown above with the appropriate values:

    * **\<IoT-Hub-Name\>**: The Name of the Azure IoT Hub.
    * **\<Primary-Key-for-IoT-Device\>**: The Primary Key for the **sensor-th-0072** IoT device in Azure IoT Hub.
    * **\<Hostname-Name-for-IoT-Edge-Device\>**: The Hostname of the **vm-az220-training-gw0001-{your-id}** Edge device.

    The **connectionString** variable with the assembled connection string value will look similar to the following:

    ```csharp
    private readonly static string connectionString = "HostName=iot-az220-training-abc201119.azure-devices.net;DeviceId=sensor-th-0072;SharedAccessKey=ygNT/WqWs2d8AbVD9NAlxcoSS2rr628fI7YLPzmBdgE=;GatewayHostName=vm-az220-training-gw0001-{your-id}";
    ```

1. On the **File** menu, click **Save**.

1. Scroll down to locate the **Main** method, and then take a minute to review the code.

    This method contains the code that instantiates the **DeviceClient** using the configured Connection String, and specifies `MQTT` as the transport protocol to use for communicating with the Azure IoT Edge Gateway.

    ```csharp
    deviceClient = DeviceClient.CreateFromConnectionString(connectionString, TransportType.Mqtt);
    SendDeviceToCloudMessagesAsync();
    ```

    The Main method also:

    * calls the **InstallCACert** method which includes the code to automatically install the root CA X.509 certificate to the local machine.
    * calls the **SendDeviceToCloudMessagesAsync** method that sends event telemetry from the simulated device.

1. Locate the **SendDeviceToCloudMessagesAsync** method, and then take a minute to review the code.

    This method contains the code that generates the simulated device telemetry, and sends the events to the IoT Edge Gateway.

1. Locate the **InstallCACert** and browse the code that installs the root CA X.509 certificate to the local machine certificate store.

    > **Note**: Remember that this certificate is used to secure the communication from the device to the Edge Gateway. The device uses the symmetric key within the connection string for authentication with the IoT Hub.

    The initial code within this method is responsible for ensuring the **azure-iot-test-only.root.ca.cert.pem** file is available. Of course, in production applications you might consider using an alternative mechanism to specify the path to the X.509 certificate, such as an environment variable, or using TPM.

    Once the presence of the X.509 certificate has been verified, the **X509Store** class is used to load the certificate into the current user's certificate store. The certificate will then be available on-demand to secure communication to the gateway - this occurs automatically within the device client, so there is no additional code.

    > **Information**: You can learn more about the **X509Store** class [here](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.x509certificates.x509store?view=netcore-3.1).

1. On the **Terminal** menu, click **New Terminal**.

1. At the TERMINAL command prompt, enter the following command:

    ```bash
    dotnet run
    ```

    This command will build and run the code for the **sensor-th-0072** simulated device, which will start sending device telemetry.

    > **Note**: When the app attempts to install the X.509 certificate on the local machine (so that it can use it to authenticate with the IoT Edge Gateway), you may see a Security Warning asking about installing the certificate. You will need to click **Yes** to allow the app to continue.

1. If you are asked if you want to install the certificate, click **Yes**.

1. Once the simulated device is running, the console output will display the events being sent to the Azure IoT Edge Gateway.

    The terminal output will look similar to the following:

    ```text
    IoT Hub C# Simulated Cave Device. Ctrl-C to exit.

    User configured CA certificate path: azure-iot-test-only.root.ca.cert.pem
    Attempting to install CA certificate: azure-iot-test-only.root.ca.cert.pem
    Successfully added certificate: azure-iot-test-only.root.ca.cert.pem

    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    ```

    > **Note**: If the device send seems to pause for longer than a second on the first send, you likely did not add the NSG incoming rules correctly earlier, so your MQTT traffic is being blocked.  Check your NSG configuration.

1. Leave the simulated device running while you move on to the next exercise.

#### Task 5: Verify Event Flow

In this task, you will use the Azure CLI to monitor the events being sent to Azure IoT Hub from the downstream IoT Device through the IoT Edge Gateway. This will validate that everything is working correctly.

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. If Cloud Shell is not running, on the Azure portal toolbar, click **Cloud Shell**.

1. If you are still connected to the Edge device with an SSH connection in Cloud Shell, exit that connection.

1. At the Cloud Shell command prompt, to monitor the stream of events flowing to the Azure IoT Hub, run the following command:

    ```bash
    az iot hub monitor-events -n iot-az220-training-{your-id}
    ```

    Be sure to replace the `{your-id}` placeholder for the `-n` parameter with the name of your Azure IoT Hub.

    The `az iot hub monitor-events` command enables you to monitor device telemetry & messages sent to an Azure IoT Hub. This will verify that events from the simulated device, being sent to the IoT Edge Gateway, are being received by the Azure IoT Hub.

    > **Note**: If prompted `Dependency update (uamqp 1.2) required for IoT extension version: 0.10.13.`, enter **Y*.

1. With everything working correctly, the output from the `az iot hub monitor-events` command will look similar to the following:

    ```text
    chris@Azure:~$ az iot hub monitor-events -n iot-az220-training-1119
    Starting event monitor, use ctrl-c to stop...
    {
        "event": {
            "origin": "sensor-th-0072",
            "module": "",
            "interface": "",
            "component": "",
            "payload": "{\"temperature\":29.995470051651573,\"humidity\":70.47896838303608}"
        }
    }
    {
        "event": {
            "origin": "sensor-th-0072",
            "module": "",
            "interface": "",
            "component": "",
            "payload": "{\"temperature\":28.459910635584922,\"humidity\":60.49697355390386}"
        }
    }
    ```

Once you have completed this lab and verified the event flow, exit the console application by pressing **CTRL+C**.
