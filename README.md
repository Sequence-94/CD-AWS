
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


















