pipeline {
    agent any

    tools {
        jdk 'JDK11'
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/IpshitaCT/harish-jenkins-task.git'
            }
        }
        stage('List Files') {
            steps {
               sh 'ls -al'
                 
            }
        }
        stage('file path') {
            steps {
               sh 'pwd'
                 
            }
        }
        
        stage('SonarCloud Scan') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('SonarCloud') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=IpshitaCT_harish-jenkins-task \
                            -Dsonar.organization=ipshitact \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=bf0a5119898648af390b5aa8868fe35ecdbd6f74"
                    }
                }
            }
        }
        
        stage('Build and Test') {
            steps {
                // Run Maven build which includes JaCoCo code coverage
                sh 'mvn clean test'
            }
        }
        
        stage('Code Coverage') {
            steps {
                // Change directory to where your pom.xml is located
                dir('/') {
                    // Run Maven build which includes JaCoCo code coverage
                    sh 'mvn clean test'
                    jacoco execPattern: 'target/jacoco.exec', classPattern: 'target/classes', sourcePattern: 'src/main/java', exclusionPattern: '**/Test*', changeBuildStatus: true
                }
            }
        }

        stage('Cyclomatic Complexity') {
            steps {
                sh 'lizard .'
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'lizard-report',
                    reportFiles: 'index.html',
                    reportName: 'Cyclomatic Complexity Report'
                ])
            }
        }
        
        stage('Security Vulnerability Scan') {
            steps {
                sh 'dependency-check.sh --project your-project --scan .'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
    }
}
