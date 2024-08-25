pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/asatishpaul/FullStack-Blogging-App'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh '''$SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectName=Blogging-app  -Dsonar.projectKey=Blogging-app -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('docker Build & Tag') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                
                sh "docker build -t asatishpaul/bloggingapp:latest ." 
                }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html asatishpaul/bloggingapp:latest"
            }
        }
        stage('docker Push Image') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                
                sh "docker push asatishpaul/bloggingapp:latest" 
                }
                }
            }
        }
        stage('k8s-deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-secret', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://1e66084e878db6b4baf1f7e79c2f55be.gr7.ap-south-1.eks.amazonaws.com/') {
                  sh 'ls -al'
                  sh 'kubectl apply -f /var/lib/jenkins/workspace/ci-cd/deployment-service.yml'
                  sleep 20
                }
            }
        }
        stage('Verify the deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-secret', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://1e66084e878db6b4baf1f7e79c2f55be.gr7.ap-south-1.eks.amazonaws.com/') {
                  sh 'kubectl get pods'
                  sh 'kubectl get svc'
                }
            }
        }
    }
}
