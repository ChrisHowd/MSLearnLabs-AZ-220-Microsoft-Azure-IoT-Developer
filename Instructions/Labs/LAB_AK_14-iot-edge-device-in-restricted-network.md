---
lab:
    title: 'Lab 14: Run an IoT Edge device in restricted network and offline'
    module: 'Module 7: Azure IoT Edge Module'
---

# Run an IoT Edge device in restricted network and offline

## Lab Scenario

The conveyor belt monitoring system that you've implemented at Contoso's cheese packaging and shipping facilities is paying off. The system is now sending telemetry data to Azure IoT Hub that helps operations to manage the vibration levels of the belts, and the new IoT Edge devices helping to manage inventory by tracking the number of cheese package that pass through the system.

Your manager wants the system to be resilient to network outages, which do still occur occasionally in some areas of the cheese processing facilities. In addition, the IT department has requested that you optimize the system to bulk upload any non-critical telemetry data at specific times in the day to help load balance network usage.

You propose configuring IoT Edge to support an offline scenario in case the network drops, and you will look into storing telemetry from sensors locally (on device) and configuring the Edge devices for regular syncs at given times.

The following resources will be created:

![Lab 14 Architecture](media/LAB_AK_14-architecture.png)

## In this Lab

In this lab, you will complete the following activities:

* Verify that the lab prerequisites are met (that you have the required Azure resources)

  * The script will create an IoT Hub if needed.
  * The script will create a new device identity needed for this lab.

* Deploy Azure IoT Edge Enabled Linux VM
* Setup an IoT Edge Parent device with a Child IoT device
* Configure the IoT Edge device as Gateway
* Open the IoT Edge Gateway device inbound ports using Azure CLI
* Configure the IoT Edge Gateway device Time-to-Live and Message Store
* Connect the Child IoT device to the IoT Edge Gateway
* Test the device connectivity and offline support

## Lab Instructions

### Exercise 1: Verify Lab Prerequisites

This lab assumes the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name |
| IoT Hub | iot-az220-training-{your-id} |
| IoT Edge Device | vm-az220-training-gw0002-{your-id} |
| IoT Device | sensor-th-0084 |

To ensure these resources are available, complete the following tasks.

