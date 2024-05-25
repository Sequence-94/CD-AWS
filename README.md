
![Continous delivery on AWS Cloud(Java App)](https://github.com/Sequence-94/CD-AWS/assets/53806574/eaaf8798-40e4-48a5-90fd-777feacdecfa)



# SCENARIO:

We have a Product Development Team working in an agile environment
Regular changes are made to the code
These commits need to be Buit & Tested
The team does not have System Admins or Cloud Engineers.
Hence running short on Operations(such as the Deployment team).
For Package/Software/Artifact Deployment on the server

Regular Code Changes => Regular Packaging of Software => Regular Deployment of Servers

Additionally, we need Software Testing/Integration Testing after this deployment. 


# PROBLEM:

In an Agile Software Development Life Cycle, there will be frequent code changes, so Manual code deployment is time-consuming.
Additionally, Developers and Testers are not equipped with Ops Knowledge.

We therefore need to Hire Ops Professionals or Outsource but then there is a Dependency on Ops Team for
CI/CD server Maintenance
Creating Operational Overhead to maintain servers like Jenkins,Nexus,Sonar,GIT, QA Server for testing.


# SOLUTION:

Instead of Ops Team we can use Platform As A Service(Like Elastic Beanstalk) and Software As A Service.
In AWS we can create a Disposable Environment
We can Automate CI/CD Process to these disposable environments once it is tested and Delete so we need to manage them.
Now we can Build,Test, Deploy & Test For Every Commit.

# BENEFITS(CD PIPELINE):

Short MTTR - can repair issues quickly,low downtime
Complements Agile Process - soon as code is changed, the process runs and gives us the result.
Automated - little to no human intervention
Fault Isolation - we can easily decipher is the problem was made in the src code, build process, deployment process or testing
No Ops team needed


# AWS SERVICES:

CODE COMMIT - version control system
CODE ARTIFACT - maven repository for dependencies
CODE BUILD - run sonar scanner and fetch dependencies -> build our artifact + sonar scanner to run code analysis and fetch dependencies.
CODE DEPLOY - we can deploy our artifact to Beanstalk
BEANSTALK - for hosting application
RDS - database for application (paas)
CODEPIPELINE - integration service (paas)



# EXTERNAL SERVICES:

SONARCLOUD - code analysis sonarqube server on the cloud.
CHECKSTYLE - code analysis in CodeBuild 
SELENIUM - software testing in CodeBuild



NO/LESS Ops Overhead
Short MTTR
Fast turnaround on feature changes
Less disruptive


# FLOW OF EXECUTION:

# CI

Login to AWS account
Code Commit
	Create codecommit repo
	sync with local repo
Code Artifact
	create repo
	update settings.xml and pom.xml with repo details
	generate token and store in SSM Parameter Store

Sonar Setup
	Create sonar cloud account
	Generate token and store in ssm parameter store
	create build project
	update codebuild role to access SSM parameter store
Create SNS
Build Project
	Create variables in SSM => parameterstore
	Create build project
Create Pipeline
	Codecommit
	Testcode
	Build
	Deploy to S3 bucket

# CD

Test Pipeline
Create Beanstalk(deploy artifact) & RDS(database)
Update RDS security group so that beanstalk can access it
Deploy DB in RDS
Switch to cd-aws branch
Update settings.xml and pom.xml
Create another build job to create artifact with buildspec file in cd-aws
Create a deploy job to beanstalk
Create a build job for software testing
Upload screenshot to s3 bucket
Update Pipeline
	Codecommit
	Testcode
	Build & Store
	Deploy to s3 bucket
	Build & Release
	Deploy to Beanstalk
	Build Job for Selenium test scripts
	Upload result to s3
Test Pipeline


	BeanStalk Setup: Creating this is fairly easy on AWS console, click create application , I chose a name and picked Tomcat as my webserver. Eveything else it default.
 	Instance Type - Load Balanced , t2.micro
  	Min instances =2
   	Max instances =4
	Created key-pair
 	Created a role for beanstalk as well

  
![Screen Shot 2024-05-24 at 08 11](https://github.com/Sequence-94/CD-AWS/assets/53806574/59d58a0a-8f2d-4e6c-be1a-3f7f3a39fc96)

![Screen Shot 2024-05-24 at 08 11 - 2](https://github.com/Sequence-94/CD-AWS/assets/53806574/68696d6a-d6f9-4f6c-99f2-926ac5354dd3)

![Screen Shot 2024-05-24 at 08 11 - 3](https://github.com/Sequence-94/CD-AWS/assets/53806574/50ac459a-480e-4900-af98-e7feeb6e4b4f)

![Screen Shot 2024-05-24 at 08 11 - 4](https://github.com/Sequence-94/CD-AWS/assets/53806574/b5625eff-da50-4115-af4c-ce1309619bd4)

	RDS Setup: 
 	Standard Create
	Engine = MySQL
 	V5.7.44
  	Free tier template
   	DB name = "vprofile-cicd-mysql"
    	Auto generate password
     	Create new security group
      	Initial databse name = "accounts"

       Rest is defaults.
![Screen Shot 2024-05-24 at 08 48](https://github.com/Sequence-94/CD-AWS/assets/53806574/9b7017d2-6a44-435a-89cb-a2ac5ce6fdf5)

ERROR:
![Screen Shot 2024-05-24 at 08 58](https://github.com/Sequence-94/CD-AWS/assets/53806574/f8105544-b2e9-45bd-951a-876ebc1ae203)

After doing a quick google search it came to my attention that I should recreate the inbound rule from scratch by deleteing the default one I was editting:
![Screen Shot 2024-05-24 at 09 00](https://github.com/Sequence-94/CD-AWS/assets/53806574/892c40c0-c291-418f-a2b1-a448352aa239)
I was basically updating the inbound rule for my database to allow access to the ec2 instances created by elastic beanstalk.


	Now I will SSH into my ec2's and from there I'll login to the rds and perform a test so I can deploy the schemas.
 	First I will login as Root
  	Second I will install mysql client as well as git to clone source code where I have the SQL file

    	ERROR:
     	unable to install mysql client 
![Screen Shot 2024-05-24 at 09 20](https://github.com/Sequence-94/CD-AWS/assets/53806574/2e2ebbc5-db5a-455d-bdc0-2abd375b2db5)

      	Solution:
       	MYSQL was not in my repository so I had to download it and install it from there.
	
![Screen Shot 2024-05-24 at 09 22](https://github.com/Sequence-94/CD-AWS/assets/53806574/92a1455f-5f85-4250-88cb-a2a95f0b642e)

	Then after i was able to run "yum install mysql-community-client"
![Screen Shot 2024-05-24 at 09 55](https://github.com/Sequence-94/CD-AWS/assets/53806574/20a76409-020b-42b3-841f-f782b7946aef)



	So since I am able to login and execute sql commands I have a script that will create my schemas for me in github so I will 
 	First clone the github repo into the ec2 and use the files inside to execute the commands to create the database tables:
  	
![Screen Shot 2024-05-24 at 10 03](https://github.com/Sequence-94/CD-AWS/assets/53806574/7ae6b25b-a0a5-457d-a2ff-756cbe750e6f)


	Now I update the pom.xml & settings.xml file with the repository path and mirror paths.
 	

	Then I create a new Build Job this is specifically for the Build&Release stage.
 	This stage will changed the default password and username of the database to our own from secrets
  	It will download jdk8 with maven and all the dependencies using the buildspec file shown below:
![Screen Shot 2024-05-24 at 13 11](https://github.com/Sequence-94/CD-AWS/assets/53806574/20d7d15a-3824-43e3-bb41-bd9a13245eb4)
	

	I need to update my SSM Parameter store with maven-central-store token

	ERROR:
 	My VPROfILE-CODE-ADMIM user was not allowed the codeartifact:GetAuthorizationToken action and sts:GetServiceBearerToken actions so I had to update the policy to retrive the token and 	put it inside my secrets.
![Screen Shot 2024-05-24 at 13 25](https://github.com/Sequence-94/CD-AWS/assets/53806574/17568116-30d0-4f91-8312-76335ef3e1ca)


	After updating my parameter store with the required secrets it looks like this:

![Screen Shot 2024-05-24 at 13 37](https://github.com/Sequence-94/CD-AWS/assets/53806574/c74c9115-1ce1-40d1-8867-48edf29decf4)

	Now my buildspec file can use these as variables.

 ERROR:
 ![Screen Shot 2024-05-24 at 14 27](https://github.com/Sequence-94/CD-AWS/assets/53806574/7654d12e-5081-4e53-895f-83c6b8fc2eac)

This error seems to suggest that I am not authorised to read from codeartifacts so I looked into my Authentication token that I read from the cli and it
seems there is something that goes wrong when I copy and paste into SSM parameter store so instead of exporting and reading the token from cli I did it via the code: like this
![Screen Shot 2024-05-24 at 15 26](https://github.com/Sequence-94/CD-AWS/assets/53806574/8a2623f0-346e-4ce6-a1fc-e08449dec176)

This resolved the issue:
![Screen Shot 2024-05-24 at 15 27](https://github.com/Sequence-94/CD-AWS/assets/53806574/af211787-665e-4f2c-822f-73d22d81b021)


Now I will create another Build Job for testing using Selenium from windows.
It will take screenshots and upload them into s3 bucket. These are screenshots confirming that application works.
So it will login for me into the app and take screenshots confirming not only the build pipeline was successful but the app actually works and is deployed.

# Running the pipeline

ERROR:
![Screen Shot 2024-05-25 at 12 28](https://github.com/Sequence-94/CD-AWS/assets/53806574/6592d71c-3be9-495d-bf0b-81a4ac1c22c7)
I got a 404 error because my target group was unhealthy - It seems beanstalk could not find the "Health check path" or couldn't validate that one exists as well.
To resolve the issue I had to downgrade my Tomcat from 10 to 8.5 because that's the one with Corretto8 and that's the jdk version I ran with all my other build projects.
![Screen Shot 2024-05-25 at 12 32](https://github.com/Sequence-94/CD-AWS/assets/53806574/c2b7bf9e-ca4a-4298-92c1-71d2dab49375)


![Screen Shot 2024-05-25 at 12 34](https://github.com/Sequence-94/CD-AWS/assets/53806574/3da45c7b-aabf-40dc-b010-7eec3f793c03)

ERROR:
At first I couldn't login because I needed to enable stickysessions 
![Screen Shot 2024-05-25 at 12 34 - 2](https://github.com/Sequence-94/CD-AWS/assets/53806574/85f2a202-6942-4669-9933-dfba1bbdaa5c)

SONARCLOUD RESULT:
![Screen Shot 2024-05-25 at 12 58](https://github.com/Sequence-94/CD-AWS/assets/53806574/3b3b0054-53d0-4b41-b097-d7fdd4df1750)


Selenium Test also took a screenshot and saved it into my s3 bucket:
![Screen Shot 2024-05-25 at 12 37](https://github.com/Sequence-94/CD-AWS/assets/53806574/3d4f12c7-b53a-457b-821d-0cdfc5e40c06)

After downloading the object to my local computer i am able to view the screenshots confirming that everything worked:

![Screen Shot 2024-05-25 at 12 38](https://github.com/Sequence-94/CD-AWS/assets/53806574/0b0a2187-ac84-4b9c-a144-e3570d8fad42)

















