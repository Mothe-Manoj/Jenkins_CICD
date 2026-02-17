# CI/CD Pipeline Script

Add the following `Jenkinsfile` content to your repository root or copy it directly into a Jenkins Pipeline job.

```groovy
pipeline {
    agent any
    
    tools {
        jdk 'JDK'
        maven 'maven'
    }

    environment {
        // SonarQube Scanner Tool Name from Jenkins Global Tool Configuration
        SCANNER_HOME = tool 'sonar-scanner'
        
        // Define Docker Image Name
        DOCKER_IMAGE = "manojhub2002/ekart"
    }

    stages {
        stage('Git Checkout') {
            steps {
                // Clones the source code from GitHub
                git branch: 'main', url: 'https://github.com'
            }
        }
        
        stage('Compile') {
            steps {
                // Compiles the Java source code
                sh "mvn clean compile"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                // Performs static code analysis and sends report to SonarQube server
                sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.host.url=http://3.110.145.238 \
                    -Dsonar.login=squ_ddcea8b2ffb044e14830787bbb7b584b5db59cce \
                    -Dsonar.projectKey=Ecart \
                    -Dsonar.projectName=Ecart \
                    -Dsonar.java.binaries=.
                """
            }
        }
        
        stage('Build Application') {
            steps {
                // Packages the application into a JAR/WAR file, skipping unit tests
                sh "mvn clean package -Dmaven.test.skip=true"
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Uses 'Dockerhub' credentials stored in Jenkins to push the image
                    withDockerRegistry(credentialsId: 'Dockerhub') {
                        sh "docker build -t ${DOCKER_IMAGE}:latest -f docker/Dockerfile ."
                        sh "docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    }
                }
            }
        }
        
        stage('Trigger CD Pipeline') {
            steps {
                // Automatically triggers the downstream Continuous Delivery pipeline
                build job: "CD_pipeline", wait: true
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "CI Pipeline completed successfully!"
        }
        failure {
            echo "CI Pipeline failed. Check the logs."
        }
    }
}