1. Select **Deploy to Azure**:

    [![Deploy To Azure](media/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab14.json)

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

1. In the **Vm Resource Group** field, enter **@lab.CloudResourceGroup(ResourceGroup1).Namevm**.

1. In the **Admin Username**, enter the account name you wish to use.

1. In the **Authentication Type** field, select **password**.

1. In the **Admin Password Or Key** field, enter the password you wish to use for the admin account.

1. To validate the template, click **Review and create**.

1. If validation passes, click **Create**.

    The deployment will start.

1. Once the deployment has completed, in the left navigation area, to review any output values from the template,  click **Outputs**.

    Make a note of the outputs for use later:
    * connectionString
    * deviceConnectionString
    * gatewayConnectionString
    * devicePrimaryKey
    * publicFQDN
    * publicSSH

The resources have now been created.

> **Note**: In addition to provisioning the VM and IoT Edge, the ARM template also configured the firewall rules for inbound traffic and created the child device.

### Exercise 2: Download Device CA Certificate

In this exercise, you will explore the **vm-az220-training-gw0002-{your-id}** Virtual Machine that you just created and download the generated test certificates to the cloud shell.

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

    > **Note**: If the cloud shell has not been configured, follow the steps in **Lab 3 - Exercise 2 - Task 3: Configure cloud shell storage & Task 4: Install Azure CLI Extension - cloud environment**.

1. At the Cloud Shell command prompt, paste the **ssh** command that you noted in the earlier task, similar to **ssh vmadmin@vm-az220-training-gw0002-dm080321.centralus.cloudapp.azure.com**, and then press **Enter**.

1. When prompted with **Are you sure you want to continue connecting?**, type **yes** and then press **Enter**.

    This prompt is a security confirmation since the certificate used to secure the connection to the VM is self-signed. The answer to this prompt will be remembered for subsequent connections, and is only prompted on the first connection.

1. When prompted to enter the password, enter the administrator password that you created when the Edge Gateway VM was provisioned.

1. Once connected, the terminal will change to show the name of the Linux VM, similar to the following. This tells you which VM you are connected to.

    ``` bash
    username@vm-az220-training-gw0002-{your-id}:~$
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

#### Task 3: Download SSL certs

Next, you need to "download" the **MyEdgeDeviceCA** certificate from the **vm-az220-training-gw0002-{your-id}** virtual machine so that it can be used to encrypt communications between a leaf device and the IoT Edge gateway.

1. At the Cloud Shell command prompt, to download the **/tmp/lab12** directory from the **vm-az220-training-gw0002-{your-id}** virtual machine to the **Cloud Shell** storage, enter the following commands:

    ```bash
    mkdir lab12
    scp -r -p <username>@<FQDN>:/tmp/lab12 .
    ```

    > **Note**: Replace the **<username>** placeholder with the username of the admin user for the VM, and replace the **<FQDN>** placeholder with the fully qualified domain name for the VM. Refer to the command that you used to open the SSH session if needed.
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

    Once the files are copied to Cloud Shell storage from the **vm-az220-training-gw0002-{your-id}** virtual machine, you will be able to easily download any of the IoT Edge Device certificate and key files to your local machine as necessary. Files can be downloaded from the Cloud Shell using the `download <filename>` command. You will do this later in the lab.

1. To download the root cert for use later in this lab, enter the following command:

    ```bashd
    download ~/lab12/
    ```

### Exercise 3: Configure IoT Edge Device Time-to-Live and Message Storage

Configuring your IoT Edge Devices for extended offline scenarios includes specifying the supported period of time that you may be offline, often referred to as Time-to-Live, and specifying your local storage settings.

The default value for Time-to-Live (TTL) is `7200` (7200 seconds, which is 2 hours). This is plenty of time for quick interruptions, but there are cases when two hours may not be long enough, when a device or solution needs to function in Offline mode for an extended period of time. For the solution to operate without telemetry data loss when extended periods in a disconnected state can occur, you can configure the TTL property of the IoT Edge Hub module to a value up to 1,209,600 seconds (a 2 week TTL period).

The IoT Edge Hub module (`$edgeHub`) is used to coordinate communications between the the Azure IoT Hub service and the IoT Edge Hub running on the gateway device. Within the Desired Properties for the Module Twin, the `storeAndForwardConfiguration.timeToLiveSecs` property specifies the time in seconds that IoT Edge Hub keeps messages when in a state disconnected from routing endpoints, such as the Azure IoT Hub service. The `timeToLiveSecs` property for the Edge Hub can be specified in the Deployment Manifest on a specific device as part of a single-device or at-scale deployment.

The IoT Edge Device will automatically store messages when in a disconnected / offline state. The storage location can be configured using a `HostConfig` object.

In this exercise, you will use the Azure Portal user interface for Azure IoT Hub to modify the `timeToLiveSecs` property for the Edge Hub (`$edgeHub`) module on the single IoT Edge Gateway device. You will also configure the storage location on the IoT Edge Device where the messages are to be stored.

#### Task 1: Configure the $edgeHub Module Twin

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On your **@lab.CloudResourceGroup(ResourceGroup1).Name** resource group tile, click **iot-az220-training-{your-id}**.

1. On left-side menu of your IoT hub blade, under **Automatic Device Management**, click **IoT Edge**.

    This pane allows you to manage the IoT Edge devices connected to the IoT Hub.

1. Under **Device ID**, click **vm-az220-training-gw0002-{your-id}**.

1. Under **Modules**, click **$edgeHub**.

    The Module Identity Details blade of the **Edge Hub** module provides access to the Module Identity Twin and other resources for your IoT Edge Device.

1. On the **Module Identity Details** blade, click **Module Identity Twin**.

    This blade contains the module identity twin for `vm-az220-training-gw0002-{your-id}/$edgeHub` displayed as JSON in an editor pane.

1. Take a moment to review the contents of the $edgeHub module identity twin.

    Notice that since this is a new device the desired properties are essentially empty.

1. Close the **Module Identity Twin** blade.

1. Navigate back to the **vm-az220-training-gw0002-{your-id}** blade.

1. At the top of the blade, click **Set Modules**.

    The **Set modules on device** blade enables you to create and configure the IoT Edge Modules deployed to this IoT Edge Device.

1. On the **Set modules** blade, under **Iot Edge Modules**, click **Runtime Settings**.

1. On the **Runtime Settings** pane, select the **Edge Hub** tab.

1. Locate the **Store and forward configuration - time to live (seconds)** field.

1. In the **Store and forward configuration - time to live (seconds)** textbox, enter **1209600**

    This specifies a message Time-to-Live value of 2 weeks for the IoT Edge Device, which is the maximum time.

    > **Note**:  There are several things to consider when configuring the **Message Time-to-Live** (TTL) for the Edge Hub (`$edgeHub`) module. When the IoT Edge Device is disconnected, the messages are stored on the local device. You need to calculate how much data will be stored during the TTL period, and make sure there is enough storage on the device for that much data. The amount of storage and TTL configured will need to meet the solutions requirements if you want to avoid the loss of important data.
    >
    > If the device does not have enough storage, then you need to configure a shorter TTL. Once the age of a message reaches the TTL time limit, it will be deleted if it has not yet been sent to Azure IoT Hub.

    The IoT Edge Device will automatically store messages when in a disconnected / offline state. The storage location can be configured using a `HostConfig` object.

1. Locate the **Environment Variables** area.

    You need to add a new environment variable in order to complete the configuration of the message storage location.

1. Under **Environment Variables**, in the **Name** textbox, enter **storageFolder**

1. Under **Environment Variables**, in the **Value** textbox, enter **/iotedge/storage/**

1. Locate the **Container Create Options** field.

    Notice that this field contains a `HostConfig` JSON object that can be configured. You will create a `HostConfig` property and an Environment Variable to configure the storage location for your Edge device.

1. In the `HostConfig` object, below the closing bracket of `PortBindings` property, add the following `Binds` property:

    ```json
    "Binds": [
        "/etc/aziot/storage/:/iotedge/storage/"
    ]
    ```

    > **Note**: Be sure to separate the `PortBindings` property from the `Binds` property with a comma.

    The resulting JSON in the **Create Options** textbox should look similar to the following:

    ```json
    {
        "HostConfig": {
            "PortBindings": {
                "443/tcp": [
                {
                    "HostPort": "443"
                }
                ],
                "5671/tcp": [
                {
                    "HostPort": "5671"
                }
                ],
                "8883/tcp": [
                {
                    "HostPort": "8883"
                }
                ]
            },
            "Binds": [
                "/etc/aziot/storage/:/iotedge/storage/"
            ]
        }
    }
    ```

    This `Binds` value configures the `/iotedge/storage/` directory in the Docker container for the Edge Hub Module to be mapped to the `/etc/aziot/storage/` host system directory on the physical IoT Edge Device.

    The value is in the format of `<HostStoragePath>:<ModuleStoragePath>`. The `<HostStoragePath>` value is the host directory location on the IoT Edge Device. The `<ModuleStoragePath>` is the module storage path made available within the container. Both of these values must specify an absolute path.

1. At the bottom of the **Runtime Settings** pane, click **Apply**.

1. On the **Set modules on device** blade, click **Review + create**.

1. Take a minute to review the contents of the deployment manifest.

    Find your updates within the deployment manifest. You will need to look under both `$edgeAgent` and `$edgeHub` to find them.

1. At the bottom of the blade, click **Create**.

    Once the change is saved, the **IoT Edge Device** will be notified of the change to the Module configuration and the new settings will be reconfigured on the device accordingly.

    Once the changes have been passed to the Azure IoT Edge device, it will restart the **edgeHub** module with the new configuration.

    >**Note**: In the **Modules** list, the **$edgeHub** module's **Runtime Status** will display an error.

1. To review the error message, click **Error**.

    The **Troubleshoot** page will display the error log. It will include an exception similar to the following:

    ```log
    Unhandled exception. System.AggregateException: One or more errors occurred. (Access to the path '/iotedge/storage/edgeHub' is denied.)
    ```

    The next task will resolve this error.

#### Task 2: Update Directory Permissions

Before continuing, it is essential for you to ensure that the user profile for the IoT Edge Hub module has the required read, write, and execute permissions to the **/etc/aziot/storage/** directory.

1. On the Azure portal toolbar, click **Cloud Shell**

1. At the Cloud Shell command prompt, paste the **ssh** command that you noted in the earlier task, similar to **ssh vmadmin@vm-az220-training-gw0002-dm080321.centralus.cloudapp.azure.com**, and then press **Enter**.

1. When prompted with **Are you sure you want to continue connecting?**, type **yes** and then press **Enter**.

    This prompt is a security confirmation since the certificate used to secure the connection to the VM is self-signed. The answer to this prompt will be remembered for subsequent connections, and is only prompted on the first connection.

1. When prompted to enter the password, enter the administrator password that you created when the Edge Gateway VM was provisioned.

1. Once connected, the terminal will change to show the name of the Linux VM, similar to the following. This tells you which VM you are connected to.

    ``` bash
    username@vm-az220-training-gw0002-{your-id}:~$
    ```

1. To view the running IoT Edge modules, enter the following command:

    ```bash
    iotedge list
    ```

1. Take a moment to review the output of the `iotedge list` command:

    You should see that the *edgeHub* has failed to start:

    ```text
    NAME             STATUS           DESCRIPTION                 CONFIG
    edgeAgent        running          Up 4 seconds                mcr.microsoft.com/azureiotedge-agent:1.1
    edgeHub          failed           Failed (139) 0 seconds ago  mcr.microsoft.com/azureiotedge-hub:1.1
    ```

    This is due to the fact that the *edgeHub* process does not have permission to write to the **/etc/aziot/storage/** directory.

1. To confirm the issue with the directory permission, enter the following command:

    ```bash
    iotedge logs edgeHub
    ```

    The terminal will output the current log - if you scroll through the log you will see the relevant entry that looks similar to the following:

    ```text
    Unhandled Exception: System.AggregateException: One or more errors occurred. (Access to the path '/iotedge/storage/edgeHub' is denied.) ---> System.UnauthorizedAccessException: Access to the path '/iotedge/storage/edgeHub' is denied. ---> System.IO.IOException: Permission denied
    ```

1. To update the directory permissions, enter the following commands:

    ```sh
    sudo chown $( whoami ):iotedge /etc/aziot/storage/
    sudo chmod 775 /etc/aziot/storage/
    ```

    The first command sets the owner of the directory to the current user and the owning user group to **iotedge**. The second command enables full access to both the current user and members of the **iotedge** group. This will ensure that the *edgeHub* module is able to create directories and files within the **/etc/iotedge/storage/** directory.

    > **NOTE**: If you see an error stating **chown: cannot access '/etc/iotedge/storage/': No such file or directory**, create the directory using the following command and then re-run the commands above:

    ```sh
    sudo mkdir /etc/iotedge/storage
    ```

1. To restart the *edgeHub* module, and then verify that it is started, enter the following commands:

    ```bash
    iotedge restart edgeHub
    iotedge list
    ```

    >**Note**: The module name for the restart is case-sensitive - **edgeHub**

    You should see that the *edgeHub* module is now running:

    ```text
    NAME             STATUS           DESCRIPTION      CONFIG
    edgeAgent        running          Up 13 minutes    mcr.microsoft.com/azureiotedge-agent:1.1
    edgeHub          running          Up 6 seconds     mcr.microsoft.com/azureiotedge-hub:1.1
    ```

You are now ready to connect an IoT device (the child/leaf) to this IoT Edge Gateway device.

### Exercise 4: Connect Child IoT Device to IoT Edge Gateway

The process to authenticate regular IoT devices to IoT Hub with symmetric keys also applies to downstream (or child / leaf) devices. The only difference is that you need to add a pointer to the Gateway Device to route the connection or, in offline scenarios, to handle the authentication on behalf of IoT Hub.

> **Note**: You will be using the connection string value for **sensor-th-0084** that you saved earlier in the lab. If you need a new copy of the connection string, it can be accessed from your Azure IoT Hub in the Azure portal. Open the **IoT devices** pane of your IoT Hub, click **sensor-th-0084**, copy the **Primary Connection String**, and then save it to a text file.

#### Task 1: Create hosts file entry

In earlier versions of this lab, the FQDN would be used as the value from the **GatewayHostName** in the device connectionstring, however the test x509 certificate generated by the current versions of the test scripts no longer supports this. Instead, only the hostname is used and an entry must be created in the local machine's **hosts** file to resolve the hostname to an IP Address. Complete the following steps to add the required entry to the hosts file.

1. Open Visual Studio Code.

1. On the **File** menu, click **Open File**.

1. Navigate to the following folder **c:\\Windows\\System32\\Drivers\\etc\\** file, and open the **hosts** file.

    > **Note**: the **hosts** file has no extension.

1. Add the following line to the **hosts** file, followed by an empty line:

    ```text
    {VM Public IP Address} vm-az220-training-gw0002-{your-id}
    {blank line}
    ```

    For example,

    ```text
    168.61.181.131 vm-az220-training-gw0002-dm090821

    ```

1. Save the file - when prompted that the save failed, click **Retry as Admin...** and in the **User Account Control** dialog, click **Yes**.

The local machine can now resolve the VM name to the appropriate IP Address.

#### Task 1: Configure device app

In this task, you will configure the downstream IoT device (child or leaf device) to connect to IoT Hub using Symmetric Keys. The devices will be configured to connect to IoT Hub and the parent IoT Edge Device using a Connection String that contains the Symmetric Key (in addition to the Gateway Hostname for the Parent IoT Edge Device).

1. Open the Windows **File Explorer** app, and then navigate to your **Downloads** folder.

    Your Downloads folder should contain the X.509 certificate file that was downloaded when you configured the IoT Edge Gateway. You need to copy this certificate file to the root directory of your IoT device app.

1. In the **Downloads** folder, right-click **azure-iot-test-only.root.ca.cert.pem**, and then click **Copy**.

    > **Note**: If you already had an azure-iot-test-only.root.ca.cert.pem file in your Downloads folder, the file that you need may be named azure-iot-test-only.root.ca.cert (1).pem. You will need to rename it to azure-iot-test-only.root.ca.cert.pem once you've added it to the destination folder.

    This file is the X.509 certificate file that you downloaded and will be adding to the lab 14 /Starter/ChildIoTDevice directory (where the source code for the Child IoT Device is located).

1. Navigate to the lab 14 Starter folder, and then paste the copied file into the **ChildIoTDevice** folder.

1. Ensure that the copied certificate file is named **azure-iot-test-only.root.ca.cert.pem**

    If you already had an azure-iot-test-only.root.ca.cert.pem file in your Downloads folder, the file may have been named azure-iot-test-only.root.ca.cert (1).pem.

1. Open a new instance of Visual Studio Code.

1. On the **File** menu, click **Open Folder**.

1. In the **Open Folder** dialog, navigate to the lab 14 **Starter** folder, click **ChildIoTDevice**, and then click **Select Folder**.

    You should now see the project files listed in the EXPLORER pane.

1. In the Visual Studio Code **EXPLORER** pane, click **Program.cs**.

1. In the **Program.cs** file, locate the declaration for the **connectionString** variable.

1. Replace the placeholder value with the Primary Connection String for the **sensor-th-0084** IoT Device.

1. Append the assigned **connectionString** value with a **GatewayHostName** property, and then set the value of GatewayHostName to be the full DNS name for your IoT Edge gateway device.

    The full DNS name of your Edge gateway device is the Device ID, **vm-az220-training-gw0002-{your-id}**, appended with your specified region and the Azure commercial cloud domain name, for example: **.westus2.cloudapp.azure.com**.

    The completed connection string values should match the following format:

    ```text
    HostName=<IoT-Hub-Name>.azure-devices.net;DeviceId=sensor-th-0072;SharedAccessKey=<Primary-Key-for-IoT-Device>;GatewayHostName=<DNS-Name-for-IoT-Edge-Device>
    ```

    Be sure to replace the placeholders shown above with the appropriate values:

    * **\<IoT-Hub-Name\>**: The Name of the Azure IoT Hub.
    * **\<Primary-Key-for-IoT-Device\>**: The Primary Key for the **sensor-th-0084** IoT device in Azure IoT Hub.
    * **\<DNS-Name-for-IoT-Edge-Device\>**: The Hostname name of the **vm-az220-training-gw0002-{your-id}** Edge device.

    The **connectionString** variable assignment code should look similar to the following:

    ```csharp
    private readonly static string connectionString = "HostName=iot-az220-training-1119.azure-devices.net;DeviceId=sensor-th-0084;SharedAccessKey=ygNT/WqWs2d8AbVD9NAlxcoSS2rr628fI7YLPzmBdgE=;GatewayHostName=vm-az220-training-gw0002-{your-id}";
    ```

1. On the **File** menu, click **Save**.

1. On the **View** menu, click **Terminal**.

    Ensure that the **Terminal** command prompt lists the `/Starter/ChildIoTDevice` directory.

1. To build and run the **ChildIoTDevice** simulated device, enter the following command:

    ```cmd/sh
    dotnet run
    ```

    > **Note**: When the app installs the **X.509 certificate** on the local machine (so it can use it to authenticate with the IoT Edge Gateway), you may see a popup window that asks if you would like to install the certificate. Click **Yes** to allow the app to install the certificate.

1. Notice the output displayed in the Terminal.

    Once the simulated device is running, the console output will display the events being sent to the Azure IoT Edge Gateway.

    The terminal output will look similar to the following:

    ```cmd/sh
    IoT Hub C# Simulated Cave Device. Ctrl-C to exit.

    User configured CA certificate path: azure-iot-test-only.root.ca.cert.pem
    Attempting to install CA certificate: azure-iot-test-only.root.ca.cert.pem
    Successfully added certificate: azure-iot-test-only.root.ca.cert.pem
    11/27/2019 4:18:26 AM > Sending message: {"temperature":21.768769073192388,"humidity":79.89793652663843}
    11/27/2019 4:18:27 AM > Sending message: {"temperature":28.317862208149332,"humidity":73.60970909409677}
    11/27/2019 4:18:28 AM > Sending message: {"temperature":25.552859350830715,"humidity":72.7897707153064}
    11/27/2019 4:18:29 AM > Sending message: {"temperature":32.81164186439088,"humidity":72.6606041624493}
    ```

1. Leave the simulated device running while you move on to the next Exercise.

#### Task 2: Test Device Connectivity and Offline Support

In this task, you will monitor events from the **sensor-th-0084** that are being sent to Azure IoT Hub through the **vm-az220-training-gw0002-{your-id}** IoT Edge Transparent Gateway. You will then interrupt connectivity between the **vm-az220-training-gw0002-{your-id}** and Azure IoT Hub to see that telemetry is still sent from the child IoT Device to the IoT Edge Gateway. After this, you will resume connectivity with Azure IoT Hub and monitor that the IoT Edge Gateway resumes sending telemetry to Azure IoT Hub.

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On the Azure portal toolbar, click **Cloud Shell**.

    Ensure that the Environment dropdown is set to **Bash**.

1. At the Cloud Shell command prompt, to start monitoring the Events being received by the Azure IoT Hub, enter the following command:

    ```cmd/sh
    az iot hub monitor-events --hub-name iot-az220-training-{your-id}
    ```

    Be sure to replace the `{your-id}` placeholder with your unique suffix for our Azure IoT Hub instance.

1. Notice that telemetry from the **sensor-th-0084** that is getting sent to Azure IoT Hub.

    Keep in mind that the **sensor-th-0084** simulated device application is configured to send telemetry to the **vm-az220-training-gw0002-{your-id}** IoT Edge Transparent Gateway virtual machine, which is then sending the telemetry on to Azure IoT Hub.

    The Cloud Shell should begin displaying event messages similar to the following:

    ```text
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

    > **Note**: Next, you will need to test the **Offline** capabilities. To do this, you need to make the **vm-az220-training-gw0002-{your-id}** device go offline. Since this is a Virtual Machine running in Azure, this can be simulated by adding an **Outbound rule** to the **Network security group** for the VM.

#### Task3: Add rule to block traffic

1. Within the **Azure portal**, navigate to your Dashboard, and then locate the **@lab.CloudResourceGroup(ResourceGroup1).Namevm** resource group tile.

1. In the list of resources, to open the **Network Security Group** for the **vm-az220-training-gw0002-{your-id}** virtual machine, click **nsg-vm-az220-training-gw0002-{your-id}**.

1. On the **Network security group** blade, on the left side navigation pane under **Settings**, click **Outbound security rules**.

1. At the top of the blade, click **+ Add**.

1. On the **Add outbound security rule** pane, set the following field values:

    * Destination port ranges: **\***
    * Action: **Deny**
    * Priority: 100
    * Name: **DenyAll**

    A **Destination port range** of "**\***" will apply the rule to all ports.

1. At the bottom of the blade, click **Add**.

1. Go back to the **Cloud Shell** in the Azure portal.

1. If the `az iot hub monitor-events` command is still running, end it by pressing **Ctrl + C**.

1. At the Cloud Shell command prompt, to connect to the **vm-az220-training-gw0002-{your-id}** VM using `ssh`, enter the following command:

    ```sh
    ssh <username>@<ipaddress>
    ```

    Be sure to replace the placeholders with the required values for the `ssh` command:

    | Placeholder | Value to replace |
    | :--- | :--- |
    | `<username>` | The admin **Username** for the **IoTEdgeGateaway** virtual machine. This should be **vmadmin**.
    | `<ipaddress>` | The **Public IP Address** for the **vm-az220-training-gw0002-{your-id}** virtual machine.

1. When prompted, enter the admin **Password** for the **vm-az220-training-gw0002-{your-id}**.

    The command prompt will be update once you are connected to the **vm-az220-training-gw0002-{your-id}** VM via `ssh`.

1. To reset the IoT Edge Runtime, enter the following command:

    ```sh
    sudo iotedge system restart
    ```

    This will force the IoT Edge Runtime to disconnect from the Azure IoT Hub service, and then attempt to reconnect.

1. To verify that the *edgeHub* module has restarted correctly, enter the following command:

    ```bash
    iotedge list
    ```

    If the *edgeHub* module failed to restart successfully, retry by entering the following commands:

    ```bash
    iotedge restart edgeHub
    iotedge list
    ```

1. To end the `ssh` session with the **vm-az220-training-gw0002-{your-id}**, enter the following command:

    ```cmd/sh
    exit
    ```

1. At the Cloud Shell command prompt, to start monitoring the Events being received by the Azure IoT Hub, enter the following command

    ```cmd/sh
    az iot hub monitor-events --hub-name iot-az220-training-{your-id}
    ```

    Be sure to replace the `{your-id}` placeholder with your unique suffix for our Azure IoT Hub instance.

1. Notice there are no longer any events being received by the **Azure IoT Hub**.

1. Switch to the Visual Studio Code window.

1. Open the **Terminal** where the **sensor-th-0084** simulated device application is running, and notice that it's still sending device telemetry to the **vm-az220-training-gw0002-{your-id}**.

    At this point the **vm-az220-training-gw0002-{your-id}** is disconnected from the Azure IoT Hub. It will continue to authenticate connections by the **sensor-th-0084**, and receive device telemetry from child device(s). During this time, the IoT Edge Gateway will be storing the event telemetry from the child devices on the IoT Edge Gateway device storage as configured.

1. Switch to you **Azure portal** window.

1. Navigate back to the **Network security group** blade for the **vm-az220-training-gw0002-{your-id}**.

1. On the left side navigation menu, under **Settings**, click **Outbound security rules**.

1. On the **Outbound security rules** pane, click **DenyAll**.

1. On the **DenyAll** pane, to remove this deny rule from the NSG, click **Delete**.

1. On the **Delete security rule** prompt, click **Yes**.

    Once the **vm-az220-training-gw0002-{your-id}** IoT Edge Transparent Gateway is able to resume connectivity with Azure IoT Hub, it will sync the event telemetry from all connected child devices. This includes the saved telemetry that couldn't be sent while disconnected, and all telemetry still being sent to the gateway.

    > **Note**:  The IoT Edge Gateway device may take a couple minutes to reconnect to Azure IoT Hub and resume sending telemetry. After waiting, you will see events showing up in the `az iot hub monitor-events` command output again.

In this lab we have demonstrated that an Azure IoT Edge Gateway can utilize local storage to retain messages that can't be sent due to an interruption in the connection to the IoT Hub. Once connection is reestablished, we saw that messages are then sent.

> **Note**:  Once you have finished with the lab, ensure you exit the device simulation application by pressing **CTRL+C** in the terminal.
