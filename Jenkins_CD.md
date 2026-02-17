# CI/CD Pipeline Script

Add the following `Jenkinsfile` content to your repository root or copy it directly into a Jenkins Pipeline job.

```groovy
pipeline {
    agent any

    stages {
        stage('Deploy the container') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'Dockerhub') {
                        sh "docker stop ekart"
                        sh "docker rm ekart"
                        sh "docker run -d --name ekart -p 8070:8070 manojhub2002/ekart:latest" 
                }
                }
            }
        }
    }
}



