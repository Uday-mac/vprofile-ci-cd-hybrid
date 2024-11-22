def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any 
    tools {
        jdk "OracleJDK8"
        maven "MAVEN3"
    }
    environment {
        sonar_scanner = 'sonar4.7'
        sonar_server = 'sonar'
        NEXUS_IP = '18.208.247.120'
        NEXUS_PORT = '8081'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-mvn-central'
        NEXUS_GRP_REPO = 'vprofile-group'
        NEXUS_LOGIN = 'nexus_login'
    }
    stages {
        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('sonar analysis') {
            environment {
                sonarHome = tool "${sonar_scanner}"
            }
            steps {
                withSonarQubeEnv("${sonar_server}"){
                    sh ''' ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.source=src/ \
                    -Dsonar.java.binaries=target/classes/com/visualpathit/account/controller \
                    -Dsonar.junit.reportsPath=target/surefire-reports \
                    -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
            }
        }

        stage('Quality_gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage('build artifact') {
            steps {
                sh 'mvn -s settings.xml install'
            }
        }
        
        stage('upload to nexus') {
            steps {
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${NEXUS_URL}:${NEXUS_PORT}",
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}",
                credentialsId: 'nexus_login',
                artifacts: [
                    [artifactId: 'vproapp',
                    classifier: '',
                    file: 'target/vprofile-v2.war',
                    type: 'war']
                ]
            )
            }
        }
        
    }

    post {
        always {
            steps {
                slackSend channel: "#jenkins-hybrid",
                color: COLOR_MAP[currentBuild.currentResult],
                message: "Find Status of Pipeline:- ${currentBuild.currentResult} ${env.JOB_NAME} ${env.BUILD_NUMBER} \n more info at  ${BUILD_URL}"
            }
        }
    }
}