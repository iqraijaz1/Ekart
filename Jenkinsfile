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
           stage('manifest repo checkout') {
          steps {
             dir('Ekart-manifest') {
                  git branch: 'main', url: 'https://github.com/iqraijaz1/Ekart-manifest.git'

             }
             
          }
         
        }
      stage('update tag push to manifest repo') {
        steps {
         dir('Ekart-manifest') {
           withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
             sh '''
               git config user.email "shadow7860@gmail.com"
               git config user.name "iqraijaz1"
               cat deployment.yaml
               sed -i "s+iqraijaz/Ekart-manifest.*+iqraijaz/Ekart-manifest:${BUILD_NUMBER}+g" deployment.yaml
               git add -A
               git commit -m "Updated by Jenkins: ${BUILD_NUMBER}" || echo "No changes to commit"
               git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/iqraijaz1/Ekart-manifest.git HEAD:main
             '''
           }
         }
    }
}
}
}

