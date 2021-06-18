# WebLogic Basic Lab

## Intro to WebLogic Server

## Typical topology
Admin Server + Managed server

## Connectivity to Database

DataSources + Connection poooling

## Running and deploying JavaEE applications

Paragraph about this


## Running WebLogic Server in Docker

This lab uses the Docker image with WebLogic domain inside the image deployment. We do it as this is the easiest way nowadays to provision lab environments and provision Weblogic in testing and development environments. We avoid process of installing Java, WebLogic Server binaries and configuring the domain. This means that all the artifacts and domain-related files are stored within the image. 

## WebLogic-Remote-Console

During these labs we will be using the latest WebLogic Server console that runs outside of the context of the Admin Server. It will run on the desktop of the Administrator. It is called WebLogic Remote Console  and it is a lightweight, open-source console that you can use to manage your WebLogic Server domain running anywhere, such as on a physical or virtual machine, in a container, Kubernetes, or in the Oracle Cloud. The Remote Console does not need to be co-located with the WebLogic Server domain.

You can install and run the Remote Console anywhere, and connect to your domain using WebLogic REST APIs. You simply launch the desktop application and connect to the Administration Server of your domain. Or, you can start the console server, launch the console in a browser and then connect to the Administration Server.

The Remote Console is fully supported with WebLogic Server 12.2.1.3, 12.2.1.4, and 14.1.1.


### Key Features of the WebLogic Remote Console

The WebLogic Remote Console provides an alternative WebLogic Server administration GUI that enables REST-based access to WebLogic management information, in alignment with current cloud-native trends. When connected to a WebLogic domain using the Remote Console, you can:
- Configure WebLogic Server instances
- Configure WebLogic Server clusters
- Configure WebLogic Server services, such as database connectivity (JDBC), and messaging (JMS)
- Deploy and Un-deploy applications
- Start and stop servers and applications
- Monitor server and application performance
- View server and domain log files
- View application deployment descriptors
- Edit selected runtime application deployment descriptor elements

### Difference with the WebLogic Server Administration Console

If you are already familiar with the WebLogic Server Administration Console deployed as part of your WebLogic domain, you'll notice these key differences in the WebLogic Remote Console:

