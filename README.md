# Linux Server Configuration

> This project takes a baseline installation of a Linux server and prepares it to host a web application. The project also shows how to secure a server from a number of attack vectors, how to install and configure a database server, and how to deploy an existing web application onto it.

![](https://upload.wikimedia.org/wikipedia/commons/f/f8/Python_logo_and_wordmark.svg)
![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Postgresql_elephant.svg/200px-Postgresql_elephant.svg.png)
![](https://upload.wikimedia.org/wikipedia/commons/9/9d/Ubuntu_logo.svg)

![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)

The Linux Server Configuration project will address the following:

1. How to secure a server from a number of attack vectors. 

2. How to install and configure a database server.

3. How to deploy an existing web application onto the server.

## Project Configuration

The project has been configured with the below settings:

* Linux Server Config: Amazon Lightsail Ubuntu 18.04 server image
* Website URL: http://ec2-35-177-16-5.eu-west-2.compute.amazonaws.com/
* Server public ip address: 35.177.16.5
* SSH port: 2200
* Enabled ports in firewall: 2200, 80 and 123
* Linux user: grader

## Project Walkthrough

### Amazon Lightsail Setup
The project uses VirtualBox, Vagrant, Python and PostgreSQL. 

* Log into AWS Console [RegisteredUsers]('https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fnc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fhomepage&forceMobileApp=0'). If not please register to [NewRegistration]('https://portal.aws.amazon.com/billing/signup#/start')
* Click on **Create Instance**
* Select **OS Only**
* Select **Ubuntu 16.04 LTS**
* Select base instance 
* Change the name if you would like but is not mandatory
* Click on **Create Instance**
* Go to **Account**
* Go to SSH Keys and download the key (which is a .pem file) onto your local machine.
* Within the instance you just created, you can connect to it by clicking "Connect using SSH". Alternatively, take the following steps to connect via your own SSH client:
  1. Move the downloaded LightsailDefaultKey.pem public key file into the local folder ~/.ssh and rename it lightsail.pem
  2. In your terminal, type: chmod 600 ~/.ssh/lightsail.pem . This secures the public key while also making it accessible.
  3. To connect to the instance via the terminal: $ ssh -i ~/.ssh/lightsail.pem ubuntu@35.177.16.5, where 35.177.16.5 is the public IP address of the instance.
  
  

