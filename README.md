# Jenkins Declarative Pipeline Setup Guide for Windows

This comprehensive guide walks you through setting up Jenkins with GitHub integration using a declarative pipeline approach. We'll cover everything from initial installation to automated webhook triggers.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Jenkins Installation](#jenkins-installation)
3. [ngrok Setup](#setting-up-ngrok-for-webhook-access)
4. [GitHub Integration](#setting-up-github-integration)
5. [Pipeline Configuration](#pipeline-configuration)
6. [Usage Guide](#usage)
7. [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software
- Java (JDK 11 or 17)
  - Verify your Java installation:
    ```bash
    java -version
    ```
  - If not installed, download from [Adoptium](https://adoptium.net/)
- Git (for version control)
- Chocolatey (for ngrok installation)

### System Requirements
- Windows 10 or later
- Minimum 2GB RAM (4GB recommended)
- At least 10GB free disk space

## Jenkins Installation

### Step 1: Download Jenkins
1. Visit the [Jenkins Download Page](https://www.jenkins.io/download/)
2. Navigate to the Windows section
3. Download the Windows installer (.msi)

### Step 2: Install Jenkins
1. Run the downloaded .msi installer
2. Follow the setup wizard
   - Accept the default installation directory
   - Allow Jenkins to run as a Windows service
   - Note the default port (8080)

### Step 3: Initial Jenkins Setup
1. Open your browser and navigate to:
   ```
   http://localhost:8080
   ```
2. Locate the initial admin password:
   - Open this file:
     ```
     C:\Program Files\Jenkins\secrets\initialAdminPassword
     ```
   - Copy the password
3. Install suggested plugins when prompted
4. Create your admin user
   - Fill in required details (username, password, email)
   - Save and continue

## Setting up ngrok for Webhook Access

### Step 1: Install ngrok
1. Open PowerShell as Administrator
2. Install ngrok using Chocolatey:
   ```bash
   choco install ngrok
   ```
3. Accept all prompts during installation

### Step 2: Configure ngrok
1. Create an account at [ngrok Dashboard](https://dashboard.ngrok.com/signup)
2. After logging in, locate your AuthToken in the dashboard
3. Configure ngrok with your token:
   ```bash
   ngrok config add-authtoken <your_token_here>
   ```
   - Replace `<your_token_here>` with your actual token
   - The command should show a success message

### Step 3: Expose Jenkins
1. Start ngrok to create a tunnel to Jenkins:
   ```bash
   ngrok http 8080
   ```
2. Look for the forwarding URL in the output:
   ```
   Forwarding    https://abc123.ngrok.io -> http://localhost:8080
   ```
3. Important notes:
   - Keep this terminal window open
   - The URL changes each time you restart ngrok
   - Save the URL for GitHub webhook configuration

<div align="center">
  <img src="/images/Screenshot 2025-04-05 024023.png">
</div>

## Setting up GitHub Integration

### Step 1: Create GitHub Repository
1. Create a new repository on GitHub
   - Name: `Jenkins-Declarative-Pipeline`
   - Initialize with a README (optional)
2. Clone the repository locally:
   ```bash
   mkdir Jenkins-Declarative-Pipeline
   cd Jenkins-Declarative-Pipeline
   git init
   ```

### Step 2: Configure GitHub Access
1. Generate Personal Access Token:
   - Go to GitHub → Settings → Developer settings → Personal access tokens
   - Click "Generate new token"
   - Select permissions:
     - `repo` (full control)
     - `workflow`
     - `read:org`
   - Generate and immediately copy the token
   - ⚠️ Store this token securely - it won't be shown again

<div align="center">
  <img src="/images/Screenshot 2025-04-05 021823.png">
</div>

2. Add Token to Jenkins:
   - Navigate to Jenkins → Manage Jenkins → Credentials → Global
   - Click "Add Credentials"
   - Configure as follows:
     - Kind: Username with password
     - Scope: Global
     - Username: Your GitHub username
     - Password: The GitHub token you just generated
     - ID: github-token
     - Description: GitHub Personal Access Token
   - Click "Create"

<div align="center">
  <img src="/images/Screenshot 2025-04-05 022037.png">
</div>
<div align="center">
  <img src="/images/Screenshot 2025-04-05 022055.png">
</div>

### Step 3: Create Jenkinsfile
1. In your local repository, create a file named `Jenkinsfile`:
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

2. Commit and push to GitHub:
   ```bash
   git add Jenkinsfile
   git commit -m "Add declarative Jenkinsfile"
   git remote add origin https://github.com/YOUR_USERNAME/Jenkins-Declarative-Pipeline.git
   git push -u origin master
   ```

## Pipeline Configuration

### Step 1: Create Jenkins Pipeline
1. Go to Jenkins Dashboard
2. Click "New Item"
3. Enter name: "Declarative-GitHub-Pipeline"
4. Select "Pipeline" and click OK

<div align="center">
  <img src="/images/Screenshot 2025-04-05 022205.png">
</div>

### Step 2: Configure Pipeline Settings
1. In the pipeline configuration:
   - Description: Add a meaningful description
   - GitHub project: Check and add your repository URL
   - Build Triggers: 
     - Scroll to "Build Triggers" section
     - ✅ Check "GitHub hook trigger for GITScm polling"
     - This enables automatic builds when GitHub sends webhook events
   - Pipeline Definition: Select "Pipeline script from SCM"
   - SCM: Select "Git"
   - Repository URL: Add your HTTPS repository URL
   - Credentials: Select the github-token credential
   - Branch Specifier: */master
2. Click "Save"

<div align="center">
  <img src="/images/Screenshot 2025-04-05 022331.png">
</div>

### Step 3: Configure GitHub Webhook
1. In your GitHub repository:
   - Go to Settings → Webhooks
   - Click "Add webhook"
   - Configure webhook:
     - Payload URL: `https://YOUR_NGROK_URL/github-webhook/`
     - Content type: application/json
     - Secret: Leave empty
     - Events: Select "Just the push event"
   - Click "Add webhook"

<div align="center">
  <img src="/images/Screenshot 2025-04-05 024243.png">
</div>

### Step 4: Enable GitHub Trigger in Jenkins
1. Go to your Jenkins job → Configure
2. Scroll to Build Triggers section
3. ✅ Check "GitHub hook trigger for GITScm polling"
4. Save the configuration

This setup ensures Jenkins will automatically start builds when it receives webhook events from GitHub.

<div align="center">
  <img src="/images/Screenshot 2025-04-05 024352.png">
</div>

## Usage

### Manual Builds
1. Go to your pipeline in Jenkins
2. Click "Build Now"
3. Monitor the build progress in "Build History"

<div align="center">
  <img src="/images/Screenshot 2025-04-05 023026.png">
</div>
<div align="center">
  <img src="/images/Screenshot 2025-04-05 023128.png">
</div>

### Automatic Builds
1. Make changes to your repository
2. Commit and push to GitHub
3. Jenkins will automatically trigger the build
4. Monitor the build status in Jenkins

<div align="center">
  <img src="/images/Screenshot 2025-04-05 025728.png">
</div>

## Troubleshooting

### Common Issues and Solutions
1. Jenkins Service Issues
   - Check if Jenkins service is running in Windows Services
   - Restart Jenkins service if needed

2. Webhook Problems
   - Verify ngrok is running and URL is correct
   - Check GitHub webhook delivery logs
   - Ensure Jenkins URL is correctly configured

3. Authentication Issues
   - Verify GitHub token permissions
   - Check Jenkins credentials configuration
   - Ensure token hasn't expired

### Logs and Debugging
- Jenkins logs: Manage Jenkins → System Log
- Build logs: Available in individual build pages
- GitHub webhook logs: Repository Settings → Webhooks

## Additional Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [GitHub Webhook Documentation](https://docs.github.com/en/webhooks)
- [ngrok Documentation](https://ngrok.com/docs) 