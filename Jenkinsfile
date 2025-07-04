pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    
    }
    environment {
        SCANNER_HOME= tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/iqraijaz1/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                  -Dsonar.java.binaries=. '''
               }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy to Nexus') {
            steps {
             withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
               sh "mvn deploy -DskipTests=true"
               }   
            }
        }
        stage('Dokcer Build & Tag Image') {
            steps {
     
                   sh "docker build -t iqraijaz/ekart:latest -f docker/Dockerfile ."
                 
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image iqraijaz/ekart:latest > trivy-report.txt"
            }
        }
          stage('Dokcer Push Image') {
            steps {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                   sh "docker push iqraijaz/ekart:latest"
                 }

            }
        }
    }
}
