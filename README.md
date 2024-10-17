# Simple end-to-end CI/CD Pipeline on AWS

## 🌍 Overview

Simulating an end-to-end CI/CD Pipeline for a Java web app using AWS Deployment Tools, using CodeArtifact to cache dependencies, CodeBuild to run the build servers, CodeDeploy to deploy to production, CodePipeline to automate these, and CloudFormation to set-up the resource stacks. The journey is documented in the pdfs through parts 1 to 7.

## 🏗️ Architecture

![diagram](awsProject2.drawio.png)

## 🧱 Setup Instructions

### Step 1: Set-up project using Maven in EC2 instance

1. Create a new EC2 instance and install maven

```
wget https://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz

sudo tar -xzf apache-maven-3.5.2-bin.tar.gz -C /opt

echo "export PATH=/opt/apache-maven-3.5.2/bin:$PATH" >> ~/.bashrc

source ~/.bashrc
```

2. Install Java 8

```
sudo dnf install -y java-1.8.0-amazon-corretto-devel

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64

export PATH=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre/bin/:$PATH

```

3. Generate webapp template

```
mvn archetype:generate \
   -DgroupId=com.nextwork.app \
   -DartifactId=nextwork-web-project \
   -DarchetypeArtifactId=maven-archetype-webapp \
   -DinteractiveMode=false
```

### Step 2A: Upload template to CloudFormation

Use this step if you want to use the CloudFormation template directly. Otherwise, do Step 2B to set things up manually for the first time.

1. Create Build and Deploy stack using `CICDpipeline-NextWorkWebAppSetup-template.yaml`
2. Configure CodeArtifact connection using connection instructions from AWS console (create `settings.xml`) and compile project
3. Set CodeBuild's source as your CodeCommit/Github repo

### Step 2B: Set-up CodeCommit, CodeArtifact, CodeBuild

1. Create CodeCommit/Github repo, add url to project folder, and push
2. Configure CodeArtifact connection using connection instructions from AWS console (create `settings.xml`) and compile project
3. Set-up IAM Policy for accessing CodeArtifact
4. Set-up S3 bucket for storing build artifact
5. Create CodeBuild project and add above CodeArtifact IAM policy to CodeBuild service role

### Step 3: Upload deploy server template to CloudFormation

1. Create Deployment server with a VPC stack using `DeployServer-EC2-VPC-template.yaml`

### Step 4: Create configuration files

1. Upload Appspec.yml
2. Upload Bash scripts for the Deployment server
3. Upload full Buildspec.yml (includes `appspec.yml` and `scripts/**/*` folder in build artifacts)

### Step 5: Set-up CodeDeploy

1. Create CodeDeploy service role to allow access to EC2 deployment server, with `AWSCodeDeployRole` permission
2. Create CodeDeploy application and deployment group
3. Create a Deployment

### Step 6: Set-up CodePipeline

1. Follow the prompts to set-up a new pipeline
2. Commit new code to repo, and watch the pipeline trigger itself automatically
3. Click "rollback" on deployment stage to rollback to a previous version, or "release changes" to go to ship the most updated version

## 🛠️ Configuration Details

## 🍽️ Usage Instructions

## 🚨 Troubleshooting

### Common Issues

- **Issue 1:** Unable to compile and publish dependencies to CodeArtifact
  - **Solution:** If necessary, start from a fresh instance or uninstall Maven first. Switch your Java runtime and compiler to Java 8 (openjdk 1.8) with `sudo alternatives --config java` and `sudo alternatives --config javac` before installing Maven, and ensure that you get at least Maven version 3.2.5 or above. Recompile again with `mvn -s settings.xml compile`.
- **Issue 2:** Unable to create stack on CloudFormation due to	 circular reference
  - **Solution:** resolve dependencies by removing the Managed Policies in the CodeBuild IAM Service Role entry
