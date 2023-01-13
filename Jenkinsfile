pipeline {
    agent {label "linux"}
    // Build Environment Variables
    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ECR_URL = "815599370552.dkr.ecr.us-east-1.amazonaws.com"
        GIT_COMMIT_SHORT = sh(
                script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
                returnStdout: true
        )
        PROJECT_NAME = sh(
                script: "basename $GIT_URL .git",
                returnStdout: true
        ).trim()
    }
    // Build options for Jenkins.
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    // Stages of pipeline execution
    stages {
        stage('ECR Login') {
            steps {
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWS_ECR_URL"
            }
        }
        stage('Helm Update') { 
            agent {
              docker { 
                registryUrl 'https://815599370552.dkr.ecr.us-east-1.amazonaws.com'
                image '815599370552.dkr.ecr.us-east-1.amazonaws.com/helm-utility-image:latest' 
                args '--entrypoint=\'\' -u root --privileged'
              }
            }
            steps {
                sh 'helm repo add koddi-one-debug-service s3://helm-koddi/stable/koddi-one-debug-service/'
                sh 'helm package --app-version `cat koddi-one-debug-service/Chart.yaml | grep "version: " | cut -d" " -f2` ./koddi-one-debug-service'
                sh 'helm s3 push --force ./koddi-one-debug-service-`cat koddi-one-debug-service/Chart.yaml | grep "version: " | cut -d" " -f2`.tgz koddi-one-debug-service'
            }
        }
    }
}
