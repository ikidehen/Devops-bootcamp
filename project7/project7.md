# AWS VPC Project

# Documentation

- Login to your AWS management console and search VPC

- Click on Create VPC  

![1](img7/1.png)

![2](IMG7/2.png)

![3](img7/3.png)

- Now we create an Internet Gateway

![4](img7/4.png)

![5](img7/5.png)

- Click on Create internet gateway

![6](img7/6.png)

- Click on Action and Select Attach to VPC
![7](IMG7/7.png)

![8](img7/8.png)

We will follow the following subnet naming schemes

```
EnvName-AppType-RouteType-AZ
```

For example,

```
Prod-Web-Public-2a
```

 Create some subnet

 - Public Subnets

| Subnet Name|	Availability Zone|	CIDR Block|	Type|
|------------|-------------------|------------|-----|
|Prod-Web-Public-2a	|us-west-2a	|10.0.0.0/28  |Public
|Prod-Web-Public-2b	|us-west-2b	|10.0.0.16/28 |Public
|Prod-Web-Public-2c	|us-west-2c	|10.0.0.32/28 |Public

 - Using the data above create a public subnet

 ![10](img7/10.png)

 ![11](img7/11.png)

 ![12](img7/12.png)

 ![13](img7/13.png)

 ![14](img7/14.png)

 ![15](img7/15.png)

 - Using the above steps, I created the other subnets with the data below

 - Application Subnets

|Subnet Name|	Availability Zone|	CIDR Block|	Type|
|-----------|--------------------|------------|-----|
|Prod-App-Private-2a|	us-west-2a|	10.0.0.48/28|	Private|
|Prod-App-Private-2b|	us-west-2b|	10.0.0.64/28|	Private|
|Prod-App-Private-2c|	us-west-2c|	10.0.0.80/28|	Private|

- Database Subnets

|Subnet Name|	Availability Zone|	CIDR Block|	Type|
|-----------|--------------------|------------|-----|
|Prod-DB-Private-2a|	us-west-2a|	10.0.0.96/28|	Private|
|Prod-DB-Private-2b	|us-west-2b|	10.0.0.112/28|	Private|
|Prod-DB-Private-2c	|us-west-2c|	10.0.0.128/28|	Private|

- Management Subnets

|Subnet Name|	Availability Zone|	CIDR Block|	Type|
|-----------|--------------------|------------|-----|
|Prod-Mgmt-Private-2a|	us-west-2a|	10.0.0.144/28|	Private|
|Prod-Mgmt-Private-2b|	us-west-2b|	10.0.0.160/28|	Private|
|Prod-Mgmt-Private-2c|	us-west-2c|	10.0.0.176/28|	Private|

- Platform Subnets

|Subnet Name|	Availability Zone|	CIDR Block|	Type|
|-----------|--------------------|------------|-----|
|Prod-Platform-Private-2a|	us-west-2a|	10.0.0.192/28|	Private|
|Prod-Platform-Private-2b|	us-west-2b|	10.0.0.208/28|	Private|
|Prod-Platform-Private-2c|	us-west-2c|	10.0.0.224/28|	Private|

This is what it looks like after creating all subnets

![16](img7/16.png)

## Route Table Design

- Click Route table in VPC dashboard

- Click create route table

![17](img7/17.png)

- Give it a name and select the VPC you created earlier

![18](img7/18.png)

- Click on subnet associations

![19](img7/19.png)

- Click on edit subnet associations

![20](img7/20.png)

- Add 3 public subnets (prod-web-public a,b,c)

![21](img7/21.png)

After using the table and steps above to create the other route tables and subnet association it should all look like this

![22](img7/22.png)

## NAT Gateway

- Create a NAT gateway and attach it to all our route tables created earlier

![23](img7/23.png)

- select a subnet and allocate an elastic ip

![24](img7/24.png)

![25](img7/25.png)

- Now add the NAT gateway to our route tables one by one

![26](img7/26.png)

- Click on edit routes

![27](img7/27.png)

- follow exactly process as the images

![28](img7/28.png)

![29](img7/29.png)

![30](img7/30.png)

![31](img7/31.png)

- Do the above steps for the other route tables which are DB, APP, MANAGEMENT ROUTE TABLES

## AWS VPC Topology

The following diagram shows the high-level VPC topology for our design.

Note: Both the internet Gateway (IGW) and NAT gateway(NAT-GW) gets deployed in the public subnet.

To check our VPC topology:

![32](img7/32.png)

## Network ACLs

The following are the tables for inbound and outbound rules for the DB NACL.

## DB NACL (Inbound Rules)

|Rule Number|	Type|	Protocol|	Port Range|	Source IP|	Allow/Deny|
|-----------|-------|-----------|-------------|----------|------------|
|100|	MySQL/Aurora(3306)|	TCP|	3306|	10.0.0.96/28|	Allow|
|110|	MySQL/Aurora(3306)|	TCP|	3306|	10.0.0.112/28|	Allow|
|120|	MySQL/Aurora(3306)|	TCP|	3306|	10.0.0.128/28|	Allow|
*|	All| Traffic|	All|	All|	0.0.0.0/0|	Deny|

## DB NACL (Outbound Rules)

|Rule Number|	Type|	Protocol|	Port Range|	Destination IP|	Allow/Deny|
|100|	Custom TCP|	TCP|	3306|	10.0.0.192/28|	Allow|
|110|	Custom TCP|	TCP|	3306|	10.0.0.208/28|	Allow|
|120|	Custom TCP|	TCP|	3306|	10.0.0.224/28|	Allow|
*|	All Traffic|	All|	All|	0.0.0.0/0|	Deny|

![33](img7/33.png)

![34](img7/34.png)

![35](img7/35.png)

![36](img7/36.png)

![37](img7/37.png)

![38](img7/38.png)

![39](img7/39.png)

![40](img7/40.png)

# The End.
