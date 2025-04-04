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

## Setting up ngrok for Webhook Access

### Step 1: Install ngrok
```bash
choco install ngrok
```
Accept any prompts. Once installed, ngrok will be available from any terminal.

### Step 2: Configure ngrok
1. Sign up at [ngrok Dashboard](https://dashboard.ngrok.com)
2. Once logged in, copy your AuthToken from the dashboard
3. Configure ngrok with your token:
   ```bash
   ngrok config add-authtoken <your_token_here>
   ```
   Replace `<your_token_here>` with your actual token

### Step 3: Expose Jenkins
1. Start ngrok to expose Jenkins (port 8080):
   ```bash
   ngrok http 8080
   ```
2. You'll see output like:
   ```
   Forwarding    https://abc123.ngrok.io -> http://localhost:8080
   ```
3. Copy your unique ngrok URL (e.g., `https://abc123.ngrok.io`)
   - This URL will be used for GitHub webhook configuration
   - Note: The URL changes each time you restart ngrok (unless you have a paid plan)

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

1. **Generate GitHub Personal Access Token**:
   - Go to GitHub → Settings → Developer settings → Personal access tokens
   - Click "Generate new token"
   - Select the following permissions:
     - `repo`
     - `workflow`
     - `read:org`
   - Generate and copy the token (store it safely, as you won't be able to see it again)

2. **Add Token to Jenkins**:
   - Navigate to Jenkins → Manage Jenkins → Credentials → Global → Add Credentials
   - Configure:
     - Kind: Username with password
     - Username: Your GitHub username
     - Password: The GitHub token you just generated
     - ID: github-token (or any memorable name)
     - Description: GitHub Personal Access Token
   - Click "Create"

### Step 3: Create Jenkins Pipeline

1. From Jenkins Dashboard, click "New Item"
2. Enter name: "Declarative-GitHub-Pipeline"
3. Select "Pipeline" and click OK
4. Configure Pipeline:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: 
     - HTTPS: `https://github.com/YOUR_USERNAME/Jenkins-Declarative-Pipeline.git`
   - Credentials: Select appropriate credentials
   - Branch: */master
5. Save configuration

### Step 4: GitHub Webhook Setup (Automated Triggers)

1. In GitHub:
   - Go to repository Settings → Webhooks
   - Click "Add webhook"
   - Payload URL: `https://YOUR_NGROK_URL/github-webhook/`
     (Replace YOUR_NGROK_URL with the URL from ngrok, e.g., `https://abc123.ngrok.io/github-webhook/`)
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