#!/usr/bin/env groovy
pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        ECR_REPO_URL = 'java-app'
        IMAGE_REPO = "${ECR_REPO_URL}/java-maven-app"
        ANSIBLE_INVENTORY = 'inventory/hosts'
        APP_PORT = '8090'
        CONTAINER_PORT = '8080'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    echo "############ ${IMAGE_REPO}"
                }
            }
        }
        stage('build app') {
            steps {
               script {
                   echo "building the application..."
                   sh 'mvn clean package'
               }
            }
        }
        stage('deploy with ansible') {
            steps {
                script {
                    echo "Running Ansible playbook for deployment..."
                    sh """
                    ansible-playbook -i ${ANSIBLE_INVENTORY} deploy-container.yml \
                    --extra-vars "image_repo=${IMAGE_REPO} \
                    image_name=${IMAGE_NAME} \
                    app_port=${APP_PORT} \
                    container_port=${CONTAINER_PORT}"
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