- The user interface has been completely redesigned to conform to the Oracle Alta UI Design system and the Oracle Redwood theme included with Oracle JET.
- The configuration and monitoring content is separated into separate pages in the Remote Console. In the WebLogic Server Administration Console, the configuration and runtime information are presented on one page. See [Separation of Configuration and Runtime Data](https://github.com/oracle/weblogic-remote-console/blob/master/site/console_uidesign.md#separation).
- The Change Center is now expressed as a shopping cart. See [Use the Shopping Cart](https://github.com/oracle/weblogic-remote-console/blob/master/site/console_uidesign.md#cart).
- Instead of logging directly into the Administration Console deployed in a WebLogic domain, the Remote Console connects to the Administration Server in a WebLogic domain, with the credentials supplied by the user, using WebLogic REST APIs.

##  Step 1: Login to OCI console and Navigating through Resources

Due to the limitations of the Luna environment we can't start the docker directly on Luna desktop. So we need to ask you to connect to the "external" compute VM where Docker was already provisioned. This step indicates how to find out the IP of that compute VM where you will be starting WebLogic Server

- On the Desktop, Click on **Luna-Lab.html** file.

![](images/1.png)

- This Luna-Lab.html file contains the Credentials for your oracle cloud account which you will use for this lab.

![](images/2.png)

- Click on **OCI CONSOLE**. It opens a new tab. Click on **Copy** for copying **Username** and **Password** and paste it on new tab as shown below. Click on **Sign In**.

![](images/2.png)

![](images/3.png)
- Get Started page open up, where you have Quick Actions links for creating various resources.

![](images/4.png)

- Let’s navigate through the resources, which we provisioned for you as part of this Lab. 
- Click on **Hamburger menu** on the left upper corner, then click on **Compute -> Instances**.

![](images/5.png)

- In the **Compartment**, Select the compartment which is under the **Luna-Labs** as shown below.

![](images/6.png)

- You can see the Instance in this section, which we will use in this lab. Click on the **Instance**.

![](images/7.png)

- This page contains the details about this Instance, just note down the **Public IP Address**, which we will use later stage. <ANKIT TO DO hide the IP on 7 and 8 images)

![](images/8.png)

> Note: In Furthers steps, we will mention this Public IP Address as **<Public_Ip_Of_Instance>**. So, you just need to replace <Public_Ip_Of_Instance> with this Public IP Address of the VM where will be starting WebLogic domains.

> Leave this tab open in browser. We can use this to copy Public IP of Instance.

## Step 2: Starting Admin Server in Docker Container

Once we collected information about the IP of the VM where we will start WebLogic Server in that step you will connect to that VM (through SSH) and download (pull) image that contains the binaries of WebLogic and some sample domain confirguration. After this you will start Admin Server from sample WebLogic domain and you will try to open WebLogic Admin console to verify that the Admin server is up and running.

- Right Click on Luna Desktop and Click on Open Terminal Here.

![](images/9.png)

- Connect to VM as opc user using SSH

```bash
ssh -i ../.ssh/id_rsa opc@[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]
```

> Note: Here, **id_rsa** is private key which we need to access to the Instance. **[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]** part of the command should be replaced with the Public IP of your VM that you have identified in Step 1. Type **yes** when asked as shown below.

![](images/10.png)

- Start WebLogic Admin Server inside Docker.

The image with the sample domain is stored in OCIR registry (registry in Oracle Cloud). The server will open inside the container port 7001. And this port will be mapped to port 7001 of the VM. So the admin server will be accessible through port 7001 of our VM.

```bash
docker run -p 7001:7001 --network='bridge' --name='WLSADMIN' --rm iad.ocir.io/weblogick8s/weblogic-operator-tutorial-store:1.0 /u01/oracle/user_projects/domains/sample-domain1/bin/startWebLogic.sh
```

![](images/11.png)

![](images/12.png)

![](images/13.png)

> Note: This docker command initially search for this image locally. If not found, it downloads (pulls) the image from **Oracle Public Repository**. This image has WebLogic Domain embedded in it. Later this command also start the admin server in container on port number 7001 and also map this port to 7001 port in host machine.

- Open a new tab in Chrome Browser and type the **http://[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]:7001/console**. It will open Admin console. Enter **weblogic/welcome1** as **Username/Password** then click on **Login**.

![](images/14.png)

- Verify the configuration of the domain and verify that the Admin Server is properly up and running. To do this please click on **Environment-> Servers** and check that all the Managed servers are in **SHUTDOWN** state and Admin Server is in the **RUNNING** state.

![](images/15.png)

## Step 3: Starting Managed Server in Docker Container

In this step you will start another Docker container that will run one of the configured Managed Servers. We will use the same VM. So the Admin Server and the Managed Server will run in the same VM. Admin Server had openned the port 7001. While the managed server will listen on the port 8001.

- Open a new Terminal.
<ANKIT TO DO - again screenshot how to open>

- Connect through SSH to the VM where we are planning to run WebLogic
	
```bash
ssh -i ../.ssh/id_rsa opc@[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]
```

![](images/16.png)
	
- Start Managed Server as Docker container

We will use the same image as we were using in Step 2. This time we will start another process within the context of that Docker container (Managed Server instead of Admin Server). The Managed Server intends to open and bind to port 8001 of our VM	

```bash
docker run -p 8001:8001 --network='bridge' --name='WLSMS1' --rm iad.ocir.io/weblogick8s/weblogic-operator-tutorial-store:1.0 /u01/oracle/user_projects/domains/sample-domain1/bin/startManagedWebLogic.sh managed-server1 t3://<Public_Ip_Of_Instance>:7001 -Dweblogic.management.username=weblogic -Dweblogic.management.password=welcome1
```

![](images/17.png)

![](images/18.png)

> Note: As image is already downloaded in local repositories. Thus, it starts the **managed-server1** at port number 8001 in container (map to port 8001 in host machine). Here we need to authenticate while connecting to admin server.

- Go to the WebLogic Admin Console (that you have oopened in Chrome browser in step 2). You have left console on the Servers page in Step 2. So if you haven't navigate away from that page then it is enough just to click on **Refresh icon** to verify Managed Server named "manged-server1" is in **RUNNING** State. If you navigated away on the console from the Servers page you need navigate back to that page (as described in the Step 2)

![](images/19.png)

## Step 4: Creating JDBC Data-Source through Admin Console

WebLogic Server connects to Datbase(s) through mechanism called connection pooling. So the application server opens set of precreated connections that are reused by the applications. So once the application needs to utilize the precrated connection then it uses the API to find proper Connection Pool (the API is called DataSource API) and asks Connection Pool to "borrow" a connection for a moment. It is the Application developer responsibility to "return" the connection to the pool once the connection is not needed anymore. In this step we will explain how to configure DataSource object in WebLogic Server so the applications my interact with the Connection Pool mechanism and find proper DataSource.
	
In the Luna environment we created for you simple running Oracle DB. And in this lab we will demonstrate how to connect to that DB.

- On the Desktop, Click on **Luna-Lab.html** file.

- This Luna-Lab.html file contains the Credentials for your oracle cloud account which you will use for this lab.

- Click on **OCI CONSOLE**. It opens a new tab. Click on Copy for copying Username and Password and paste it on new tab as shown below.

- Click on **Sign In**.

- Click on **Hamburger menu** in upper left corner, then click on **Oracle Database -> Bare Metal, VM and Exadata**.

![](images/20.png)

- Select the Same **Compartment** and Click on DB System **DBWLS** which is created for you.

![](images/21.png)

- Click on the Databases **DBWLS** as shown below.

![](images/22.png)

- Click on **DB Connection** tab, Then Click on **Show** in **Easy Connect**. It will provide the **Connection String** Which we will use for using this database.

![](images/23.png)

![](images/24.png)

> Note: You need to note some information like **Hostname**, **DB name** and **User name**. This information will be required, when we will create JDBC data source in next steps.
	[ANKIT TO DO - THIS DOESN'T EXPLAIN WHERE I SHOULD READ IT FROM I MEAN HOSTNAM DBNAME and USERNAME???? ]

- Go back to browser and open a WebLogic Admin Console at **http://[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]:7001/console** . Use **weblogic/welcome1** as **Username/Password** then click on **Login**.

- The configuration changes require to open the "Edit" session. So you need to click on **Lock & Edit**.

![](images/25.png)

- Click on **Services-> Data Sources-> New-> Generic Data Source**.

![](images/26.png)

- Enter the Following Details and click on **Next**.

> 	Name:     			  testDS

> 	Scope:		      		  Global

> 	JNDI Name:		      	  jdbc/testDS (this is the name that Application will use to find Connection Pool)

> 	Database Type:			  Oracle

![](images/27.png)


- Select the Database Driver as **“Oracle Driver (Thin) for Service Connection; Version: Any”** and click on **Next**.

![](images/28.png)

- Leave Default on Next Screen and click on **Next**.

![](images/29.png)

- Enter the following information for Database connection and click on **Next**.

>  Database Name, Host Name details can be found in DB Connection String. [ANKIT TODO- YOU NEED TO EXPLAIN HOW TO READ THIS FROM THE CONNECTION STRING]

>   	Port:	 			1521

>   	Database User Name:		system

>   	Password:			AAaa11##1

![](images/30.png)


	[ANKIT TO DO THESE CREDENTIALS SHOULD BE PLACED IN ANOTHER MD file]
	
> Note: If you find out that you don’t have Oracle Database configured in your oracle cloud account. Then for completing this lab, you can use the below credentials.


>           Hostname:		129.146.60.235 <Public_Ip_of_Instance_Containing_DB>

>           DB Name:		DBWLS_phx16j.sub06110437450.ankitvcn.oraclevcn.com

>           Username:		system

>	    Password:		AAaa11##1

- Click on **Test Configuration**.  You will see message **“Connection test succeeded”**. Click on **Next**.

![](images/31.png)

![](images/32.png)

- Select **‘admin-server’** as **Select Targets** and click on **Finish**.

![](images/33.png)

- Click on **“Activate Changes”**. This will end "Edit" session and apply changes

![](images/34.png)

![](images/35.png)

- The demo image that we are using is having one preconfigured web application that is runnin on Managed Server. This preconfigured web app allows you test and verify all JDBC Connection parameters from DataSource. Click on **Deployments**.

![](images/36.png)

> Note: We have **testwebapp** deployed in **cluster**

- Go to Browser and type the following URL **http://[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]:8001/opdemo/?dsname=testDS**.

![](images/37.png)

> This application will provide you information about **Datasource name**, **Database URL** and **Database User**.

> Note: Here **managed-server1** is running at port number **8001** and context for application is **‘/opdemo’**. So, once you provide the **Datasource name** in URL, it will provide you the Database URL for this Datasource.

  
## Step 5: Deployment of Application on Managed Server

[ANKIT TO DO - 4-5 sentences what we do in this lab	
	
- Open a new terminal.
[ANKIT TO DO screenshot for openning termina]
	
- Download demo application archive
	
```bash
curl -LSs https://github.com/pandey-ankit/WebLogic-Basic-Lab/blob/main/aussie-tripper-v1.ear?raw=true > aussie-tripper-v1.ear
```
 
![](images/38.png)

> Note: This command downloads the application **aussie-tripper-v1.ear** from git repository and put it in Desktop. 

  
- Go to Admin Console **http://[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]:7001/console**  and Click on **Deployments**.

![](images/39.png)

- Making application deployment requires making changes in the Weblogic configuration so you need to click on **Lock & Edit** first to start "Edit" session.

![](images/40.png)

- Click on **Install**.

![](images/41.png)

- Click on **“Upload your file(s)”**.

![](images/42.png)

- Click on **“Choose File”** in **Deployment Archive**.

![](images/43.png)

- Select the **aussie-tripper-v1.ear** file in **/home/luna.user/Desktop/** folder and click on **Open**.

![](images/44.png)

- Click on **Next**.

![](images/45.png)

- On Next Page, Select the **aussie-tripper-v1.ear** file and then click on **Next**.

![](images/46.png)

- Select **“Install this deployment as an application”** and then click on **Next**.

![](images/47.png)

- Select **“cluster-1”** as Available targets for aussie-tripper-v1 and then click on **Next**.

![](images/48.png)

- On Next page, Click on **Finish**.

![](images/49.png)

- Click on **“Activate Changes”**.

![](images/50.png)

- Select the **Control tab** and then check the box for **“aussie-tripper-v1”** app and click on **Start->Servicing all request**.

![](images/51.png)

- Click on **“Yes”**.

![](images/52.png)
  
- Open a new tab in Chrome Browser and open **http://[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]:8001/aussie-tripper/**

![](images/53.png)

- You can add Trips by Click on Different Destination and you can also Clear Trips as well. Or you can play in another way wth that demo app

## Step 6: Accessing Admin Server using WebLogic-Remote-Console

[ANKIT TO DO - 5-6 sentence what we do in that step]	
	
- We need to install new lightway console to WebLogic. To do that first Copy the following URL[WebLogic-Remote-Console](https://github.com/oracle/weblogic-remote-console/releases)  and paste it on new tab in browser. Download the file **console.zip**.

![](images/54.png)

- Open a new terminal and navigate to the folder where you have downloaded binaries of the console.

```bash
cd /home/luna.user/Downloads/
```

- Extract the archive with the binaries	
	
```bash
unzip console.zip
```
![](images/55.png)

- Start console as below	
	
```bash
java -jar console/console.jar
```
![](images/56.png)

- It starts the WebLogic Remote Console. Go to Browser and paste the URL **http://localhost:8012/** This time console works directly on the desktop of luna (not inside VM) so you are using localhost as Hostname for the application. The port 8012 is default port of WebLogic Remote Console

![](images/57.png)

- You need to connect WebLogic Remote Console with WebLogic Admin Server. The Remote Console relies on REST API. So please enter the Following details and click on **Connect**.

> 	Username:		weblogic

> 	Password:		welcome1

> 	URL:			http://[REPLACE_THIS_WITH_THE_Public_Ip_Of_Instance]:7001

![](images/58.png)

## Re-Configuration of JDBC Datasource through WebLogic-Remote-Console

[ANKIT TO DO 3-4 sentences What we are planning to do here] 
	
- As we connect to Admin Server, we observe two options: **Configuration** and **Monitoring**.

![](images/59.png)

- Click on the **Configuration Icon -> Services-> JDBC System Resources-> testDS-> Connection Pool** as shown in below image.

![](images/60.png)

- Click on **Hamburger menu** in the upper left corner and then press **F11** to enter full screen.

![](images/61.png)

- Change the **Initial Capacity** and **Minimum Capacity** from **1** to **2** in **Connection Pool** tab and then click on **Save**.

![](images/62.png)
 
- It shows **“Changes added to the cart”**.

![](images/63.png)

- Click on the **Shopping Cart Icon** and then click on **Commit Changes**.

![](images/64.png)

![](images/65.png)

- Go to Admin Console **http://<Public_IP_Of_Instance>:7001/console** and verify the changes we made through **WebLogic-Remote-Console**.

![](images/66.png)

- Click on **Services-> Data Sources-> testDS-> Connection Pool**.

![](images/67.png)
 
- Verify the **Initial Capacity** and **Minimum Capacity**.

![](images/68.png)

## Monitoring through WebLogic-Remote-Console

	[ANKIT TO DO again 3-4 sentences what we are planning to do here]
	
- Click on **Monitoring Icon -> Running Servers-> managed-server1->Deployments-> aussie-tripper-v1_v1->Component Runtimes-> managed-server1/aussie-tripper**. 

![](images/69.png)

- Click on **Hamburger menu** of upper left corner and you may observe the **Session Opened Total Count** is equal to **1**.

![](images/70.png)
	
- Go to browser and open a **New Incognito window**.

![](images/71.png)

- Open aussie-tripper home page: **http://<Public_IP_of_Instance>:8001/aussie-tripper/**.

![](images/72.png)

  
- Go back to **WebLogic-Remote-Console** and click on **Refresh Icon**.

![](images/73.png)

- Verify the **open session count** as **2**.

![](images/74.png)
