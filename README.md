## Continuous Deployment to Kubernetes using AWS CodeCommit, AWS CodeBuild, AWS CodePipeline and AWS ECR.

#### Creating a CI/CD pipeline on AWS to deploy a simple application on Kubernetes. 

The following content will walk you through the steps to deploy a simple application when developers commit the source code.

CodeCommit is AWS managed Git service. It will automatically encrypt your files in transit and at-rest. 

CodeBuild is AWS managed build service. It is referred as a Continuous Integration tool. It will get the source code from the source provider and run them in a container for us. 

CodePipeline is AWS managed deployment pipeline service. It is a CD pipeline orchestrator. We don't need to give scripts to run instead give it as a sequence of actions, which are links to other services.

AWS ECR is AWS managed docker registry service. We can use the docker CLIs to push, pull and manage images. It will transfer the container images over HTTPs and automatically encrypt the images at-rest. 

This is a basic CI/CD flow to demonstrate how to deploy a simple application when developers commit the source code.

1.	The developer will commit code on CodeCommit/Github.
2.	The CodePipeline will poll source code when there is any change.
3.	The CodeBuild will build a docker image and push the image to ECR.

### Who should use this?

This will be helpful for developer, Ops or DevOps person to migrate their application, apply the changes in code with minimal manual work. 

**Key benefits:**

-	 Easily deploy the application.
-	 Codecommit automatically encrypts all the files in transit and at-rest. 
-	 Provide private git repositories for developers to use. 

### Pre-requisites

-	Client Server - Ubuntu OS. EC2 instance used as a client server for codecommit updates.
-	Required Dockerfiles and build specification files for base image and application image setup. Copy or clone the files from github repository. 
-	CodeCommit Repositories to upload the source codes. 
-	IAM service roles for AWS CodeBuild and AWS CodePipeline.
-	Amazon ECR Repositories to upload the new images.

#### CodeCommit Repositories

Create new repositories in AWS CodeCommit to upload the source codes for both base image and application image setup. 

Clone/download the github repository files to your local root directory. 

In this example, we use simple Dockerfile to build the base-image and application-image. Update the base-image URI in application image's dockerfile.

    FROM 0123456789017.dkr.ecr.us-east-1.amazonaws.com/base-image 
    ENV PORT=80
    EXPOSE $PORT
    COPY app.js /app/
    CMD ["node", "/app/app.js"]

#### IAM Service Roles

AWS CodeBuild and AWS CodePipeline will be used to build the docker images and push them to Amazon ECR. It requires IAM service roles to make calls to Amazon ECR API operations.

Follow the below steps to create CodeBuild Service role,

	1. Create new role as CodeBuildServiceRole
	2. Select CodeBuild service
	3. Select CodeBuildServiceRolePolicy and Amazon EC2ContainerRegistryPowerUser policies
	4. Click on Create Role to complete. 

#### Build Specification Files Setup

Create a build specification file for base image and upload it to source code repository to build the base image using CodeBuild.

buildspect.yml: This file is used for building docker image by CodeBuild.

Follow the below steps to repository in Amazon ECR and push buildspec.yml file. 

	1. Create new repository in Amazon ECR
	2. Replace the REPOSITORY_URI value with newly created Amazon ECR repository URI. 
	3. Replace base-image with the name of your docker image. 
	4. Commit and push the buildspec.yml file to source repository using below commands,
        	git add . 
        	git commit -m “Adding Build Specification”
        	git push

Create a build specification file for application and upload it to source code repository to build the application image using CodeBuild. 

Follow the below steps to repository in Amazon ECR and push buildspec.yml file. 

	1. Create new repository in Amazon ECR
	2. Replace the REPOSITORY_URI value with newly created Amazon ECR repository URI. 
	3. Replace base-image with the name of your docker image. 
	4. Commit and push the buildspec.yml file to source repository. 
        	git add . 
        	git commit -m “Adding Build Specification”
        	git push

#### Continuous deployment pipeline for Base Image

