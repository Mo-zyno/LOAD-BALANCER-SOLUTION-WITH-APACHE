# LOAD-BALANCER-SOLUTION-WITH-APACHE

**When we access a website in the Internet we use an URL and we do not really know how many servers are out there serving our requests, this complexity is hidden from a regular user, but in case of websites that are being visited by millions of users per day (like Google or Reddit) it is impossible to serve all the users from a single Web Server**

**Each URL contains a domain name part, which is translated (resolved) to IP address of a target server that will serve requests when open a website in the Internet. Translation (resolution) of domain names is perormed by DNS servers, the most commonly used one has a public IP address 8.8.8.8 and belongs to Google**

*When you have just one Web server and load increases – you want to serve more and more customers, you can add more CPU and RAM or completely replace the server with a more powerful one – this is called "vertical scaling". This approach has limitations – at some point you reach the maximum capacity of CPU and RAM that can be installed into your server*

**Another approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers. This approach is much more common and can be applied almost seamlessly and almost infinitely (you can imagine how many server Google has to serve billions of search requests).**

**Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. Adjustment of number of servers can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory load).**

*Property of a system (in our case it is Web tier) to be able to handle growing load by adding resources, is called "Scalability".*

**In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers**

**In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.**

*Let us take a look at the updated solution architecture with an LB added on top of Web Servers (for simplicity let us assume it is a software L7 Application LB, for example – Apache, NGINX or HAProxy)*



![3tier](https://user-images.githubusercontent.com/97651517/158739203-6038f056-2b97-4973-9cbf-30cc7f4d14b1.png)


## Prerequisites


1.Two RHEL8 Web Servers


2.One MySQL DB Server (based on Ubuntu 20.04)


3.One RHEL8 NFS server



![pre-requisite for this project](https://user-images.githubusercontent.com/97651517/158739320-0dfb4761-da4d-4d67-b1de-a76bf56eca39.png)


# Configure Apache as a Load Balanacer


## STEP 1.


**Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:**


![lb-sever](https://user-images.githubusercontent.com/97651517/158739527-641b45aa-1b58-4c22-b7d6-9995ab711a39.png)


*ssh into terminal*



![ssh into terminal](https://user-images.githubusercontent.com/97651517/158739613-9b2560a3-3086-4cbc-8101-d76a0ae3c8d6.png)


## STEP 2.

**update and upgrade the server**

`sudo apt update -y`

`sudo apt upgrade`


![sudo apt update](https://user-images.githubusercontent.com/97651517/158739701-01d55a88-bca2-4f32-92ab-c1d802addf7d.png)



![sudo apt upgrade](https://user-images.githubusercontent.com/97651517/158739718-5dc17e44-9574-45aa-8f49-c358907cd116.png)


## STEP 3.

**open TCP Port 80**

*http 0.0.0.0*


![open http port 80 ](https://user-images.githubusercontent.com/97651517/158739817-25a7b4a7-f538-4acf-8178-2dbce202eeac.png)


## STEP 4.

**Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers**


`sudo apt install apache2 -y`


![sudo apt install apache 2](https://user-images.githubusercontent.com/97651517/158739925-5d5f2d13-3922-4a3e-992d-7c09bca9bdcf.png)


`sudo apt-get install libxml2-dev`


*install dependencies*


![install dependencies](https://user-images.githubusercontent.com/97651517/158740166-e73a1956-f988-41ef-b352-7416c9962a8c.png)


**Enable Modules/Dependencies**

`sudo a2enmod rewrite`

`sudo a2enmod proxy`

`sudo a2enmod proxy_balancer`

`sudo a2enmod proxy_http`

`sudo a2enmod headers`

`sudo a2enmod lbmethod_bytraffic`


![enabling apache modules](https://user-images.githubusercontent.com/97651517/158740392-31ff966d-7894-4a5d-92fb-91b442dd6dbe.png)


`sudo systemctl restart apache2`


![sudo systemctl restart apache2](https://user-images.githubusercontent.com/97651517/158740533-a9498b1f-46de-4b8d-b6b2-372f05704bdf.png)


**make sure Apache is up an running**


`sudo systemctl status apache2`



![apache status](https://user-images.githubusercontent.com/97651517/158740660-d0831b58-4f9d-4028-9626-09e983c2aa05.png)



## STEP 5.

# Configure load balancing


*insert private IP address of webserver1 and webserver2*


`sudo vi /etc/apache2/sites-available/000-default.conf`

![configure LB](https://user-images.githubusercontent.com/97651517/158741000-a05a2325-ef99-463b-a74e-8c4466334f75.png)


`sudo systemctl restart apache2`


![restart apache LB](https://user-images.githubusercontent.com/97651517/158741107-de0075a1-6907-41cd-988f-f1da1ed56989.png)


**check that Apache is up and running**

`sudo systemctl status apache2`



![apache status](https://user-images.githubusercontent.com/97651517/158741190-407d3f35-f3f1-4f06-8f89-d02ad18322ca.png)


*NB: bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.*


## STEP 6.


### Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:


*<Load Balancer Public IP addr/index.php*



![browsing with LB ip addr](https://user-images.githubusercontent.com/97651517/158741314-a29ff05f-d166-490f-ad25-fd3e650318d9.png)


*Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.*

`sudo tail -f /var/log/httpd/access_log`

*Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.*


  
![unmount webserver1](https://user-images.githubusercontent.com/97651517/158741433-fba535be-028f-43b5-9cf0-7594ee0cb5a0.png)


  
![unmount webserver2](https://user-images.githubusercontent.com/97651517/158741442-52b1d2e0-2c59-41fe-8323-5f7dce8747d9.png)
  

## STEP 7.

# Configure Local DNS Names Resolution


  **Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB**


  *Open this file on your LB server*


  ### Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers


*<WebServer1-Private-IP-Address> Web1*


  *<WebServer2-Private-IP-Address> Web2*


  `sudo vi /etc/hosts`
 

  ![config dns resolution](https://user-images.githubusercontent.com/97651517/158741638-b9550157-8d55-4914-8e6e-0ad79d7989e3.png)
  
  
**Now you can update your LB config file with those names instead of IP addresses.**


  `sudo vi /etc/apache2/sites-available/000-default.conf`
  

  ![web1 and web2](https://user-images.githubusercontent.com/97651517/158741803-de56757a-629d-4743-8905-7286a28d84e0.png)


  *BalancerMember http://Web1:80 loadfactor=5 timeout=1*


  *BalancerMember http://Web2:80 loadfactor=5 timeout=1*


  **You can try to curl your Web Servers from LB locally curl http://Web1 or curl http://Web2 – it shall work.**


  `curl http://Web1`


  `curl http://Web2`
  

  ![curl web1](https://user-images.githubusercontent.com/97651517/158741931-a53c1d26-c19c-46bc-b9da-3be3569f6663.png)


  ![curl web2](https://user-images.githubusercontent.com/97651517/158741941-b4038e8d-3885-4abc-86b6-4407d33c9bc5.png)


  **Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally not 
  from the Internet.**


  *NB: Image of What we succeeded in implimenting in this project*


  ![apache LB](https://user-images.githubusercontent.com/97651517/158742136-d24211c0-92f6-43fe-b456-be3fa429c746.png)


  ## And that concludes this project.




