# Jenkins Declarative Pipeline Setup Guide for Windows

This guide walks you through setting up Jenkins and creating a declarative pipeline that integrates with GitHub.

## Prerequisites

- Java (JDK 11 or 17)
  - Verify installation: `java -version`
  - If not installed, download from [Adoptium](https://adoptium.net/)

## Jenkins Installation

1. **Download Jenkins**
   - Visit [Jenkins Download Page](https://www.jenkins.io/download/)
   - Navigate to Windows section
   - Download the .msi installer

2. **Installation Process**
   - Run the .msi installer
   - Follow the setup wizard
   - Jenkins will be installed as a Windows service by default

3. **Initial Jenkins Setup**
   - Open your browser and navigate to: `http://localhost:8080`
   - Locate the initial admin password:
     ```
     C:\Program Files\Jenkins\secrets\initialAdminPassword
     ```
   - Choose "Install Suggested Plugins"
   - Create your admin user credentials

## Setting up GitHub Integration

### Step 1: Create GitHub Repository with Jenkinsfile

1. Create a new GitHub repository (e.g., Jenkins-Declarative-Pipeline)
2. Clone and set up locally:
   ```bash
   mkdir Jenkins-Declarative-Pipeline
   cd Jenkins-Declarative-Pipeine
   git init
   ```
3. Create a Jenkinsfile in the root directory with the following content:
   ```groovy
   pipeline {
       agent any
       
       stages {
           stage('Checkout') {
               steps {
                   // This will checkout the code from the configured SCM
                   checkout scm
               }
           }
           
           stage('Build') {
               steps {
                   echo 'Building...'
                   // Add your build steps here
               }
           }
           
           stage('Test') {
               steps {
                   echo 'Testing...'
                   // Add your test steps here
               }
           }
           
           stage('Deploy') {
               steps {
                   echo 'Deploying...'
                   // Add your deployment steps here
               }
           }
       }
       
       post {
           always {
               echo 'Pipeline completed'
               // Add post-build actions here
           }
           success {
               echo 'Pipeline succeeded!'
           }
           failure {
               echo 'Pipeline failed!'
           }
       }
   }
   ```
4. Push to GitHub:
   ```bash
   git add .
   git commit -m "Add declarative Jenkinsfile"
   git remote add origin https://github.com/YOUR_USERNAME/Jenkins-Declarative-Pipeline.git
   git push -u origin master
   ```

### Step 2: Configure GitHub Credentials in Jenkins

#### For HTTPS:
1. Navigate to Jenkins → Manage Jenkins → Credentials → Global → Add Credentials
2. Configure:
   - Kind: Username with password
   - Username: Your GitHub username
   - Password: GitHub Personal Access Token
   - ID: github-token

#### For SSH:
1. Navigate to Jenkins → Manage Jenkins → Credentials → Global → Add Credentials
2. Configure:
   - Kind: SSH Username with Private Key
   - Username: git
   - Private key: Your SSH private key
   - ID: github-ssh

### Step 3: Create Jenkins Pipeline

1. From Jenkins Dashboard, click "New Item"
2. Enter name: "Declarative-GitHub-Pipeline"
3. Select "Pipeline" and click OK
4. Configure Pipeline:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: 
     - HTTPS: `https://github.com/YOUR_USERNAME/Jenkins-Declarative-Pipeline.git`
     - SSH: `git@github.com:YOUR_USERNAME/Jenkins-Declarative-Pipeline.git`
   - Credentials: Select appropriate credentials
   - Branch: */master
5. Save configuration

### Step 4: GitHub Webhook Setup (Automated Triggers)

1. In GitHub:
   - Go to repository Settings → Webhooks
   - Click "Add webhook"
   - Payload URL: `http://YOUR_JENKINS_SERVER/github-webhook/`
   - Content type: application/json
   - Select: Just the push event
   - Save webhook

2. In Jenkins Pipeline:
   - Configure job
   - Under Build Triggers
   - Enable "GitHub hook trigger for GITScm polling"
   - Save

## Usage

- Manual Build: Click "Build Now" in Jenkins pipeline
- Automatic Build: Push changes to GitHub repository

## Troubleshooting

- Ensure Jenkins service is running
- Check webhook payload URL is accessible
- Verify GitHub credentials are correct
- Review Jenkins logs for any error messages

## Additional Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [GitHub Webhook Documentation](https://docs.github.com/en/webhooks) 