Follow the below steps to create CodePipeline in AWS Console,

	1. Navigate to AWS CodePipeline service.
	2. Click on Create Pipeline.
	3. Type the name for your pipeline and choose Next Step. 
	4. On the Source page, select **AWS CodeCommit ** as the source provider. 
		a. For Repository name, choose a repository from the dropdown, where you have pushed source code for base image. 
		b. For Branch name, select Master and then choose Next Step. 
	5. On the Build page, choose **AWS CodeBuild** as a Build Provider and click on Create Project. 
	6. Choose Continue to CodePipeline and Next.
	7. On the Deploy page, choose to skip and acknowledge the pop-up warning.
		
| Parameter  |   Value|
| :------------: | :------------: |
| Project Name  | base-image   |
| Operating System  | Ubuntu  |
| Runtime | Standard |
| Image | aws/codebuild/standard:1.0 |
| Image version | Always use the latest image for this runtime version |
| Privileged flag | Enabled |
| Service role | CodeBuildServiceRole | 

 For Service role, choose Existing service role, choose the CodeBuild service role you have created earlier and then enable the Allow AWS CodeBuild to modify this service role so it can be used with this build project.
 		

#### Continuous deployment pipeline for Application Image

Follow the below steps to create CodePipeline in AWS Console,

	1. Navigate to AWS CodePipeline service.
	2. Click on Create Pipeline.
	3. Type the name for your pipeline and choose Next Step. 
	4. On the Source page, select **AWS CodeCommit ** as the source provider. 
		a. For Repository name, choose a repository from the dropdown, where you have pushed source code for application image. 
		b. For Branch name, select Master and then choose Next Step. 
	5. On the Build page, choose **AWS CodeBuild** as a Build Provider and click on Create Project. 
	6. Choose Continue to CodePipeline and Next.
	7. On the Deploy page, choose to skip and acknowledge the pop-up warning.
		
| Parameter  |   Value|
| :------------: | :------------: |
| Project Name  | base-image   |
| Operating System  | Ubuntu  |
| Runtime | Standard |
| Image | aws/codebuild/standard:1.0 |
| Image version | Always use the latest image for this runtime version |
| Privileged flag | Enabled |
| Service role | CodeBuildServiceRole | 

 For Service role, choose Existing service role, choose the CodeBuild service role you have created earlier and then enable the Allow AWS CodeBuild to modify this service role so it can be used with this build project.
 		
This pipeline will fail, because it is missing the application source code. 

Now, edit the application pipeline to add an additional source stage.

	1. Navigate to AWS CodePipeline and select application pipeline. 
	2. On the pipeline page, choose Edit.
	3. On the Editing: application-image page, in Edit Source, choose Edit stage.
	4. Choose the existing source action and click on edit icon to update the Output artifacts.
	5. Update the Output artifacts as BaseImage and then Save. 
	6. Add action, and then enter a name for the action. (Eg. application)
		a. For Action provider, choose **AWS CodeCommit**. 
		b. For Repository name, choose the repository where you have already pushed application source code. 
		c. For Branch name, choose the branch. (Eg. Master) 
		d. For Output artifacts, specify SourceArtifact, and then choose Save.
	7. Click on Save to update the changes.
	

#### Kubernetes cluster setup on EC2
Kubernetes is an open source container orchestration and management tool. Follow the below steps to create Kubernetes cluster in AWS (EC2) using kops and install the application. 

Be sure to have AWS CLI, Kops and Kubectl installed on Ubuntu OS client server.

Execute the below commands from ubuntu OS client server to deploy 2 node Kubernetes cluster. 

	 `$ export KOPS_STATE_STORE=s3://bucket-name `
	 `$ kops create cluster --name=code-build.k8s.local --state=s3://bucket-name --zones=us-east-1a --vpc vpc-xxxxx --network-cidr=172.x.x.x/16 --subnets subnet-xxxxxx --topology public --associate-public-ip=true --networking flannel-vxlan --node-count=2 --master-size t2.micro --node-size t2.micro `

After few minutes, ensure the cluster is up and running,

`$ kops validate cluster --name <cluster-name> `

Verify kubernetes cluster nodes state

`$ kubectl get nodes `

Now, execute the deployment scripts from application-deployment folder to install the application on Kubernetes cluster,

`$ kubectl create -f my-dep.yaml `
	
`$ kubectl create -f my-dep-svc.yaml `

#### Conclusion
In this article, we showed how to create a complete, end-to-end continuous deployment (CD) pipeline with Amazon ECR and AWS CodePipeline.

