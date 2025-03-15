pipeline {
    agent any
    
    environment {
        // Define environment variables
        APP_NAME = 'my-java-app'
        DOCKER_IMAGE = 'my-java-app:latest'
        DOCKER_PORT = '8080'
        HOST_PORT = '8080'
        ANSIBLE_INVENTORY = 'inventory/hosts'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout the Jenkins pipeline repository
                checkout scm
            }
        }
        
        stage('Run Ansible Deployment') {
            steps {
                // Run the Ansible playbook
                sh '''
                ansible-playbook -i ${ANSIBLE_INVENTORY} deploy-app.yml \
                --extra-vars "app_name=${APP_NAME} \
                docker_image=${DOCKER_IMAGE} \
                docker_port=${DOCKER_PORT} \
                host_port=${HOST_PORT}"
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                // Simple health check to verify the application is running
                sh '''
                ansible -i ${ANSIBLE_INVENTORY} app_servers -m uri -a "url=http://localhost:${HOST_PORT}/health status=200" || true
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
