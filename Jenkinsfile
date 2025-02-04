pipeline {
    agent any
    
    environment{
        SCANNER_HOME = tool "sonarqube"
    }

    stages {
        stage('clean-ws') {
            steps {
                cleanWs()
            }
        }
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/psslalitha/Python-System-Monitoring.git'
            }
        }
        stage('sonarqube-analysis') {
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=python \
                    -Dsonar.projectKey=python'''
                }
            }
        }
        stage('sonar-quality-gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 's-id'
                }
            }
        }
        stage('trivy'){
            steps{
                sh 'trivy fs . > trivyresults.txt'
            }
        }
        stage('DP-check'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'dp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("docker"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'doc-id', toolName: 'docker'){   
                       sh "docker build -t python ."
                       sh "docker tag python srilalithac/python:latest "
                       
                    }
                }
            }
        }
        stage("trivy image"){
            steps{
                sh 'trivy image srilalithac/python:latest > trivyimageresults.txt'
            }
        }
         stage("docker-push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'doc-id', toolName: 'docker'){   
                       sh "docker push srilalithac/python:latest"
                       
                    }
                }
            }
        }
        stage(deploy){
            steps{
                sh 'docker run -d -p 5000:5000 --name pythonC srilalithac/python:latest'
            }
        }
        
    
    }
}
