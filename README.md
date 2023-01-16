# On-Prem. App and Database Migration to AWS

#### I will be detailing the process of migrating an app with its database previously hosted in a local data-center to the Cloud (AWS).

Preceding every migration, proper **_Planning_** has to be done to ensure feasibility of the migration. The planning phase is usually exhaustive and factors in a number of things such as *timing, cost, strategy, is moving to the Cloud the best for your customers? Will code need to be refactored?* etc. As much time should be taken here and reviewing before proceeding with caution!

### Implementation
#### App Server (EC2)
For the networking, you can choose to put this in your default VPC or if you prefer to customize later, create a VPC.

![image](https://user-images.githubusercontent.com/64602124/212645110-ba4a6991-d4e0-4c7d-b050-bb0c378650f6.png)

Next, create 3 subnets in this VPC. 2 Private subnets and 1 Public subnet. For the Public, use CIDR 10.0.0.0/24, and use 10.0.1.0/24 & 10.0.2.0/24 for the Private subnets. When completed, your subnets should look like this:

![image](https://user-images.githubusercontent.com/64602124/212647761-385e464e-dd51-4ad7-9155-2e17ac5a1c8d.png)

Next, an EC2 instance will be provisioned to host the app files. *Based on details from the Planning phase, you are able to determine and select the appropriate Instance type that has the capacity to host the app.* For the **Network Settings**, place this instance in the VPC created earlier and the Public Subnet and then Enable Auto-assign public IP (since we want the app to be publicly accessible). In the *Security Group (SG) settings*, I am configuring a new SG and allowing source only from my IP see Network Settings below:

![image](https://user-images.githubusercontent.com/64602124/212653946-24eb3ef2-7d47-417a-bcd6-1aaba87f0654.png)

**_Prepping your app server:_**


#### Database Server (RDS)
Once again, based on findings from the Planning phase, I have been able to determine that the appropriate database Engine type to be used for hosting the database server is MySQL. In the Network settings, place the database in the Prod VPC earlier created, however choose No for Public Access. *Best practices recommends not to expose your db server to the public. All communications with your DB, should originate from the App Server*. Then Create database.
