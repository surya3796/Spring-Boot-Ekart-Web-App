pipeline {
    agent any

    tools{
        maven 'maven3'
        jdk 'jdk17'
        
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/surya3796/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. 
                    '''
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
    }
         stage('Deploy to Nexus') {
            steps {
                script {
                    withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('Build and tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t surya030596/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        
            
        stage('Trivy Scan') {
            steps {
                sh "trivy image surya030596/ekart:latest > trivy-report.txt"
            }
        }
        
        stage('Push Image to Dockerhub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push surya030596/ekart:latest"
                    }
                }
            }
        }
        
                stage('Kubernetes Deploy') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'k8s-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.93.192:6443') {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sh "kubectl get svc -n webapps"
                    }
                }
            }
        }
    }