pipeline {
    agent any

    tools {
        jdk 'JDK11'
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/IpshitaCT/Jenkins_Task.git'
            }
        }
        stage('Build and Test') {
            steps {
                // Run Maven build which includes JaCoCo code coverage
                sh 'mvn clean test'
            }
        }
        stage('SonarCloud Scan') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('SonarCloud') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=IpshitaCT_Jenkins_Task\
                            -Dsonar.organization=ipshitact \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=a2c51ef132f9abc7dce60942dc62050418e4040f"
                    }
                }
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                script {
                    // Run OWASP Dependency-Check
                    def report = 'dependency-check-report/dependency-check-report.xml'
                    sh "mvn org.owasp:dependency-check-maven:check -Ddependency-check.reportFormat=XML -Ddependency-check.outputDirectory=dependency-check-report"
                    
                    // Archive the Dependency-Check report
                    archiveArtifacts artifacts: report, allowEmptyArchive: true

                    // Fail the build if critical vulnerabilities are found
                    def result = readFile(report)
                    if (result.contains('Critical')) {
                        error "Critical vulnerabilities found in dependencies."
                    }
                }
            }
        }
        stage('Publish JaCoCo Report') {
            steps {
                // Publish JaCoCo coverage report
                recordIssues(tools: [jacoco(pattern: '**/target/site/jacoco/jacoco.xml')])
            }
        }
        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install lizard
                '''
            }
        }
        stage('Cyclomatic Complexity') {
            steps {
                sh '''
                    . venv/bin/activate
                    venv/bin/lizard .
                '''
            }
        }
    }

    post {
        success {
            mail to: 'ipshitachaudhary2000@gmail.com',
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' succeeded."
        }
        failure {
            mail to: 'ipshitachaudhary2000@gmail.com',
                subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed."
        }
    }
}
