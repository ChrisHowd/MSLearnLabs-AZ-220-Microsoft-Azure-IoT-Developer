---
lab:
    title: 'Lab 01: Getting Started with Azure'
    module: 'Module 1: Introduction to IoT and Azure IoT Services'
---

# Getting Started with Azure

## Lab Scenario

You work for a gourmet cheese company named Contoso. The company's Chief Technology Officer has evaluated the business opportunity for implementing IoT and has concluded that Contoso can realize significant benefits by implementing an IoT solution. Contoso has selected the Microsoft Azure IoT tools based on their evaluations.

As one of the individuals assigned to the project, you need to become familiar with the Azure tools.

## In This Lab

In this lab, you will become familiar with the Azure portal and you will setup a Resource Group. The lab includes the following exercises:

* Explore the Azure Portal
* Create an Azure Dashboard and Resource Group

## Lab Instructions

### Exercise 1: Explore the Azure Portal and Dashboard

Before you begin working with the Azure IoT services, it's good to be familiar with how Azure itself works.

Although Azure commonly referred to as a 'cloud', it is actually a web portal that is designed to make Azure resources accessible from a single web site. All of Azure is accessible through the Azure portal.

#### Task 1: Examine the Azure portal Home page

1. In your Web browser, to open the Azure portal, navigate to [portal.azure.com](http://portal.azure.com).

    When you log into Azure you will arrive at the Azure portal. The Azure portal provides you with a customizable UI that you can use to access your Azure resources.

1. In the upper left corner of the portal window, to open the Azure portal menu, click the hamburger menu icon.

    At the top of the portal menu, you should see a section containing four menu options:

    * The **Create a resource** button opens a page displaying the services available through the Azure Marketplace, many of which provide free options. Notice that services are grouped by technology, including "Internet of Things", and that a search box is provided.
    * The **Home** button opens a customized page that displays links to Azure services, your recently accessed services, and other tools.
    * The **Dashboard** button opens a page displaying your default (or most recently used) dashboard. You will be creating a dashboard later in this lab.

    * The **All services** button opens a page similar to the **Create a resource** button described above.

    The bottom section of the portal menu is a **FAVORITES** section that can be customized to show your favorite, or most commonly used, resources. Later in this lab, you will learn how to customize this default list of common services to make it a list of your own favorites.

1. On the Azure portal menu, click **Home**.

    The Home page provides a customized view of recently used resources and services, as well as other helpful links.

1. On the home page, under **Tools**, click **Azure Monitor**.

    Azure Monitor is a tool that can help you to manage your Azure resources. You will be using Azure Monitor later in this course when you have implemented the services that comprize your IoT solution.

1. On the left side navigation menu, to display a map of data center regions, click **Service Health**.

    Notice the dropdowns for **Subscription**, **Region**, and **Service**. When you subscribe to a resource in Azure you'll pick a region to deploy it to. Azure is supported by a series of regions placed all around the world.

    This map shows the current status of regions associated with your subscription(s). A green circle is used to indicate that services are running normally at that region.

    With any cloud vendor (Azure, AWS, Google Cloud, etc.), services will go down from time to time. If you see a blue 'i' next to a region on the Service Health map, it means the region is experiencing a problem with one or more services. Azure mitigates these issues by running multiple copies of your application in different regions (a practice referred to as *Geo-redundancy*). If a region experiences an issue with a particular service, those requests will roll over to another region to fulfill the request. This is one of the big advantages of hosting apps in the Azure cloud. Azure deals with the issues, so you don't have to.

1. In the upper-left corner of your Azure portal, to navigate back to your home page, click **Home**.

    You can also use the portal menu to perform some simple navigation. You will have a chance to try out some options for portal navigation shortly.

#### Task 2: Explore the Azure Service options

1. Open the Azure portal menu, and then click **All services**.

    The All services page provides you with a few different viewing options and access to all of the PaaS, IaaS, and SaaS services that Azure offers. The first time that you open the All services page, you will see the Overview page. This view is accessible from the left side menu.

    > **Definitions**: The term **PaaS** is an acronym for **Platform as a Service**, the term **IaaS** is an acronym for **Infrastructure as a Service**, and the term **SaaS** is an acronym for **Software as a Service**

1. On the **All services** page, on the left side menu under **Categories**, click **All**.

    This view displays all of the services organized into groups corresponding to each Category. The Search box at the top can be very helpful.

1. On the left side menu, under **Categories**, click **Internet of Things**.

    The list of services is now limited to the services directly related to an IoT solution.

    Service/Resource pages on the Azure portal are sometimes referred to as _blades_. When you opened the Service Health page a couple of steps back, you opened a Service Health blade.

    The Azure portal uses blades as a kind of navigation pattern, opening new blades to the right as you drill deeper and deeper into a service. This gives you a form of breadcrumb navigation as you navigate horizontally, and Azure provides a File Explorer style path at the top of the blade that is clickable. For example: Home > Monitor > Service Health. But not every page is a blade. You will get used to it pretty quickly.

1. On the **All services** page, hover your mouse pointer over **IoT Hub**.

    A dialog box should be displayed. In the top-right corner, notice the "star" shape. When the star shape is filled-in, the service is selected as a favorite. Favorites will appear on the list of your favorite services on the left navigation menu of the portal window. This makes it easier to access the services that you use most often. You can customize your favorites list by selecting the services that you use most.

1. In the top-right corner of the IoT Hub dialog, to add IoT Hub to the list of your favorite services, click the star shaped icon.

    The star should now appear filled. If the star is shown as an outline, click the star icon again.

    > **Tip**: When you add a new item to your list of favorites, it is placed at the bottom of the favorites list on the Azure portal menu. You can rearrange your favorites into the order that you want by using a drag-and-drop operation.

1. Use the same process to add the following services to your favorites: **Device Provisioning Services**, **Logic Apps**, **Stream Analytics jobs**, and **Storage Accounts**.

    > **Note**: You can remove a service from the list of your favorite services by clicking the star of a selected service.

1. On the left side menu, under **Categories**, click **General**.

1. Ensure that the following services are selected as favorites:

    * **Subscriptions**
    * **Resource groups**

    The favorites that you've added are enough to get you started, but you can use the Internet of Things category to add additional favorites to the portal menu if you want.

#### Task 3: Examine the Toolbar menu

1. Notice the toolbar at the top of the portal that runs the full width of the window.

    In addition to the hamburger menu icon on the far left of this toolbar, there several tool items that you will find helpful.

    First, notice that you have a _Search resources_ tool that can be used to quickly find a particular resource.

    To the right of the search tool are several buttons that provide access to common tools. You can hover the mouse pointer over a button to display the button name.

    * The _Cloud Shell_ button opens an interactive, authenticated shell right in the portal window that you can use to manage Azure resources. The Azure Cloud Shell supports Bash and PowerShell.
    * The _Directory + Subscriptions_ button opens a pane that you can use to manage your Azure subscriptions and account directory (the Azure Active Directory authentication mechanism).
    * The _Notifications_ button that opens a notifications pane. The notifications pane is useful when working with a long running process. You will be monitoring notifications when you create and configure resources throughout this course.
    * There are also buttons for *Settings*, *Help*, and *Feedback*. The *Help* button contains links to help documents and a list of useful keyboard shortcuts.

    On the far right is a button for your account information, providing you with access to things like your account password and billing information.

1. On the toolbar, click **Help**, and then click **Help + support**

1. On the **Help + support** blade, notice the four Tiles for _Getting started_, _Documentation_, _Billing FAQs_, and _Support plans_.

    The Help + support blade gives you access to lots of great resources. You may want to come back to this later for further exploration.

1. On the **Help + support** blade, click **Billing FAQs**

    A new browser tab should open to display Azure billing documentation.

1. Take a moment to scan the contents of the **Prevent unexpected charges with Azure billing and cost management** page. 

    If *you* are using a paid Azure subscription and you are responsible for billing (you are the Account Administrator), you can set up cost alerts to help manage your billing.

### Exercise 2: Create an Azure Dashboard and Resource Group

On the Azure portal, dashboards are used to present a customized view of your resources. Information is displayed through the use of tiles which can be arranged and sized to help you organize your resources in useful ways. You can create many different dashboards that provide different views and serve different purposes.

Each tile that you place on your dashboard exposes one or more of your resources. In addition to tiles that expose the data of an individual resource, you can create a tile for something called a resource group.

A resource group is a logical group that contains related resources for a project or application. The resource group can include all the resources for the solution, or only those resources that you want to manage as a group. You decide how you want to allocate resources to resource groups based on what makes the most sense for your organization. Generally, add resources that share the same lifecycle to the same resource group so you can easily deploy, update, and delete them as a group.

In this exercise, you will:

* create a custom dashboard that you can use during this course
* create a Resource Group and add a Resource Group tile to your dashboard

#### Task 1: Create a Dashboard

1. If necessary, open your Web browser and navigate to your Azure portal.

    You can use the following link to open the Azure portal: [Azure portal](https://portal.azure.com)

1. On the Azure portal menu, click **Dashboard**.

1. On the **My Dashboard** page, click **+ New dashboard**

    You can create a custom dashboard to organize and access your Azure resources for a project. In this case, you will create a custom dashboard for this course.

1. To name your new dashboard, replace **My Dashboard** with **AZ-220**

    In the upcoming steps you will be adding a tile to your dashboard manually. Another option would be to use drag-and-drop operations to add tiles from the Tile gallery to the space provided.

1. At the top of the dashboard editor, click **Done customizing**

    You should see an empty dashboard at this point.

#### Task 2: Create a Resource Group and add a Resource Group tile to your Dashboard

1. On the Azure portal menu, click **Resource groups**

    This blade displays all of the resource groups that you have created using your Azure subscription(s). If you are just getting started with Azure, you probably don't have any resource groups yet.

1. On the **Resource groups** blade, in the top-left corner area, click **+ Add**

    This will open a new blade named Create a resource group.

1. Take a moment to review the contents of the **Create a resource group** blade.

    Notice that the resource group is associated with a Subscription and a Region. Consider the following:

    * How might associating a subscription with your resource group be helpful?
    * How might associating a region with your resource group affect what you include in your resource group?

1. In the **Subscription** dropdown, select the Azure subscription that you are using for this course.

1. In the **Resource group** textbox, enter **rg-az220**

    The name of the resource group must be **unique** within your subscription. A green check mark will appear if the name that you enter has not already been used and confirms to resource group naming rules.

    > **Tip**: The Azure documentation describes all Azure [naming rules and restrictions](https://docs.microsoft.com/en-us/azure/architecture/best-practices/resource-naming).

1. In the **Region** dropdown, select a region that is near you.  

    You should check with your instructor as well, [as not all regions offer all services](https://azure.microsoft.com/en-us/global-infrastructure/services/).

    You need to provide a location for the resource group because the resource group stores metadata about the resources and acts as the default location for where new resources in the resource group will be created. For compliance reasons, you may want to specify where that metadata is stored. In general, it is recommended that you specify a location where most of your resources will reside. Using the same location can simplify the template used to manage your resources.

1. At the bottom of the **Create a resource group** blade, click **Review + create**.

    You should see a message informing you that the settings for your resource group have been validated successfully.

1. To create your resource group, click **Create**.

1. On the top menu of the **Resource groups** blade, to see your new resource group, click **Refresh**

    You will learn more about managing your resources as you continue through this course.

1. In the list of named resource groups, click the box to the left of the **rg-az220** resource group that your just created.

    > **Note**:  You don't want to open the resource group in a new blade, you just want to select it (check mark on the left).

1. On the right side of the screen, click the ellipsis (...) corresponding to your resource group, and then click **Pin to dashboard**

1. Close your **Resource groups** blade.

    Your dashboard should now contain an empty Resources tile, but don't worry, you will fill it up soon enough.
