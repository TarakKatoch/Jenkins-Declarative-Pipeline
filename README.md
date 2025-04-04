# Jenkins in Docker

This project sets up Jenkins in a Docker container for CI/CD pipeline development.

## Prerequisites

- Docker
- Docker Compose
- Git

## Setup Instructions

1. Start Jenkins container:
   ```bash
   docker-compose up -d
   ```

2. Access Jenkins:
   - Open your browser and go to `http://localhost:8080`
   - The initial admin password can be found by running:
     ```bash
     docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
     ```

3. Install required plugins:
   - After logging in, go to "Manage Jenkins" > "Manage Plugins"
   - Install the following plugins:
     - Git
     - Pipeline
     - GitHub Integration

4. Configure GitHub credentials:
   - Go to "Manage Jenkins" > "Manage Credentials"
   - Add your GitHub credentials (either SSH key or token)

5. Create a new pipeline:
   - Click "New Item"
   - Enter a name and select "Pipeline"
   - In the configuration:
     - Select "Pipeline script from SCM"
     - Choose "Git" as SCM
     - Enter your repository URL
     - Select your credentials
     - Set the branch to build
     - Set the script path to "Jenkinsfile"

## Stopping Jenkins

To stop the Jenkins container:
```bash
docker-compose down
```

## Notes

- Jenkins data is persisted in a Docker volume named `jenkins_home`
- The container is configured to restart automatically unless stopped manually
- Port 8080 is used for the web interface
- Port 50000 is used for Jenkins agent communication 