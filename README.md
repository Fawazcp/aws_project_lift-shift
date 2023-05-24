# aws_project_lift-shift
AWS PROJECT
Lift and Shift Application Workloads
Step 1 
Create Security Group for the Load balancer
- Add HTTP and HTTPS inbound rules.
Select the default VPC and add HTTP and HTTPS inbound rule then click on create.

Now we have created Security group with HTTP and HTTPS traffic from anywhere IPv4

Next we need to create another security group for the TomCat instance
- Add inbound rule Port 8080 allow from the load balancer security group


Drag and dropdown to select the security group we created for load balancer.
-	Under outbound rule leave as default and click on create secuoty group



Create one more security group for backend services (RabbitMQ,memcahed and Mysql)
Here we need to add 3 inbound rules for MySQL (3306), RabbitMq (5672) and memcached (11211) allow traffic from the tomcat server SG
-	Select the load balancer security group for this as well.




Click on create security group and open the security group again. Because we want to add one more inbound rule All traffic from the same security group

Click on the hyperlink of the security group it will open the security group again
- Click on Edit inbound rules

-	add new inbound rule All traffic and select the same security group ID then click on save.

Step 2
Create key pair
In the EC2 console under Network & security click on Key pairs and again click on  create key pairs.

You can choose .ppk extension if you want to connect the instance using putty otherwise select .pem





Step 3
Create EC2 Instances
Clone the github repo using the below link
- git clone https://github.com/devopshydclub/vprofile-project.git
I have already clone this repo for different project
- Switch branch to aws-LiftAndShift (here is our userdata scripts saved for this project)
To switch branch enter the below command
- git checkout aws-LiftAndShift

- Open each and every files using vscode or sublime editor and go through with the scripts.

Task 1
Launch MySql instance with user-data script
- Name the instance as db-server 
- Select the AMI as CentOS
- Select the key pair we created
-  Under security group select the backend security group
- Add user-data

To get the user-data click on the link below
https://github.com/Fawazcp/aws_project_lift-shift/tree/main/MySQL

 
 
Click on Browse more AMIs  AWS marketplace AMI search for CentOS 7 and select the AMI as you see below;
 
 
 
-	Select the key pair we created.
Under security group click on existing security group and select the backend sg
 
Expand 	advanced details and drop down towards user-data
 
 
Add the userdata mentioned earlier and click Launch instance.
To get user-data click on the link below;
https://github.com/Fawazcp/aws_project_lift-shift/tree/main/MySQL
-	Wait for sometime to get the instance up and running because we have added the user-data to provision the mysql databse.
-	Go to security group of this instance and add port 22 (SSH) from MyIP
 
 
 

To login to the machine using gitbash follow the below steps;
-	Go to the file where you have downloaded the key.
-	ssh  -i Prod_key.pem centos@publicIP of the ec2 instance
 
Now we logged in to our database server in gitbash
-	sudo –i
-	systemctl status mariadb (you may need to wait for sometime 5 to 10 minutes to see the service is active)
 
Now we can see the mariadb service is active
Next we can try to login to the mysql database
To login enter the below command;
-	mysql  -u root –p
Enter the password as admin123
-	show databses;
-	use accounts;
-	show tables;
 
And we can see the database is provisioned as we expected
-	exit; (use this command to exit from the database)



Task 2
Provision the other backend service (memcahed)
-	Create new instance named mc-server
Follow the same steps we used for creating the Mysql server but this time add the below user-data script.
Click on the link to get the user-data
https://github.com/Fawazcp/aws_project_lift-shift/blob/main/Memcached/user-data.sh
Add this user-data and click on launch instance
 
Wait for sometime to get the instance up and running
Meanwhile we can set our next backend server
Task 3
Create rabbitmq server
-	Create an instance and name it rmq-server
-	Follow the same steps as we did for last 2 ec2 instance but make sure that add the below user-data.
Click on the link to get the user-data;
https://github.com/Fawazcp/aws_project_lift-shift/blob/main/rabbitmq/user-data.sh
Add this user-data and click on launch instance.
 
 
Now we have our 3 backend servers are up and running

Task 4
Login to memcached and rabbitmq server.
First we can log in to memcached server.
-	Copy the public IP of mc-server and follow the below steps;
 
The memcached service is active
-	Exit from this server and log in to rabbitmq server to verify everthing is provisioned as we expected.
 
