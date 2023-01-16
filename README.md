# On-Prem. App and Database Migration to AWS

#### I will be detailing the process of migrating an app with its database previously hosted in a local data-center to the Cloud (AWS).

### Demo of the Completed Migration
Start the app using `python3 app.py` then get the Public IP Address of the EC2 instance and add port 8080 (for Apache Tomcat Server) to access the UI of the app. You can view the demo in this video:

https://user-images.githubusercontent.com/64602124/212712679-dce45ef5-dfd7-4317-985b-08defdae92b7.mp4


Preceding every migration, proper **_Planning_** has to be done to ensure feasibility of the migration. The planning phase is usually exhaustive and factors in a number of things such as *timing, cost, strategy, is moving to the Cloud the best for your customers? Will code need to be refactored?* etc. As much time should be taken here and reviewing before proceeding with caution!

### Implementation
#### App Server (EC2)
For the networking, you can choose to put this in your default VPC or if you prefer to customize later, create a VPC. 
***
If you run into troubles later on connecting to your EC2 instance, be sure that _your VPC is attached to an Internet Gateway_ and _in the Route Tables, Edit Route and add source from any IP (0.0.0.0/0) & destination to the IGW._
***

![image](https://user-images.githubusercontent.com/64602124/212645110-ba4a6991-d4e0-4c7d-b050-bb0c378650f6.png)

Next, create 3 subnets in this VPC. 2 Private subnets and 1 Public subnet. For the Public, use CIDR 10.0.0.0/24, and use 10.0.1.0/24 & 10.0.2.0/24 for the Private subnets. When completed, your subnets should look like this:

![image](https://user-images.githubusercontent.com/64602124/212647761-385e464e-dd51-4ad7-9155-2e17ac5a1c8d.png)

Next, an EC2 instance will be provisioned to host the app files. *Based on details from the Planning phase, you are able to determine and select the appropriate Instance type that has the capacity to host the app.* For the **Network Settings**, place this instance in the VPC created earlier and the Public Subnet and then Enable Auto-assign public IP (since we want the app to be publicly accessible). In the *Security Group (SG) settings*, I am configuring a new SG and allowing source only from my IP see Network Settings below:

![image](https://user-images.githubusercontent.com/64602124/212653946-24eb3ef2-7d47-417a-bcd6-1aaba87f0654.png)

**_Prepping your app server:_**
After installing prerequisites, from the cmd line, the server is now ready for the app.
`sudo apt-get update`

`sudo apt install python3-pip`\
`sudo apt-get install python3-dev -y`\
`sudo apt-get install libmysqlclient-dev -y`\
`sudo apt-get install unzip -y`\
`sudo apt-get install libpq-dev python3-dev libxml2-dev libxslt1-dev libldap2-dev -y`\
`sudo apt-get install libsasl2-dev libffi-dev -y`\
`curl -O https://bootstrap.pypa.io/pip/3.6/get-pip.py`\
`python3 get-pip.py userexport PATH=$PATH:/home/ubuntu/.local/bin/`\
`pip3 install flask`\
`pip3 install wtforms`\
`pip3 install flask_mysqldb`\
`pip3 install passlib`\
`sudo apt install awscli -y`\
`sudo apt-get install mysql-client -y`

Download the app files and database dump files from a remote repo. I put mine in an S3 bucket.
- `wget https://eve-migrate-files.s3.us-east-2.amazonaws.com/app.zip`
- `https://eve-migrate-files.s3.us-east-2.amazonaws.com/dump.sql`

#### Database Server (RDS)
Once again, based on findings from the Planning phase, I have been able to determine that the appropriate database Engine type to be used for hosting the database server is MySQL. In the Network settings, place the database in the Prod VPC earlier created, however choose No for Public Access. *Best practices recommends not to expose your db server to the public. All communications with your DB, should originate from the App Server*. In your Security Group, create a new SG and allow access only from the Private IP of the app instance. Then Create database.

###Connecting the App Server to the Database Server
Click on the RDS instance and in the Connectivity & Security section, take note of the Endpoint:

![image](https://user-images.githubusercontent.com/64602124/212687304-3d9c9e71-d79a-46de-af5f-e3c3c8e87c92.png)

Use the command `mysql -u admin -P 3306 -h database-server.cwu1sb2kffxo.us-east-2.rds.amazonaws.com -p` to connect to the DB. *Note that this admin user and password are same as the ones created I was setting up the instance.*
When connected, create the database using `create database wikidb;` > `use wikidb;` > dump the SQL files `source dump.sql;` to import the database into our instance.

Since the app is set up to communicate with the DB Server, it make sense to make an app user in the DB specifically for this purpose. [Creating User in MySQL](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)

`CREATE USER wiki@'%' IDENTIFIED BY 'wiki123456';` > `GRANT ALL PRIVILEGES ON wikidb.*TO wiki@'%';` > `FLUSH PRIVILEGES;` which triggers the DB to reload the in-memory copy of privileges from the privilege tables. This is a good practice after making manual edits to tables.

![image](https://user-images.githubusercontent.com/64602124/212694868-445992ed-5c23-48c2-9024-b5905d0b604e.png)

Now, in the app.py file, I am updating the `MYSQL_HOST` from the on-prem host name to the RDS Endpoint.


