<h1 align="center">Deploy Application to an EC2 Instance<h1> 


# Deployment 4
October 2, 2023

By: Andrew Mullen

## Purpose:

Demonstrate the ability to deploy an application to an EC2 instance.

## Steps:

### 1. Clone, branch, and make the update to the Jenkinsfile, and merge back into main, before you start your Jenkins build!!!
- This process is to give us practice using Git and experience in the day-to-day operations of a DevOps engineer.
- During certain steps in the process, there will be subsections where git commands will need to be used to complete step 1.
   	- Create a new repository on GitHub
	- Use git commands to clone the Kura Deployment 4 repository to the local instance and push it to the new repository
	- Branch, update, and merge Jenkinsfile into the main branch

	
### 2. Create a T.2 medium in your public subnet:
- We are using a larger EC2 instance as we will host both our Jenkins server and webserver on the same machine.
- But first, we need to do the following
	- Create a VPC with the following attributes
		- 2 availability zones
		- 2 Private and Public subnets
	- Create the EC2 instance with the following network settings
		- Created VPC
		- Select the Public Subnet
		- Auto-assign Public IP
		- Create a new security group
		  - 80, 8080, 8000, 22, and 5000
		  
#### 2a. Clone the Kura repository to our local instance and push it to the new repository
	- Clone the Kura Deployment 4 repository to the local instance
		- Clone using `git clone` command and the URL of the repository
			- This will copy the files to the local instance 
		- Enter the following to gain access to GitHub repository
			- `git config --global user.name username`
			- `git config --global user.email email@address`
		- Next, you will push the files from the local instance to the new repository (Done from the local instance via the command line)
			- `git push`
			- enter GitHub username
			- enter personal token (GitHub requires this as it is more secure)
			
#### 2b. Branch, update, and merge Jenkinsfile into the main branch
	- Create a new branch in your repository
		- `git branch newbranchName`
	- Switch to the new branch and modify the Jenkinsfile
		- `git switch newbranchName`
		- Update Jenkinsfile with the script below
		```
			pipeline {
			agent any
			stages {
			stage ('Build') {
			steps {
			sh '''#!/bin/bash
			python3 -m venv test3
			source test3/bin/activate
			pip install pip --upgrade
			pip install -r requirements.txt
			export FLASK_APP=application
			'''
			}
			}
			stage ('test') {
			steps {
			sh '''#!/bin/bash
			source test3/bin/activate
			py.test --verbose --junit-xml test-reports/results.xml
			'''
			}
			post{
			always {
			junit 'test-reports/results.xml'
			}
			}
			}
			stage ('Clean') {
			steps {
			sh '''#!/bin/bash
			if [[ $(ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2) != 0 ]]
			then
			ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
			kill $(cat pid.txt)
			exit 0
			fi
			'''
			}
			}
			stage ('Deploy') {
			steps {
			keepRunning {
			sh '''#!/bin/bash
			pip install -r requirements.txt
			pip install gunicorn
			python3 -m gunicorn -w 4 application:app -b 0.0.0.0 --daemon
			'''
			}
			}
			}
			}
			}
		```
	- After modifying the Jenkinsfile commit the changes
		- `git add "filename"`
		- `git commit -m "message"`
	- Merge the changes into the main branch
		- `git switch main`
		- `git merge second main`
	- Push the updates to your repository
		- `git push`
		
### 3. Install Jenkins and install the following on the T.2 medium:
   - Install "python3.10-venv" - Needed to set up the Build Stage in Jenkins
   - Install "python3-pip" - Needed to set up the Build Stage in Jenkins
   - Install "nginx" - Needed to run the web service
   - Install the plugin "Pipeline Keep Running Step"

### 4. Edit the Nginx configuration file
	- Modify the following
		- Change the port from 80 to 5000 under server
		- Change the settings under location

### 5. Configure Cloudwatch agent
- Cloudwatch is an AWS service that collects and visualizes real-time logs, metrics, and event data. It can create alerts based on CPU, Memory, and other metrics.
	- Create Cloudwatch Agent on EC2
	- Create alerts on Memory and CPU 

### 6. Create a multibranch pipeline and run the build for the application

- Jenkins is the main tool used in this deployment for pulling the program from the GitHub repository, and then building and testing the files to be deployed to Elastic Beanstalk.
- Creating a multibranch pipeline gives the ability to implement different Jenkinsfiles for different branches of the same project.
- A Jenkinsfile is used by Jenkins to list out the steps to be taken in the deployment pipeline.

- Steps in the Jenkinsfile are as follows:
  - Build
    - The environment is built to see if the application can run.
  - Test
    - Unit test is performed to test specific functions in the application
  - Clean
    - Checks to see if gunicorn is running and kills the process
  - Deploy
    - Installs requirements and gunicorn web application 	


### 7. How is the server performing?

- The server is running well at times.  However, I do notice that memory is elevated and runs at a constant 20% of 4GB.

### 8. Can the server handle everything installed on it? if yes, how would a T.2 micro handle in this deployment? 

- Yes, the server can handle everything installed.  However, a T.2 micro would run into issues with memory as it only has 1GB of memory, and the baseline for everything running is about 1GB.

### 9. What happens to the CPU when you run another build?

- There is a slight uptick in CPU but no more than about 15% when running a build.


## System Diagram:

To view the diagram of the system design/deployment pipeline, click [HERE](https://github.com/andmulLABS01/Deployment_4AM/blob/main/Deployment_4.drawio.png)

## Issues/Troubleshooting:

#### 1. Could not edit nginx default file

Resolution Steps:
- ran sudo nano default to edit the file.


#### 2. Could not select created Security Group when creating VPC

Resolution Steps:
- Created a new security group when creating VPC.



## Conclusion:

This deployment can be improved. One improvement would be to automate the creation of the following:
- AWS instance
- Jenkins server
- Creating GitHub repository

Some of the automation would include writing Bash scripts for the installation process. We may also want to start thinking about how we will handle incident response when an alert is triggered.