Now we have our rabbitmq is active.
Task 5
Update private of backend server (database, rabbitmq and memcached) in to Route53 private DNS zones.
-	Copy the Private IP of each machines
 
These are private IP of my all three instances. To get this IP refer the below image
 
Select each instance individually then copy the private IP and paste It in notepad or any text editor.
-	Go to route53 and follow the below steps;
 
 
 
Select the region where you created all the ec2 instances. I have created in us-east-1 so selected that region and the vpc I used to create the instance.
 
 
Once you create a hosted by default you can see 2 records has been created.
-	Create new simple routing policy record.
To create follow the below steps;
 
 
Make sure to paste the db-server private IP under value.
Do the same for other 2 instances as well
 
 
 
Now we have domain name rather than a IP address in each instance.


Task 6
Launch the frontend application server (Tomcat)
-	Go to EC2 and click on launch instance and follow the below steps;
  
Make sure to select the same AMI you see in the snapshot.
 
 
Scroll down to the bottom and add the user-data
To get the user-data click on the link below;
https://github.com/Fawazcp/aws_project_lift-shift/blob/main/Tomcat/user-data.sh
 
Add the user-data and click on launch instance
After some time we can see our application server (tomcat) is up and running .
-	Log in to the tomcat server and verify it is provisioned as we expected.
We get some error says connection timeout or something. It is because in the security group we haven’t added SSH port 22.
-	Go to security group and add inboud rule SSH from MY IP.
-	Now try again and we are able to log in.
 
-	Exit from the instance

###################################################################################




Step 4
Build and Deploy Artifacts
Task 1
In order to deploy artifacts we need to install some tools
-	jdk8  (dependency for maven)
-	maven
-	awscli (we use this to upload the artifacts to the s3 bucket)
To install the above tools in windows follow the below steps
Open the windows powershell as administrator nd enter the below command to install the tools
-	choco install jdk8 –y
-	choco install maven –y
-	choco install awscli –y
 

To verfiy the packages installed;
 


Task 2
Go to the repo that we cloned.
 


 
Make the above changes as per the DNS record you created and save the file
 
This is the one we created the record for each server that I mentioned in the application.properties file.
Now this time generate our artifact.

-	Go back to the vprofile-project directory and enter the below command.
-	mvn install 
 
 
Make sure pom.xml file is there and after some time we can see the artifact bulid success
 
And now we can see a new directory named target has been created.

Task 3
Store the vprofile-v2.war file in to our s3 bucket.
We will upload this file to s3 bucket using aws cli. In order to login to aws cli we need to have IAM user credentials.
-	Create IAM User.
To create an IAM user follow the below steps;
 
 
 
 
 
Now the user has been created and we need to create an access key for this user in order to log in to the aws cli
-	Click on the user name Security credentials Create access key.
 
 

 
 
Make sure to download the .csv file at the same time when you create the access key otherwise you can’t download afterwards. 
Also don’t share your access key and creditials to anybody

Task 4
Log in to the aws cli.
Go to gitbash and follow the below steps;
 
Copy and paste the access key and secret key 
-	Enter the region where you have created the EC2 instances.
-	Out format you can leave or enter json.
Now let’s create a new s3 bucket.  Make sure you should give unique bucket name otherwise you will get some error.
To create an s3 bucket using aws cli enter the below command.
 
 
Now we have created a new bucket in the us-east-1 region.
Next we need to copy the artifact file in to s3 bucket from the target directory.
Follow the below steps to copy the file.
 

Now we have copied the artifact to the s3 bucket.
In order to download this artifact in the Tomcat instance we need to create a role.
 
Task 5
Create role
We need to create a role for our tomcat instance in order to download the files from the s3 bucket.
Follow the steps to create a role.
 
 
 
 
 
 

Once the role has been created we need to attach this role in to our app server (tomcat)
-	Go to EC2 instance and follow the below steps.
 
 

Connect the tomcar instance using gitbash
-	sudo -i 
-	apt update -y
-	apt install awscli -y
-	aws s3 ls  s3://aws-project-artifact-storage1 
 
 
 
