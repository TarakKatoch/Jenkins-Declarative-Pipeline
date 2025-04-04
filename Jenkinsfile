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