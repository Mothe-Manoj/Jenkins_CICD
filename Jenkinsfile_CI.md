#CI_PIPELINE
 
 * pipeline {
    agent any
    
    tools{
        jdk 'JDK'
        maven 'maven'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jaiswaladi246/Ekart.git'
                
            }
        }
        
        stage('compile') {
            steps {
                sh "mvn clean compile"
                
            }
        }
        
        stage('SonarQube Analysis') {
    steps {
        sh """
            $SCANNER_HOME/bin/sonar-scanner \
            -Dsonar.host.url=http://3.110.145.238:9000/ \
            -Dsonar.login=squ_ddcea8b2ffb044e14830787bbb7b584b5db59cce \
            -Dsonar.projectKey=Ecart \
            -Dsonar.projectName=Ecart \
            -Dsonar.java.binaries=.
        """
        }
     }
        
       stage('Build application') {
                    steps {
                        sh "mvn clean package -Dmaven.test.skip=true"
                
            }
        }
        
        stage('Build & push docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'Dockerhub') {
                        sh "docker build -t ekart:latest -f docker/Dockerfile ."
                        sh "docker tag ekart:latest manojhub2002/ekart:latest"
                        sh "docker push manojhub2002/ekart:latest"
                }
                }
            }
        }
        
        stage('Trigger') {
                    steps {
                        build job: "CD_pipeline" , wait: true
                
            }
        }
                
    }
} *