We are able to access the s3 bucket from the application server (tomcat instance).
Copy the artifact file in to this ec2 instance
-	aws s3 cp s3://aws-project-artifact-storage1/aws-project.war /tmp/aws-project.war 
(copy the file from s3 to /tmp directory)
In order to delpoy this code we need to stop our application (tomcat). To shutdown application enter the below command
-	systemctl stop tomcat9
-	cd /var/lib/tomcat9/webapps/
-	ls
-	rm -rf ROOT
-	cp /tmp/aws-project.war ./ROOT.war  (copying the artifact in the current directory with ROOT.war name file)
-	systemctl start tomcat9 (wait for few minutes)
-	ls
 
 
Validate  the application.properties file that everything is correct as we mentioned especially the hostname.

In order to validate our tomcat instance can connect to other instance we use a software called telnet
Telnet we are using to check the connectivity.
-	apt install telnet –y
 
As we can see the telnet is already installed in the ubuntu machine
If you are not able to connect the instance using telnet then go to backend instance security group and make sure that tomcat secuity group has been selected under source.
 

To close the telnet session use the below commad
-	cntrl+] then type ‘quit’
 
Now we are able to connect to the backend server inside the tomcat instance










Step 5
Load balancer and DNS
Task 1
 Create target group
 
 
 
 
 
 
 

 
 
Task 2
Create Application Load Balancer
 
 
 
 
Under mappings select all the regions available for high availabilty
 
Select the security group we created for the load balancer. Make sure that it is having HTTP and HTTPS inbound rule from anywhere added.
 
 
Add 2 listeners HTTP and HTTPS and select the target group we created.
When we select the HTTPS listener we should add the certificate.
To create a certificate click on this link below
Need to write a blog on creating a certificate.
 
Dropdown and select the certificate. If you cannot see the certificate then check the region you selected and where you have created the certificate.
 
 
Wait for sometime and after sometime we can see our load balancer state is active.
 





Task 3
Create CNAME
Copy the load balancer DNS name.
 
-	Go to route53
-	Create new CNAME record and add paste the load balancer DNS name.(If you registered the domain using godaddy or other platform you need to add the CNAME in that domain name, I have registered a domain using route53)
-	If you have a domain in the route53 then follow the below steps
 
 
Also make sure to add the CNAME where you have already added the certificate.
 
 
 
Copy the record name and paste it in to new browser also add /login in the last
 
Enter the username and password as admin_vp and click on log in
 
Now we have successfully logged in.
First let’s check the rabbitmq
-	Click on the rabbitmq icon
 
-	Go back and click all users to check the memcached
 
Click on any user and we can see data inserted in cache
 
-	Go back and click on the same user and we can see this time it came faster than before because it is coming from the cache
 
Wow and that was a successful attempt to the application instance








Step 5
Autoscaling Group
Task 1
Create an AMI of the application server (tomcat-server)
To create an AMI follow the below steps;
 
 
Creating an image may take some time.
Go to AMI from EC2 console and check the status of the AMI we created.
 
After sometime we can see the status as Available.
 

Task 2
Launch templates
To create a launch templates follow the below steps;
 
 
 
 
 
 
 
Task 2
Create Autosscaling group
To create autoscaling group follow the below steps;
 
 
 
 
Select the same vpc we used to create ec2 instance and select all the subnets are available in the region
 
 
 
 
 
 
 
 
Add notification you can skip if you don’t want to get email notification whenever instance launched, terminated ……..
 
Verify everything and click on create autoscaling group
 
This will create an instance automatically.
If you want to terminate the previous instance then we can because we have launched a new instance using the autoscaling group with all the configuration required.
 
As we a new instance is created.





Step 6
Validate and summerize
Final Task
  
Now we are able to login with HTTPS protocol.
What we have achieved so far is;
-	User access the app through a URL (which is we added in the domain name)
-	That URL points to the Application load balancer endpoint
-	With HTTPS connection we access application load balancer endpoint.
-	For HTTPS we have the certificate in the ACM (Amazon Certificate Manager)
-	Application load balancer is in a security group that allow HTTPS 443 traffic
-	Which will forward the request to application server (Tomcat ec2 instances) on port 8080 which is in another security group
-	For the backend servers that will access with record name we added in the route53
-	Backend server private IP is mapped in the private DNS zone
-	All the backend server are in the one security group
Whenever we want we can upload the artifact in to s3 bucket and download it in the application instances.

This is not a right way to deploy artifact. The right way to deploy artifact is using CI CD pipeline that we will learn later.

Clean UP
-	Delete Autoscaling Group
-	Delete application load balancer
-	Delete target group
-	Delete launch template
-	Delete all the intances 
-	Delete AMI
-	Delete the security groups
-	Delete the DNS records
