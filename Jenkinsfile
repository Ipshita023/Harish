pipeline {
    agent any

    tools {
        jdk 'JDK11'
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ipshita023/Harish.git'
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
                            -Dsonar.projectKey=Ipshita023_Harish \
                            -Dsonar.organization=harishscoreme \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=8a1cf4f474853e5ea9311d9c020f06e4ea27f2db"
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
                dir('/var/lib/jenkins/workspace/score-me-assignment') {
                    // Run Maven build which includes JaCoCo code coverage
                    sh 'mvn clean test'
                    jacoco execPattern: 'target/jacoco.exec', classPattern: 'target/classes', sourcePattern: 'src/main/java', exclusionPattern: '**/Test*', changeBuildStatus: true
                }
            }
        }

        stage('Cyclomatic Complexity') {
            steps {
                sh """
                export PATH=$PATH:~/.local/bin
                lizard .
                """
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
