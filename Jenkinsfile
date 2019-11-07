pipeline {
    agent any
    tools {
        maven 'ADOP Maven'
        jdk 'Java'
    }
    stages {
        stage('Build') {
            steps {
                cleanWs()
                sshagent(['adop-jenkins-master']) {
                        
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
                
                checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'adv1', url: 'http://gitlab/gitlab/advancedone/currencyconverterdts.git']]]
                
                sh 'mvn -B -DskipTests clean package'
               
                archiveArtifacts artifacts: '**/*.war', onlyIfSuccessful: true
                }
            }
        }
        
        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Quality') {
            environment {
                sonarscanner = tool name: 'ADOP SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                
                withCredentials([usernamePassword(credentialsId: 'adv1', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    
                withSonarQubeEnv ('ADOP Sonar') {
                sh '''
                #!/bin/bash
                ${sonarscanner}/bin/sonar-scanner \
                    -Dsonar.projectKey=CurrencyConverter \
                    -Dsonar.projectName=currency-converter-${BUILD_ID} \
                    -Dsonar.projectVersion=1 \
                    -Dsonar.login=$USERNAME \
                    -Dsonar.password=$PASSWORD \
                    -Dsonar.sources=.
                '''
                }
                }
            cleanWs()
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                    nexusArtifactUploader artifacts: [[artifactId: 'CurrencyConverter', classifier: '', file: '/var/jenkins_home/jobs/pipeline_test/builds/${BUILD_ID}/archive/target/CurrencyConverter.war', type: 'war']], credentialsId: 'adv1', groupId: 'sim', nexusUrl: 'nexus:8081/nexus', nexusVersion: 'nexus2', protocol: 'http', repository: 'releases', version: '1'
            }
        }
    }
}